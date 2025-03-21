# A complete-ish modern guide on snaps, rate, maxfps and physicsfps, including info for server owners/rconners

*Last updated: 2025-03-21*

I will first explain the client settings and later go into server settings and how they related to the client settings.

Some of this information applies to all JK2 versions, some other information may only apply to recent clients/servers. Especially vanilla JK2 has some things just hardcoded.

## Client settings

### com_physicsFps

This setting lets you set a fixed and perfectly regular interval for timestamps for userpackets you send to the server. This allows for perfectly stable physics behavior, unaffected by the performance of your own PC, your com_maxfps setting or certain quirks in the netcode. Note: Even a PC that can pump out 100% stable 142 fps will NOT get truly stable 142fps packet timing for consistent physics. The main reason for this is that the user packet timing is not only affected by your real frametimes, but constantly (every time you receive a snapshot packet from the server) adjusted to closely track the server's time. More details about this in the section about the snaps setting. 

Due to the imperfect nature of userpacket timestamping without com_physicsFps, the feel of com_physicsFps will be a bit unfamiliar and might for example make some movements at com_physicsFps 142 rather difficult to execute well.

At g_gravity 800, com_physicsFps 142 will result in an effective gravity of 857 (allowing for good maneuverability and low jumps), and com_physicsFps 125 will result in an effective gravity of 750. Acceleration is also affected, but due to being two-dimensional and depending on strafe and your base speed, it is hard to give a concise summary of the effects other than com_physicsFps 142 being generally a bit faster with optimal strafe and tending to lock you to straighter movement lines than 125.

### com_maxfps 

This setting determines how many frames your client draws on the screen. If com_physicsFps is disabled (set to 0), it also affects your movement physics similar to how com_physicsFps does, but the effects are more muted in comparison. The degree to which com_maxfps affects your physics in this case is dependant on the stability of your PC being able to generate the desired amount of frames per second in regular intervals, and how many snapshots per second you receive from the server. More details on this in the section about the snaps setting. 

If com_physicsFps is being used, com_maxfps simply controls how many frames are drawn on your screen, with no effect on your movement physics. E.g. you can play at maxfps 30 and see 30 frames drawn on your screen but play with 142 fps physics. Or reversely, you can set maxfps to 500 and have smooth 500 frames per second whilst still playing at 142 fps physics.

Note: The smooth behavior at high framerates depends on another cvar called cg_specialPredictPhysicsFps, which by default is set to 3. If you set it to 0, you will still get that amount of frames drawn, but they will not feel as smooth as you'd expect. 

### rate 

rate is a recommendation/wish the client sets which is shared with the server. It sets the maximum amount of data you wish to receive from the server per second, in bytes. Generally most servers tend to ignore this value, but if they don't, you will be receiving less messages/snaps from the server to not exceed this data rate. If you set a very low value and the server accepts it, you can end up with extreme lags. Typically you want to just set this to some very high value (like 200000) and forget about it. NWH already has a good default value here and you don't need to worry about it.

### snaps

snaps is a recommendation/wish the client sets which is shared with the server. It sets the maximum amount of snaps per second you wish to receive from the server. Like with rate, most servers - regrettably, see later section on server settings - ignore this value and send you whatever amount of snaps is desired by the server owner. 

The primary impact of snapshots per second is just what it implies: How many updates per second you get from the server about the game's state - positions of players, items, etc.

Secondarily, too many snapshots at too high of a ping (e.g. over 100 snaps per second with 100+ ping) will result in an inability to receive delta-snapshots because the server will phase out/forget old frames it has sent to clients to delta from. This will result in more network traffic, bloated demo files and sometimes yellow color on the lagometer. If this happens you can try a lower snaps setting and see if the server accepts it.

Thirdly, if the client is not using com_physicsFps, the amount of snaps received per second will affect movement physics. The reason is that the client slightly adjusts its local time (which is also used for timestamping userpackets and thus evaluating movement) to track the server's time. This generally happens every time a snapshot is received from the server. The more snapshots you receive, the more often this adjustment takes place. The more often this adjustment takes place, the more unstable your effective frame timing becomes, which in turn reduces the effect to which com_maxfps affects movement physics, since the effect of this constant adjustment outweighs the actual frame timing of your frames. This is why, for example, com_maxfps has a relatively low impact on movement physics when playing locally on your PC with the /map command. When playing locally, you get a fresh update from the "server" (integrated in your client) on every single client frame, no matter the snaps settings. This is hardcoded behavior.

Generally speaking, since people enjoy fps-specific behavior, a very high snaps setting without com_physicsFps usage results in somewhat unpleasant movement.

To demonstrate, this is - roughly - how snapshots per second affect effective gravity (at g_gravity 800) at com_maxfps 142:
- 25 snaps (traditional): 840
- 50 snaps (classic NWH before the 2024 update): 820
- 100 snaps (most current NWH servers): 800 (effects of com_maxfps very strongly limited)

This behavior is why NWH servers used to be locked at 50 snaps. At com_maxfps 142 for example, some jumps that are not possible with 50 snaps become possible with 100 snaps (which was seen as an exploit), as the gravity is effectively reduced and things generally feel more "floaty". But even this old NWH behavior already had a notably reduced gravity compared to the traditional 25 snaps servers. More on this in the server settings section.

Note: In my own client, TommyTernal, there is a feature called com_slowDriftAdjustMaxFPS which by default is set to 25. This limits how often the time adjustment from server snapshots takes place, letting you enjoy com_maxfps 142 with 100 snaps whilst the physics of movement remain closer to how they would be with 25 snaps.

### cl_packetdup, cl_maxpackets

cl_maxpackets controls how many packets your client sends to the server, at most. It is essentially the amount of times per second the server gets an update from you. If you set a too low value here, you can appear stuttery to other players, and you might get worse hit registration as the server will update your saber/forces/movement less often. In essence a low value will make you lag a bit, so keep this at a high value (1000). Current NWH has this value set automatically to my knowledge, so you don't need to worry about it. 

cl_packetdup determines how often each user command gets sent to the server. You generally want this to be 1 (exception see below), which means each usercommand gets sent twice (duplicated once). If your internet is extremely unstable and you experience high packet loss for your messages to the server, you can try increasing this value to reduce the negative impact of the packet loss (which to you would feel like prediction errors/stuttering/lagging).

#### cl_dynamicuserpacket

In TommyTernal, a feature called cl_dynamicuserpacket exists. If set to 1 (default), the client will automatically check the playerstate commandtime of the last snapshot received from the server and use this information to send exactly as many old userpackets to the server as necessary to not have packet loss. This eliminates the need for cl_packetdup, which should be set to 0 when using cl_dynamicuserpacket 1. Otherwise, some annoying console spam (MAX_USERCMDS) can occur.

### cl_commandsize 

On clients that support this (I believe old NWH had it, but current does not, but I might be wrong), set it to 512 to compensate negative effects on clientside prediction when experiencing lag. If you experience a short lag/delay in receiving packets from the server, this will let you keep hopping for a bit without disrupting your flow, which might save you in some situations. This setting does not affect gameplay or packet timing or anything else about you from the perspective of other players, it imply improves the prediction in your own client.

## Server settings

### sv_minRate/sv_maxRate

This regulates the allowed range of rate values accepted from a client. Generally you probably want sv_minRate to be some reasonably high value (e.g. 25000) so people don't accidentally lock themselves into an extremely stuttery/laggy experience by seting rate 0 or some nonsense like that. The max value should be pretty high. 200000 can't hurt. The classic value here was 90000 hardcoded, which should also be fine in almost all cases.

### sv_fps 

This is the framerate the server/game module runs at. This determines how often the game's state is updated, saber movements are evaluated, force powers are checked/activated/etc. It is also the maximum amount of snapshots a client can possibly receive - with the exception of playing locally with /map in the same client, in which case the client receives a new snapshot on every single client frame.

### sv_minSnaps/sv_maxSnaps/sv_enforceSnaps

Firstly, sv_enforceSnaps is a very important value. If it is set to 1, client snaps settings are always totally ignored. Keep it at 0 to take the user wish into account. 

Regardless of sv_enforceSnaps, minSnaps and maxSnaps regulate the allowed range of snaps values for any given client. The main difference with sv_enforceSnaps 1 is that the user wish is ignored and only sv_fps is considered (but still limited by minSnaps and maxSnaps). 

These values effectively control how many snapshots per second clients can receive. A client can only receive a snapshot once on every server frame, so if sv_fps is 100 and a client is set to receive 200 snaps, he will still only receive 100. There is also a slightly unintuitive matter of timing. For example, if you set snaps to 60 at an sv_fps of 100, the client will end up only receiving 50, as the delay between snapshots is checked on every frame and reset when a new snapshot is sent. 60 snaps would require a time interval of 16 milliseconds, but the server only checks every 10 milliseconds (100 times per second), so effectively a snapshot is sent every 20 milliseconds (50 times per second). 

Most server owners don't allow the client much control over how many snaps he receives. Historically, here is an overview of how servers ran:
- Traditional: 25 snaps max (min unknown, depending on server probably), sv_fps 100. 
- Older NWH before 2024 update: 50 snaps max (usually also min 50), sv_fps 100
- Current NWH servers: 100 snaps min and max, sv_fps 100 (sometimes more for experiments) 

NWH used to be capped at 50 maxSnaps because 100 maxSnaps (in the case of com_maxfps 142) makes movement "floatier" and allows higher jumps (e.g. red right balcony backflip up)/floatier rolls, which was seen as an exploit (in actuality it was just a result of extremely unstable frame timestamps). With com_physicsFps becoming an option, 100 snaps is now commonly used, allowing more server updates for clients whilst maintaining stable physics. However, given the difficulties of some movements with com_physicsFps 142, some players choose to keep playing at com_maxfps 142. For this reason I recommend setting sv_minSnaps 25 and sv_enforceSnaps 0, allowing players to experience more traditional com_maxfps 142 behavior, at either 25 or 50 snapshots per second, by setting their own snaps value accordingly. There isn't really any way to exploit a low snaps value; e.g. it cannot be used as a lag switch as it only affects how many packets the client receives, not how many packets he sends, hence even a player with 15 snaps would still look perfectly fluid to other players as long as his internet is stable and unrelated client settings aren't manipulated. It merely allows clients to play with more traditional movement physics.

Note: As mentioned higher in the snaps client setting section, too many snapshots at too high of a ping (e.g. far over 100 snaps per second with 100+ ping) will result in an inability to send delta-snapshots because the server will phase out/forget old frames it has sent to clients to delta from. This will result in more network traffic, bloated demo files and sometimes yellow color on the lagometer. Sending a lot of non-delta snaps to clients during a busy game can also lead to split messages/netcode glitches and strong stutter for the affected clients. Most servers keep a record of the last 32 messages sent to any client (hardcoded). At 100 snaps per second, that means a client has up to 320 milliseconds of time to confirm the receipt of an older snapshot to keep receiving delta frames. In other words, 100 snaps are theoretically okay for people up to a maximum of 320 ping. 200 snaps are theoretically okay for people up to a ping of 160. However this theoretical maximum will be practically lower because timing is never 100% in sync and perfect and small spikes/lags can quickly push the delay higher than that. 

Especially when using sv_fps over 100, at the very least an sv_minSnaps of 100 should be allowed for clients to not experience issues.



