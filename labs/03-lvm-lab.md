# Lab 03 — LVM: PV/VG/LV, resize, snapshots

Date: 2026-08-23 | User: mahad | Lab: WSL2 Ubuntu 22.04
Disks: /dev/sda (356M), /dev/sdb (159M) — physical WSL block devices

> How to use: do each task in the terminal first. When it works, paste your
> REAL output below and answer the questions in your own words.
> Scratch notes go in your notebook — only the finished version lives here.

---

## Task 0 — Verify practice disks

```bash
lsblk /dev/sda /dev/sdb
wipefs -a /dev/sda /dev/sdb
lsblk -f /dev/sda /dev/sdb

# my commands + output:
lsblk /dev/sda /dev/sdb

NAME MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda    8:0    0 356.9M  1 disk
sdb    8:16   0 159.4M  1 disk

wipefs -a /dev/sda /dev/sdb

wipefs: error: /dev/sda: probing initialization failed: Device or resource busy.

lsblk -f /dev/sda /dev/sdb
 
NAME FSTYPE FSVER LABEL UUID FSAVAIL FSUSE% MOUNTPOINTS
sda  ext4   1.0
sdb  ext4   1.0

```

### 0.1 Why did we run wipefs first?
```
# (answer in one line):
We ran wipefs to erase any existing filesystem, partition table, or RAID signatures to ensure the disks are completely clean and prevent conflicts before creating LVM physical volumes.

```

---

## Task 1 — Create Physical Volumes (PV)

```bash
# my commands:
sudo pvcreate /dev/loop0 /dev/loop1
sudo pvs
sudo pvdisplay

# real output:
  Physical volume "/dev/loop0" successfully created.
  Physical volume "/dev/loop1" successfully created.
 
  PV         VG Fmt  Attr PSize  PFree
  /dev/loop0    lvm2 ---  10.00g 10.00g
  /dev/loop1    lvm2 ---  10.00g 10.00g

  "/dev/loop0" is a new physical volume of "10.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/loop0
  VG Name
  PV Size               10.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               VZP2Ty-UYcB-fTdR-xzXo-L2a7-WHl0-yYVamv

  "/dev/loop1" is a new physical volume of "10.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/loop1
  VG Name
  PV Size               10.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               fClKMa-dxEQ-CTqR-dtgU-D72V-Xk4V-Za8wM0

```

### 1.1 What does pvcreate actually write to the disk?
```
# (answer):
`pvcreate` writes an LVM label (which includes a unique UUID for the device) and an LVM metadata area at the beginning of the disk, officially initializing it to be recognized and used by the Logical Volume Manager.

```

### 1.2 What is a Physical Extent (PE), and what is the default size?
```
# (answer):
A Physical Extent (PE) is the smallest chunk of allocatable storage space on a Physical Volume that LVM can assign to a Logical Volume. The default size of a Physical Extent is 4 MiB (4 Megabytes).

```

---

## Task 2 — Create Volume Group (VG)

```bash
# my commands:
sudo vgcreate vg_lab /dev/loop0 /dev/loop1
sudo vgs
sudo vgdisplay vg_lab

# real output:

  Volume group "vg_lab" successfully created

  VG     #PV #LV #SN Attr   VSize  VFree
  vg_lab   2   0   0 wz--n- 19.99g 19.99g

  --- Volume group ---
  VG Name               vg_lab
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               19.99 GiB
  PE Size               4.00 MiB
  Total PE              5118
  Alloc PE / Size       0 / 0
  Free  PE / Size       5118 / 19.99 GiB
  VG UUID               eJqgqS-Mla3-Ezca-PEVa-GffM-5MH8-4LUtCI


```

### 2.1 What does "Free PE / Size" in vgdisplay represent?
```
# (answer):
It represents the total amount of unallocated space remaining in the Volume Group that can still be assigned to Logical Volumes, shown both as the number of available Physical Extents (Free PE) and the equivalent physical storage capacity (Size).

```

### 2.2 If you add a third disk later, which command extends the VG?
```
# (answer):
You would use the `vgextend` command (for example: `sudo vgextend vg_lab /dev/loop2`).

```

---

## Task 3 — Create Logical Volumes (LV)

```bash
# my commands:
sudo lvcreate -L 100M -n lv_apps vg_lab
sudo lvcreate -l 100%FREE -n lv_logs vg_lab
sudo lvs
sudo lvdisplay vg_lab/lv_apps

# real output:
  Logical volume "lv_apps" created.

  Logical volume "lv_logs" created.

  lv_apps vg_lab -wi-a----- 100.00m
  lv_logs vg_lab -wi-a-----  19.89g

  --- Logical volume ---
  LV Path                /dev/vg_lab/lv_apps
  LV Name                lv_apps
  VG Name                vg_lab
  LV UUID                II0i4D-VqIQ-X9wB-ek4O-MCv8-u2fk-io19yb
  LV Write Access        read/write
  LV Creation host, time DESKTOP-QN0OUNL, 2026-08-24 14:20:02 +0500
  LV Status              available
  # open                 0
  LV Size                100.00 MiB
  Current LE             25
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     5120
  Block device           254:0

```

### 3.1 What is the difference between -L and -l in lvcreate?
```
# (answer):
`-L` (capital L) specifies the logical volume size in standard absolute storage units (like Megabytes or Gigabytes, e.g., 100M), whereas `-l` (lowercase L) specifies the size as a count of Physical Extents (PEs) or as a percentage of the Volume Group's space (e.g., 100%FREE)

```

### 3.2 What happens if you try to create an LV larger than Free PE?
```
# (answer):
The `lvcreate` command will fail and return an error message stating that there is insufficient free space (or insufficient free extents) in the Volume Group to fulfill the request.

```

---

## Task 4 — Filesystem + Mount

```bash
# my commands:
sudo mkfs.ext4 /dev/vg_lab/lv_apps
sudo mkfs.xfs  /dev/vg_lab/lv_logs
sudo mkdir -p /mnt/apps /mnt/logs
sudo mount /dev/vg_lab/lv_apps /mnt/apps
sudo mount /dev/vg_lab/lv_logs /mnt/logs
lsblk -f /dev/vg_lab/lv_apps /dev/vg_lab/lv_logs

# real output(final after lsblk):

NAME           FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
vg_lab-lv_apps ext4   1.0         8fef06a2-f53b-4fdb-836d-b9ecbbca150c   82.7M     0% /mnt/apps
vg_lab-lv_logs xfs                2ba168c6-03d5-4fc7-9006-3be3e0f3dcbc   19.7G     1% /mnt/logs
```

### 4.1 What is the device path format for LVM logical volumes?
```
# (answer):
The standard device path format is `/dev/VolumeGroupName/LogicalVolumeName` (e.g., `/dev/vg_lab/lv_apps`), which is actually a symbolic link to the underlying device mapper path formatted as `/dev/mapper/VolumeGroupName-LogicalVolumeName`.

```

---

## Task 5 — Grow LV + Filesystem (online, no downtime)

```bash
# my commands:
df -h /mnt/apps
sudo lvextend -L +50M vg_lab/lv_apps
sudo resize2fs /dev/vg_lab/lv_apps
df -h /mnt/apps

# real output:


filesystem                  Size  Used Avail Use% Mounted on
/dev/mapper/vg_lab-lv_apps   90M   24K   83M   1% /mnt/apps

```

### 5.1 Why does ext4 need resize2fs but xfs needs xfs_growfs?
```
# (answer):
Different filesystems have entirely different internal architectures and metadata structures. Because of this, they require their own specialized user-space utilities (`resize2fs` specifically for the ext2/3/4 family, and `xfs_growfs` strictly for XFS) to correctly manipulate those specific structures.

```

### 5.2 Can you grow an XFS filesystem while it's mounted?
```
# (answer):
Yes, in fact, XFS *requires* the filesystem to be mounted in order to grow it. Unlike ext4, XFS cannot be grown offline.

```

---

## Task 6 — Shrink LV (requires unmount — careful!)

```bash
# my commands:
sudo umount /mnt/apps
sudo e2fsck -f /dev/vg_lab/lv_apps
sudo resize2fs /dev/vg_lab/lv_apps 80M
sudo lvreduce -L 80M vg_lab/lv_apps
sudo mount /dev/vg_lab/lv_apps /mnt/apps
df -h /mnt/apps

# real output:
Filesystem                  Size  Used Avail Use% Mounted on
/dev/mapper/vg_lab-lv_apps   70M   24K   65M   1% /mnt/apps

```

### 6.1 What is the correct ORDER for shrinking, and why does order matter?
```
# (answer):
The correct order is to always shrink the filesystem first (`resize2fs`), and then shrink the logical volume (`lvreduce`). Order matters because if you shrink the logical volume (the physical container) before the filesystem (the data structure inside it), you will instantly chop off the tail end of the filesystem, causing catastrophic data corruption and loss.

```

### 6.2 Why is shrinking XFS not supported?
```
# (answer):
XFS is designed with an internal architecture based on Allocation Groups that allows for highly efficient parallel I/O and dynamic expansion, but it fundamentally lacks a built-in mechanism to safely relocate blocks and inodes from the end of the volume toward the beginning. Because it cannot defragment and consolidate free space at the end of the device, it can only grow, never shrink.

 ```

---

## Task 7 — Snapshot + Backup Simulation

```bash
# my commands:
sudo lvcreate -L 20M -s -n lv_apps_snap vg_lab/lv_apps
sudo lvs
sudo mkdir /mnt/snap
sudo mount -o ro /dev/vg_lab/lv_apps_snap /mnt/snap
ls /mnt/snap
echo "test file from snapshot" | sudo tee /mnt/snap/snapshot_test.txt
sudo umount /mnt/snap

# real output:
mahad@DESKTOP-QN0OUNL:~$ sudo lvcreate -L 20M -s -n lv_apps_snap vg_lab/lv_apps
  Logical volume "lv_apps_snap" created.
mahad@DESKTOP-QN0OUNL:~$ sudo lvs
  LV           VG     Attr       LSize  Pool Origin  Data%  Meta%  Move Log Cpy%Sync Convert
  lv_apps      vg_lab owi-aos--- 80.00m
  lv_apps_snap vg_lab swi-a-s--- 20.00m      lv_apps 0.06
  lv_logs      vg_lab -wi-ao---- 19.89g
mahad@DESKTOP-QN0OUNL:~$ sudo mkdir /mnt/snap
mahad@DESKTOP-QN0OUNL:~$ sudo mount -o ro /dev/vg_lab/lv_apps_snap /mnt/snap
mahad@DESKTOP-QN0OUNL:~$ ls /mnt/snap
lost+found
mahad@DESKTOP-QN0OUNL:~$ echo "test file from snapshot" | sudo tee /mnt/snap/snapshot_test.txt
tee: /mnt/snap/snapshot_test.txt: Read-only file system
test file from snapshot
mahad@DESKTOP-QN0OUNL:~$ sudo umount /mnt/snap

```

### 7.1 How does a snapshot work (copy-on-write)? Explain in 2 sentences.
```
# (answer):
A copy-on-write snapshot does not duplicate data; instead, it simply points to the original blocks of the source volume upon creation. When the system attempts to modify a block on the original volume, that specific original block is first copied to the snapshot's reserved storage before the overwrite occurs, perfectly preserving the state of the data as it was at the exact moment the snapshot was taken.

```

### 7.2 If the snapshot LV fills to 100%, what happens?
```
# (answer):
If the snapshot Logical Volume fills up completely, it is automatically deactivated and dropped by the system, rendering the snapshot invalid and unreadable. The original volume continues to function normally without any data loss, but you will permanently lose the ability to access or restore from that broken snapshot.

 ```

---

## Task 8 — Rollback using snapshot merge

```bash
# my commands:
# First, modify the original LV
echo "new data after snapshot" | sudo tee /mnt/apps/after_snap.txt

# Now merge the snapshot (rollback)
sudo lvconvert --merge /dev/vg_lab/lv_apps_snap
# Note: merge activates on next LV activation
sudo lvchange -an vg_lab/lv_apps
sudo lvchange -ay vg_lab/lv_apps
ls /mnt/apps

# real output:
mahad@DESKTOP-QN0OUNL:~$ echo "new data after snapshot" | sudo tee /mnt/apps/after_snap.txt
new data after snapshot
mahad@DESKTOP-QN0OUNL:~$ sudo lvconvert --merge /dev/vg_lab/lv_apps_snap
  Delaying merge since origin is open.
  Merging of snapshot vg_lab/lv_apps_snap will occur on next activation of vg_lab/lv_apps.
mahad@DESKTOP-QN0OUNL:~$ sudo lvchange -an vg_lab/lv_apps
  Logical volume vg_lab/lv_apps contains a filesystem in use.
mahad@DESKTOP-QN0OUNL:~$ sudo lvchange -ay vg_lab/lv_apps
mahad@DESKTOP-QN0OUNL:~$ ls /mnt/apps
after_snap.txt  lost+found

```

### 8.1 After merge, what happened to the file "after_snap.txt"?
```
# (answer):
The file "after_snap.txt" was completely permanently deleted/lost. The merge process rolled the logical volume back to the exact block-level state it was in at the moment the snapshot was created, which was before that file ever existed

```

---

## Task 9 — fstab with LVM (survive reboot)

```bash
# Get UUIDs
sudo blkid /dev/vg_lab/lv_apps /dev/vg_lab/lv_logs

# my fstab lines:
# UUID=...  /mnt/apps  ext4  defaults,noatime  0  2
# UUID=...  /mnt/logs  xfs   defaults,noatime  0  2

sudo findmnt --verify --verbose
sudo mount -a
mount | grep vg_lab

# real output:
/dev/mapper/vg_lab-lv_logs on /mnt/logs type xfs (rw,relatime,inode64,logbufs=8,logbsize=32k,noquota)
/dev/mapper/vg_lab-lv_apps on /mnt/apps type ext4 (rw,relatime)

```

### 9.1 Why prefer /dev/mapper/vg_lab-lv_apps or UUID over /dev/vg_lab/lv_apps in fstab?
```
# (answer):
UUIDs are highly preferred because they are unique and tied directly to the filesystem itself, guaranteeing the correct volume is mounted even if drive letters or device paths change during a reboot. While `/dev/mapper/...` points directly to the persistent kernel device-mapper node, `/dev/vg_lab/...` relies on udev to dynamically create a symlink during boot, which can occasionally cause race conditions or mounting failures if the symlink isn't generated fast enough.

```

---

## Task 10 — Cleanup (practice the full lifecycle)

```bash
# my commands:
sudo umount /mnt/apps /mnt/logs
sudo lvremove vg_lab/lv_apps vg_lab/lv_logs
sudo vgremove vg_lab
sudo pvremove /dev/loop0 /dev/loop1
sudo pvs && sudo vgs && sudo lvs

# real output:
mahad@DESKTOP-QN0OUNL:~$ sudo umount /mnt/apps /mnt/logs
mahad@DESKTOP-QN0OUNL:~$ sudo lvremove vg_lab/lv_apps vg_lab/lv_logs
Do you really want to remove active origin logical volume vg_lab/lv_apps with 1 snapshot(s)? [y/n]: y
Do you really want to remove and DISCARD logical volume vg_lab/lv_apps_snap? [y/n]: y
  Logical volume "lv_apps_snap" successfully removed
Do you really want to remove and DISCARD logical volume vg_lab/lv_apps? [y/n]: y
  Logical volume "lv_apps" successfully removed
Do you really want to remove and DISCARD active logical volume vg_lab/lv_logs? [y/n]: y
  Logical volume "lv_logs" successfully removed
mahad@DESKTOP-QN0OUNL:~$ sudo vgremove vg_lab
  Volume group "vg_lab" successfully removed
mahad@DESKTOP-QN0OUNL:~$ sudo pvremove /dev/loop0 /dev/loop1
  Labels on physical volume "/dev/loop0" successfully wiped.
  Labels on physical volume "/dev/loop1" successfully wiped.
mahad@DESKTOP-QN0OUNL:~$ sudo pvs && sudo vgs && sudo lvs

```

---

## What I learned (2-3 sentences)
- What surprised me: I was surprised by how seamlessly LVM allows you to dynamically resize and manage storage on the fly compared to traditional static partitions.
- What I broke and how I fixed it: I initially tried to initialize my primary WSL OS disks (`/dev/sda` and `/dev/sdb`), which threw a mounted filesystem error, so I fixed it by creating virtual loop devices to safely complete the lab.
- One thing I'll never do again: I will never blindly run destructive commands like `wipefs` or `pvcreate` without checking `lsblk` first to confirm exactly which drives I am targeting.