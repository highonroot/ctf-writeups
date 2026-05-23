# Hardening

## Overview

This is the defender's perspective in this series.

It covers solutions and fixes for the vulnerabilities found on your machine — not exploiting them, but securing against them. Every technique covered in the previous writeups has a corresponding fix here.

> **Note:** This series documents Linux privilege escalation techniques for educational purposes — understanding how systems break is the foundation of securing them.


## Finding to Fixing

In [Enumeration](11-enumeration.md), we covered a manual checklist for finding misconfigurations on a system — but finding them is only the starting point.

Vulnerabilities need to be fixed systematically. Left unaddressed, they become the escalation paths this entire series documents.


## Hardening Checklist

> Hardening is not about removing every privilege. It is about ensuring every privilege has a legitimate purpose and the smallest possible scope.

Each entry follows the same structure — risk, fix, verify. Same sequence as enumeration.


### 1. Identity and Access

**Risk:** Users in unnecessary privileged groups; unused accounts

**Fix:**
```bash
# Remove user from a privileged group
sudo gpasswd -d <username> <group>

# Lock unused accounts
sudo usermod -L <username>
```

**Verify:** `id <username>` — confirm group removal; `cat /etc/passwd` — check for unexpected accounts

→ [Users and Groups](03-users-and-groups.md)



### 2. Sudo Misconfigurations

**Risk:** `NOPASSWD` entries; `ALL` privilege; dangerous commands permitted through sudo; `env_keep` preserving dangerous variables like `LD_PRELOAD`

**Fix:**
```bash
# Always edit sudoers through visudo — validates syntax before saving
sudo visudo
```

Inside visudo — remove or restrict:
- `NOPASSWD` entries unless explicitly required
- `ALL` privilege — replace with specific commands only
- `env_keep` entries for `LD_PRELOAD` or `PATH`

**Verify:** `sudo -l` — confirm permissions are restricted as intended

→ [Sudo](04-sudo.md)



### 3. SUID/SGID Binaries

**Risk:** Unexpected, custom, or writable binaries with SUID or SGID bit set

**Fix:**
```bash
# Remove SUID bit
sudo chmod u-s <binary>

# Remove SGID bit
sudo chmod g-s <binary>
```

Only remove SUID/SGID from binaries that don't legitimately require it. Standard system binaries like `passwd`, `ping`, and `su` need SUID to function.

**Verify:**
```bash
find / -perm -u=s -type f 2>/dev/null    # check remaining SUID binaries
find / -perm -g=s -type f 2>/dev/null    # check remaining SGID binaries
```

→ [SUID/SGID](09-suid-sgid.md)



### 4. Capabilities

**Risk:** Dangerous capabilities — `cap_setuid`, `cap_dac_override`, `cap_sys_admin` — assigned to binaries that don't need them

**Fix:**
```bash
# Remove all capabilities from a binary
sudo setcap -r <binary>
```

**Verify:** `getcap -r / 2>/dev/null` — confirm capabilities removed; only capabilities that are operationally required should remain

→ [Capabilities](10-capabilities.md)



### 5. Cron Jobs

**Risk:** Writable cron scripts or directories; cron jobs using relative commands instead of absolute paths

**Fix:**
```bash
# Restrict permissions — only owner can read, write, execute
sudo chmod 700 <script>

# Ensure root owns the script
sudo chown root:root <script>

# Use absolute paths inside cron scripts
/usr/bin/tar    instead of    tar
/usr/bin/curl   instead of    curl
```

**Verify:**
```bash
ls -la /etc/cron*       # check script permissions and ownership
cat /etc/crontab        # confirm absolute paths used throughout
```

→ [Cron Jobs](08-cron-jobs.md)



### 6. Writable Directories and Files

**Risk:** World-writable files, directories, or root-owned scripts editable by normal users

**Fix:**
```bash
# Restore appropriate ownership to root
sudo chown root:root <file>

# Restore appropriate permissions based on file type
# For directories — owner full access, others read and execute only
sudo chmod 755 <directory>

# For scripts executed by root — owner only
sudo chmod 700 <script>

# For sensitive config files — owner read/write, others read only
sudo chmod 644 <config>
```

> **Note:** Permissions depend on what the file does. Binaries need execute permission — don't apply `644` to something that needs to run. Restore the minimum permission the file needs to function.

**Verify:**
```bash
ls -l <file>                          # check permissions and ownership
find / -writable -type f 2>/dev/null  # confirm no unexpected writable files remain
```

→ [Filesystem](01-filesystem.md)



### 7. PATH Vulnerabilities

**Risk:** Writable directories in PATH; root scripts using relative commands

**Fix:**
```bash
# Remove unnecessary or writable directories from PATH
# Example — removing /tmp if it was prepended
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Use absolute paths in all root-owned scripts
/usr/bin/python3    instead of    python3
```

> **Note:** Don't prescribe a fixed PATH universally — some systems legitimately use `/usr/local/bin` or custom tool directories. The principle is: remove any directory that is writable by non-root users.

**Verify:** `echo $PATH` — confirm no writable directories listed; `which <command>` — confirm command resolves to the expected binary

→ [PATH Hijacking](07-path-hijacking.md)



### 8. Credential Exposure

**Risk:** Hardcoded credentials in scripts, config files, or shell history; private keys with wrong permissions

**Fix:**
```bash
# Fix private key permissions — readable only by owner
chmod 600 ~/.ssh/id_rsa

# Clear current shell history if credentials were passed as arguments
history -c
```

For hardcoded credentials found in `.env`, `.conf`, or `.sh` files — remove them and replace with environment variables or a secrets manager. Rotate any credentials that may have been exposed — clearing history does not undo exposure if logs or other users captured the session.

**Verify:** `grep -r "password" /etc 2>/dev/null` — check for remaining plain text credentials; `ls -la ~/.ssh` — confirm key permissions

→ [Credential Exposure](05-credential-exposure.md)



### 9. Sensitive Files

**Risk:** `/etc/passwd` or `/etc/group` writable by non-root; `/etc/shadow` readable by normal users

**Fix:**
```bash
# Restore correct ownership
sudo chown root:root /etc/passwd
sudo chown root:root /etc/group
sudo chown root:shadow /etc/shadow

# Restore correct permissions
sudo chmod 644 /etc/passwd     # readable by all, writable only by root
sudo chmod 644 /etc/group      # same
sudo chmod 640 /etc/shadow     # readable by root and shadow group only
```

**Verify:** `ls -l /etc/passwd /etc/group /etc/shadow` — confirm ownership and permissions

→ [Users and Groups](03-users-and-groups.md)


### 10. Processes and Services

**Risk:** Unnecessary services running; services running as root when a dedicated user would suffice; internal services relying on weak trust assumptions

**Fix:**
```bash
# Stop and disable unused services
sudo systemctl stop <service>
sudo systemctl disable <service>

# Create a dedicated low-privilege service user
sudo useradd -r -s /usr/sbin/nologin <service-user>
```

Then update the service unit file to run under that user instead of root:
```ini
# Inside /etc/systemd/system/<service>.service
[Service]
User=<service-user>
```

**Verify:** `ps aux | grep root` — confirm service no longer runs as root; `ss -tlnp` — confirm unused services are no longer listening

→ [Processes and Execution](06-processes-and-execution.md)



## Verifying Fixes

After fixing misconfigurations — re-run the enumeration checklist from [Enumeration](11-enumeration.md).

Each command should return clean output without flagged vulnerabilities. If something still flags — it isn't fully fixed. Re-identify what's misconfigured and address it specifically.


## What Clean Output Looks Like

```bash
sudo -l              # no sudo access or restricted commands only
find / -perm -u=s    # only expected binaries: passwd, ping, sudo, su
getcap -r /          # no output or only operationally required binaries
ls -l <file>         # no write permission for group or others
echo $PATH           # no writable directories listed
which <command>      # resolves to the expected system binary
ls -la ~/.ssh        # private keys restricted to owner only (600)
cat /etc/shadow      # Permission denied
```


## What a Hardened System Looks Like

A system cannot have zero vulnerabilities — that's not a realistic goal.

A hardened system has the least possible privilege across every layer — intentional permissions, every privileged process justified, credentials protected, and access limited strictly to what each user and process needs to function.


## Ongoing Hardening

- Re-run enumeration after any system change — new software, new users, new cron jobs
- Audit new software installations for SUID bits and capabilities
- Rotate credentials regularly — especially after testing or sharing access
- Cleanup is mandatory — tmp files, test scripts, and debug logs left behind become exposure points


## Automated Tools

**[LinPEAS](https://github.com/carlospolop/PEASS-ng)** — most comprehensive, color coded output, highlights critical findings

**[LinEnum](https://github.com/rebootuser/LinEnum)** — simpler output, good for reading enumeration results while still learning

**[LES — Linux Exploit Suggester](https://github.com/mzet-/linux-exploit-suggester)** — focuses on kernel vulnerabilities based on system version

Run these periodically — not just once. Compare output over time. New findings after system changes indicate new risks introduced.

