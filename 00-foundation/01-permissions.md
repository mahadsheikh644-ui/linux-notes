# Lesson 01 — Permissions Mastery (Beyond chmod 755)

Date: 2026-08-18 | Live demo on WSL2 Ubuntu 22.04 | Follow-up to README index entry

## Why this matters
Most junior admins know `chmod 755`. The interview doesn't stop there:
special bits, umask, ownership defaults, and ACLs separate you from the crowd.
Every permission question at a Linux admin interview traces back to this lesson.

## 1. The permission model (review in 30 seconds)
- Every file has an **owner** (user) and a **group**.
- Three permission triads: owner / group / others.
- Each triad: read (r=4), write (w=2), execute (x=1).

```
-rwxr-xr--  1 mahad devops 2048 Aug 18 10:00 deploy.sh
 ^^^        ^^^^ ^^^^^
 owner perms ^^^  group perms + 4th bit = special bits (see below)
```

## 2. The special bits — the part most people skip

| Bit | Name | On files | On directories |
|-----|------|----------|----------------|
| 4 (setuid) | SUID | Runs as the file owner, not the runner | (ignored) |
| 2 (setgid) | SGID | Runs as the file's group | New files inherit the directory's group |
| 1 (sticky) | Sticky | (ignored) | Only the owner (or root) can delete files inside |

### SUID example (the dangerous one — interview favorite)
```
chmod u+s /usr/bin/myapp        # or: chmod 4755
ls -l → -rwsr-xr-x              # note the 's' in owner slot
```
- Classic real example: `/usr/bin/passwd` has SUID so any user can change their own password (needs to write /etc/shadow).
- **Security warning:** never put SUID on scripts or on binaries owned by root without good reason. This is a privilege-escalation vector.

### SGID example (team collaboration — extremely common in real jobs)
```
mkdir /srv/projects && chown :devops /srv/projects
chmod 2770 /srv/projects       # setgid + rwx for owner/group
# any new file created here is group devops, not your default group
```

### Sticky example (/tmp — you use it daily)
```
ls -ld /tmp → drwxrwxrwt    # 't' at the end = sticky
# anyone can write to /tmp, but can only delete their OWN files
```

### The numeric 4-digit form
```
chmod 4770 file  → setuid  + rwxrwx---
chmod 2770 dir   → setgid  + rwxrwx---
chmod 1777 dir   → sticky  + rwxrwxrwx
```

## 3. umask — the default permissions everyone forgets

```
umask 022  → new files get 644, new dirs get 755
umask 002  → new files get 664, new dirs get 775   (common in shared-team setups)
umask 077  → new files get 600, new dirs get 700   (secure default)
```
Formula: `default(666 for files, 777 for dirs) MINUS umask`.
Files never get execute by default — that's why files default to 666-mask, dirs to 777-mask.

Set it in `.bashrc` for interactive shells; `/etc/login.defs` for the system default.

## 4. chown — ownership control
```
chown user file                 # change owner
chown user:group file           # change owner AND group
chown :group file               # change group only
chown -R user:group /srv/app    # recursive — CAREFUL with /
```

## 5. ACLs — when classic permissions aren't enough
Scenario: file owned by `devops` group, but one more user needs read access
without being in the group.

```
setfacl -m u:alice:r /srv/app/config.yaml    # give alice read
setfacl -m g:auditors:rx /srv/app            # group-level ACL
setfacl -x u:alice /srv/app/config.yaml      # remove alice's ACL
getfacl /srv/app/config.yaml                 # view ACLs
```
ACL entries show as a `+` at the end of `ls -l`:
```
-rw-r-----+ 1 root root 512 Aug 18 10:00 config.yaml   ← the +
```

## 6. Reading permissions fast (interview drill)
- `-rwxr-xr--` → owner: rwx, group: r-x, others: r--
- `-rwsr-xr-x` → SUID set (s in owner slot)
- `drwxrwsr-x` → SGID on directory (s in group slot)
- `drwxrwxrwt` → sticky bit (t at the end)
- `-r-xr--r--` → no write for anyone
- `---s--x--x` → SUID + execute only — looks odd on purpose; check who owns it

## 7. Common mistakes to avoid
1. `chmod 777` "because it doesn't work otherwise" — never. Debug the actual cause.
2. `chmod -R` on the wrong directory. Double-check with `pwd` first.
3. Forgetting that dirs need `x` for traversal — you can read a file but not
   "see" it if you lack execute on its parent directory.
4. Using `sudo chmod` on user files — it changes ownership state too, sometimes.
5. Confusing file permissions with ownership — `chmod` and `chown` solve different problems.

## What I learned (fill after the lab)
- (write 2-3 sentences: what surprised you, what you broke, how you fixed it)
