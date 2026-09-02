# Lab 05 — RAID: Create, monitor, simulate failure, recover

Date: 2026-09-02 | User: mahad | Lab: WSL2 Ubuntu 22.04
Disks: ~/linux-lab/day1/disk1.img, disk2.img, disk3.img (1GB each, attached as loop devices)

> How to use: do each task in the terminal first. When it works, paste your
> REAL output below and answer the questions in your own words.
> Scratch notes go in your notebook — only the finished version lives here.

---

## Task 0 — Attach practice disks (loop devices)

```bash
# my commands:
sudo losetup -f
sudo losetup /dev/loop0 ~/linux-lab/day1/disk1.img
sudo losetup /dev/loop1 ~/linux-lab/day1/disk2.img
sudo losetup /dev/loop2 ~/linux-lab/day1/disk3.img
lsblk /dev/loop0 /dev/loop1 /dev/loop2
# real output:
mahad@DESKTOP-QN0OUNL:~$ sudo losetup -f
/dev/loop0
mahad@DESKTOP-QN0OUNL:~$ sudo losetup /dev/loop0 ~/linux-lab/day1/disk1.img
mahad@DESKTOP-QN0OUNL:~$ sudo losetup /dev/loop1 ~/linux-lab/day1/disk2.img
mahad@DESKTOP-QN0OUNL:~$ sudo losetup /dev/loop2 ~/linux-lab/day1/disk3.img
mahad@DESKTOP-QN0OUNL:~$ lsblk /dev/loop0 /dev/loop1 /dev/loop2
NAME  MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
loop0   7:0    0   1G  0 loop
loop1   7:1    0   1G  0 loop
loop2   7:2    0   1G  0 loop

```

### 0.1 Why do we use loop devices instead of real disks?
```
# (answer in one line):
We use loop devices because they allow disk image files to be used like real disks without risking the actual system disks.

```

### 0.2 Why did we run wipefs first?
```
# my commands:
sudo wipefs -a /dev/loop0 /dev/loop1 /dev/loop2
lsblk -f /dev/loop0 /dev/loop1 /dev/loop2
# real output:
mahad@DESKTOP-QN0OUNL:~$ sudo wipefs -a /dev/loop0 /dev/loop1 /dev/loop2
/dev/loop0: 8 bytes were erased at offset 0x00000200 (gpt): 45 46 49 20 50 41 52 54
/dev/loop0: 8 bytes were erased at offset 0x3ffffe00 (gpt): 45 46 49 20 50 41 52 54
/dev/loop0: 2 bytes were erased at offset 0x000001fe (PMBR): 55 aa
/dev/loop1: 4 bytes were erased at offset 0x00000000 (xfs): 58 46 53 42
/dev/loop0: calling ioctl to re-read partition table: Invalid argument
mahad@DESKTOP-QN0OUNL:~$ lsblk -f /dev/loop0 /dev/loop1 /dev/loop2
NAME  FSTYPE FSVER LABEL UUID FSAVAIL FSUSE% MOUNTPOINTS
loop0
loop1
loop2
```
# (answer in one line):
We ran wipefs first to remove existing filesystem or RAID signatures so the loop devices could be safely used for the RAID array.
```

---

## Task 1 — Create RAID 1 array

```bash
# my commands:
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/loop0 /dev/loop1
sudo mdadm --detail /dev/md0
cat /proc/mdstat
# real output:
d@DESKTOP-QN0OUNL:~$ sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/loop0 /dev/loop1
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array?
Continue creating array? (y/n) y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
mahad@DESKTOP-QN0OUNL:~$
mahad@DESKTOP-QN0OUNL:~$ sudo mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Sep  2 15:53:24 2026
        Raid Level : raid1
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Wed Sep  2 15:53:30 2026
             State : clean
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : resync

              Name : DESKTOP-QN0OUNL:0  (local to host DESKTOP-QN0OUNL)
              UUID : 45065c83:e0647b3e:b8bb2fa7:5b29d659
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       7        0        0      active sync   /dev/loop0
       1       7        1        1      active sync   /dev/loop1
mahad@DESKTOP-QN0OUNL:~$ cat /proc/mdstat
Personalities : [raid1]
md0 : active raid1 loop1[1] loop0[0]
      1046528 blocks super 1.2 [2/2] [UU]

unused devices: <none>
```

### 1.1 What RAID level are you creating and what does the output show?
```
# (answer):
we are creating RAID 1 (mirroring). The output shows that /dev/md0 is active and contains two loop devices, /dev/loop0 and /dev/loop1, with both devices synchronized and healthy.
```

### 1.2 What is the difference between "resync" and "recovery" in mdadm output?
```
# (answer):
Resync synchronizes the RAID array when it is first created, while recovery rebuilds the array after a failed disk is replaced or re-added.
```

---

## Task 2 — Add spare disk and simulate failure

```bash
# my commands:
sudo mdadm --manage /dev/md0 --add /dev/loop2
sudo mdadm --manage /dev/md0 --fail /dev/loop0
sudo mdadm --manage /dev/md0 --remove /dev/loop0
cat /proc/mdstat
# real output:
mahad@DESKTOP-QN0OUNL:~$ sudo mdadm --manage /dev/md0 --add /dev/loop2
mdadm: added /dev/loop2
mahad@DESKTOP-QN0OUNL:~$ sudo mdadm --manage /dev/md0 --fail /dev/loop0
mdadm: set /dev/loop0 faulty in /dev/md0
mahad@DESKTOP-QN0OUNL:~$ sudo mdadm --manage /dev/md0 --remove /dev/loop0
mdadm: hot removed /dev/loop0 from /dev/md0
mahad@DESKTOP-QN0OUNL:~$ cat /proc/mdstat
Personalities : [raid1]
md0 : active raid1 loop2[2] loop1[1]
      1046528 blocks super 1.2 [2/2] [UU]

unused devices: <none>
```

### 2.1 What happened to the array when one disk failed?
```
# (answer):
The RAID 1 array became degraded, but it continued working because the remaining disk still contained a complete copy of the data.
```

### 2.2 What does "degraded" mode mean?
```
# (answer):
Degraded mode means that one or more RAID disks have failed or been removed, so the array is running with fewer active disks than normal.
```

---

## Task 3 — Re-add disk and verify rebuild

```bash
# my commands:
sudo mdadm --manage /dev/md0 --add /dev/loop0
cat /proc/mdstat
# real output:
Personalities : [raid1]
md0 : active raid1 loop0[3](S) loop2[2] loop1[1]
      1046528 blocks super 1.2 [2/2] [UU]

unused devices: <none>
```

### 3.1 How long does rebuild take and how can you monitor it?
```
# (answer):
The rebuild time depends on disk size and system speed. We can monitor the rebuild using cat /proc/mdstat or sudo mdadm --detail /dev/md0.
```

---

## Task 4 — mdadm.conf

```bash
# my commands:
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
cat /etc/mdadm/mdadm.conf
# real output:
mahad@DESKTOP-QN0OUNL:~$ sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
ARRAY /dev/md0 metadata=1.2 spares=1 name=DESKTOP-QN0OUNL:0 UUID=45065c83:e0647b3e:b8bb2fa7:5b29d659
mahad@DESKTOP-QN0OUNL:~$ cat /etc/mdadm/mdadm.conf
# mdadm.conf
#
# !NB! Run update-initramfs -u after updating this file.
# !NB! This will ensure that initramfs has an uptodate copy.
#
# Please refer to mdadm.conf(5) for information about this file.
#

# by default (built-in), scan all partitions (/proc/partitions) and all
# containers for MD superblocks. alternatively, specify devices to scan, using
# wildcards if desired.
#DEVICE partitions containers

# automatically tag new arrays as belonging to the local system
HOMEHOST <system>

# instruct the monitoring daemon where to send mail alerts
MAILADDR root

# definitions of existing MD arrays

# This configuration was auto-generated on Wed, 02 Sep 2026 15:52:39 +0500 by mkconf
ARRAY /dev/md0 metadata=1.2 spares=1 name=DESKTOP-QN0OUNL:0 UUID=45065c83:e0647b3e:b8bb2fa7:5b29d659
```

### 4.1 Why is mdadm.conf important?
```
# (answer):
mdadm.conf stores RAID configuration information so the system can identify and assemble the RAID array automatically.
```

---

## Task 5 — Cleanup

```bash
# my commands:
sudo mdadm --stop /dev/md0
sudo mdadm --zero-superblock /dev/loop0 /dev/loop1 /dev/loop2
sudo losetup -d /dev/loop0 /dev/loop1 /dev/loop2
cat /proc/mdstat
# real output:
Personalities : [raid1]
unused devices: <none>
```

---

## What I learned (2-3 sentences)
- What surprised me:I was surprised that RAID 1 can continue working even when one disk fails.
- What I broke and how I fixed it:I initially tried to use the real WSL disks, but they were in use, so I switched to loop devices for safe RAID practice.
- One thing I will never do again:I will never wipe or modify system disks without first checking whether they are in use.
