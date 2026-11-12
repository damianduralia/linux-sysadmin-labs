\# Storage lab: LVM, RAID 1 disk failure, fstab troubleshooting


Rocky Linux 9 VM.



Built a structure of PVs a VG and LVs on loopback devices to emulate new disks being added on a real system. Chose loopback because the VM has no spare disks and the rocky system fs stayed safe during the lab. Grew both LVs by adding another loop device to the VG. Tar backup three ways (gz, bz2, xz) and restored one.



\## RAID 1 disk failure



Built a RAID 1 mirror array on two loop devices and put a test file on it. Emulated a disk fail on loop3. mdstat showed \[U\_] and loop3(F). Proved that the data was still available because the file was still readable.



Diagnosed the array status as degraded but wrongly called the failed device degraded too. The array is called degraded and the device is called failed.



Removed loop3 and added loop4 to the array. It synced back to \[UU] in a few minutes. Still \[UU] when I checked later.



\## fstab ticket



Used AI (Claude) to emulate the failure. It wrote the commands that broke fstab in base64 so I couldn't read them before running. The ticket said the app can't write its logs suggesting that the logs fs might be faulty.



Checked the mountpoints and the ticket complaint was right. The logs LV was unmounted. lsblk showed the LV was fine.



Tried mount -a and it printed nothing. Wrongly thought it was because systemd wasn't reloaded but it was nofail so that was the reason mount -a displayed no errors. fstab pointed to a UUID that didn't exist and nofail skips it without an error.



Found that the UUID in /etc/fstab was different from the blkid UUID. Restored the fstab backup then daemon-reload and mount -a. Logs LV mounted in \~6 min. Read the AI made script after that and it confirmed that was the fault.



First run Oct 2026. Rerun + logged in Nov 2026
Recorded the whole terminal session with script in \[storage-fstab-raid\_2026-11-12.log](../logs/storage-fstab-raid\_2026-11-12.log)

