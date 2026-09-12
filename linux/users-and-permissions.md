# Users and Permissions

## What is it?

The set of commands for finding out **who you are**, **what you're allowed to
do**, and **managing user accounts** on a Linux system.

## Why do we need it?

Linux is multi-user by design. Every process runs as some user, and permissions
are enforced based on that user's identity and group memberships. Before you can
reason about permissions, file ownership, or `sudo` access, you need to know how
to check identity and manage accounts.

## Mental Model

```
whoami    → Who am I?
id        → Who am I, and what can I access?
uname -a  → What machine/kernel is this, in full?
uname -m  → Just the CPU architecture
useradd -m → create user + home dir
userdel -r → delete user + home dir
```

## How it works

- Every user has a UID (user ID) and belongs to one primary group (GID) plus
  optionally many secondary groups.
- Group membership is what unlocks extra permissions — e.g. being in the `sudo`
  group is what lets a user run `sudo`, not anything about the username itself.
- `useradd`/`userdel` manage the account entries (in `/etc/passwd`, `/etc/shadow`,
  `/etc/group`) and optionally the user's home directory.

## Example

```
whoami
```
Output: `ubuntu`

```
id
```
Output: `uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu),27(sudo)`

## Common Commands

### `whoami`

Tells you which user account you're logged in as in the current shell.

```
whoami
```

**Mental model:** "Who am I logged in as?"

### `id`

Shows who you are **and** what groups you belong to — more detail than `whoami`.

```
id
```

**Breakdown:**
- `uid` → your user ID + username
- `gid` → your primary group ID + name
- `groups` → every group you're a member of (determines extra permissions, e.g. `sudo`)

**Mental model:** `whoami` = "who am I?", `id` = "who am I, and what am I allowed to touch?"

### `uname -a`

**Base:** `uname` prints system information.

**Flags:**
- `-a` → "all" — prints kernel name, hostname, kernel version, build info, and architecture in one line

**Mnemonic:** `-a` = **a**ll fields.

Example output broken down:
```
Linux    moo    6.8.0-1061-raspi    #65-Ubuntu SMP ...    aarch64    aarch64    aarch64    GNU/Linux
kernel   host   kernel version      build info            arch       arch       arch       OS
```

### `uname -m`

**Flags:**
- `-m` → **m**achine hardware name (CPU architecture only, e.g. `aarch64`, `x86_64`)

**Mental model:** `uname -a` = "everything about this machine", `uname -m` = "just the architecture" — useful when picking the right binary/package for this system.

### `useradd`

Creates a new user.

**Flags:**
- `-m` → also create the user's home directory (`/home/username`). Without it, the account exists but has no home folder.

```
sudo useradd -m alice
```

**Mnemonic:** `-m` = **m**ake home directory.

### `userdel`

Removes a user.

**Flags:**
- `-r` → also remove the user's home directory and mail spool. Without it, the account is gone but their files stay behind.

```
sudo userdel -r alice
```

**Mnemonic:** `useradd -m` adds the home, `userdel -r` removes it — a matched pair.

## Common Mistakes

- Assuming `whoami` tells you about permissions — it only gives the username, not group memberships. Use `id` for that.
- Running `useradd` without `-m` and then wondering why the new user has no home directory.
- Running `userdel` without `-r` and leaving orphaned home directories behind.

## Interview Questions

**Q: What's the difference between `whoami` and `id`?**
> `whoami` just gives the username. `id` gives the full identity — UID, GID, and every group — which matters for permission troubleshooting.

**Q: If a user can't run `sudo`, what's the first thing you'd check?**
> Run `id` on that user and check whether `sudo` appears in their `groups` list.

**Q: What does `useradd -m` do that plain `useradd` doesn't?**
> `-m` creates the home directory for the new user; without it, the account exists but has no home folder.

**Q: How do you fully remove a user, including their files?**
> `userdel -r username` — the `-r` flag also deletes the home directory and mail spool.

## Interview Answer

> "`whoami` tells me the current username, and `id` gives the fuller picture — UID, GID, and group memberships, which is what actually determines permissions like `sudo` access. For system-level info I'd use `uname -a` for everything or `uname -m` just for architecture. When managing accounts, `useradd -m` creates a user with a home directory, and `userdel -r` removes a user along with their home directory."

## Related Concepts

- [[linux/shell-basics]]
- [[linux/file-permissions]]
- [[linux/processes]]

## Remember

```
whoami    → Who am I?
id        → Who am I, and what can I access?
uname -a  → What machine/kernel is this, in full?
uname -m  → Just the CPU architecture
useradd -m → create user + home dir
userdel -r → delete user + home dir
```
