---
layout: post
title: "What's Actually Running in My Homelab"
date: 2025-01-18 14:30:00 +0000
icon: ti-server
accent: "#28B78D"
author: Darragh Crawley-Doyle
tags: Homelab, Self-Hosted, Networking, Unraid
---

**January is homelab season. Too cold to be outside, too early in the year for anything else to demand attention, so the annual "tidy up the server and actually document what's running on it" project happens now.**

The short version: an Unraid server sits behind a UniFi router, with Virgin Media as the main internet connection and a 5G connection configured as failover only. A custom SLA check watches a handful of external targets rather than trusting the router's default monitoring, which was tripping into failover far too easily. The main network and the IoT devices sit on separate VLANs, mostly so a compromised smart plug can't see anything interesting on the main network.

Everything on the server runs in Docker, but not all in the same way. Anything that needs to discover other devices on the network, like Home Assistant, runs on host networking. Most of the standalone utility containers sit behind a standard Docker bridge. The bigger group, media and the self-hosted apps I actually rely on day to day, all live on their own Docker network behind Caddy as a reverse proxy, with Authentik handling single sign-on in front of anything that doesn't already need its own native login for a TV or mobile app.

The media side is the part that actually gets used by other people: a shared library that plays back through an app on every TV in the house. None of that is unusual as far as homelabs go. What I'd actually call the interesting part is designing it so someone with zero interest in how any of it works can still use it. My grandparents can open an app and press play. That's the actual product requirement, and it's the same lesson I apply at work: the technology is only good if the person who doesn't care how it works still has a good experience.

Past the media stack there's a small pile of other things I've ended up depending on: Immich for private photo backup instead of paying a cloud provider, Vaultwarden for passwords, and a meal-planning app I built from scratch for a friend, front end, API and database, because nothing off the shelf did quite what he wanted. Remote access to all of it runs through Tailscale rather than opening anything to the public internet directly, which also means it keeps working even when the connection fails over to 5G.

Resilience was the part I spent the most time on this round. Two separate UPS units, one for networking gear and one for the server, so the network and Tailscale access stay up even if the server itself has to shut down cleanly on battery. Local backups of all the container configuration for fast recovery, and an encrypted offsite copy of anything that actually can't be replaced, personal documents rather than the media library, since the media collection matters a lot less than photos or paperwork if something ever goes wrong.

None of this needed to be this resilient for a home setup. I built it this way because it's the same set of questions I'd ask about any production system: what happens when this fails, who notices, and how long before it's back. Turns out that's a fun question to ask about your own house too.
