# qnap-recovery
This is an attempt to recover data from a RAID-1 drive pair that was attaced to a QNAP TS-451+ NAS that bricked thanks to an LPC clock failure.  This is a known failure on Celeron-based QNAP NAS systems.

The two initial contributors are 1000 miles away from each other, so need a way to quickly give each other information and suggestions.  At the moment, we have had no success in reading any data from the drives.  The now-bricked QNAP was host to a 10Tb and a 6Tb RAID-1 pair.

The drives will not mount automatically under Windows 10 or Ubuntu 22.04, but Ubuntu detects that the drives are RAID members.  We currently have two potential strategies.

### Strategy 1:  Force mount as "typical" drive.  
Depending on how the RAID-1 drive was formatted, it may be mountable a a standalone drive if 'forced.'  A forced mount should not make any changes to the drive simply by mounting and reading from it.

In order to perform this, the user needs to know the partition which is the RAID member as well as the filesystem type it was formatted as.
In terminal:
1. Identify the partition that is the RAID member (/dev/sd...).
```
lsblk /dev/sd*
```
2. Identify the TYPE value (most likely ext4 or NTFS) in the returned information (double check it's displaying info on theright partition ID).  If it displays something other than ext4 or NTFS, you may need to assume the format.  Since QNAP is a Linux-based system, ext4 is very likely the right format.
```
blkid <partition id>
```
3. Create a folder the drive can be mounted to so that if mount succeeds, you can read the data from the drive (such as /mnt/raid)
```
sudo mkdir /mnt/raid
```
4. Try to force-mount the drive (change /dev/sdb2 to the right partition label, and /mnt/raid to the directory path you created
```
sudo mount -f ext4 /dev/sdb2 /mnt/raid
-or-
sudo mount ext4 /dev/sdb2 /mnt/raid -o force
```
5. If there is an error, write it down.  If not, see if the mount was successful and the data can be read
```
ls /mnt/raid
```
6. If you see files and directories from the last command, you should be able to copy the files to the replacement drive via the GUI.
7. If the drive mounted properly, ensure you unmount the drive before you disconnect.  If a button isn't provided in Nautilus, in terminal, type
```
sudo umount /mnt/raid
```

### Strategy 2:  Access array with mdadm
1.  Check if mdadm is installed on your system - install if it is not
```
sudo apt-install -y mdadm
```
2.  If you haven't created a mount point for the drive, do it now
 ```
 sudo mkdir /mnt/raid
 ```
3. See if mdadm will mount the drive
```
sudo mdadm --assemble --scan
```
4.  If you see something like below, you are in luck!  Pay attention to the md number displayed
```
mdadm: /dev/md/1 has been started with 1 drive (out of 2)
```
5. Assuming that md1 was displayed, enter the following.  If 1 is not the right number, change it as you type it into terminal
```
sudo mount /dev/md1 /mnt/raid
```
6. Check if the mount was successful
```
ls -la /mnt/raid
```
7.  If the mount was successful, you should be able to copy the files to the replacement drive via the GUI.
8. If the drive mounted properly, ensure you unmount the drive before you disconnect.  If a button isn't provided in Nautilus, in terminal, type
```
sudo umount /mnt/raid
```
9. If the drive mounted properly, ensure you unmount the drive before you disconnect.  If a button isn't provided in Nautilus, in terminal, type
```
sudo umount /mnt/raid
```
