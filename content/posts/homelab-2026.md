+++
title = 'Homelab 2026 Update'
date = 2026-02-02T12:00:00+00:00
draft = false
tags = ['homelab', 'self-hosting', 'infrastructure', 'backups']
categories = ['tech']
+++

I adore self-hosting. It got me into my current career path and keeps me there. I love control and ownership of my data — though my OCD means I'm constantly consolidating and simplifying. Recently I've been focused on minimalism: reducing mental overhead across infrastructure, data, and digital presence. Here's the current state.

## Network

Unifi everything. Simple and it just works.

- **Firewall:** UDM Pro
- **Switch:** USW-24-PoE
- **APs:** U6-LR, U6-Lite
- **Living room:** USW-Flex Mini

I'm a huge fan of VLANs. Security is good, right? ^_^

VLANs for segmentation:

| VLAN | Purpose |
|------|---------|
| 1 | Management (infrastructure only) |
| 10 | Trusted (my devices, full access) |
| 20 | Less Trusted (family, limited) |
| 30 | IoT (isolated) |
| 100 | Servers (accessible from trusted) |

## Services

I used to host a bunch of services, but now I like to keep things lean. I tried trimming the fat, and here is what I absolutely rely on.

- **Bitwarden Lite** - password manager, local network only. This is a new one. I've used Bitwarden Cloud for a few years, but my paranoia got me to finally self-host Bitwarden when their self-hosted unified edition came out of beta and rebranded to lite. (Yay for SQLite)
- **Pi-hole/Unbound + Redis Cache** - DNS ad-blocking/recursive local DNS server, HA pair via keepalived.
- **Traefik** - reverse proxy
- **Grafana + Prometheus** - monitoring mainly for general system stats and monitoring my Unbound statistics. I built a custom dashboard and application based on this. [unbound-dashboard](https://github.com/ar51an/unbound-dashboard)
- **Matrix** - I've been hosting this forever. I mainly use it to consolidate my Signal chats and Matrix stuff in one application (Element).
- **Homepage** - Pretty simple home dashboard that allows me to quickly access my services.
- **Nebula-Sync** - Keeps my Pi-holes in sync when I update DNS records and blocklists, syncs every 15 minutes.

I have tried hundreds of other self-hosted applications, but these are the ones that stuck.

## Hardware

- **Arch Workstation** (9900X3D, 9070XT, 64GB RAM) - Just built a new rig, and this thing rips. I've been running Arch for years now. I might eventually switch to NixOS, but for now Arch does everything I need and more. I also might have a plan to go to macOS (yes, FOSS people can crucify me now). This fits with my ideology of consolidation and simplicity.

- **Synology DS218+** (2x 12TB shucked drives, SHR1) - Honestly, this does the job. I just use this mainly for cold-ish storage and backups. Don't need anything too performant and I've had it forever. Synology was attempting to lock people into only using their drives, but reversed this stance. Still leaves a horrible taste in my mouth. Will re-evaluate TrueNAS when the time comes.

- **Intel NUC 14 Pro** (32GB RAM, 2TB NVMe) - My one and only Proxmox host, currently. All my services are hosted here.

- **Raspberry Pi 4** (4GB) - I used to use this for services before I got the NUC. Currently only being used for redundant Pi-hole/Unbound server with Keepalived for failover.

- **Hetzner VPS** - Hosts my blog and Matrix server, does the job. I love Hetzner for their pricing and performance. They've never let me down.

- **OVH Dedicated Server** - I have this server to host some other things, notably a full archival Monero node. Still deciding what else I should do with this.

- **iPhone 14 Pro Max** - Yeah... ugh, just works. Way too heavy though. GrapheneOS is cool and shit, but I don't have the time to fuck around with it. Knowing I might go macOS in the future, this also keeps me on the iPhone.

## Backups

Arguably the most important of the bunch. I used to use Borg, but switched to Restic as I find it simpler with more backend support. Some cool projects to check out if you don't want to use the CLI are [Autorestic](https://github.com/cupcakearmy/autorestic) and [Backrest](https://github.com/garethgeorge/backrest).

Everything flows through the NAS to offsite:

```
Workstation ──rsync──→ NAS ──Restic──→ B2
```

| Job | Schedule | Tool |
|-----|----------|------|
| Workstation → NAS | Nightly | rsync (-a --delete --backup) |
| Proxmox VMs → NAS | Nightly | Proxmox Backup |
| Bitwarden → NAS | Nightly | sqlite3 + tar |
| NAS → B2 | Nightly | Restic |

Workstation backup runs via systemd user timer with `Persistent=true` — if I miss the nightly window (workstation asleep), it catches up on next boot. The `--backup` flag keeps deleted files for 7 days before purging.

The key: my workstation has no B2 credentials. If it gets compromised, offsite backups are untouched. Monitoring via healthchecks.io alerts me if any job fails.

Recovery is paper-based — a read-only B2 key and passphrase get me back to zero.

## What's next?

Good question, ha. The setup works, I don't have too many issues — but I need a laptop. I am trying to get away from constantly working at my desk. Yes, in theory this should be better for my posture, but I find that moving around more helps more than a highly ergonomic setup. I already have a lovely Steelcase Leap V2 — but I suffer from neck and posture issues, so I think this will help. When looking at laptops, the state is bleak. Apple is killing the game currently, and the M4/5 MacBook Pros just look too promising. I have been eyeing [AeroSpace](https://github.com/nikitabobko/AeroSpace) (an i3-like window manager) for macOS. I can't imagine going back to a floating window manager.

I will continue to add to this as my setup changes, so check back at another date. :)

*Last updated: February 2026*
