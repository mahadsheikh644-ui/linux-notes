# Cheatsheet — Backups

## Tools

### tar (archive + compress)
\tar -czf backup.tar.gz /path/to/dir    # compress (gzip)
tar -czf backup.tar.gz /path/to/dir    # create archive
tar -xzf backup.tar.gz                  # extract
tar -tzf backup.tar.gz                  # list contents
tar -cf backup.tar /path/to/dir         # archive without compression
\
### rsync (sync/incremental)
\sudo rsync -avz /source/ /backup/        # archive mode, verbose, compress
sudo rsync -avz --delete /source/ /backup/ # mirror (delete extras)
sudo rsync -avz --progress /source/ /backup/ # with progress
\
### dump / restore (filesystem-level)
\sudo dump -0uf /backup/partition.dump /dev/sdb1
sudo restore -if /backup/partition.dump
\
## Scheduled backups (cron)
\# crontab -e
0 2 * * * tar -czf /backup/$(date +%F).tar.gz /important/data
0 2 * * 0 rsync -avz /data/ /backup/weekly/
\
## Verification
\tar -tzf backup.tar.gz | head -20    # verify archive contents
diff <(cd /source && find .) <(cd /backup && find .)  # compare
\
## Backup types
- Full: entire dataset
- Incremental: only changed since last backup
- Differential: changed since last full backup

## 3-2-1 rule
- 3 copies of data
- 2 different storage media
- 1 offsite/cloud
