# sudo

`sudo` is a terminal command that runs other commands temporarily with high privilege — usually root — which are otherwise prohibited for normal users.

Without `sudo`, every admin task would require logging in as root directly — which is dangerous. `sudo` is the controlled alternative.

Example:
```
sudo apt update
```


## How sudo Works

`sudo` works as a gatekeeper — it determines who can be given root privilege to run desired commands, as stated in `/etc/sudoers`.

Linux checks if you're allowed to run `sudo` in the sudoers file — if yes, the command runs as root; if no, access is denied and the attempt gets logged.


## The sudoers File

`/etc/sudoers` is a configuration file that stores who can run what as whom.

An entry looks like:
```
user ALL=(ALL:ALL) ALL
```

Breaking it down:

- `user` — which user this applies to
- `ALL` — on all hosts in the system
- `(ALL:ALL)` — can execute as any user and any group
- `ALL` — can execute any command

This simply states that the specified user can run any command as any user (including root).
> Never edit `/etc/sudoers` directly with editors like `vim` or `nano` — a syntax error can break sudo entirely and lock you out. Always use `visudo`, which validates syntax before saving.


## Common sudo Configurations

**Full access** — user can do anything as root using sudo:
```
user ALL=(ALL:ALL) ALL
```

**NOPASSWD** — user can run the mentioned command without any password (dangerous if shell, vim, find etc. have NOPASSWD):
```
user ALL=(ALL) NOPASSWD: <command>
```

Dangerous — full access, no restrictions:
```
user ALL=(ALL) NOPASSWD: ALL
```

**Controlled restrictions** — user can run only specifically mentioned commands:
```
user ALL=(ALL) /bin/systemctl restart nginx
```
Can only restart nginx as root — nothing else.


## Where Trust Breaks

**NOPASSWD on dangerous commands** — user can run commands as root without any password — extremely dangerous.

Example:
```
user ALL=(ALL) NOPASSWD: /bin/bash
```
User can run a shell as root without a password — gets full system access.


**Wildcard in restrictions:**
```
user ALL=(ALL) /usr/bin/vim /var/www/*
```

`*` is a wildcard — matches any file at that path.

Looks harmless? Vim can open a shell from inside it. An attacker runs vim with sudo as root, spawns a shell — gets a root shell, and unrestricted root access achieved.

Example:
```
sudo vim /var/www/index.html
```
Just editing a file? Type `:!/bin/bash` inside vim — you get a root shell.


## What Attackers Look For

`sudo -l` — shows exactly what a user can run with sudo. An attacker checks this first — each entry in the output is a potential escalation path.

Example output:
```
Matching Defaults entries for user on machine:
    env_reset, mail_badpass

User user may run the following commands on machine:
    (ALL : ALL) ALL
    (root) NOPASSWD: /usr/bin/find
```

- First section — default settings
- Second section — actual permissions: `(ALL:ALL) ALL` means full sudo access, `(root) NOPASSWD: /usr/bin/find` means find can be run as root without a password

> Take any listed binary to GTFOBins and check if it can be abused.
> 

## GTFOBins

[GTFOBins](https://gtfobins.github.io) is a website that lists common Linux binaries and how each one can be abused when run with sudo — simply put: "If you can run THIS with sudo, here's how you get root."

`vim`, `find`, `python`, `perl`, `awk` — all on GTFOBins, all can spawn shells.

Example: If `sudo -l` shows you can run vim as root, GTFOBins shows exactly how to spawn a root shell from inside vim.


## Commands

```bash
sudo -l                  # shows sudo settings and permissions for current user
sudo -u root <command>   # runs command as root
cat /etc/sudoers         # displays sudoers file — usually restricted
sudo -V                  # checks sudo version
```


## What Comes Next

`sudo` shows what you can run and how to reach high privilege — but elevated access isn't even required if credentials are stored carelessly, which can give full access directly.
