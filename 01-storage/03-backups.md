# Lesson 06 — Backups

Date: 2026-09-02 | User: mahad | Lab: WSL2 Ubuntu 22.04

## Why this matters
Backups are the only safety net against data loss. Disk failure, accidental deletion, ransomware — without a backup, data is gone forever. Interviewers ask: What is the 3-2-1 rule? What is the difference between full, incremental, and differential backups? How do you verify a backup actually works?

## Types of backups

TODO: explain full, incremental, differential backups with diagram

| Type | What it backs up | Speed | Restore complexity |
|------|-----------------|-------|--------------------|
| Full | Everything | Slow | Simple |
| Incremental | Changed since last backup | Fast | Complex |
| Differential | Changed since last full | Medium | Medium |

## tar — archive and compress

TODO: fill in commands after hands-on practice

\# my commands:
# tar -czf backup.tar.gz /path/to/dir
# tar -tzf backup.tar.gz
# tar -xzf backup.tar.gz
# real output:

\
## rsync — sync and incremental backup

TODO: fill in commands after hands-on practice

\# my commands:
# sudo rsync -avz /source/ /backup/
# real output:

\
## Scheduled backups (cron)

TODO: explain cron jobs for automated backups

\# crontab -e
# 0 2 * * * tar -czf /backup/$(date +%F).tar.gz /important/data
# real output:

\
## Verification

TODO: explain how to verify backups work

## The 3-2-1 rule

TODO: explain 3-2-1 backup rule

- 3 copies of data
- 2 different storage media
- 1 offsite/cloud

## Common mistakes to avoid
1. Never back up to the same physical disk
2. Always verify backups by extracting and checking
3. Never skip testing the restore process
4. Do not rely on a single backup location

## What I learned (fill after the lab)
- (write 2-3 sentences: what surprised you, what you broke, how you fixed it)
