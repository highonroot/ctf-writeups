## What is the Linux Filesystem

The Linux filesystem is how the operating system organizes everything — executables, configurations, temporary files, and system binaries. Unlike Windows, Linux treats almost everything as a file, which means file permissions directly control what users can access, modify, or execute.

This matters in PrivEsc because misconfigured files and directories are one of the most common ways attackers gain higher access.


## Why Everything is a File

In Linux, almost everything is represented as a file — commands, binaries, hardware devices, and even keyboard input. This design allows users and programs to interact with the system through the filesystem itself.

In PrivEsc, this becomes important because Linux trusts files based on their ownership, permissions, and execution context. If a trusted file becomes writable by normal users, that trust can break.

Example: A root-owned script executed automatically by cron becomes world-writable. Any user can modify the script, and when cron executes it as root, the malicious code runs with root privileges.


## Key Directories and What They Hold

- `/etc` — System configuration files. Contains sensitive files like `/etc/passwd`, service configs, and sometimes stored credentials. High-value target during enumeration.

- `/tmp` — World-writable temporary directory used by applications and users. Often abused to store or execute temporary malicious scripts.

- `/var` — Contains logs, caches, mail, and application data. Logs may expose sensitive information, credentials, or application behavior.

- `/bin` and `/sbin` — Essential system binaries and administrative commands. These locations are trusted by the system and should never be writable by normal users.

- `/home` — User home directories. Common place for bash history, SSH keys, configuration files, and personal scripts.

- `/root` — Root user's home directory. Normally inaccessible to standard users and worth checking for permission mistakes.


## Where Trust Boundaries Exist

Linux relies heavily on filesystem trust. The system assumes that files in trusted locations are owned and controlled correctly.

Trust boundaries commonly involve:
- file ownership
- writable directories
- executable files
- scripts executed by privileged users
- directories searched through `$PATH`

If these boundaries are misconfigured, normal users may gain control over something executed with higher privileges.

Example: A root cron job executes a script stored inside `/tmp`. Since `/tmp` is writable by everyone, another user can replace or modify the script and gain root execution when the cron job runs.


## What Attackers Look For

Attackers commonly search for:
- world-writable files and directories
- sensitive configuration files with weak permissions
- readable logs containing credentials or tokens
- writable scripts executed by root
- dangerous writable locations inside `$PATH`

The goal is not just finding files, but finding places where trust can be manipulated.


## Commands

```bash
# Inspect important directories
ls -la /etc
ls -la /tmp

# View user account entries
cat /etc/passwd

# Find writable directories
find / -writable -type d 2>/dev/null

# Find writable files
find / -writable -type f 2>/dev/null
```

## What Comes Next

Now that you understand how Linux organizes files and where trust boundaries exist, the next step is understanding how permissions control access to those files — and how those permissions can be misconfigured.
