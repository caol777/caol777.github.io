---
layout: post
title:  "Automating Competition Prep: Our Club Script Repo"
tags: [automation, bash, python, powershell]
---
Automating Competition Prep: Our Club Script Repo
---

One of the projects I'm proudest of is the **central automation repo** our club uses for competitions. The idea is simple: when the clock starts, nobody should be typing the same fifteen commands they typed at last month's practice. If we've done a task twice, it belongs in the repo. This post walks through how it's organized and the philosophy behind it — not to hand out copy-paste answers, but to show how thinking in scripts changes how you compete.

I write the **Bash and Python** side, and our Windows lead Marcos owns the **PowerShell** integration so the Windows fleet gets the same treatment.

---

## Why Automate At All?

In a defense competition you're doing three things at once: hardening boxes, hunting an active attacker, and answering business injects. Your attention is the scarcest resource in the room. Every task you can turn into a single command is attention you get back for the problems a script *can't* solve.

The rule we settled on: **a script should do the boring, repeatable, error-prone stuff so a human can do the judgment stuff.** Changing 40 passwords is boring and error-prone — automate it. Deciding *whether* a weird cron job is malicious is judgment — keep that human.

---

## How the Repo Is Laid Out

Roughly:

```text
comp-repo/
├── linux/
│   ├── enum/          # read-only situational awareness
│   ├── harden/        # firewall, ssh, password policy, etc.
│   └── recover/       # restart scored services, restore baselines
├── windows/
│   ├── enum/
│   ├── harden/
│   └── recover/
├── baselines/         # known-good configs to diff against
├── injects/           # report + memo templates
└── README.md          # run order + gotchas
```

The split between **enum**, **harden**, and **recover** matters. Enumeration scripts are strictly read-only — you run them the moment you get on a box and they change nothing. Hardening scripts change state and are the ones you have to run carefully and coordinate with the team. Recovery scripts get the scored service breathing again when it drops.

---

## Principle 1: Enumeration Before Anything

The first script anyone runs is a read-only situational-awareness sweep. In Bash that's the usual suspects wrapped so the output is consistent and saved to a file:

```bash
#!/usr/bin/env bash
# enum/quicklook.sh — read-only. Changes nothing.
out="enum_$(hostname)_$(date +%s).txt"
{
  echo "=== users with shells ==="
  grep -vE '/(nologin|false)$' /etc/passwd
  echo "=== logged in now ==="
  who
  echo "=== listening ports ==="
  ss -tulpn
  echo "=== cron ==="
  for u in $(cut -f1 -d: /etc/passwd); do crontab -l -u "$u" 2>/dev/null; done
  echo "=== sudoers extras ==="
  grep -vE '^#|^$' /etc/sudoers
} | tee "$out"
```

Nothing clever here — the value is that it's *the same every time*, the output is saved for the debrief, and no one forgets a step under pressure.

---

## Principle 2: Idempotent Hardening

A hardening script has to be safe to run twice. If re-running it breaks something or double-applies a rule, it's worse than no script. So they check state before they change it, and — importantly — **they never lock the operator out.** Password-rotation tooling, for example, always skips the account you're currently using and prints exactly what it changed so the team can log it.

We keep hardening steps small and single-purpose (one script enables the firewall, one enforces the password policy, one fixes SSH config) rather than one giant "secure everything" button. When something goes wrong you want to know *which* change did it.

---

## Principle 3: Cross-Platform Parity with PowerShell

The Windows side mirrors the Linux layout so the team's mental model is identical no matter which box they're on. Marcos's PowerShell enumeration produces the same kind of read-only snapshot:

```powershell
# windows/enum/QuickLook.ps1 — read-only
"=== Local users ==="            ; Get-LocalUser | Select Name,Enabled
"=== Admins ==="                 ; Get-LocalGroupMember Administrators
"=== Listening ports ==="        ; Get-NetTCPConnection -State Listen |
                                     Select LocalPort,OwningProcess
"=== Scheduled tasks ==="        ; Get-ScheduledTask |
                                     Where State -ne 'Disabled' |
                                     Select TaskName,TaskPath
```

Same idea, same output shape, same discipline. Someone who learned the flow on Linux can read the Windows output without re-learning anything.

---

## Principle 4: Everything Gets Logged

Every script tees its output to a timestamped file. This does two things. During the event, it's your paper trail — when the scoreboard says a service dropped, you can see what you ran and when. After the event, those logs *are* the debrief. We read them, find the step that was slow or manual, and that becomes next week's automation.

---

## The Payoff

The scripts themselves aren't the point. The point is that building them forces you to actually understand the systems — you can't automate hardening a service you don't understand. And when game day comes, the team spends its energy on the attacker and the injects instead of re-deriving syntax under a clock.

If you run a club, start your own repo this week. Make the rule "if we did it twice, it goes in the repo," and let it grow one debrief at a time.

---

*The public club repos are on my [GitHub](https://github.com/caol777) — the Proxmox setup and CCDC scripts live there.*
