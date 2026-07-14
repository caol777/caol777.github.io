---
layout: post
title:  "How We Prep for CCDC & CPTC"
tags: [competitions, blue team, red team]
---
How We Prep for CCDC & CPTC
---

Every year our club throws itself at two very different beasts: the **Collegiate Cyber Defense Competition (CCDC)**, where you defend a network you didn't build against a professional red team, and the **Collegiate Penetration Testing Competition (CPTC)**, where *you* are the red team and have to write a professional report at the end. They pull on opposite muscles, but the way we prepare for them rhymes. This post is the playbook we've landed on after a few seasons of trial and error.

---

## The Two Games Are Not the Same

It's worth being blunt about this up front, because people show up to their first practice assuming "cyber competition" means one thing.

**CCDC is a defense grind.** You inherit a pile of misconfigured, out-of-date boxes running services the scoreboard checks every few minutes. The red team already has a foothold. Your job is to keep services up, kick attackers out, and answer business "injects" (tasks from the fake company's management) — all at the same time. Points come from uptime and injects; you *lose* points when a service goes down or a red-teamer proves they own you.

**CPTC is an offense-plus-communication grind.** You're a consulting firm hired to assess a client. Finding the vulnerabilities is only half of it. The teams that win are the ones who write a clean, prioritized, executive-readable report and can brief it like real consultants. Raw shells don't score — *documented, contextualized findings* do.

The overlap is the important part: **both reward preparation, teamwork, and not panicking.** So that's what we drill.

---

## Roles Beat Heroes

The single biggest lesson: a team of six generalists all touching the same box loses to a team where everyone owns a lane. We split roughly like this.

- **Linux lead** — owns the *nix boxes, hardening, and service recovery. (This is usually my lane.)
- **Windows / AD lead** — owns Active Directory, GPOs, and the Windows fleet. Marcos runs our PowerShell side.
- **Network/firewall** — owns segmentation, ACLs, and pcap when something looks weird.
- **Injects / comms** — owns the business tasks and the scoreboard clock so the technical people don't have to context-switch.
- **Captain** — doesn't touch keyboards much during crunch; tracks what's down, what's owned, and what's due.

For CPTC we flip the same idea toward offense: someone owns recon, someone owns exploitation, and — critically — **someone owns the report from minute one**, not the last hour.

---

## The Repo Is the Real Weapon

The thing that changed our results the most wasn't a tool, it was a **central competition repository** every member clones before the event. It holds:

- Hardening scripts for Linux and Windows (more on these in a separate post)
- Enumeration one-liners and checklists
- A known-good baseline of configs to diff against
- Inject templates (incident report, memo, remediation plan)
- A password-change playbook so nobody fat-fingers themselves out of a box

The point is that during the competition, **nobody should be writing a script from scratch or googling syntax.** Everything you might reach for is already in the repo, tested, and documented. We rehearse pulling it down and running it cold so game day is muscle memory.

---

## A Practice Cadence That Actually Works

We run practice on a simple loop:

1. **Skills week** — everyone drills fundamentals on free platforms. Linux on OverTheWire Bandit and pwn.college, Windows/PowerShell on UnderTheWire, web on PortSwigger, and general CTF reps on picoCTF. (These all live in our intern guide.)
2. **Scenario week** — we spin up a scenario on our own infrastructure and *time-box* it. For blue team that means "here's a rooted box, harden it and keep the service up." For red team that means "here's a subnet, get domain admin and write it up."
3. **Debrief** — the most underrated hour. What went down, who got locked out, which script failed, what we'd automate next time. Every debrief adds something back to the repo.

We host the scenario side on a **Proxmox server** I've been building out so we're not dependent on someone's laptop or a paid range. Being able to snapshot, break, and reset a whole environment on demand is what makes the scenario/debrief loop repeatable.

---

## Blue-Team Muscle Memory (CCDC)

A few things we hammer until they're reflexes:

- **Change every default and inherited credential first.** Before hunting for the attacker, close the front door. Do it in a way that won't lock *you* out — coordinate with the captain.
- **Establish a baseline, then hunt the delta.** `who`, `ss -tulpn`, running services, cron/scheduled tasks, and new users. Attackers hide in the diff from normal.
- **Keep the scored service up above all else.** A perfectly hardened box that's offline still scores zero. Recovery speed beats perfection.
- **Persistence is a hydra.** A stolen box usually has more than one way back in — cron jobs, extra sudoers lines, rogue accounts, SSH keys. Assume there's a second one after you find the first.

---

## Red-Team Muscle Memory (CPTC)

- **Enumerate like it's the whole game — because it mostly is.** Most footholds come from careful recon, not exotic exploits.
- **Write as you go.** Screenshot the command, the output, and the impact *at the moment you get it.* Reconstructing findings at 11pm is how teams lose points.
- **Think client, not scoreboard.** "I got a shell" is not a finding. "An unauthenticated SMB share exposed plaintext credentials that led to domain compromise; here's the business risk and the fix" is a finding.

---

## The Mindset

Underneath the tooling, the teams that do well share one trait: they stay calm and keep working the process when things go sideways — and they *will* go sideways. Someone will get locked out. A service will drop. A shell will die. The prepared team shrugs, checks the repo, and keeps moving. That composure is a skill, and the whole practice cadence above exists to build it.

If you're just starting a club or a team, don't over-index on the flashiest tools. Build the repo, assign the roles, run the loop, and debrief honestly. The results follow.

---

*Questions or want the club resources? Hit me on [LinkedIn](https://www.linkedin.com/in/shaamad-allison) or check the [intern training guide](/2026/02/01/Guide.html).*
