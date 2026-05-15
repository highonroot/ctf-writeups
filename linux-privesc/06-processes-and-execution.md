## What a Process Is

A process is a running instance of a program.

Every process runs under a user's permissions — it can only do what that user is allowed to do. A process started by a normal user cannot perform actions restricted to root, and a process started by root carries full system access.

Every process has a PID (Process ID) — a unique number the kernel uses to track and manage it.


## How Processes Run Under Users

Every process inherits the identity of whoever started it — the user, their permissions, and their environment.

- Processes running under normal users — custom scripts, user applications, shell commands
- Processes running under root — system services, administrative commands like `adduser`, scheduled maintenance tasks

This inheritance is what makes process-based escalation possible — if a root process can be influenced, the attacker inherits root's identity through it.


## Parent and Child Processes

The process that starts first is the parent. Any process spawned by it is a child. A child process inherits the identity and environment of its parent.

**Example:**
```
root runs backup.sh → backup.sh runs tar → tar runs as root
```
- `backup.sh` — parent process
- `tar` — child process

> **Note:** If any part of this chain is writable by a non-root user, an attacker can replace or modify it — and whatever runs next executes as root.


## Environment Variables in Execution

Environment variables are temporary settings passed to a process at runtime. Processes inherit them from their parent.

`PATH`, `LD_PRELOAD`, and `HOME` directly influence how processes find and load things.

**`PATH`** — tells the system where to search for commands when a name is typed without a full path.

Example:
```bash
echo $PATH
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```
If an attacker can prepend a writable directory to PATH, a command like `tar` can be shadowed by a malicious binary of the same name — root runs it instead of the real one.


**`LD_PRELOAD`** — tells the dynamic linker to load a specified library before anything else when a program starts.

Example:
```bash
# Attacker sets LD_PRELOAD in their own user environment
export LD_PRELOAD=/tmp/evil.so

# Runs a command they're allowed to run with sudo
sudo /usr/bin/find

# env_keep in some systems — ensures user's preload is executed instead of root's
```
If `LD_PRELOAD` is preserved when running sudo and an attacker controls what it points to, a crafted shared library executes with root privileges before the actual program even starts.


## Where Trust Breaks

**Writable files executed by root** — if a script or binary executed by root can be modified by a normal user, the attacker replaces its content with arbitrary commands. Root executes it on the next run.

Example:
```bash
ls -la /opt/backup.sh
# -rwxrwxr-x 1 root root 512 Jan 10 backup.sh
```
World-writable script run by root — attacker overwrites it, root executes attacker's commands.


**Inherited environment variables** — a root process that inherits `PATH` or `LD_PRELOAD` from a user's environment can be redirected without touching the process itself.

Example:
```bash
export LD_PRELOAD=/tmp/evil.so
sudo some-allowed-command
```
The allowed command runs as root but loads the attacker's library first.


## What Attackers Look For

Scripts or binaries running as root that are writable, or that reference commands and libraries through controllable paths.

```bash
ps aux                          # list all running processes
ps aux | grep root              # filter processes running as root
```

Attackers cross-reference root processes with writable files — any overlap is a potential escalation path.


## Commands

```bash
ps aux                                          # list all processes with details
ps aux | grep root                              # filter processes owned by root
ps -eo pid,user,command                         # list PID, user, and command
cat /proc/<PID>/environ                         # environment of a specific process
echo $PATH                                      # show PATH variable
echo $LD_PRELOAD                                # check preload library setting
find / -writable -type f 2>/dev/null            # find all writable files
find / -writable -type f 2>/dev/null | grep -v proc   # exclude /proc noise
ls -la /etc/cron* 2>/dev/null                   # check cron script permissions
```


## What Comes Next

Process execution can be manipulated by controlling what root runs and what it loads — next we look at PATH hijacking specifically, where controlling the search order of commands is enough to escalate privileges.
