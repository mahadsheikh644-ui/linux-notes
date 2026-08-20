# Lab 02 — Filesystems: partitions, ext4/xfs, mount, fstab, swap

Date: 2026-08-20 | User: mahad | Lab: WSL2 Ubuntu 22.04
Disks: ~/linux-lab/day1/disk1.img, disk2.img, disk3.img (1GB each)

> How to use: do each task in the terminal first. When it works, paste your
> REAL output below and answer the questions in your own words.
> Scratch notes go in your notebook — only the finished version lives here.

---

## Task 0 — Attach the practice disks (loop devices)

```bash
# my commands (losetup -f shows the next free loop device):
sudo losetup -f
sudo losetup /dev/loop0 ~/linux-lab/day1/disk1.img
sudo losetup /dev/loop1 ~/linux-lab/day1/disk2.img
sudo losetup /dev/loop2 ~/linux-lab/day1/disk3.img
lsblk /dev/loop0 /dev/loop1 /dev/loop2

# real output:
#
#
#
```

## Task 1 — Explore block devices
Run `lsblk -f` and `blkid`. Note what exists, and the UUID/FSTYPE columns.

```bash
# my commands + output:
#
```

### 1.2 Why is the FSTYPE column empty on a fresh disk?
```
# (answer in one line):
```

---

## Task 2 — Partitioning with fdisk (GPT)

Partition `/dev/loop0`:
```bash
sudo fdisk /dev/loop0
# then: g (new GPT table), n (new partition, accept defaults = whole disk), w (write)
sudo partprobe /dev/loop0
lsblk /dev/loop0

# real output (partition should appear as /dev/loop0p1):
#
```

### 2.1 Why GPT and not MBR?
```
# (one line):
```

---

## Task 3 — Creating filesystems

```bash
# my commands:
sudo mkfs.ext4 -L DATA /dev/loop0p1
sudo mkfs.xfs -L LOGS /dev/loop1
lsblk -f /dev/loop0 /dev/loop1
```

### 3.1 What is the journal for? Why is it a crash-safety feature?
```
# (answer):
```

### 3.2 When would you pick xfs over ext4, and vice versa?
```
# (answer):
```

---

## Task 4 — Mounting

```bash
# my commands:
sudo mkdir -p /mnt/data /mnt/logs
sudo mount /dev/loop0p1 /mnt/data
sudo mount /dev/loop1 /mnt/logs
# write a test file in each:
echo "hello from disk1" | sudo tee /mnt/data/test.txt
lsblk -f
sudo umount /mnt/data
ls /mnt/data        # what do you see now, and why?
```

### 4.1 What happened to test.txt after umount? Where is it?
```
# (answer + output):
```

### 4.2 Mount it back. Why is a mount hiding what's below it if the target dir has content?
```
# (answer):
```

---

## Task 5 — fstab with UUIDs (survive reboot)

Get the UUIDs: `blkid /dev/loop0p1 /dev/loop1`

```bash
# my fstab lines (via sudo tee -a /etc/fstab):
#
# verify before reboot:
sudo findmnt --verify --verbose /etc/fstab
sudo mount -a
mount | grep -E 'loop|data|logs'
```

### 5.1 Why UUID instead of /dev/loop0p1 in fstab?
```
# (answer):
```

### 5.2 What does the last field (fsck pass) mean: 0, 1, 2?
```
# (answer):
```

### 5.3 When would you use the `nofail` option?
```
# (answer):
```

---

## Task 6 — Swap (the overflow)

```bash
# my commands:
sudo fallocate -l 512M /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
free -h
grep swap /etc/fstab
```

### 6.1 Why 600 permissions on a swapfile? What does mkswap check for?
```
# (answer):
```

### 6.2 What is swappiness, and when would you lower it?
```
# (answer):
```

---

## Task 7 — Health checks

```bash
df -h
df -i /mnt/data
du -sh /var/log /home
# unmount /mnt/logs, then run fsck on the xfs disk (read-only check):
sudo umount /mnt/logs
sudo fsck.xfs -n /dev/loop1
```

### 7.1 A disk shows 20% used in `df -h` but you can't create files. Why? (hint: df -i)
```
# (answer):
```

---

## What I learned (2-3 sentences)
- What surprised me:
- What I broke and how I fixed it:
- One thing I'll never do again:
