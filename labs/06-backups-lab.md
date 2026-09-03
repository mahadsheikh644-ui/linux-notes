# Lab 03 — Backups: tar, rsync, cron, verify

Date: 2026-09-02 | User: mahad | Lab: WSL2 Ubuntu 22.04

> How to use: do each task in the terminal first. When it works, paste your
> REAL output below and answer the questions in your own words.
> Scratch notes go in your notebook — only the finished version lives here.

---

## Task 1 — Create a tar archive

```bash
# my commands:
# sudo mkdir -p /tmp/backup-lab
# echo "test file 1" | sudo tee /tmp/backup-lab/file1.txt
# echo "test file 2" | sudo tee /tmp/backup-lab/file2.txt
# tar -czf /tmp/backup.tar.gz /tmp/backup-lab
# tar -tzf /tmp/backup.tar.gz
# real output:
mahad@DESKTOP-QN0OUNL:~$ sudo mkdir -p /tmp/backup-lab
[sudo] password for mahad:
mahad@DESKTOP-QN0OUNL:~$  echo "test file 1" | sudo tee /tmp/backup-lab/file1.txt
test file 1
mahad@DESKTOP-QN0OUNL:~$  echo "test file 2" | sudo tee /tmp/backup-lab/file2.txt
test file 2
mahad@DESKTOP-QN0OUNL:~$ tar -czf /tmp/backup.tar.gz /tmp/backup-lab
tar: Removing leading `/' from member names
mahad@DESKTOP-QN0OUNL:~$  tar -tzf /tmp/backup.tar.gz
tmp/backup-lab/
tmp/backup-lab/file2.txt
tmp/backup-lab/file1.txt
```

### 1.1 What is the difference between `tar -c` and `tar -czf`?
```
# (answer):
tar -c creates an archive, while tar -czf creates a gzip-compressed archive and saves it to a specified filename.
```

---

## Task 2 — Extract and verify

```bash
# my commands:
# mkdir /tmp/restore-test
# tar -xzf /tmp/backup.tar.gz -C /tmp/restore-test
# ls -la /tmp/restore-test/backup-lab/
# diff <(cd /tmp/backup-lab && find . | sort) <(cd /tmp/restore-test/backup-lab && find . | sort)
# real output:
mahad@DESKTOP-QN0OUNL:/$ diff <(cd /tmp/backup-lab && find . | sort) <(cd /tmp/restore-test/tmp/backup-lab && find . | sort)
mahad@DESKTOP-QN0OUNL:/$ tar -xzf /tmp/backup.tar.gz -C /tmp/restore-test
mahad@DESKTOP-QN0OUNL:/$ ls -la /tmp/restore-test/tmp/backup-lab/
total 16
drwxr-xr-x 2 mahad mahad 4096 Sep  3 14:01 .
drwxr-xr-x 3 mahad mahad 4096 Sep  3 14:07 ..
-rw-r--r-- 1 mahad mahad   12 Sep  3 14:01 file1.txt
-rw-r--r-- 1 mahad mahad   12 Sep  3 14:01 file2.txt
mahad@DESKTOP-QN0OUNL:/$ diff <(cd /tmp/backup-lab && find . | sort) <(cd /tmp/restore-test/tmp/backup-lab && find . | sort)
```

### 2.1 How would you verify a backup is complete and correct?
```
# (answer):
I would check that the backup file exists, verify its size/checksum, and restore a few files to confirm they work correctly.
```

---

## Task 3 — rsync for incremental backup

```bash
# my commands:
# echo "new file" | sudo tee /tmp/backup-lab/file3.txt
# sudo rsync -avz --delete /tmp/backup-lab/ /tmp/rsync-backup/
# ls -la /tmp/rsync-backup/
# real output:
mahad@DESKTOP-QN0OUNL:/$ ls -la /tmp/rsync-backup/
total 20
drwxr-xr-x 2 root root 4096 Sep  3 14:23 .
drwxrwxrwt 9 root root 4096 Sep  3 14:24 ..
-rw-r--r-- 1 root root   12 Sep  3 14:01 file1.txt
-rw-r--r-- 1 root root   12 Sep  3 14:01 file2.txt
-rw-r--r-- 1 root root    9 Sep  3 14:23 file3.txt
```

### 3.1 What does --delete do in rsync?
```
# (answer):
rsync --delete removes files from the destination that no longer exist in the source, keeping both directories exactly in sync.
```

---

## Task 4 — Cron scheduled backup

```bash
# my commands:
# crontab -l
# echo "0 2 * * * tar -czf /tmp/daily-$(date +\%F).tar.gz /tmp/backup-lab" | crontab -
# crontab -l
# real output:
mahad@DESKTOP-QN0OUNL:~$ crontab -l
no crontab for mahad
mahad@DESKTOP-QN0OUNL:~$ echo "0 2 * * * tar -czf /tmp/daily-$(date +\%F).tar.gz /tmp/backup-lab" | crontab -
mahad@DESKTOP-QN0OUNL:~$ crontab -l
0 2 * * * tar -czf /tmp/daily-$(date +%F).tar.gz /tmp/backup-lab
```

### 4.1 What does the cron schedule mean?
```
# (answer):
The schedule "0 2 * * *" means run at 2:00 AM every day. The command creates a dated tar.gz backup of the lab folder automatically each night.
```

---

## Task 5 — Backup verification

```bash
# my commands:
# ls -lh /tmp/backup.tar.gz
# file /tmp/backup.tar.gz
# tar -tzf /tmp/backup.tar.gz | wc -l
# real output:
mahad@DESKTOP-QN0OUNL:/$  ls -lh /tmp/backup.tar.gz
-rw-r--r-- 1 mahad mahad 187 Sep  3 14:01 /tmp/backup.tar.gz
mahad@DESKTOP-QN0OUNL:/$  file /tmp/backup.tar.gz
/tmp/backup.tar.gz: gzip compressed data, from Unix, original size modulo 2^32 10240
mahad@DESKTOP-QN0OUNL:/$  tar -tzf /tmp/backup.tar.gz | wc -l
3
```

---

## Task 6 — Cleanup

```bash
# my commands:
# rm -rf /tmp/backup-lab /tmp/backup.tar.gz /tmp/restore-test /tmp/rsync-backup
# real output:
mahad@DESKTOP-QN0OUNL:/tmp$ sudo rm -rf /tmp/backup-lab /tmp/backup.tar.gz /tmp/restore-test /tmp/rsync-backup
mahad@DESKTOP-QN0OUNL:/tmp$
```

---

## What I learned (2-3 sentences)
- What surprised me:I was surprised that cron can automatically run commands and scripts at a specific time.
- What I broke and how I fixed it:I had an issue with the backup restore path, so I checked the extracted files and corrected the path used for comparison.
- One thing I will never do again:I will never assume the backup contains the same directory structure without checking it first.
