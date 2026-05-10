## Why Users and Groups Exist

Linux is a multiuser system — each user has their own files and each file, directory, and process runs under the identity of the logged-in user.

A group is a collection of users — every user in that group can access files owned by the group. Example: a `devs` group created for a shared project.


## How Linux Identifies Users

Linux doesn't identify users by name — instead it assigns a unique ID (UID) to each user. Linux uses this number internally, not the name.

Root always has UID 0.


## Important Users to Know

- **UID 0** — root, predefined. Full system access.
- **UID 1-999** — system users, created automatically for services. Not real people.
- **UID 1000+** — real human users.

In PrivEsc, the goal is almost always reaching UID 0.


## What Groups Are

A group is a collection of users. Groups can own files — users in the group inherit access to files owned by that group.

Every group also has a unique ID — **GID**.


## Important Files

**`/etc/passwd`**
Contains user account information — username, password placeholder, UID, GID, home directory, and shell. If writable, an attacker can add a new entry with UID 0, creating another root account.

Example:
```

root:x:0:0:root:/root:/bin/bash
user:x:1000:1000::/home/user:/bin/bash
```


**`/etc/shadow`**
Stores password hashes and password aging information. The `$6$` prefix indicates SHA-512 hashing. Dangerous if readable — hashes can be cracked offline.

Example:
```
root:$6$randomsalt$hashedpassword...:18000:0:99999:7:::
user:$6$randomsalt$hashedpassword...:18000:0:99999:7:::
```


**`/etc/group`**
Contains group information and members in format — `groupname:password:GID:members`. If writable, an attacker can add themselves to privileged groups without needing a password.

Example:
```
sudo:x:27:user1,user2
docker:x:999:user1
```


## Where Trust Breaks

- **Writable `/etc/passwd`** — attacker adds a new user with UID 0, instantly granting root access.
- **Readable `/etc/shadow`** — attacker reads password hashes, posing a direct threat to account security.


## What Attackers Look For

**Readable `/etc/shadow`** :
If readable by a normal user, password hashes can be extracted and cracked offline using tools like John the Ripper — no lockouts, no alerts.

**Writable `/etc/passwd`** :
Attacker adds this entry:
```

hacker::0:0::/root:/bin/bash
```
Instant root access — no password required.

**Users in Privileged Groups** :
- `sudo` — can run commands as root
- `docker` — can mount filesystem as root through containers
- `disk` — raw disk access, bypasses normal file permissions

**Service Accounts with Shells** :
System accounts shouldn't have login shells. Their shell is normally set to `/usr/sbin/nologin`. If misconfigured as `/bin/bash` — compromising that service gives an interactive shell as that account. Further escalation may follow.


## Commands

```bash
id                            # shows username, UID, GID and group memberships
whoami                        # shows current username
cat /etc/passwd               # list all user account entries
cat /etc/passwd | grep <user> # search for specific user entry
cat /etc/group                # list all groups and members
cat /etc/shadow               # show password hashes (root only normally)
getent passwd                 # alternative to cat /etc/passwd
```

> **NOTE:** If `cat /etc/shadow` returns output without sudo — that is a misconfiguration worth investigating.


## What Comes Next

Now that you know who users are and what files store their identity — next is what they are allowed to run with elevated privileges. That is where `sudo` comes in.
