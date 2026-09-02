# Lesson 03 — LVM (Logical Volume Manager)

Date: 2026-08-23 | Live demo on WSL2 Ubuntu 22.04 | Follow-up to README index entry

## Why this matters
Production Linux servers almost never use raw partitions directly. LVM gives you:
- Resize filesystems online (grow instantly, shrink with care)
- Snapshots for consistent backups / rollback before risky changes
- Pool multiple disks into one big volume group
- Migrate data across physical disks without downtime (pvmove)

Interviewers ask: How do you grow /var without downtime? What is a snapshot? Explain PV/VG/LV.

---
## 1. The LVM stack (bottom to top)

Physical Disks/Partitions
        |
        v
+-------------------+
|  Physical Volume  |  <- pvcreate /dev/sdb  (writes LVM header)
|      (PV)         |
+--------+----------+
         |
         v
+-------------------+
|  Volume Group     |  <- vgcreate vg_name /dev/sdb /dev/sdc
|      (VG)         |     (pool of PVs = one big storage bucket)
+--------+----------+
         |
         v
+-------------------+
|  Logical Volume   |  <- lvcreate -L 10G -n lv_name vg_name
|      (LV)         |     (carved out of VG, formatted + mounted)
+-------------------+

Key unit: Physical Extent (PE)
- Default PE size = 4 MiB
- LV size = number of PEs x PE size
- vgdisplay shows Free PE / Size - that is your expansion room

---

## 2. PV — Physical Volume

```
sudo pvcreate /dev/sdb /dev/sdc    # initialize disks/partitions for LVM
sudo pvdisplay                     # show all PVs
sudo pvs                           # short view
sudo pvremove /dev/sdb             # undo (removes LVM header)
```

What pvcreate actually does: writes an LVM label in the first 512 bytes
of the device (contains UUID, size, metadata area). The rest is free for
data extents.

### pvdisplay fields that matter
```
PV Name               /dev/sdb
VG Name               vg_data           (empty = not in a VG yet)
PV Size               159.00 MiB
PE Size               4.00 MiB          (default, can change at VG creation)
Total PE              39
Free PE               39                (this is what you can allocate)
Allocated PE          0
```

---

## 3. VG — Volume Group

```
sudo vgcreate vg_data /dev/sdb /dev/sdc     # pool them together
sudo vgs                                     # short view
sudo vgdisplay vg_data                       # detailed view
sudo vgextend vg_data /dev/sdd               # add another PV later
sudo vgreduce vg_data /dev/sdc               # remove a PV (must be empty!)
```

### vgdisplay fields that matter
```
VG Name               vg_data
VG Access             read/write
VG Status             resizable
VG Size               515.00 MiB            (sum of all PVs)
PE Size               4.00 MiB
Total PE              128
Alloc PE / Size       0 / 0
Free  PE / Size       128 / 515.00 MiB      <-- THIS IS YOUR BUDGET
```

---

## 4. LV — Logical Volume

```
# Create (size in G, M, or extents)
sudo lvcreate -L 100M -n lv_apps vg_data
sudo lvcreate -l 100%FREE -n lv_logs vg_data   # use all remaining space

# View
sudo lvs
sudo lvdisplay vg_data/lv_apps

# Resize (GROW — online, no unmount needed for ext4/xfs)
sudo lvextend -L +50M vg_data/lv_apps
sudo resize2fs /dev/vg_data/lv_apps            # ext4
sudo xfs_growfs /mnt/apps                      # xfs (mounted path!)

# Resize (SHRINK — MUST unmount first, fsck, then shrink FS, then LV)
# DANGER: data loss if done wrong. Backup first.
sudo umount /mnt/apps
sudo e2fsck -f /dev/vg_data/lv_apps
sudo resize2fs /dev/vg_data/lv_apps 50M
sudo lvreduce -L 50M vg_data/lv_apps
sudo mount /dev/vg_data/lv_apps /mnt/apps

# Remove
sudo lvremove vg_data/lv_apps
```

### lvdisplay fields that matter
```
LV Path                /dev/vg_data/lv_apps
LV Name                lv_apps
VG Name                vg_data
LV Size                100.00 MiB
Current LE             25
Segments               1
Allocation             inherit
```

---

## 5. Snapshots — the backup/rollback superpower

A snapshot is a **point-in-time copy** using copy-on-write (COW).
It starts tiny (just metadata) and grows only when original blocks change.

```
# Create snapshot (size = how much CHANGE you expect)
sudo lvcreate -L 50M -s -n lv_apps_snap vg_data/lv_apps

# Mount snapshot read-only for backup
sudo mkdir /mnt/snap
sudo mount -o ro /dev/vg_data/lv_apps_snap /mnt/snap
tar -czf /backup/apps-$(date +%F).tar.gz -C /mnt/snap .

# Rollback to snapshot (original LV becomes snapshot content)
sudo lvconvert --merge /dev/vg_data/lv_apps_snap
# Note: merge happens on next activation (reboot or lvchange -an/-ay)
```

### Snapshot size math
- If you expect 10% of the LV to change during backup window, make
  snapshot 10% of LV size + margin.
- If snapshot fills 100% -> it becomes invalid (stops tracking changes).

---

## 6. Filesystem on LV + fstab

```
# Format
sudo mkfs.ext4 /dev/vg_data/lv_apps
sudo mkfs.xfs  /dev/vg_data/lv_logs

# Mount
sudo mkdir -p /mnt/apps /mnt/logs
sudo mount /dev/vg_data/lv_apps /mnt/apps

# fstab (use UUID from blkid, or /dev/mapper/vg_data-lv_apps)
blkid /dev/vg_data/lv_apps
# UUID=abc-123  /mnt/apps  ext4  defaults,noatime  0  2
```

---

## 7. Common operations cheat sheet

| Task | Command |
|------|---------|
| Scan for LVM devices | sudo pvscan / vgscan / lvscan |
| Show free space in VG | vgs -o +vg_free_count,vg_free |
| Move data off a disk (replace disk) | pvmove /dev/sdb (can run online!) |
| Rename VG | vgrename old_name new_name |
| Rename LV | lvrename vg_data old_name new_name |
| Activate/deactivate VG | vgchange -ay / vgchange -an |

---

## Common mistakes to avoid
1. **Shrinking without unmount + fsck first** -> guaranteed corruption.
2. **Forgetting resize2fs/xfs_growfs after lvextend** -> LV grows but FS doesn't see it.
3. **Running out of snapshot space** -> snapshot invalidates, backup inconsistent.
4. **Using /dev/sdX in fstab instead of UUID or /dev/mapper/ path** -> boot fails if disk order changes.
5. **Not checking Free PE in vgs before creating LV** -> "Insufficient free space" error.

---

## What I learned (fill after the lab)
- (write 2-3 sentences: what surprised you, what you broke, how you fixed it)