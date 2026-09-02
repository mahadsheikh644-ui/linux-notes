# Cheatsheet — RAID (Redundant Array of Independent Disks)

## RAID Levels

| Level | Name | Min Disks | Description |
|-------|------|-----------|-------------|
| 0 | Striping | 2 | Performance, no redundancy |
| 1 | Mirroring | 2 | Redundancy, 50% capacity |
| 5 | Distributed parity | 3 | Balance of performance and redundancy |
| 6 | Dual parity | 4 | Tolerates 2 disk failures |
| 10 | Mirroring + Striping | 4 | Best performance + redundancy |

## Setup practice disks (loop devices)

```bash
sudo losetup -f
sudo losetup /dev/loop0 ~/linux-lab/day1/disk1.img
sudo losetup /dev/loop1 ~/linux-lab/day1/disk2.img
sudo losetup /dev/loop2 ~/linux-lab/day1/disk3.img
sudo wipefs -a /dev/loop0 /dev/loop1 /dev/loop2
lsblk /dev/loop0 /dev/loop1 /dev/loop2
```

## mdadm commands

### Create
```bash
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/loop0 /dev/loop1
sudo mdadm --create /dev/md1 --level=5 --raid-devices=3 /dev/loop0 /dev/loop1 /dev/loop2
sudo mdadm --detail /dev/md0
cat /proc/mdstat
```

### Monitor and manage
```bash
sudo mdadm --detail /dev/md0
sudo cat /proc/mdstat
sudo mdadm --manage /dev/md0 --add /dev/loop2
sudo mdadm --manage /dev/md0 --fail /dev/loop0
sudo mdadm --manage /dev/md0 --remove /dev/loop0
```

### Assemble / Stop / Cleanup
```bash
sudo mdadm --assemble /dev/md0 /dev/loop0 /dev/loop1
sudo mdadm --stop /dev/md0
sudo mdadm --zero-superblock /dev/loop0
sudo losetup -d /dev/loop0 /dev/loop1 /dev/loop2
```

## mdadm.conf
```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
cat /etc/mdadm/mdadm.conf
```

## fstab (use UUID from mdadm --detail)
```bash
UUID=...  /mnt/raid  ext4  defaults,noatime  0  2
```

## Common operations
- Check status: `cat /proc/mdstat`
- Rebuild after disk failure: `mdadm --manage /dev/md0 --add /dev/newdisk`
- Grow array: add spare disk, then grow
