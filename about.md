---
title: About
permalink: /about/
layout: page
comments: false
excerpt: >-
  Benham Haynes (xPOPE): hairstylist turned sysadmin, and the guy
  running a student hacking club on the side.
---

I'm Pope, aka Benham Haynes. I build things, break things, and write about how both went. (Want the formal version? There's a [resume]({{ "/resume/" | relative_url }}) for that. This page is just me.)

Right now I'm a Systems Administrator at a managed service provider, I lead a student security club, and I'm about to start a B.S. in Cybersecurity at WGU. The long game is a career that lives on both sides of security: offense that finds the gaps, defense that closes them.

## Before any of this

I didn't come up through the usual pipeline. For four-plus years I was a professional hairstylist, and I still cut hair part-time at a salon in Costa Mesa. Nobody tells you that cutting hair and running IT support use the same muscle: someone is stressed, they don't care about the technical details, and your job is half fixing the actual problem and half making them feel like it's handled. Turns out that's most of helpdesk work too.

At some point the curiosity about how networks actually worked stopped being a hobby and became the plan. I went back to school and built the skill set from the ground up.

## The day job

I handle the real mess of supporting multiple client networks at an MSP: SonicWall firewalls, Microsoft 365 and Exchange Online, Copilot policy decisions nobody else wants to make, IPv6 hardening so a "new" protocol doesn't quietly open a hole, and running Gophish campaigns to see who still clicks the bad link. The project I'm proudest of there is a captive portal I built for a hospitality client on IronWifi and Ruckus hardware, SMS login and all. It broke in a way that took real digging to explain: two layers of MAC/IP encryption stepping on each other. Finding that is exactly the kind of problem I like chasing.

On the side I've been building an AI help-desk agent with a privacy gateway at the center of it, because "just use ChatGPT on client data" is a bad answer. Real identifiers never leave the building; the model only ever sees reversible, scoped pseudonyms; secrets get redacted for good; and if anything looks uncertain, it fails closed instead of guessing. Every request gets logged without ever storing the mapping itself.

## Competing and breaking stuff on purpose

I compete when I can. SoCal Cyber Cup, CIRCUS (I captained Forensics and Incident Response), HackCCDC defending live boxes against an actual red team, and a steady diet of Hack The Box and TryHackMe. I write up the machines I crack because if I can't explain how I got in, I don't actually understand it yet.

## SCBBG

I lead SCBBG, the student cyber club at Saddleback. The name says the whole mission: we build, and we break.

## The homelab

Most of what I actually know, I learned by breaking it at home first. The lab runs on a Proxmox box alongside a second server on Hyper-V, built like a small business network: a real SonicWall at the edge, self-hosted UniFi, AdGuard for DNS-wide filtering, a WireGuard gateway routed through Mullvad, Tailscale instead of open ports, and a fully air-gapped attack range where Kali and Parrot get to misbehave against Metasploitable without touching anything real. I wrote up the [full tour]({{ "/xpope-lab-homelab/" | relative_url }}) if you want the details. There's also a field repair toolkit, a jailbroken test phone for mobile security work, and a couple of dumb-fun automation projects I'm not quite ready to talk about yet.

## And before all that, QA

I did freelance QA for a web app called PrayerConnect, running structured test passes and writing bug reports people could actually reproduce. QA is where I learned to look for the edge case nobody thought to check, which is basically the whole job in security too.

## What it comes down to

I don't trust a system until I know how it fails. Firewall rule, captive portal, AI pipeline, a VM in my lab, doesn't matter, I want to know where the seam is. I bring the same people skills from a decade of client-facing work into every ticket, every team, and every competition.

If you're building something interesting, or just want to talk labs, CTFs, or privacy-first AI, [hit me up](https://www.linkedin.com/in/benhamhaynes){:target="_blank" rel="noopener"}.
