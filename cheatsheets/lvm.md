# Cheatsheet — LVM (Logical Volume Manager)

## Commands

### Physical Volumes (PV)
\sudo pvcreate /dev/sdb          # initialize a disk for LVM
sudo pvdisplay                  # show all PVs
sudo pvs                        # short view
sudo pvremove /dev/sdb          # undo (removes LVM header)
sudo pvscan                     # scan for PVs
\
### Volume Groups (VG)
\sudo vgcreate vg_name /dev/sdb  # pool PVs together
sudo vgs                        # short view
sudo vgdisplay vg_name          # detailed view
sudo vgextend vg_name /dev/sdc  # add a PV to a VG
sudo vgreduce vg_name /dev/sdc  # remove a PV (must be empty)
sudo vgremove vg_name           # remove a VG
sudo vgscan                     # scan for VGs
\
### Logical Volumes (LV)
\sudo lvcreate -L 100M -n lv_name vg_name    # create by size
sudo lvcreate -l 100%FREE -n lv_name vg_name # use all remaining space
sudo lvs                                      # short view
sudo lvdisplay vg_name/lv_name                # detailed view
sudo lvremove vg_name/lv_name                 # remove an LV
sudo lvrename vg_name old_name new_name       # rename
sudo lvscan                                   # scan for LVs
\
### Resize
\sudo lvextend -L +50M vg_name/lv_name     # grow LV
sudo resize2fs /dev/vg_name/lv_name        # grow ext4 FS
sudo xfs_growfs /mnt/apps                  # grow xfs FS (mounted path)
sudo umount /mnt/apps                      # MUST unmount before shrinking
sudo e2fsck -f /dev/vg_name/lv_name        # check FS before shrink
sudo resize2fs /dev/vg_name/lv_name 50M   # shrink FS
sudo lvreduce -L 50M vg_name/lv_name       # shrink LV
\
### Snapshots
\sudo lvcreate -L 20M -s -n lv_snap vg_name/lv_name  # create snapshot
sudo lvconvert --merge /dev/vg_name/lv_snap           # merge (rollback)
\
### Other
\sudo mkfs.ext4 /dev/vg_name/lv_name     # format LV
sudo mount /dev/vg_name/lv_name /mnt    # mount
sudo pvmove /dev/sdb                    # move data off a disk (online)
sudo vgrename old_name new_name         # rename VG
sudo lvchange -an / -ay vg_name         # deactivate/activate VG
\
## Exit status math
- killed by signal N → exit status = 128 + N
- LV growth: lvextend + resize2fs/xfs_growfs (both needed)
- LV shrink: unmount → e2fsck → resize2fs → lvreduce (order matters)
