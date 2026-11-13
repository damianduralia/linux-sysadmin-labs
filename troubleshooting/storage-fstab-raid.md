# Storage lab: LVM, RAID 1 disk failure, fstab troubleshooting

Rocky Linux 9 VM.

Built a structure of PVs a VG and LVs on loopback devices to emulate new disks being added on a real system. Chose loopback because the VM has no spare disks and the rocky system fs stayed safe during the lab. Grew both LVs by adding another loop device to the VG. Copied /etc onto the data LV as test data and backed it up with tar in 3 formats (gz, bz2, xz).

```console
[root@rocky ~]# lsblk
NAME         MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
loop0          7:0    0  600M  0 loop
├─labvg-data 253:2    0  700M  0 lvm   /mnt/data
└─labvg-logs 253:3    0  252M  0 lvm   /mnt/logs
loop1          7:1    0  600M  0 loop
├─labvg-data 253:2    0  700M  0 lvm   /mnt/data
└─labvg-logs 253:3    0  252M  0 lvm   /mnt/logs
loop2          7:2    0  300M  0 loop
└─md0          9:0    0  299M  0 raid1
loop3          7:3    0  300M  0 loop
└─md0          9:0    0  299M  0 raid1
loop4          7:4    0  300M  0 loop
sda            8:0    0   20G  0 disk
├─sda1         8:1    0    1G  0 part  /boot
└─sda2         8:2    0   19G  0 part
  ├─rlm-root 253:0    0   17G  0 lvm   /
  └─rlm-swap 253:1    0    2G  0 lvm   [SWAP]
...
```

## RAID 1 disk failure

Built a RAID 1 mirror array on two loop devices and put a test file on it. Emulated a disk fail on loop3. mdstat showed [U_] and loop3(F). Proved that the data was still available because the file was still readable.

Removed loop3 and added loop4 to the array. It synced back to [UU] right away and was still [UU] when I checked later, so the array was healthy again.

```console
[root@rocky ~]# mdadm --manage /dev/md0 --fail /dev/loop3
[root@rocky ~]# cat /proc/mdstat
Personalities : [raid1]
md0 : active raid1 loop3[1](F) loop2[0]
      306176 blocks super 1.2 [2/1] [U_]
...
[root@rocky ~]# cat /mnt/raid/data.txt
data before the failure
...
[root@rocky ~]# mdadm --manage /dev/md0 --remove /dev/loop3
mdadm: hot removed /dev/loop3 from /dev/md0
...
[root@rocky ~]# mdadm --manage /dev/md0 --add /dev/loop4
mdadm: added /dev/loop4
[root@rocky ~]# cat /proc/mdstat
Personalities : [raid1]
md0 : active raid1 loop4[2] loop2[0]
      306176 blocks super 1.2 [2/2] [UU]
...
```

## fstab ticket

Used AI (Claude) to emulate the failure. It wrote the commands that broke fstab in base64 so I couldn't read them before running. The ticket said the app can't write its logs suggesting that the logs fs might be faulty.

Checked the mountpoints and confirmed the issue described in the ticket was present. The logs LV was unmounted. lsblk showed the LV was fine.

Tried mount -a and it printed nothing. My intuition was that it was because the break script didn't reload systemd but that wasn't the case, the script reloaded systemd. mount -a printed nothing because the logs fs was configured in /etc/fstab with nofail, so the error passed silently. fstab pointed to a UUID that didn't exist.

Found that the UUID in /etc/fstab was different from the blkid UUID.

```console
[root@rocky ~]# cat /etc/fstab
...
UUID=ed492aba-4677-4624-aab5-bc69b5976eed /mnt/logs ext4 defaults,nofail 0 2
[root@rocky ~]# blkid
...
/dev/mapper/labvg-logs: UUID="c7c4353b-a233-4a80-8142-476f5113a5f7" TYPE="ext4"
...
[root@rocky ~]# mount -a
[root@rocky ~]# # logs disk not mounted, fstab and blkid UUID dont match.
[root@rocky ~]# cp /etc/fstab.prebreak /etc/fstab
cp: overwrite '/etc/fstab'? y
...
[root@rocky ~]# systemctl daemon-reload
[root@rocky ~]# mount -a
[root@rocky ~]# df -hT
Filesystem             Type      Size  Used Avail Use% Mounted on
...
/dev/mapper/labvg-logs ext4      230M   14K  215M   1% /mnt/logs
```

Restored the backup the break script left (/etc/fstab.prebreak) then daemon-reload and mount -a. Logs LV mounted in ~6 min. Read the break script after that and that confirmed the right diagnosis and fix.

First run Oct 2026. Rerun + logged in Nov 2026

Recorded the whole terminal session with script in [storage-fstab-raid_2026-11-12.log](../logs/storage-fstab-raid_2026-11-12.log)
