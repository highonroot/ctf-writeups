## What cron is
`cron` is a job scheduler in Linux — it schedules jobs/scripts to run automatically at the defined time without human intervention.
Used for backup, cleanup, monitoring, maintenance — many Linux machines use `cron`

**Example:**
```bash
0 2 * * * /opt/backup.sh
# Run `/opt/backup.sh` daily at 2AM
```

## How cron jobs are configured
Cron uses a specific syntax to schedule programs :
```bash
* * * * * /path/to/example.sh
```
Five fields represents — minutes, hours, days, months, days of the week. `*` means every value for that field.

## Types of cron jobs
- User crontab — specific to one user. Runs as that user.
```bash
crontab -e    # Edit your crontab
crontab -l     # List cron jobs 
```

- System cron — stored in `/etc/cron*` directories, often runs as root
```bash
/etc/crontab            # Stores main cron jobs
/etc/cron.d              # Additional cron jobs
/etc/cron.daily        # Cron jobs running daily 
/etc/cron.hourly     # Cron jobs running hourly
```

## Where trust breaks
- **Writable cron scripts** : Scripts running as root via cron that are world writable — attacker can modify it directly.
  ```bash
  ls -la /opt/backup.sh
  -rwxrwxrwx 1 root root backup.sh    # world writable, owned by root
  ```
  Attacker modifies it directly — next time cron runs it, malicious code executes as root.

- **Writable cron directories** : Cron script directories that are world-writable — attacker can replace the script with malicious version.
  ```bash
  ls -la /etc/cron.daily/
  drwxrwxrwx    # world writable directory
  ```
  Attacker replaces existing script or plants new one.

- **Commands used without full path** : Scripts using commands relatively instead of using full path — directly connects to [PATH Hijacking](07-path-hijacking.md) we discussed in previous writeup.
  ```bash
  # Inside backup.sh
  tar -czf backup.tar.gz /home    # relative — vulnerable to PATH hijacking
  /usr/bin/tar -czf backup.tar.gz /home    # full path — safe
  ```

## What attackers look for
```bash
cat /etc/crontab    # displays crontab details
ls -la /etc/cron.d/    # list additional cron scripts
ls -la /etc/cron.daily/   # list daily running scripts
crontab -l    # list all cron jobs
```
For each cron job to be found —
- What the script executes?
- Who owns the script?
- What permissions script have?
- Does the script use relative commands?

If script is writable — modify it directly. If directory is writable — replace the script. If relative commands found — apply PATH hijacking. Each finding is a direct escalation path.

## Commands
```bash
cat /etc/crontab                    # main system crontab
ls -la /etc/cron.d/                 # additional cron jobs
ls -la /etc/cron.daily/             # daily scripts
ls -la /etc/cron.hourly/            # hourly scripts
crontab -l                          # current user's crontab
find / -name "*.sh" 2>/dev/null     # find all shell scripts
cat /var/log/syslog | grep cron     # check cron execution logs
```

## What comes next
Cron abuses scheduled execution for escalation. SUID/SGID takes a different approach — privilege is baked into the file itself, not the schedule. Next we look at how that works and where it breaks.
