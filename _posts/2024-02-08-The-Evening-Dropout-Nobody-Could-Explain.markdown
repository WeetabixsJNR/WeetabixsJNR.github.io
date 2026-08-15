---
layout: post
title: "The Evening Dropout Nobody Could Explain"
date: 2024-02-08 19:40:00 +0000
icon: ti-search
accent: "#286FD6"
author: Darragh Crawley-Doyle
tags: Troubleshooting, Networking, Problem Management, IT Support
---

**"The internet drops for evening classes" is a genuinely difficult ticket. Not because it's hard to believe, but because by the time anyone reports it, the moment has already passed.**

This one had been logged a handful of times over a few weeks, always the same shape: evening class starts, room fills up, and at some point mid-session the wifi just stops working for a chunk of the room. Nobody could say exactly when. Nobody could reliably reproduce it. The easy, wrong response to a ticket like that is to treat each report as its own one-off incident, apologise, and move on. The problem doesn't go away, it just goes quiet for a while and comes back the next time a room fills up.

So instead of waiting for the next report, I went looking for the pattern. The common thread across every report was room occupancy, a lot of devices joining the network in a short window at the start of class. That pointed at DHCP rather than the wifi hardware itself, so I started watching lease behaviour during an actual evening session rather than trying to reconstruct it after the fact from a support ticket written the next morning.

It turned out to be lease timing. The lease pool and renewal timing hadn't been sized for a room's worth of devices all joining in the same few minutes, several evenings a week, on top of the daytime device load already using the same pool. Under the right conditions, new devices joining right at class start could end up waiting on leases that hadn't actually freed up yet from the daytime session. Adjusting the lease configuration to account for that pattern fixed it. No new hardware, no drama, just a lease pool that was quietly undersized for how the room was actually being used.

Around the same time I was chasing a completely different recurring problem: laptops kept coming back through reception in the wrong state, wrong accessories, not logged correctly, sometimes not even flagged as returned at all. Same underlying mistake as the network issue, just wearing a different outfit. Treating each bad laptop return as its own isolated incident wasn't fixing anything, because the actual problem lived in the process at reception, not in any individual laptop. Fixing that meant working with reception on the process itself, not writing a better incident note each time.

Both of these come back to the same lesson, and it's one I still lean on: a support ticket tells you that something happened, not why. If the same shape of problem keeps showing up, the fix usually isn't a faster response to the next ticket, it's figuring out what's actually feeding all of them.
