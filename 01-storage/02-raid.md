# Lesson 05 — RAID (Redundant Array of Independent Disks)

Date: 2026-09-02 | User: mahad | Lab: WSL2 Ubuntu 22.04

## Why this matters
RAID provides redundancy and/or performance by combining multiple disks into a single array. Interviewers ask: What is the difference between RAID 0, 1, 5, 6, 10? How do you monitor array health? What happens when a disk fails?

## What is RAID?

TODO: explain RAID basics, redundancy, types

## RAID Levels

TODO: explain RAID 0, 1, 5, 6, 10 with diagram showing how data is distributed

| Level | Name | Min Disks | Redundancy | Use Case |
|-------|------|-----------|------------|----------|
| 0 | Striping | 2 | No | Performance |
| 1 | Mirroring | 2 | Yes (1 disk) | Safety |
| 5 | Distributed Parity | 3 | Yes (1 disk) | Balance |
| 6 | Dual Parity | 4 | Yes (2 disks) | High safety |
| 10 | Mirror+Stripe | 4 | Yes (1 disk) | Performance + Safety |

## mdadm — managing RAID arrays

TODO: fill in commands after hands-on practice

### Create an array
```bash
# my commands:
# sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/loop0 /dev/loop1
# sudo mdadm --detail /dev/md0
# real output:

```

### Monitor and manage
TODO: commands for monitor, add disk, fail disk, remove disk

### fstab with RAID
TODO: how to add RAID to fstab (use UUID from mdadm --detail)

## mdadm.conf
TODO: explain mdadm.conf and why it matters

## Common mistakes to avoid
1. Forgetting to update mdadm.conf after creating an array
2. Not waiting for resync to complete before rebooting
3. Using the wrong disk count for the RAID level
4. Not monitoring array health with cat /proc/mdstat

## What I learned (fill after the lab)
- (write 2-3 sentences: what surprised you, what you broke, how you fixed it)
