---
layout: post
title: "Debugging a Guest WiFi Captive Portal: Two Layers of Encryption Fighting Each Other"
date: 2026-11-17
tags: [networking, wifi, captive-portal, troubleshooting]
description: "Deploying an SMS-authenticated guest WiFi captive portal on cloud-managed access points, and the debugging process behind a sign-in failure caused by two systems independently encrypting the same client identifier."
---

I designed and deployed a guest WiFi captive portal for a client, SMS-based sign-in, a custom splash page, the works, built on the IronWifi captive-portal platform sitting in front of Ruckus-managed wireless access points. It worked in testing. Then it didn't work for real guests, intermittently, in a way that made no sense until I found it.

## The setup

The architecture is standard for this kind of deployment: wireless access points broadcast an open guest SSID, any client that associates gets redirected to a captive-portal landing page instead of the internet, the guest enters a phone number, receives a code by SMS, and once verified gets released onto the network for a configured session length.

The two systems involved are the Ruckus AP controller, which manages the access points and enforces the captive-portal redirect at the network layer, and the IronWifi captive-portal platform itself, which owns the splash page, SMS verification, and the actual "let this device online" decision. They talk to each other through a standard mechanism: the controller tells the portal who's trying to connect, using the client's MAC address and IP as the identifier, and the portal tells the controller who's now allowed through.

## The wall

Some guests would authenticate successfully, get the "you're connected" confirmation on the splash page, and then just... not get online. Refreshing did nothing. It wasn't every device, and it wasn't consistent, which is the worst kind of bug to chase because it rules out the obvious explanations first.

I started where you'd expect: checking that the SMS codes were actually being delivered and verified correctly on the portal side. They were. Session logs on the portal showed successful authentications for the exact clients that were stuck. So the portal thought it had authorized the device. The problem was downstream of that.

Next was the controller side: pulling live client tables and checking whether the authorized MAC addresses were actually being granted access at the AP level. Some were, some weren't, and there was no obvious pattern by device type, OS, or signal strength, which pointed away from a wireless-layer issue and toward something in how the two systems were identifying the same client to each other.

## What was actually happening

The controller had a setting that encrypted MAC and IP addresses before including them in the authorization exchange with the portal, a privacy feature, reasonable on its own. The portal, independently, was also doing its own encoding of client identifiers for its session tracking.

When both of those were active at once, the identifier the portal sent back to authorize a session didn't match what the controller expected to see, because each side had transformed the value differently and neither was aware the other was doing it too. For some clients, by coincidence of how the encodings interacted, the identifiers happened to still line up well enough to pass. For others, they didn't, and the device stayed stuck in limbo: authenticated according to the portal, invisible according to the controller.

That's why it looked intermittent. It wasn't random, it was deterministic per client, just not in a way that was visible from either system's logs in isolation. You had to be looking at both sides of the handshake at once to see the mismatch.

## The fix

The real fix was disabling the redundant encryption layer on the controller side and letting the portal own client-identifier handling on its own, since it already had to do that correctly to issue valid sessions in the first place. Once the controller stopped re-encoding values the portal was already handling, the authorization handshake matched consistently and the stuck-session behavior disappeared across every device type that had been affected.

## What this one taught me

The honest lesson here isn't about captive portals specifically. It's that when two systems are supposed to agree on an identifier and only sometimes do, the bug almost never lives entirely inside one of them. You have to look at the handshake, not just each side's internal state, because each side can be completely correct about its own logs and still disagree with the other. That's a debugging instinct that's come in handy well outside of wireless networking.
