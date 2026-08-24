# Lab 01 — Answers & Log (fill this in as you work)

Date: 2026-08-18 | User: mahad | Lab: WSL2 Ubuntu 22.04

> How to use: do each task in the terminal first. When it works, paste your
> REAL output below and answer the questions in your own words.
> Scratch notes go in your notebook — only the finished version lives here.

---

## Task 1 — SUID/SGID/sticky
sudo mkdir -p /tmp/perm-lab
sudo touch /tmp/perm-lab/tool
sudo chown root:root /tmp/perm-lab/tool
verify:ls -l /tmp/perm-lab/tool
### 1.3 SUID on a file
```bash
# my commands:
# (sudo chmod u+s /tmp/perm-lab/tool)
verify:ls -l /tmp/perm-lab/tool
# real output:
# (-rwsr--r-- 1 root root ... /tmp/perm-lab/tool)
```

### 1.4-1.5 Shared dir with SGID
```bash
# commands + output (create group, dir, set SGID, verify 's'):
sudo groupadd devops
sudo mkdir -p /srv/team
sudo chown root:devops /srv/team
sudo chmod 770 /srv/team
sudo chmod g+s /srv/team
ls -ld /srv/team

Output:drwxrws--- ... root devops /srv/team
```

### 1.6 Sticky bit test 
sudo chmod +t /tmp/perm-lab
What happened when you tried to delete another user's file?
```
# (answer + output):
A different normal user cannot delete a file owned by another user
from a sticky-bit directory.

Output:rm: cannot remove '/tmp/perm-lab/otherfile': Operation not permitted
```

---

## Task 2 — umask

### 2.1-2.2 Demonstrate and explain
```bash
# commands + outputs for umask 077 and umask 002
umask 077
touch file077
ls -l file077

output: -rw------- 1 <user> <group> 0 ... file077

umask 002
touch file002
ls -l file002

output: -rw-rw-r-- 1 <user> <group> 0 ... file002
```
Difference explained:With umask 077, new files are private to the owner and have no
permissions for the group or other users.
With umask 002, the owner and group can write, while other users
can only read the new file.
```

```

### 2.3 Where to set persistent umask
One user:
```
~/.profile
```
All users:
```
# /etc/profile
```

---

## Task 3 — ACLs

### 3.1-3.3 Grant alice read via ACL
```bash
# commands + output (setfacl, getfacl, ls -l with the +)
sudo mkdir -p /srv/app
sudo touch /srv/app/config.yaml
sudo chown root:devops /srv/app/config.yaml
sudo chmod 640 /srv/app/config.yaml
sudo setfacl -m u:alice:r-- /srv/app/config.yaml
getfacl /srv/app/config.yaml
ls -l /srv/app/config.yaml
```

### 3.4 Why not just add alice to the group?
```
# (one-line answer)
Adding alice to the devops group would give her the group's permissions on all files/directories where devops has access, not just config.yaml.
```

---

## Task 4 — Permission reading drill

| ls -l output | Numeric mode |
|---|---|
| `-rwxr-xr--` | `(750)` |
| `-rwsr-xr-x` | `(4755)` |
| `drwxrwsr-x` | `(2775)` |
| `drwxrwxrwt` | `(1777)` |

Difference between `chmod 1777` and `chmod 3777` on a directory:

```
# (chmod 1777 sets the sticky bit, while chmod 3777 sets both SGID)
```

---

## Task 5 — The sysadmin scenario (interview question)

Situation: user can't write to `/var/www/html` (root:www-data, 755). Worked last week.

### 5.1 Diagnosis commands + real output
```bash
# run these and paste the REAL output below
ls -ld /var/www/html
id
getfacl /var/www/html
df -h /var/www/html
Output:
drwxr-xr-x 2 root root 4096 Sep 20  2025 /var/www/html
uid=1000(mahad) gid=1000(mahad) groups=1000(mahad),27(sudo),121(docker)
# file: var/www/html
# owner: root
# group: root
user::rwx
group::r-x
other::r-x

Filesystem      Size  Used Avail Use% Mounted on
/dev/sdd       1007G  7.0G  949G   1% /
```

### 5.2 Possible causes (at least 3)
1.The directory has 755 permissions, so only root has write permission.
2.The user may not belong to the www-data group.
3.An ACL may be restricting or changing the effective permissions.
4.The filesystem may be mounted as read-only or the disk may be full.

### 5.3 The fix + why not 777
Commands I would run:
sudo chown root:www-data /var/www/html
sudo chmod 775 /var/www/html
```bash
# (This gives root full access and the www-data group read/write/execute
access without giving every user write permission.)
```
Why NOT `chmod 777`:
```
# (Do NOT use chmod 777 because it gives read/write/execute permission
to everyone and creates an unnecessary security risk.)
```

---

## What I learned (2-3 sentences)
- What surprised me:I learned that small permission changes like SUID, SGID, sticky bit, and ACLs can significantly change who can access or modify files
- What I broke and how I fixed it:I had permission problems while testing files and directories, so I checked ownership and permissions using ls -l, getfacl, and corrected them with chmod and chown.
- One thing I'll never do again:I will never use chmod 777 as a quick fix because it gives unnecessary permissions to everyone.
