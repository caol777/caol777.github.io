---
layout: post
title:  "Competition Write-Up Template (Fill Me In)"
tags: [template, competitions]
---
Competition Write-Up Template
---

> **Heads up:** This is a reusable skeleton for your next competition write-up. Copy this file, rename it to `YYYY-MM-DD-Event-Name.md`, replace the bracketed prompts, and delete this note. Everything in **[brackets]** is a prompt for you, not final text.

**[One-sentence hook: what was this competition and how did it go? e.g. "A weekend CTF where I chased a single web bug for six hours and it was worth it."]**

---

## 1. Overview

- **Event:** [Name of competition]
- **Type:** [CTF / King of the Hill / CCDC-style defense / CPTC-style pentest / other]
- **Date:** [Month Day, Year]
- **Format:** [Solo / team of N, remote / in-person, categories]
- **Result:** [Placement, or what you got out of it]

**[Two or three sentences on the setup: what the environment looked like, how you connected, what "winning" meant — uptime, flags, a report, etc.]**

---

## 2. Recon / Getting Oriented

**[What did you look at first? Scans, given credentials, the scoreboard, the rules. Show the command and what it told you.]**

```bash
# example
nmap -sC -sV [target]
```

**[Screenshot placeholder — drop the image here]**

**[What did this tell you, and what did you decide to chase because of it?]**

---

## 3. The Main Path

**[This is the heart of the write-up. Walk through what you actually did, step by step. For each step: the command, the result, and *why* you did it. The "why" is what makes a write-up worth reading — anyone can paste commands.]**

```bash
# step 1: [what and why]
```

```bash
# step 2: [what and why]
```

**[Call out the moment it clicked — the credential you found, the misconfig you spotted, the pivot that worked.]**

---

## 4. Roadblocks

**[What slowed you down or fooled you? Dead ends are often the most useful part for a reader, because they'll hit the same ones. Be honest about what you got wrong.]**

---

## 5. Defense / Cleanup *(if applicable)*

**[For KOTH or defense events: how did you hold what you took? Flag persistence, kicking other users, hardening. For a pure CTF, delete this section.]**

---

## 6. Takeaways

**[Three or four bullets: what you'd do faster next time, a tool or trick you'll keep, a concept you finally understood. This is the part *you'll* reread before the next event.]**

- [Takeaway 1]
- [Takeaway 2]
- [Takeaway 3]

---

## 7. Result

**[How it ended, and a sentence of reflection. Placement, what you're proud of, what's next.]**

---

*Thanks for reading. More write-ups on the [blog archive](/archive.html).*
