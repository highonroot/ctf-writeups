## What PATH is

`PATH` is an environment variable that tells the shell where to find commands. When a command is typed, the shell searches directories listed in `PATH` from left to right until it finds a match.

**Example:**
When you type `ls`, the shell searches these directories in order:
```
/usr/local/bin:/usr/bin:/bin:/home/user/bin
```
The first match found gets executed — even if it is malicious. Search stops there.

> `:` is used as a separator between directories.


## How Command Resolution Works

Command resolution moves left to right through PATH.

**Example:**
```
PATH = /usr/local/bin:/usr/bin:/bin
```
Linux checks `/usr/local/bin/ls` → not found
Linux checks `/usr/bin/ls` → found, executes it, stops searching.

**The attack setup:**
If PATH is `/tmp:/usr/bin:...` and attacker can write to `/tmp` — they place a fake command there. Linux checks `/tmp` first, finds it, executes it. Never reaches the real command.


## How PATH Hijacking Works

Three conditions must exist simultaneously:

**1. Program calls commands without full path** :
Using `ls` instead of `/usr/bin/ls` in scripts means Linux must search PATH — making it vulnerable to substitution.

**2. Attacker controls a writable directory that appears before the real path** :
Attacker writes to `/tmp`, places malicious `ls` there, modifies PATH to `/tmp:/usr/bin`. Linux finds fake `ls` first.

**3. Program runs with high privilege** :
Script or binary runs as root via cron, sudo, or SUID.

If all three conditions exist — attacker places a malicious binary in their controlled directory with the same name as the target command. Root executes the malicious version.


## Where Trust Breaks

Scripts that call commands without full paths while running with root privilege — via cron or sudo.

**Example:**
`backup.sh` contains:
```bash
tar -czf backup.tar.gz /home
```

Attacker adds a writable directory before the real `tar` location in PATH and places a fake `tar` there. Root executes the attacker's version.


## What Attackers Look For

- Scripts running commands without full paths
- Writable directories that can be placed before actual command directories in PATH
- SUID binaries that call commands internally without full paths — same vulnerability as scripts


## Commands

```bash
echo $PATH                            # show current PATH
cat <script>                          # read script contents to check for relative commands
which <command>                       # shows which version of command would actually run
env | grep PATH                       # alternative PATH check
find / -writable -type d 2>/dev/null  # find writable directories
sudo -l | grep env_keep               # check if PATH is preserved across sudo
sudo env | grep PATH                  # verify if your PATH survives into sudo session
```


## What Comes Next

PATH hijacking often works through cron jobs running root scripts that call commands without full paths. Next, we look at cron jobs specifically — how they are configured, and how they become escalation paths.
