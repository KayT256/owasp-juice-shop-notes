# OWASP Juice Shop — Hands-On Security Notes

Raw, working notes from solving the [OWASP Juice Shop](https://owasp-juice.shop/) challenges — a deliberately vulnerable web application maintained by OWASP for security training. This repo is my personal lab notebook, not a polished writeup collection: every file captures the actual step-by-step reasoning, dead ends, and pivots I went through while working each challenge, in the order I actually thought them.

## Why this exists

I'm a rising junior with a software engineering background who's moved toward offensive security and penetration testing. I built this repo to get real, structured reps in web application security — reading requests critically, forming hypotheses, testing them, and being wrong a lot before being right — and to have something concrete to show for that practice while I keep building toward internship-level experience.

I'm intentionally publishing these **unedited**. Cleaned-up writeups are easy to find for Juice Shop; what's harder to show is the actual thought process behind solving something you don't already know the answer to. These notes are that process, warts and all — including notes where I got stuck, mis-scoped a problem, or had to backtrack after a wrong assumption.

## Environment

- Juice Shop, official Docker image, run locally
- macOS (Apple Silicon)
- Burp Suite Community Edition as primary interception/testing proxy
- Supplementary CLI tooling introduced as needed (`ffuf`, scripting with Python, etc.) — noted per-challenge where relevant

## Structure

Notes are organized by OWASP vulnerability category, mirroring how Juice Shop itself groups its challenges:

```
├── broken-access-control
├── broken-anti-automation
├── broken-authentication
├── cryptographic-issues
├── improper-input-validation
├── injection
├── insecure-deserialization
├── miscellaneous
├── observability-failures
├── security-misconfiguration
├── security-through-obscurity
├── sensitive-data-exposure
├── unvalidated-redirects
├── vulnerable-components
├── XSS
└── XXE
```

Each note generally follows the shape of: what I tried → what happened → what I learned from the result → next hypothesis — rather than a clean linear "solution."

## Current progress

As of this writing:

|Track|Progress|
|---|---|
|Hacking Challenges (overall)|53%|
|★ (1-star)|27/27|
|★★ (2-star)|23/28|
|★★★ (3-star)|34/48|
|★★★★ (4-star)|16/37|
|★★★★★ / ★★★★★★|0 / 0|
|Coding Challenges|61%|

This is a snapshot, not a ceiling — the harder tiers are exactly where the real learning is happening, and this repo will keep growing as I work through them.

## Methodology notes

A few things that show up repeatedly across these notes and reflect how I actually approach a challenge:

- **Strictly black-box.** All findings here — including things like an exposed `package.json`, backup files, or config artifacts — were discovered through live recon against the running app itself (directory brute-forcing, probing for misconfigurations, reading raw responses), not by consulting the public Juice Shop source repository on GitHub. If a file was reachable, it was reachable to any external attacker, and that's the only way I looked for it.
- **Tool-agnostic reasoning.** Several notes show the same problem approached with Burp Intruder, `ffuf`, and hand-written Python scripts, deliberately, to build fluency across manual and automated tooling rather than depending on one.
- **No AI assistance in solving challenges.** I didn't use AI to find vulnerabilities, generate payloads, or solve any challenge in this repo — that would defeat the point of building this foundation myself. The one place AI shows up in my process is as a concept reference (e.g. asking "what is a CSAF/VEX document" or "how does JWT signature verification work" when I hit an unfamiliar term), the same way I'd use a textbook or documentation. The actual testing, hypotheses, and conclusions are entirely my own.
- **Challenges aren't solved in isolation.** Juice Shop's challenges frequently overlap — working one can incidentally satisfy the conditions for another. Where that happened, I noted it directly rather than writing a separate redundant entry (e.g. a note in `Database Schema.md` pointing back to `Password Strength.md`, where that challenge was completed as a side effect). I kept these cross-references in rather than cleaning them out, since recognizing when one exploit chain satisfies multiple objectives is itself part of the skill.

## Disclaimer

All testing documented here was performed exclusively against a local, intentionally vulnerable, sandboxed instance of OWASP Juice Shop for educational purposes. Nothing in this repository targets or was performed against any real, production, or third-party system.

## About me

Rising junior, software engineering background, actively building toward a career in offensive security / penetration testing. Open to internship opportunities — feel free to reach out.

_([LinkedIn](https://www.linkedin.com/in/trieu-khang-trat/) / [Email](mailto:tratr1-28@rhodes.edu) / [Portfolio](https://www.trieukhangtrat.com/))_
