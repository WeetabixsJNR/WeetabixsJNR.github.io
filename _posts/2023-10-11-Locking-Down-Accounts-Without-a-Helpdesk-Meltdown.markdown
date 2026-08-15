---
layout: post
title: "Locking Down Accounts Without a Helpdesk Meltdown"
date: 2023-10-11 13:45:00 +0100
icon: ti-lock
accent: "#D64040"
author: Darragh Crawley-Doyle
tags: Security, MFA, Entra ID, Rollout
---

**Rolling out mandatory multifactor authentication across an organisation is technically simple. Rolling it out without every single support request that week starting with "I can't log in" is the actual project.**

The technical side genuinely is straightforward. Enforce MFA, pick sensible exceptions for the handful of accounts and devices where a hard requirement would break something more than it protects, and the policy does its job. Where it gets harder is the couple of hundred people on the other end of that policy who now have to set up an authenticator app on a personal phone, for a work account, having never been asked to before.

I rolled it out in stages rather than flipping it on for everyone at once. Early groups first, close enough to IT that any confusion gets caught and fixed fast without turning into a wider panic. Clear guidance sent ahead of time explaining exactly what was changing and why, in plain language rather than a policy notice nobody reads properly. And a genuine allowance for the early weeks being messier than the steady state, with limited exceptions while people got their setup sorted rather than locking someone out of their own account on day one over a technicality.

The support load did spike, just not anywhere near as badly as it would have with a single flip-the-switch rollout. Most of it was the same handful of questions repeated: why is this happening, what do I actually need to do, what happens if I lose my phone. Answering those clearly and early, before they turned into tickets, did more to keep the rollout smooth than any part of the Entra ID configuration.

It's an easy trap to treat a security rollout as purely a technical task and let the people side sort itself out. It doesn't. The configuration is the easy 20 percent. Getting several hundred people through a change to how they log in, without it souring how they feel about IT generally, is the actual work.
