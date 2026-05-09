## Why Permissions Exist
Permissions are rules that define who can read, write and execute files and directories.
Without these, anyone can read, write or execute files or directories, which can expose sensitive system data and control to everyone.


## Permission Structure
Permission structure breaks down into 4 parts. For example:

`-rwxr-xr--`

- First character defines type — `-`: file, `d`: directory, `l`: symbolic link
- Remaining characters are grouped in 3, representing permissions for owner, group and others respectively

```
rwx  →  Owner  : Can read, write and execute
r-x  →  Group  : Can read and execute, write prohibited
r--  →  Others : Can read only

```


## File vs Directory Behavior
Same character behaves differently for files and directories:

| Character | File | Directory |
|---|---|---|
| r | Read content | List files inside |
| w | Modify content | Create/delete files |
| x | Execute | Enter directory (cd) |

**Example:** Directory `/scripts/` is world-writable. Even if `script.sh` inside is root-owned and not writable, any user can delete `script.sh` and replace it with a malicious version. Directory write = control over what lives inside, not just content.


## Numeric Representation
Permissions are also represented numerically:

`r` = 4, `w` = 2, `x` = 1

| Value | Combination | Access |
|---|---|---|
| 7 | 4+2+1 | Full access: rwx |
| 6 | 4+2+0 | Read and write: rw- |
| 5 | 4+0+1 | Read and execute: r-x |
| 4 | 4+0+0 | Read only: r-- |

**Example**: 755 → `-rwxr-xr-x`


## Special Permissions (SUID, SGID, Sticky Bit)

### SUID (`-rwsr-xr-x`)
- `s` appears in owner execute position
- File runs as its owner, not the user executing it
- If owned by root, anyone running it gets root privileges temporarily
- Most dangerous special permission in PrivEsc

### SGID (`-rwxr-sr-x`)
- `s` in group execute position
- On files: runs with the group's privileges, not the executing user's group (similar to SUID but for group identity)
- On directories: files created inside automatically inherit the directory's group, not the creator's group

### Sticky Bit (`drwxrwxrwt`)
- `t` in others execute position
- Used on shared directories like `/tmp`
- Users can only delete their own files even if directory is writable
- Without sticky bit, anyone can delete anyone's files


## Where Trust Breaks
If a script is stored in a world-writable directory and executes automatically as root, any user can modify or replace it. Root executes it without question — broken trust through a single misconfiguration.

SUID binaries are another trust threat. If a binary is owned by root with SUID set, any user can trigger root-level execution, even without sudo.


## What Attackers Look For
Attackers look for weak permissions on files that run with higher privileges:

- **World-writable sensitive files** : attacker modifies content directly
- **World-writable directories** : attacker creates or replaces files with malicious ones
- **SUID binaries owned by root** : attacker executes as root


## Commands

```bash
ls -l                                      # list files with permissions
stat <file>                                # detailed file information
find / -perm -u=s -type f 2>/dev/null      # find files with SUID set
find / -perm -o=w -type f 2>/dev/null      # find world-writable files
chmod                                      # change permissions
chown                                      # change ownership
```


## What Comes Next
Permissions define what can be done but who owns what matters just as much. Next we'll look at users, groups, and how Linux assigns identity to every process and file.
