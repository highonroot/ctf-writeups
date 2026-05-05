## What is Privilege Escalation

Privilege Escalation (PrivEsc) refers to finding misconfigurations on a system, identifying what's vulnerable, and manipulating those weaknesses to gain higher access — often root.

PrivEsc works in two directions:
- **Vertical PrivEsc:** Gaining higher privileges, such as moving from a regular user to root
- **Horizontal PrivEsc:** Gaining access to another user's files or directories at the same privilege level


## Why It Exists

Linux is a multi-user system — multiple users share the same machine, each with different levels of access. Everything runs as a user, and each user has limited permissions. These boundaries exist by design.

But boundaries can be misconfigured. Privilege escalation is what happens when those boundaries break — intentionally or not.


## How to Think About It

Everything in Linux runs as a user. Every user has limits. Privilege escalation is about crossing those limits.

All of PrivEsc comes down to four questions:
- Who am I?
- What can I run?
- What can I modify?
- What runs with higher privileges?


## What It Is NOT

Most beginners think PrivEsc is only about getting root. In reality:
- It shows how a system actually works under the hood
- It reveals what can break a system and how to prevent it
- It demonstrates the consequences of misconfigured permissions


## Mental Model

Find vulnerabilities or broken permissions → Manipulate files, commands, or permissions → Gain higher access → Fix the misconfiguration


## What Comes Next

Before abusing permissions, you need to understand how Linux organizes everything as files and where trust boundaries exist in the filesystem. That's where escalation paths actually begin.
