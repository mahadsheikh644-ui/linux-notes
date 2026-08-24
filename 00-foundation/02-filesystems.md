# Lesson 02 — Filesystems: Partitions, ext4/xfs, mount, fstab, swap

Date: 2026-08-20 | Live demo on WSL2 Ubuntu 22.04

## Why this matters
"Disk is full" is the #1 sysadmin panic call. Fixing it — or better, setting up
storage so it never happens — is core admin work: partitioning, choosing a
filesystem, mounting, and making it survive reboots (fstab). Interviewers probe
ext4 vs xfs and "why do you use UUIDs" constantly.

## 1. Block devices and partitioning

```
lsblk                       # tree view: size, type, mountpoint
lsblk -f                    # + filesystem type, UUID, label
blkid /dev/sda1             # raw UUID/FSTYPE info
sudo fdisk -l               # classic partition table view
```

Device naming:
- `/dev/sda`, `/dev/sdb` — first/second SATA/SCSI/USB disk
- Partitions: `/dev/sda1`, `/dev/sda2` (number = partition)
- NVMe: `/dev/nvme0n1p1` (nvme0 controller, n1 namespace, p1 partition)
- `sr0` = optical, `loop*` = loop devices (used by Snaps and our lab)

Partition tables:
- **MBR** (old): max 2TB, max 4 primary partitions. Compatibility only.
- **GPT** (modern): 9 zettabytes, ~128 partitions, required for UEFI boot.
- `fdisk` is interactive but scriptable: `sudo fdisk /dev/sdb` then `g` (GPT),
  `n` (new), `w` (write). `partprobe` re-reads the table without rebooting.

## 2. Filesystems — choosing and creating

| FS | Good at | Weak at |
|----|---------|---------|
| ext4 | The default on most distros; rock solid, mature | Large files (2TB max per file) |
| xfs | Huge files/filesystems, scalability | Can't shrink (only grow) |

```
sudo mkfs.ext4 /dev/sdb1          # -L label  to name it
sudo mkfs.xfs  /dev/sdb2
```

Key concepts:
- **Journaling**: ext4/xfs keep a journal; after a crash, replay the journal
  instead of scanning the whole disk. That's why they're "safe" vs the old
  non-journaled ext2.
- **Inodes**: metadata blocks (name, owner, permissions, block list) — one per
  file. A disk can be full even when df shows free space (`df -i` = inode view).
- `df -h` and `df -i` tell different stories. Check both.

## 3. Mounting

```
sudo mkdir -p /mnt/data
sudo mount /dev/sdb1 /mnt/data
lsblk /dev/sdb1              # now shows /mnt/data as mountpoint
sudo umount /mnt/data        # or: sudo umount /dev/sdb1
```

- `/mnt` = sysadmin hand-mounts · `/media` = GUI/auto-mounted removable disks.
- Options: `mount -o noatime` (skip access-time updates → less disk I/O),
  `ro` (read-only), `noexec` (security: no binaries from that mount).
- **A mount hides what's underneath** — mounting over a non-empty dir makes the
  old content invisible until unmount.

## 4. fstab — mounts that survive reboot

`/etc/fstab` — one line per filesystem:
```
UUID=abc-123-xyz  /mnt/data  ext4  defaults,noatime  0  2
```

Six fields:
1. Device or **UUID** — always UUID, never `/dev/sda1` (names can change
   between boots when disks are added/removed)
2. Mountpoint
3. Filesystem type
4. Options (`defaults` = rw,suid,dev,exec,auto,noatime-ish; comma list)
5. Dump (0 = don't back up)
6. **Fsck pass** (0 = never check, 1 = root only, 2 = everything else)

```
sudo mount -a        # mount everything in fstab (test after every edit!)
sudo findmnt -verify --verbose /etc/fstab   # syntax check before reboot
```
Edit fstab wrong and the box may not boot. Add `nofail` for non-essential
drives so boot continues even if the disk is missing.

## 5. Swap — the overflow

- Swap = disk pretending to be RAM. Kernel moves idle pages there.
- If swap is hit hard (`free -h`), the box is starved → add RAM or fix the app.
- Swapfile vs swap partition — swapfile is easier to resize on modern systems.

```
sudo fallocate -l 2G /swapfile       # or dd if=/dev/zero of=/swapfile bs=1M count=2048
sudo chmod 600 /swapfile             # swapon refuses insecure perms
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```
`swappiness` (0-100) = how eagerly the kernel swaps. 10 is a common sane value:
`sysctl vm.swappiness=10` (persist in `/etc/sysctl.d/`).

## 6. Health checks

```
df -h            # space by filesystem
df -i            # inodes by filesystem
du -sh /var/log  # directory usage
sudo fsck /dev/sdb1   # check+repair — NEVER on a mounted filesystem
```

## Common mistakes to avoid
1. Putting `/dev/sda1` in fstab instead of UUID → broken boot after disk order changes.
2. `mkfs` on the wrong device — **verify with lsblk twice before formatting.**
3. Running fsck on a mounted filesystem → corruption. Unmount first (or use
   a rescue environment for root).
4. Full root disk when there's plenty on `/home` — separate mounts, or a full
   disk because inodes are exhausted (`df -i`).
5. Forgetting `nofail` on secondary drives → boot hangs if a disk is absent.

## What I learned (fill after the lab)
- (write 2-3 sentences: what surprised you, what you broke, how you fixed it)
