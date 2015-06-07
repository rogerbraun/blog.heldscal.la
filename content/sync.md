Title: Analog Video Synchronisation
Date: 2015-06-07 20:00
Category: Retro
Slug: video-sync
Tags: RGB, Sony PVM, CRT
Author: Roger Braun
Summary: An explanation of the different ways that an RGB monitor can get a sync signal.

This is a short one. One of the nice things about living in a big city is that you can find somewhat rare devices like a Sony PVM RGB monitor. These things were used in professional video production, but now they are sought after by people playing games on old computers and gaming consoles. These monitors have one of the best CRT images available. They are also super heavy. I nearly broke my back bringing it home...

![The PVM in action]({filename}/images/pvm.jpg)

Now, one of the harder parts of this exercise is actually getting the signal to the monitor. Many PVMs only have BNC inputs, a connector you might remember from mid-90s LAN parties. The monitor will need four signals: Red, green, blue and sync. What's sync? It's the signal that's telling the monitor when a new frame or line starts. All video signals have this, but with composite video (i.e. 'the yellow cable') it's integrated with the image signal, so you don't need an extra cable.

If you want to learn everything there is about sync, you should visit [RetroRGB](http://retrorgb.com/sync.html). Here, I just want to talk about the two different kinds of sync signals that you can encounter:

1. Composite Video: The 'yellow cable' I've mentioned before also carries the sync signal. Your monitor may be able to sync on this signal, but not all can do it.

2. Composite Sync: This is _just_ the sync part of the composite video signal. There is no image data being transmitted. Some monitors need this signal. You can get it from composite video by using a sync stripper circuit.

My advice would be to just try if your monitor can sync with composite video. If it can not, you can easily add a sync stripper circuit after the fact.


