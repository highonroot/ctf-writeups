## What Capabilities Are

Capabilities are a way to grant specific privileged operations to a program or binary — unlike SUID which gives full root access.

Capabilities provide a more granular and controlled alternative to SUID — SUID either gives full root privilege or no access at all. Capabilities are often a safer and controlled alternative.


## Why They Exist

Sometimes a program or binary requires only specific root permissions rather than full root access — capabilities are used here to grant only the specific privileges needed.

**Example:**
`ping` needs only raw network access — instead of setting SUID to give full root access, capabilities provide it only that specific permission — `cap_net_raw`.


## How Capabilities Work

A binary has capability labels attached to it — when that binary runs, Linux checks those labels and grants only those permissions, nothing else.

**Example:** `ping` has `cap_net_raw`

```bash
/usr/bin/ping = cap_net_raw+ep
# e — effective (currently active)
# p — permitted (allowed to use)
# +ep means the capability is permitted and immediately usable
# When getcap shows +ep after a capability — that capability is active and exploitable
```

Linux grants ping the ability to create raw network packets, nothing else.

> **Note:** If ping used SUID instead — it would run with full root privileges even though it only needs raw network access.


## Common Capabilities Worth Knowing

- **`cap_setuid`** — allows a process to change its effective UID to any valid user, including root (0).
  If assigned to a flexible binary like Python — it can be used to become root.


- **`cap_dac_override`** — bypasses file permission checks — can read any file.
- Bypasses root-owned and restricted files — can even read `/etc/shadow` which stores password hashes.


- **`cap_net_raw`** — provides raw network access — what ping uses legitimately.
  Allows creating raw network packets for ICMP. Attackers can use it for network sniffing or crafting packets.


- **`cap_sys_admin`** — broad system administration — considered nearly equivalent to root access.
  Covers mounting filesystems, managing namespaces, kernel operations and many other system operations.


- **`cap_chown`** — allows a process to change ownership of any file to any user or group.
  Attacker can change ownership of `/etc/passwd` or `/etc/shadow` to themselves — then read or modify sensitive files directly.


## Where Trust Breaks

**Dangerous capability on the wrong binary** — `cap_setuid` assigned to `/usr/bin/python3`:

```bash
/usr/bin/python3 = cap_setuid+ep
```

```python
import os
os.setuid(0)          # set UID to root (0)
os.system('/bin/bash') # spawn root shell
```


**Overly broad capability** — `vim` with `cap_dac_override`:

```bash
/usr/bin/vim = cap_dac_override+ep
```

`vim` can now read any file regardless of permissions — including `/etc/shadow`.


## What Attackers Look For

After finding binaries with capabilities set, attackers filter for:

- **`cap_setuid`** — immediate root access by changing UID to 0
- **`cap_dac_override`** — bypass file permissions and read anything, including restricted files
- **`cap_sys_admin`** — broad system administration, often enough for full escalation

Visit [GTFOBins](https://gtfobins.github.io) for capability-specific exploitation paths.


## Commands

```bash
getcap <binary>                      # show capabilities for a specific binary
getcap -r / 2>/dev/null              # list all files with capabilities set
cat /proc/$$/status | grep Cap       # check current process capabilities (hex output)
capsh --decode=<hex>                 # decode hex capability value to human readable
                                     # e.g. capsh --decode=0000003fffffffff
                                     # outputs: cap_chown,cap_dac_override,...
```

> **Note:** Only `root` or a process with `CAP_SETFCAP` can assign capabilities using `setcap`.


## What Comes Next

Capabilities showed how granular privilege control can be helpful as well as dangerous — next we run all checks systematically through enumeration to detect every potential escalation path covered so far.
