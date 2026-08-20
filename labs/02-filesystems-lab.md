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
#NAME  MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
loop0   7:0    0   1G  0 loop 
loop1   7:1    0   1G  0 loop 
loop2   7:2    0   1G  0 loop

```

## Task 1 — Explore block devices
Run `lsblk -f` and `blkid`. Note what exists, and the UUID/FSTYPE columns.

```bash
# my commands + output:
# lsblk -f /dev/loop0 /dev/loop1 /dev/loop2
 NAME      FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
loop0
├─loop0p1 ext4   1.0   DATA  9b547d0d-b136-4365-8f10-c793d956b765
└─loop0p2 xfs                12abb8c4-8b62-471d-befe-2b595dd6cf4b
loop1     xfs          LOGS  5c5c9370-116e-4ed2-9b80-7e6cf722826d
 sudo blkid /dev/loop0 /dev/loop1 /dev/loop2
 No output is returned for completely unformatted raw disks)
```

### 1.2 Why is the FSTYPE column empty on a fresh disk?
```
# (answer in one line):
The column is empty because the disk is raw and has not yet been formatted with a file system (like ext4 or xfs).
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
#NAME      MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
loop0       7:0    0   1G  0 loop 
└─loop0p1 259:2    0   1G  0 part
```

### 2.1 Why GPT and not MBR?
```
# (one line):
GPT supports storage drives larger than 2TB and removes MBR's restriction of only four primary partitions.
```

---

## Task 3 — Creating filesystems

```bash
# my commands:
sudo mkfs.ext4 -L DATA /dev/loop0p1
sudo mkfs.xfs -L LOGS /dev/loop1
lsblk -f /dev/loop0 /dev/loop1
# output:
NAME      FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
loop0
├─loop0p1 ext4   1.0   DATA  9b547d0d-b136-4365-8f10-c793d956b765
└─loop0p2 xfs                12abb8c4-8b62-471d-befe-2b595dd6cf4b
loop1     xfs          LOGS  5c5c9370-116e-4ed2-9b80-7e6cf722826d
```

### 3.1 What is the journal for? Why is it a crash-safety feature?
```
# (answer):
The journal acts as a log that records intended file system changes before they are permanently written. In the event of a crash or power loss, the system reads this journal to quickly restore consistency without needing a full, time-consuming disk check (fsck).
```

### 3.2 When would you pick xfs over ext4, and vice versa?
```
# (answer):
Pick XFS for systems dealing with massive files, extreme storage capacities, and high-performance parallel I/O workloads (like database servers). Pick ext4 for general-purpose use, smaller desktop environments, and when you need maximum backward compatibility and ease of recovery.
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
i see nothing it just executes as unmounting disconnects the partition
```

### 4.1 What happened to test.txt after umount? Where is it?
```
# (answer + output):
Output of `ls /mnt/data` will be completely empty. The file `test.txt` still safely exists, but it is physically stored on the `/dev/loop0p1` filesystem. Unmounting disconnects that filesystem from `/mnt/data`, so you are now looking at the original, empty directory on your root drive.
```

### 4.2 Mount it back. Why is a mount hiding what's below it if the target dir has content?
```
# (answer):
When you mount a filesystem to a directory, the Linux kernel essentially lays the new filesystem over the top of the mount point. Any existing content in that target directory isn't deleted, but it becomes completely inaccessible (hidden in the background) until the overlying filesystem is unmounted.
```

---

## Task 5 — fstab with UUIDs (survive reboot)

Get the UUIDs: `blkid /dev/loop0p1 /dev/loop1`

```bash
# my fstab lines (via sudo tee -a /etc/fstab):
/swapfile none swap sw 0 0
UUID=9b547d0d-b136-4365-8f10-c793d956b765  /mnt/data  ext4  defaults,noatime  0  2
UUID=5c5c9370-116e-4ed2-9b80-7e6cf722826d  /mnt/logs  xfs  defaults,noatime  0  2

# verify before reboot:
sudo findmnt --verify --verbose
Output:
0 parse errors, 0 errors, 1 warning
none
   [W] non-bind mount source /swapfile is a directory or regular file
   [ ] FS type is swap
/mnt/data
   [ ] target exists
   [ ] VFS options: noatime
   [ ] UUID=9b547d0d... translated to /dev/loop0p1
   [ ] source /dev/loop0p1 exists
   [ ] FS type is ext4
/mnt/logs
   [ ] target exists
   [ ] VFS options: noatime
   [ ] UUID=5c5c9370... translated to /dev/loop1
   [ ] source /dev/loop1 exists
   [ ] FS type is xfs

sudo mount -a
mount | grep -E 'loop'
Output:
/dev/loop0p1 on /mnt/data type ext4 (rw,noatime)
/dev/loop1 on /mnt/logs type xfs (rw,noatime,inode64,logbufs=8,logbsize=32k,noquota)
```

### 5.1 Why UUID instead of /dev/loop0p1 in fstab?
```
# (answer):
Device names like /dev/loop0p1 or /dev/sdb1 are dynamically assigned by the kernel at boot time based on the order drives are detected. If you add or remove a drive, the name might change, causing the system to fail to mount or mount the wrong drive. UUIDs are permanently baked into the filesystem itself, ensuring the exact correct partition is mounted every time.
```

### 5.2 What does the last field (fsck pass) mean: 0, 1, 2?
```
# (answer):
This field tells the `fsck` utility what order to check filesystems for errors at boot. 
0 = Do not check (used for swap, temporary filesystems, and often XFS which handles its own consistency). 
1 = Check first (reserved exclusively for the root `/` filesystem). 
2 = Check after root (used for all other standard data partitions).
```

### 5.3 When would you use the `nofail` option?
```
# (answer):
You use `nofail` for external or non-essential drives so the system continues booting normally even if the device is unplugged or fails to mount.
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
Output:
/swapfile none swap sw 0 0

free -h
Output:
               total        used        free      shared  buff/cache   available
Mem:           7.6Gi       482Mi       6.1Gi       6.0Mi       1.0Gi       6.9Gi
Swap:          2.5Gi          0B       2.5Gi
```

### 6.1 Why 600 permissions on a swapfile? What does mkswap check for?
```
# (answer):
Permissions of 600 prevent regular users from reading sensitive RAM data temporarily written to disk, and `mkswap` checks that the file has secure permissions, valid sizing, and no underlying file holes.
```

### 6.2 What is swappiness, and when would you lower it?
```
# (answer):
Swappiness determines how aggressively the kernel moves memory to disk; you lower it to force the system to keep applications running in fast physical RAM, which is ideal for responsive desktops or database servers.
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
Output:
Filesystem      Size  Used Avail Use% Mounted on
none            3.8G     0  3.8G   0% /usr/lib/modules/6.18.33.2-microsoft-standard-WSL2
none            3.8G  4.0K  3.8G   1% /mnt/wsl
drivers         238G  211G   28G  89% /usr/lib/wsl/drivers
/dev/sdd       1007G  6.3G  950G   1% /
none            3.8G   52K  3.8G   1% /mnt/wslg
none            3.8G     0  3.8G   0% /usr/lib/wsl/lib
rootfs          3.8G  2.8M  3.8G   1% /init
none            3.8G  864K  3.8G   1% /run
none            3.8G     0  3.8G   0% /run/lock
none            3.8G     0  3.8G   0% /run/shm
none            3.8G   80K  3.8G   1% /mnt/wslg/versions.txt
none            3.8G   80K  3.8G   1% /mnt/wslg/doc
C:\             238G  211G   28G  89% /mnt/c
snapfuse         21M   21M     0 100% /snap/astral-uv/1392
snapfuse         20M   20M     0 100% /snap/astral-uv/1662
snapfuse        128K  128K     0 100% /snap/bare/5
snapfuse         74M   74M     0 100% /snap/core22/2133
snapfuse         67M   67M     0 100% /snap/core24/1349
snapfuse         74M   74M     0 100% /snap/core22/2139
snapfuse         67M   67M     0 100% /snap/core24/1643
snapfuse         92M   92M     0 100% /snap/gtk-common-themes/1535
snapfuse         51M   51M     0 100% /snap/snapd/25202
snapfuse         50M   50M     0 100% /snap/snapd/24792
tmpfs           775M  8.0K  775M   1% /run/user/1000
tmpfs           775M  8.0K  775M   1% /run/user/0
Filesystem       Inodes IUsed    IFree IUse% Mounted on
/dev/sdd       67108864 75237 67033627    1% /
du: cannot read directory '/var/log/private': Permission denied
du: cannot read directory '/var/log/apache2': Permission denied
du: cannot read directory '/var/log/installer': Permission denied
du: cannot read directory '/var/log/unattended-upgrades': Permission denied
808M    /var/log
du: cannot read directory '/home/tester': Permission denied
1.1G    /home
If you wish to check the consistency of an XFS filesystem or
repair a damaged filesystem, see xfs_repair(8).
```

### 7.1 A disk shows 20% used in `df -h` but you can't create files. Why? (hint: df -i)
```
# (answer):
The filesystem has run out of available inodes (usually due to millions of tiny files), meaning it cannot register any new files even though raw physical disk space is still available.
```

---

## What I learned (2-3 sentences)
- What surprised me:I was surprised that mounting a filesystem over a directory completely hides that directory's original contents, and that Linux commands like umount or swapon are completely silent when they execute successfully.
- What I broke and how I fixed it:I couldn't format my new partition because WSL didn't automatically detect /dev/loop0p1, which I fixed by detaching and reattaching the loop device using the -P flag to force a partition scan. I also forgot to add my swap file to /etc/fstab, resulting in empty grep output, which I fixed by appending the configuration using echo and tee -a.
- One thing I'll never do again:I will never assume a newly created filesystem or swap file will automatically survive a reboot without explicitly adding its configuration to /etc/fstab.
