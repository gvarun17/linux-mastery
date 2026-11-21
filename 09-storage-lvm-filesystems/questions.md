# 09 — Storage and LVM: Practice Questions

## Level 1 — Concepts

1. Draw the LVM stack from disk to mount point.
2. Why use UUID in `/etc/fstab` and not `/dev/sdb1`?
3. Can you shrink an xfs filesystem? An ext4 one?
4. What does the `nofail` option do? `_netdev`?
5. What do the last two fstab fields mean?
6. RAID 1 vs RAID 5 vs RAID 10: which survives what?

<details><summary>Answers</summary>

1. Disk → (partition) → PV → VG → LV → filesystem → mount point.
2. Device names can change between boots or when disks are added. UUIDs are stable.
3. xfs: no. ext4: yes, offline.
4. `nofail`: boot continues if the device is missing. `_netdev`: mount after network is up.
5. dump (backup flag, usually 0) and fsck order (0 skip, 1 root, 2 others).
6. RAID 1: 1 disk failure (per mirror). RAID 5: 1 disk. RAID 10: 1 disk per mirror pair.
</details>

## Level 2 — Commands

1. Show all disks with filesystem types and UUIDs.
2. Create a GPT partition table and one full-size partition on `/dev/sdb`.
3. Create VG `vgapp` from `/dev/sdb1` and `/dev/sdc1`. Create LV `lvweb` of 10 GB with xfs. Mount on `/web` permanently.
4. Grow `lvweb` by 5 GB including the filesystem.
5. Add a 2 GB swap file permanently.
6. Find which process blocks `umount /data`.
7. Check `/etc/fstab` for errors without reboot.
8. On AWS, the root EBS volume grew from 8 GB to 20 GB. Use the new space (xfs).

<details><summary>Answers</summary>

1. `lsblk -f`
2. `parted /dev/sdb --script mklabel gpt mkpart primary 1MiB 100%`
3. `pvcreate /dev/sdb1 /dev/sdc1; vgcreate vgapp /dev/sdb1 /dev/sdc1; lvcreate -n lvweb -L 10G vgapp; mkfs.xfs /dev/vgapp/lvweb; mkdir /web;` add `/dev/vgapp/lvweb /web xfs defaults 0 0` (or UUID) to fstab; `mount -a`
4. `lvextend -r -L +5G /dev/vgapp/lvweb`
5. `fallocate -l 2G /swapfile; chmod 600 /swapfile; mkswap /swapfile; swapon /swapfile;` add `/swapfile none swap defaults 0 0` to fstab
6. `lsof +D /data` or `fuser -vm /data`
7. `findmnt --verify` and `mount -a`
8. `growpart /dev/nvme0n1 1 && xfs_growfs /`
</details>

## Level 3 — Scenarios

1. `/var` is full on an LVM host. The VG has no free space. Steps to grow `/var` online?
2. After a reboot, the server is in emergency mode. `journalctl -xb` shows a timeout on `/data`. Fix it.
3. You must move data off an old disk in a VG to a new disk with no downtime.
4. `lvextend` worked, `lvs` shows 100G, but `df -h` shows 50G. Why?
5. Before a risky upgrade on `/opt` (LVM), you want a fast rollback point. How?

<details><summary>Answers</summary>

1. Attach a new disk → `pvcreate /dev/sdX` → `vgextend vg /dev/sdX` → `lvextend -r -L +NG /dev/vg/var`.
2. The disk is missing or UUID changed. Log in, `mount -o remount,rw /`, fix UUID (`blkid`) or add `nofail`, `systemctl daemon-reload`, `mount -a`, reboot.
3. `pvcreate new; vgextend vg new; pvmove old new; vgreduce vg old; pvremove old`.
4. The filesystem was not grown. Run `xfs_growfs /mount` or `resize2fs /dev/vg/lv`. Next time use `-r`.
5. `lvcreate -s -n optsnap -L 5G /dev/vg/opt`. Roll back with `lvconvert --merge`. Remove with `lvremove` if all is fine.
</details>

## Lab

1. Use 3 loop devices. Build a VG, two LVs, grow one, snapshot one, merge the snapshot.
2. Build RAID 1 with mdadm on loop devices. Fail one disk (`mdadm --fail`), replace it, watch resync.
3. Create a LUKS encrypted volume and mount it at boot with `/etc/crypttab`.
