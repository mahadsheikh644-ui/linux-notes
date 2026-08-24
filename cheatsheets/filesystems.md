# Cheatsheet — Filesystems

## Discover
```
lsblk -f        # tree + FS type + UUID + mountpoint
blkid           # raw UUID/FSTYPE per device
df -h           # space        df -i  # inodes
du -sh dir      # dir usage
sudo fdisk -l   # partition tables
```

## Partition (GPT)
```
sudo fdisk /dev/sdb     # g = new GPT, n = new partition, w = write
sudo partprobe          # re-read table without reboot
```

## Format
```
sudo mkfs.ext4 -L LABEL /dev/sdb1     # default, journaled, mature
sudo mkfs.xfs  -L LABEL /dev/sdb2     # huge files, can GROW but NOT shrink
```

## Mount / unmount
```
sudo mkdir -p /mnt/data
sudo mount /dev/sdb1 /mnt/data        # or -o noatime,ro,noexec
sudo umount /mnt/data                 # or: umount /dev/sdb1
mount | grep sdb1                     # what's mounted where
```

## fstab — /etc/fstab (UUID, never /dev/sdX)
```
# device         mountpoint  fstype  options               dump  pass
UUID=abc-123     /mnt/data   ext4    defaults,noatime      0     2
/swapfile        none        swap    sw                    0     0

sudo mount -a                    # apply all — test after edits!
sudo findmnt --verify --verbose  # syntax check before reboot
# options: nofail (boot continues if disk missing) | ro | noexec
# pass: 0=never check, 1=root, 2=everything else
```

## Swap
```
sudo fallocate -l 2G /swapfile      # then: chmod 600
sudo mkswap /swapfile && sudo swapon /swapfile
sysctl vm.swappiness=10             # persist in /etc/sysctl.d/
free -h                             # watch swap usage
```

## Repair / inodes
```
sudo fsck /dev/sdb1     # ONLY unmounted
df -i                    # inodes exhausted? "no space" with free GBs
```

## Diagnose order (when "disk full")
1. `df -h` (which mount is full?) → 2. `df -i` (inodes?) →
3. `du -sh /*` (what ate it) → 4. check deleted-but-open files:
`lsof +L1` (space held by open deleted files) → 5. logrotate / empty big logs
