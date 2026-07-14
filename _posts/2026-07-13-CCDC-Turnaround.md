---
layout: post
title:  "The CCDC Turnaround: From Last Place to Top 15"
tags: [competitions, blue team, ccdc, proxmox]
---

I joined the FAU Cybersecurity Club in February of 2025. My first real event with them was CCDC, and I'll be honest with you: we got last place. Dead last. But instead of being discouraged, I walked away from that competition seeing something else entirely — potential. This club had smart, motivated people and no ceiling on how good it could get. Last place wasn't a verdict, it was a starting line.

So I made a decision. If we wanted to get better, we had to compete more. A lot more.

## Signing Up for Everything

I started signing us up for as many competitions as I could find. That meant reaching out to other collegiate cybersecurity clubs, introducing myself, asking what they were competing in, and figuring out how to get our members a seat at the table. Along the way I made friends across a bunch of different schools' programs — people who were just as obsessed with this stuff as we were.

That networking paid off in ways I didn't expect. It got our members into more competitions, more bootcamps, more projects, and more homelabs. Every event was another chance to learn something we couldn't have learned sitting in a classroom, and every new connection opened another door. We weren't just grinding — we were growing together and getting closer as a team.

## Horse Plinko Changed Everything

We did a lot of competitions before this, but nothing made the whole thing *click* like UCF's Horse Plinko.

We had an absolute blast, and more importantly we met a ton of cool fellow cyber nerds who genuinely wanted to see us do well. People handed us tips that shaped how we approached every competition after that. It was the moment where the abstract idea of "getting better" turned into concrete strategy.

On the technical side, Horse Plinko is where our defense actually came together. I was administering high-availability Apache, DNS, and MySQL services and holding strict uptime SLAs in the 80–100% range while the scoring engine hammered us. We ended up placing **5th**, and I got to mentor a peer team that landed **3rd**. Seeing a team I coached beat us was one of the proudest moments I've had in this whole journey.

## The Lessons That Hurt (WRCCDC)

We also ran WRCCDC as practice, and that one taught us a lesson the hard way: **dependencies matter.** Especially when your Linux boxes are AD-joined.

If you've never felt the pain of "fixing" one service and watching three others fall over because they all depended on a domain component you just touched, you haven't lived. That experience completely reframed how we think about hardening — you can't treat boxes as islands. You have to understand the whole environment before you start changing things, or you'll take yourself down faster than any red team could.

## Building the Range and Automating the Grind

To practice all of this without renting infrastructure or relying on someone's laptop, I built out a cyber range on **Proxmox**, with pfSense handling routing and Tailscale for remote access. I provisioned labs through Ludus and Ansible so we could spin up CCDC-style active-defense scenarios and practice live hardening *while under attack* — which is the only way it ever actually happens.

The other big win was automation. I wrote Bash scripts that took our initial hardening time from about **50 minutes down to 5** — a 90% cut. That's not a vanity metric; in a competition, those 45 minutes are the difference between closing the front door before the red team walks in and spending the whole round chasing them around your own network.

## Where We Landed

The results speak for themselves. From that last-place CCDC finish, we clawed our way to **15th out of 45 in the SECCDC Qualifiers** (and 7th in Incident Response), **7th out of 28 at the Cal Poly Mock CCDC**, and **5th at NCAE CyberGames** with me as Linux team captain.

But the placements aren't really the point. The point is that a group of people decided to take last place personally, and turned it into a reason to get closer, work harder, and lift each other up. That's the turnaround I'm actually proud of.
