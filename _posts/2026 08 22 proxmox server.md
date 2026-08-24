---
layout: post
title: "Building the Club a Proxmox Server: From Bare Metal to Live Competition Ranges"
date: 2026-08-22
tags: [projects, proxmox, homelab, infrastructure, ccdc, blue team]
---

For a while, our club practiced on whatever we could scrape together. Someone's spare laptop, a cloud instance we'd spin up and kill before the free credits ran out, a range that only existed until whoever built it needed their machine back. It worked, but it wasn't *ours*. There was no permanent home for the labs, the practice environments, or the infrastructure we kept rebuilding from scratch every single time.

So I built one. This is the story of standing up a Proxmox server for the club: from configuring the bare-metal networking, through clustering two nodes and setting up remote access, to the part I'm proudest of, actually getting real WRCCDC competition environments running on it that the whole team can attack from home.

The full step-by-step with screenshots and commands lives in the [GitHub repo](https://github.com/caol777/Club-proxmox-server). This post is the why and the highlights. (If you found this from my resume, this is the project I list there as *FAU Cyber Range Infrastructure*.)

## Why Build It At All

The goal was simple to say and hard to do: give the club a persistent, self-hosted virtual infrastructure that we own and control. Somewhere to host environments for penetration testing, cyber defense, and SOC practice, and have them still be there next week, next semester, next competition season.

Proxmox was the obvious pick. It's free, open-source, and lets me run a whole fleet of VMs and containers on our own hardware, carve up the resources however we need, and manage it from one web console. No per-hour billing, no borrowed laptops, no "sorry, I needed my machine back." Just a real server that's always on and always ours.

## Bare Metal and Networking

The first job was the least glamorous and the most important: networking. I set the Proxmox host's IP in `/etc/network/interfaces` and `/etc/hosts` to run off the gateway of the switch in our room, then built out a separate LAN so our VMs could talk to each other in isolation.

Then came pfSense as our router and firewall, running as its own VM with bridges for both the WAN and the internal LAN. Getting the bridging right here is what makes everything downstream work, so it was worth being careful.

## Storage, and a Drive That Wouldn't Die

Adding storage taught me a lesson I won't forget. Wiping one SSD with `wipefs` was easy. The second one refused, because it had an old Linux LVM sitting on it. I had to deactivate and remove the logical volumes with `lvchange -an` and `lvremove` before the disk would let go. Once it was clean, I added both drives into Proxmox.

I did the first drive the hard way from the command line, then found out the web UI does the whole thing in about four clicks. Both ways are in the repo, because knowing the CLI path is useful even when the GUI is faster. We ended up with dedicated storage for VMs, snapshots for quick rollbacks and box resets, templates for automation, and a backups pool.

## Two Nodes, One Cluster

To get more capacity, I joined a second server as a Proxmox cluster. This part was genuinely painless: create the cluster on the first node, copy the join info, paste it into the second. Suddenly both machines showed up under one datacenter with all their storage pooled together. (The one gotcha: if a node join ever goes sideways, you fix it by hand in `/etc/pve/corosync.conf`. Learned that one the hard way too.)

## Letting the Whole Club In

A server nobody can reach is useless. Since we're on the free Proxmox repos, step one was actually fixing the repositories so the box could get updates at all, which means disabling the enterprise repo and adding the no-subscription one.

Then remote access. Instead of poking holes in the firewall, I set up a Tailscale VPN running inside a container on the server, advertising the server's subnet across the tailnet. Now any club member can sign in and reach the environments from home like they're sitting in the room. This is the thing that turned it from "my server" into "the club's server."

## The Payoff: Live WRCCDC Ranges

This is where it stopped being infrastructure for its own sake and started being a training ground.

WRCCDC publishes their competition images, so I pulled the full 2026 invitational dump straight onto the server. Getting them to actually *run* took some doing. The images have to be renamed to `.vma` so Proxmox recognizes them, then either extracted and provisioned by hand or restored through the UI. I wrote a provisioning script that builds each VM (CPU, RAM, disk import, cloud-init networking) so I could stand up a whole team's worth of machines without clicking through the wizard fifty times.

These environments aren't simple, either. One of the WRCCDC images I hosted brought up a Kubernetes control plane along with several services running in Docker, so getting it healthy on the server meant dealing with a real containerized stack, not just a handful of Windows boxes. That's exactly the kind of environment worth practicing against, because it looks like something you'd actually have to defend.

The nastiest part was the networking after restore. The competition environment expects 1:1 NAT, which I could not get working cleanly no matter what I threw at the iptables NETMAP rules. So I built a workaround: a Tailscale container with one foot in the real WAN and one in the isolated competition LAN, forwarding between them and advertising the competition subnet. It simulates the 1:1 NAT behavior well enough that the environment runs and the team can practice against it remotely.

That's the whole point of the build in one sentence. We now have real, retired competition environments running on our own hardware, reachable from anywhere, that we can reset and re-attack as many times as we want.

## When It Actually Solved a Problem for Someone

The moment the server proved its worth wasn't a competition. It was a student who couldn't get their virtual machines running on their Mac.

That's a wall a lot of people hit early, and it's exactly the kind of thing that quietly pushes someone out of the field before they even get started. Instead of leaving them to fight their laptop, I stood up a CyberPatriot practice image for them directly on the server, just standard Proxmox, no local setup required. They could log in over the VPN and start working, with no hardware limits and nothing to install on their end.

That's the version of this project I care about most. The clustering and the NAT workaround are fun engineering, but the server earning its keep looks like a member who was stuck getting unstuck, and getting to actually learn.

## What's Next

The foundation is solid, so now it's about building on top of it. This semester the focus is automation with Ansible: taking the provisioning I've done with Ludus and scripts so far and turning a full practice range into a single command instead of an afternoon of setup. The goal is that any environment, for any member, is one push-button away. Beyond that, more environment types for the blue-team and SOC side, not just CCDC-style defense.

If you're standing up something similar, the [repo](https://github.com/caol777/Club-proxmox-server) has every command and screenshot. And if you've actually gotten 1:1 NAT working on a restored WRCCDC image without the VPN workaround, please tell me how, because that one still bugs me.
