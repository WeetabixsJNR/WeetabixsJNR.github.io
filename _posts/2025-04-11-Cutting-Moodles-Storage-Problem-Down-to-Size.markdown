---
layout: post
title: "Cutting Moodle's Storage Problem Down to Size"
date: 2025-04-11 09:50:00 +0100
icon: ti-cloud-down
accent: "#286FD6"
author: Darragh Crawley-Doyle
tags: SharePoint, Moodle, Process Improvement, Automation
---

**Nobody flagged this as a problem. It just quietly kept costing time and storage until it was worth actually fixing.**

Class recordings were being uploaded individually to Moodle, one file at a time, for every single class, every term. On its own, uploading one recording is nothing. Multiplied across every class running through the organisation, it turned into a genuinely significant chunk of staff time spent on a manual upload process, and a steadily growing Moodle storage bill that eventually needed active cleanup rather than just sitting there quietly.

Nobody had asked me to fix this. It surfaced the way most of the improvements I actually care about do, by noticing a repeated cost that had become normal enough that people had stopped questioning it. Once I actually looked at where the time and the storage were going, the fix wasn't complicated: recordings didn't need to live inside Moodle at all, they just needed to be reachable from it.

The solution was a view-only SharePoint site, organised by class, with the relevant folder linked directly from the corresponding Moodle course. Staff drop recordings into the right class folder once, in a structure that already made sense to them, and Moodle simply points at it rather than storing a duplicate copy of every file itself. No new system to learn, no separate login, just the existing structure staff already understood, hosted somewhere that wasn't quietly eating Moodle's storage.

The result was less upload effort for staff and a lot less ongoing Moodle storage cleanup for me, which is exactly the kind of trade I'll take every time: a small amount of upfront restructuring in exchange for a recurring cost that simply stops recurring. It's not a project that will ever show up on a highlight reel, but it's the kind of fix that keeps paying for itself term after term without anyone having to think about it again.
