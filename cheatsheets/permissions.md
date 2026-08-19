# Cheatsheet — Permissions

## Reading ls -l
```
-rwxr-xr--  owner: rwx | group: r-x | others: r--
-rwsr-xr-x  SUID (s in owner slot)      -rwxr-sr-x  SGID (s in group slot)
-rwxrwxrwt  sticky (t at end)           -rw-r-----+  has ACL entries (+)
```

## chmod
```
chmod 755 file         # numeric
chmod u+x file         # symbolic: user +x
chmod g-w,o-r file     # group -w, others -r
chmod a+x file         # all +x
chmod 4755 file        # setuid + rwxr-xr-x
chmod 2770 dir         # setgid + rwxrwx---
chmod 1777 dir         # sticky + rwxrwxrwx (like /tmp)
chmod -R 750 /srv/app  # recursive
```

## chown / chgrp
```
chown user file
chown user:group file
chown :group file
chown -R user:group dir
```

## umask
```
umask                    # show current
umask 022                # 644 files / 755 dirs (default)
umask 002                # 664 / 775 (team setups)
umask 077                # 600 / 700 (secure)
files: 666 - umask | dirs: 777 - umask
Persistent: ~/.bashrc (user) or /etc/login.defs (system)
```

## ACLs
```
setfacl -m u:alice:r file    # user ACL
setfacl -m g:auditors:rx dir # group ACL
setfacl -x u:alice file      # remove
setfacl -b file              # remove all ACLs
getfacl file                 # view
```

## Diagnose order (when "permission denied")
1. `ls -l` (owner/group/perms) → 2. `id` (am I owner? which group?) →
3. check parent dirs for `x` → 4. `getfacl` → 5. check SUID/SGID/sticky →
6. `sudo ls -la` (see what you can't see)
