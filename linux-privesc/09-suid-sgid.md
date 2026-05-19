## What SUID and SGID are
### SUID ( Set User ID )
SUID bit is placed at owner's execute permission — executes the file as its owner not the one executing it, even if it's root.
```bash
-rwsr-xr-x   # `s` at owner execute — SUID bit 
```
### SGID ( Set Group ID )
SGID bit is placed at group's execute permission — behaves differently on files and directories.
- On files — File executing runs as group privilege, not the user executing.
```bash
-rwxrwsr-x  # `s` at group execute — SGID bit; `-` — regular file
```
- On directories — Any file created in the directory inherits the group's identity, not the creator's.
```bash
drwxrwsr-x   # `d` — directory
```
**NOTE:**  For learning more about permissions, visit [Permissions Writeup](02-permissions.md)

## Why they exist
Some programs need temporary root privilege to perform operations.

**Example:** `passwd`

Normal users cannot write `/etc/shadow` file — but they still need to change their password.

`passwd` binary has the SUID bit set and owned by root — any user executing it runs the `passwd` program with root privilege temporarily.

If the SUID bit is not set — user would need direct root access everytime to change their password.

## How they work in execution
Normally, executing a program — it runs as the logged in user.
```bash
-rwxr-xr-x
```

With SUID or SGID, executing program runs as its owner's privilege or group respectively.
```bash
-rwsr-xr-x   //   -rwxrwsr-x
```
**Example:** 
```bash
-rwsr-xr-x 1 root root /usr/bin/passwd
```
Normal user executing it — real user : current logged-in user; effective user : root

Program temporarily runs as root's privilege.

## Finding SUID and SGID binaries
Usually two commands are used to locate SUID/SGID bit set binaries.
```bash
find / -perm -u=s -type f 2>/dev/null   # find files with SUID bit set 
find / -perm -g=s -type f 2>/dev/null   # fund files with SGID bit set 
```

## Where trust breaks
- Writable SUID set binary — If SUID set binaries are writable, attacker can replace the code to execute it with root privilege.
  ```bash
  -rwsrwxrwx root root /usr/local/bin/backup
  ```
  World writable and SUID — attacker replaces binary content. Next execution runs their code as root.

- Unnecessary SUID set binaries — some binaries like vim, python, find doesn't require SUID set. No legitimate reason for it.
  ```bash
  -rwsr-xr-x 1 root root /usr/bin/vim
  ```
  No legitimate reason for vim to have SUID. Anyone running it gets root shell via `:!/bin/bash`

## What attackers look for
After finding SUID set binaries, attackers filter out the expected ones and look for unusual binaries that are documented PrivEsc techniques on GTFOBins.

**Expected binaries** — these are ignored 
```bash
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/ping
/usr/bin/su
```
They look for unusual/custom binaries or programs that invoke shell or other commands, such as — find, vim, python, bash, cp, tar

## GTFOBins connection
Always check for potential escalation SUID binaries on [GTFOBins](https://gtfobins.github.io), it documents ways to abuse common Unix binaries when they are run with elevated privileges.

**Usage:** Search binary name → Look for SUID section → Follow the steps

## Commands
```bash
ls -l /usr/bin/passwd    # example of legitimate SUID
stat /path/to/binary     # detailed info including permissions 
find / -perm -u=s -type f 2>/dev/null    # find files with SUID set
find / -perm -g=s -type f 2>/dev/null   # find files with SGID set
```

## What comes next
SUID gives full owner privileges when executed — capabilities take a more granular approach, granting specific privileges only. Next we look at how capabilities work and where they break.
