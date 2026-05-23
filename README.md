# ctf-writeups

Structured notes documenting my Linux Privilege Escalation learning journey — built from first principles, not tutorials.

This is not a cheatsheet. It is a progression through how Linux systems are structured, where trust breaks, how attackers exploit those breaks, and how defenders fix them.

Every writeup was written after understanding the concept — not during it.

---

## What This Covers

Linux PrivEsc is not a list of tricks. It is pattern recognition — landing on an unknown system and systematically identifying where trust is misconfigured.

This series builds that pattern recognition layer by layer:

| # | Topic | What It Covers |
|---|---|---|
| 00 | [Overview](linux-privesc/00-overview.md) | Mindset, mental model, what PrivEsc actually is |
| 01 | [Filesystem](linux-privesc/01-filesystem.md) | How Linux organizes everything and where trust boundaries exist |
| 02 | [Permissions](linux-privesc/02-permissions.md) | rwx, numeric modes, SUID/SGID/sticky bit, where permissions break |
| 03 | [Users and Groups](linux-privesc/03-users-and-groups.md) | Identity model, /etc/passwd, /etc/shadow, privileged groups |
| 04 | [Sudo](linux-privesc/04-sudo.md) | sudoers file, NOPASSWD, GTFOBins, wildcard abuse |
| 05 | [Credential Exposure](linux-privesc/05-credential-exposure.md) | Where credentials hide — history, configs, SSH keys, env vars |
| 06 | [Processes & Execution](linux-privesc/06-processes-and-execution.md) | How processes run under users, parent-child inheritance, LD_PRELOAD |
| 07 | [PATH Hijacking](linux-privesc/07-path-hijacking.md) | Command resolution, hijacking execution through PATH manipulation |
| 08 | [Cron Jobs](linux-privesc/08-cron-jobs.md) | Scheduled execution abuse, writable scripts, relative command risks |
| 09 | [SUID/SGID](linux-privesc/09-suid-sgid.md) | Privileged execution through file bits, GTFOBins connection |
| 10 | [Capabilities](linux-privesc/10-capabilities.md) | Granular root powers, cap_setuid, cap_dac_override, cap_sys_admin |
| 11 | [Enumeration](linux-privesc/11-enumeration.md) | Systematic checklist for finding escalation paths on unknown systems |
| 12 | [Hardening Guide](linux-privesc/12-hardening-guide.md) | Defensive fixes for everything covered — risk, fix, verify |

---

## How Each Writeup Is Structured

Every writeup follows the same structure — no exceptions:

- **Why it exists** — system context before any attack surface
- **How it normally works** — what correct looks like before covering what broken looks like
- **Where trust breaks** — the specific misconfiguration and why it matters
- **What attackers look for** — enumeration commands and suspicious signals
- **Commands** — practical reference
- **What comes next** — how this connects to the following writeup

The series ends with two standalone documents:

**[Enumeration](linux-privesc/11-enumeration.md)** — a prioritized checklist connecting all 10 topics into a systematic approach for unknown systems. Every item links back to its writeup for deeper context.

**[Hardening Guide](linux-privesc/12-hardening-guide.md)** — the defensive companion. Same surface, opposite perspective. Risk, fix, verify for every escalation path covered in the series.

---

## Approach

The goal was never to memorize techniques.

The goal was to understand Linux well enough that misconfigured trust becomes visible — without being told where to look.

> Finding a vulnerability is the first step. Fixing it before someone else finds it is the actual goal.

---

## Notes

- Written after understanding each concept — not during, not copied
- Attacker and defender perspective combined throughout
- Connects every topic to real exploitation and real remediation
- No tool dependency — manual understanding before automated enumeration

---

## Coming Soon

**TryHackMe** — practical room writeups after the conceptual foundation is complete

**Real World** — college lab and personal system findings documented as discovered
