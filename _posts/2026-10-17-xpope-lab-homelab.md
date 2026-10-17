---
layout: post
title: "xPOPE-Lab: Building a Segmented Security Homelab on a Retired Workstation"
date: 2026-10-17
tags: [homelab, proxmox, networking, segmentation, security, lab, tailscale, wireguard]
image: /assets/img/posts/xpope-lab/network-map.svg
description: "A full tour of my Proxmox homelab: an air-gapped attack range, VPN-gatewayed containers, network-wide DNS filtering routed through a VPN, a dual-homed pivot box, and local AI inference, all on one hand-me-down Dell."
---

Most homelabs grow like weeds. A box here, a container there, and six months
later nobody including the owner can say what can talk to what. I built
xPOPE-Lab around one rule that keeps that from happening:

> **Anything I attack lives on a network that cannot reach anything I use.**

Everything below follows from that. This is the full tour: the hardware, how
the network is carved up, what runs where, and the design decisions that make it
a lab instead of a liability.

![xPOPE-Lab network map](/assets/img/posts/xpope-lab/network-map.svg)

## The hardware

**The host: Dell Precision T7910.** A dual-socket Xeon workstation with ECC
memory and a stack of drive bays. It was a hand-me-down, which is the ideal
price for a lab box. It runs **Proxmox VE 9.1** as the hypervisor, carving one
physical machine into every container and VM in this writeup. Proxmox calls the
node `popes`, reachable at `192.168.50.33:8006`.

**The daily driver: "Alert Raven."** My main desktop (`192.168.50.30`), running
a Ryzen 9 9950X3D and an RX 9070 XT 16GB. It does double duty as the lab's AI
box (more on that below). On the Tailscale side it shows up as
`newdesktop-pope`, same machine, two names.

**The edge.** Spectrum feeds a modem, which hands off to a **SonicWall TZ500**
acting as the gateway and firewall at `192.168.50.1`. Everything on the LAN
sits behind it. No services are exposed to the public internet (see the remote
access section for how I reach the lab from outside without opening a port).

**Wireless.** A **UniFi AP (AC HD)** broadcasts the `xPOPE-Lab` SSID and carries
the phones, laptops, and TVs. It is managed by a controller running inside the
lab rather than in somebody's cloud.

**ShadowApple.** A **GL-AR300M16** flashed into a DIY Wi-Fi Pineapple (OpenWrt
plus PineAP), admin at `172.16.42.1`. It is portable kit for wireless auditing,
not a permanent fixture, so on the map it hangs off the LAN on its own rather
than living inside the Proxmox host.

## Network architecture: the part that actually matters

Proxmox gives me three virtual bridges, and the split between them is the whole
design. Each is a separate broadcast domain with its own gateway on the host:

| Bridge | Subnet | Gateway | Internet? | Purpose |
| --- | --- | --- | --- | --- |
| `vmbr0` | `192.168.50.0/24` | `.1` (SonicWall) | Yes | Main LAN: real services and devices |
| `vmbr2` | `10.10.20.0/24` | `.1` (host) | **No** | Vuln-lab: isolated attack range |
| `vmbr3` | `10.10.30.0/24` | `.1` (host) | NAT only | Experiment net with internet, no LAN access |

The important detail is in how `vmbr2` and `vmbr3` differ, because it is the
difference between "separate subnet" and "actually contained."

**`vmbr2` is genuinely air-gapped.** On the host it has no physical bridge port
and no NAT rule. Nothing routes it to the LAN and nothing masquerades it out to
the internet. A box on `10.10.20.x` can talk to other boxes on `10.10.20.x` and
that is the entire universe it knows. When I pop a shell on something there, the
blast radius is a machine I built to be broken into.

**`vmbr3` is deliberately the opposite.** It carries a NAT rule so its traffic
masquerades out through the LAN to the internet, but it still sits on its own
subnet away from everything on `vmbr0`:

```bash
# On the Proxmox host, vmbr3 gets internet via NAT but stays off the LAN
iptables -t nat -A POSTROUTING -s 10.10.30.0/24 -o vmbr0 -j MASQUERADE
```

So `vmbr3` is for throwaway things that need to pull packages or phone home
without my trusting them anywhere near the main network.

## Core services (running 24/7)

These run as lightweight **LXC containers** rather than full VMs, because
single-purpose services do not need the overhead of a whole kernel each.

**AdGuard Home** (`192.168.50.200`). Network-wide DNS filtering. Every device on
the LAN resolves through it, so ad and tracker blocking happens once, centrally.
The non-obvious part: AdGuard's own default gateway is set to `.201`, the Mullvad
container, not the SonicWall. That means the resolver the whole network depends
on egresses through the VPN tunnel. DNS for the LAN goes out encrypted, filtered,
and from Mullvad's exit instead of my ISP's.

**Mullvad VPN gateway** (`192.168.50.201`). A container running raw **WireGuard**
out to Mullvad, acting as a gateway so I can route selected machines through the
VPN without installing a client on each one. Point a box's default route at `.201`
and its traffic tunnels out. Its own DNS is pointed back at AdGuard on `.200`, so
VPN-routed traffic still gets filtered instead of going out raw. A second Mullvad
account drives a separate container so that project's traffic exits on its own
identity.

**UniFi Controller** (`192.168.50.202:8443`). Manages the AP, SSIDs, and client
visibility. Running it as its own container means the controller survives reboots
of anything else and never depends on a vendor cloud.

## The attack range

On the isolated `vmbr2` (`10.10.20.0/24`):

- **Kali** (VM 104) as the offensive workstation.
- **HTBPARROT** (VM 107), a Parrot build I use for HackTheBox-style work.
- **Metasploitable 2** (VM 106) as the intentionally vulnerable target.

Kali and HTBPARROT are **dual-homed on purpose.** Each has one NIC on the main
LAN (`vmbr0`) and a second NIC on the attack net (`vmbr2`). I manage and update
them from the LAN side, and I attack across the isolated side. Metasploitable
has both of its interfaces on `vmbr2` only, so the target has no path off the
range at all. That is the model: the attacker can reach in, the target cannot
reach out.

This is the same workflow I run on live HackTheBox boxes, except the target is
local, offline, and air-gapped from everything real. If I break it, I redeploy
it in minutes.

## The rest of the VM zoo

A pile of lab VMs live on the LAN (`vmbr0`), powered off until I need them, so
they cost nothing but disk when idle:

- **Parrot** (100), another offensive distro build
- **TempleOS** (101), because why not
- **testthesenutes** (102), a scratch box with an honest name
- **WinSrv2022** (103) and **Win10** (105), Windows targets and AD practice
- **Cs16serv** (202), a Counter-Strike 1.6 dedicated server sitting on the
  NAT'd `vmbr3` experiment net at `10.10.30.10`
- **InstaBOT** (201), an old automation container, parked

## Local AI inference over the LAN

Alert Raven runs **LM Studio** serving local models on the LAN at port `1234`
with an OpenAI-compatible API. Any machine in the lab, including boxes on the
attack side, can point at that endpoint for inference instead of calling a cloud
provider. It keeps experiments self-contained and keeps the data on my own
hardware, which matters when the "data" is lab traffic I would rather not ship
to a third party.

## Remote access with zero open ports

**Tailscale** stitches the machines that matter into a private mesh: the
desktop, the Proxmox host, a couple of Parrot boxes, a Windows server, and more.
Eleven nodes in total. I can reach the lab from anywhere without forwarding a
single port on the SonicWall. Nothing is exposed to the public internet, there
is no public-facing admin panel, and if a device is not on the tailnet, it does
not get in. That is the whole security pitch for remote access: the attack
surface from the outside is a WireGuard handshake, not a login page.

## Design principles, if you are building one

1. **Segment first, add second.** The bridges and their isolation rules came
   before the interesting VMs. It is far easier to drop a vulnerable box onto a
   network that already cannot reach your real stuff than to retrofit isolation
   after something owns you.
2. **Decide internet access per segment, explicitly.** `vmbr2` gets none,
   `vmbr3` gets NAT'd internet but no LAN. Writing that rule down (literally, as
   an iptables line) means the posture is intentional, not accidental.
3. **Route the services that leak the most through the VPN.** DNS is the chatty
   one, so AdGuard egresses through Mullvad and filters on the way.
4. **Close the front door entirely.** Tailscale instead of port forwards.
5. **Let targets be cattle, not pets.** Everything on the attack range is
   redeployable in minutes, so breaking it is free.

## What's next

A few deep-dives I want to break out into their own posts:

- The Mullvad-over-WireGuard gateway with the AdGuard DNS routing, as a how-to.
- A full attack-range walkthrough: deploying a target and running a clean
  engagement end to end on `vmbr2`.
- The ShadowApple build and a wireless assessment done entirely inside the lab.

If you take one thing from this: start with the segmentation. Every other piece
gets easier to add once the things you break and the things you use physically
cannot see each other.
