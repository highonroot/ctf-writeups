## What Enumeration Is

Enumeration is the systematic process of gathering information on an unknown machine.

It's about identifying and framing vulnerabilities and misconfigurations only — not exploiting them. Exploitation comes after.


## The Methodology

PrivEsc follows a defined sequence:

- **Land** — gain initial access to the machine
- **Enumerate** — gather systematic information about the machine
- **Identify** — identify misconfigurations and vulnerabilities
- **Exploit** — exploit those vulnerabilities to escalate privileges

This writeup covers the first three steps — landing on a system, enumerating throughout the machine, and finding what's misconfigured.


## Manual Enumeration Checklist

The main section of enumeration. Ordered by priority — what you check first matters.


### 1. Identity

Who you are — first thing after landing on a system. Understand your current position before anything else.

```bash
id        # shows username, UID, GID, and all groups you belong to
whoami    # shows current username
groups    # shows all groups current user belongs to
```

**What to look for** —
- Are you in any privileged group? — `sudo`, `docker`, `disk`, `adm` all have escalation paths
- Is your UID something other than 1000+? — unexpected UIDs are suspicious
- Are you already root (UID 0)? — escalation already complete

→ [Users and Groups](03-users-and-groups.md)


### 2. sudo Permissions

Check what you are permitted to run using sudo.

```bash
sudo -l    # shows sudo settings and permissions for current user
```

**What to look for** —
- What commands can you run with sudo? — defines your escalation options
- Does it show `NOPASSWD`? — listed commands run as root without a password
- Does it have `ALL`? — full sudo access, immediate escalation
- Does `env_keep` preserve `LD_PRELOAD` or `PATH`? — environment based escalation possible

→ [Sudo](04-sudo.md)


### 3. SUID/SGID Binaries

Check if dangerous binaries have the SUID or SGID bit set.

```bash
find / -perm -u=s -type f 2>/dev/null    # binaries with SUID set
find / -perm -g=s -type f 2>/dev/null    # binaries with SGID set
```

**What to look for** —
- Any unexpected binary with SUID set? — cross reference with GTFOBins for shell spawning paths
- Any SUID binary that is world-writable? — can rewrite the binary itself
- Any custom or non-standard binary with SUID? — standard binaries are expected, custom ones are suspicious

→ [SUID SGID](09-suid-sgid.md)


### 4. Capabilities

Check for dangerous capabilities assigned to non-legitimate binaries.

```bash
getcap -r / 2>/dev/null    # list all files with capabilities set
```

**What to look for** —
- `cap_setuid` on a flexible binary like `python` or `perl`? — can change UID to root directly
- `cap_dac_override` on an editor like `vim`? — can read any file including `/etc/shadow`
- `cap_sys_admin` on anything? — nearly equivalent to full root access

→ [Capabilities](10-capabilities.md)


### 5. Cron Jobs

Look for what runs automatically as root with wrong permissions.

```bash
cat /etc/crontab          # main system crontab
ls -la /etc/cron.d        # additional system cron jobs
crontab -l                # current user's crontab
ls -la /var/spool/cron    # cron spool directory
```

**What to look for** —
- Any writable cron script or directory? — replace content, root executes your code
- Cron job using relative commands instead of absolute paths? — PATH hijacking possible
- For each cron entry ask — what does it run? who owns it? what permissions does it have? does it use relative commands?

→ [Cron Jobs](08-cron-jobs.md)


### 6. Writable Directories and Files

Look for world-writable directories or files containing or executing sensitive operations.

```bash
find / -writable -type d 2>/dev/null    # all writable directories
find / -writable -type f 2>/dev/null    # all writable files
```

**What to look for** —
- Any writable directory containing root-executed scripts? — `/etc/cron.d`, `/opt/app/scripts`
- Any sensitive writable file? — `/etc/passwd` writable means direct escalation
- Any world-writable directory used by privileged processes?

→ [Filesystem](01-filesystem.md)


### 7. PATH Check

Check if any writable directory exists in `PATH`.

```bash
echo $PATH                              # show current PATH
find / -writable -type d 2>/dev/null    # cross reference with writable directories
```

**What to look for** —
- Any writable directory listed in `PATH`? — plant a binary with the same name as a legitimate command
- Can you prepend a writable directory before the real one? — command resolution follows order, first match wins

→ [PATH Hijacking](07-path-hijacking.md)


### 8. Credential Exposure

Look for openly exposed credentials or credentials stored with weak security.

```bash
cat ~/.bash_history                   # previously run commands including passwords passed as arguments
env                                   # all environment variables including tokens and keys
find / -name "*.conf" 2>/dev/null     # all config files
find / -name "*.env" 2>/dev/null      # all .env files
grep -r "password" /etc 2>/dev/null   # lines containing "password" in /etc
cat ~/.ssh/id_rsa                     # private SSH key — if readable, attacker authenticates as this user anywhere the public key is trusted
```

**What to look for** —
- Passwords passed as command arguments? — visible in `~/.bash_history`
- Credentials stored as plain text? — config files, `.env` files, scripts with hardcoded DB passwords or API keys
- Private SSH key without a passphrase? — direct authentication without a password

→ [Credential Exposure](05-credential-exposure.md)


### 9. Users, Groups & Sensitive Files

Look for sensitive files with wrong permissions, users with suspicious UIDs, and wrong users in privileged groups.

```bash
cat /etc/passwd    # list all users with UID, GID, home directory, shell
cat /etc/shadow    # password hashes — normally restricted to root
cat /etc/group     # list all groups and their members
```

**What to look for** —
- Is `/etc/shadow` readable? — exposes password hashes, crackable offline
- Is `/etc/passwd` writable? — can append a crafted root user directly
- Is `/etc/group` writable? — can add yourself to `sudo`, `docker`, or any privileged group
- Any user with UID 0 other than root? — duplicate root account, immediate escalation
- Any home directory readable? — may contain credentials, keys, or history files

→ [Users and Groups](03-users-and-groups.md)


### 10. Network and Processes

Look for root-owned processes running writable scripts and internal services running with weak security.

```bash
ps aux | grep root    # all processes owned by root
ss -tlnp              # listening TCP ports and the process behind each
ss -ulnp              # listening UDP ports
cat /etc/hosts        # internal hostnames and IPs
```

**What to look for** —
- Any root process running a writable script or binary? — modify it, root executes your code on next run
- Any service listening only on `127.0.0.1`? — internal services often have weaker auth assuming they're unreachable from outside. From inside the machine, they're reachable.
- Cross reference listening ports with running processes — identify what's running internally as root

→ [Processes and Execution](06-processes-and-execution.md)


## Automated Tools

Run these only after completing manual enumeration — understanding what they flag requires knowing why it's flagged, which the previous writeups cover.

**[LinPEAS](https://github.com/carlospolop/PEASS-ng)** — most comprehensive automated enumeration script. Color coded output — red means critical, yellow means worth investigating, green means informational. Covers everything in this checklist and more. Most attackers run this first then dig into what it flags.

**[LinEnum](https://github.com/rebootuser/LinEnum)** — simpler, plain text output. Easier to read when learning enumeration before trusting automated output blindly.

**[LES — Linux Exploit Suggester](https://github.com/mzet-/linux-exploit-suggester)** — different category entirely. Takes your kernel version and cross references it against known public kernel exploits. Outputs a list of CVEs your system may be vulnerable to. Not about misconfigurations — about unpatched kernel vulnerabilities.

> **Note:** Automated tools are faster but noisy. Manual enumeration first builds the instinct to know what matters in the output.


## What Comes Next

Enumeration maps the full attack surface — every open door, every misconfiguration, every exploitable path. The final writeup looks at the same surface from the defender's side. Every vulnerability found here has a fix — next we cover hardening against everything in this series.
