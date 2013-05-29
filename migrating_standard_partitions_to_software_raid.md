Migrating from standard partitions to Software RAID
===================================================

I was asked a while back to analyse and fix a misconfigured server running Red Hat Enterprise Linux 6.2. The server had been provisioned with a standard partitioning scheme, and during the kickstart installation, Software RAID had not been configured and the second disk had not been touched. Although I could have fixed the kickstart script, I wanted to avoid having to reinstall and hence I migrated the data from the standard partitions to Software RAID partitions.

This tutorial shows how to migrate from the standard boot partition to the RAID array and how to migrate the root filesystem to the new RAID array.

When migrating from standard partitioning to Software RAID, the first step is to configure the partition table on the second drive. You can issue the fdisk command with the -l flag to list all disks and their partition table:

```bash
# fdisk -l
 
Disk /dev/sda: 5368 MB, 5368709120 bytes
255 heads, 63 sectors/track, 652 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0008f34d
 
   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *           1          26      204800   83  Linux
/dev/sda2              26          90      512000   82  Linux swap / Solaris
/dev/sda3              90         653     4525056   83  Linux
 
Disk /dev/sdb: 5368 MB, 5368709120 bytes
255 heads, 63 sectors/track, 652 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0009b157
 
   Device Boot      Start         End      Blocks   Id  System
```

As you can see from the output, there is no partition table on the secondary drive (/dev/sdb). Copy over the partition table from /dev/sda so that the partition tables on both drives are identical, using the `sfdisk` command:

```bash
# sfdisk -d /dev/sda | sfdisk --force /dev/sdb 
Checking that no-one is using this disk right now ...
OK
 
Disk /dev/sdb: 652 cylinders, 255 heads, 63 sectors/track
Old situation:
Units = cylinders of 8225280 bytes, blocks of 1024 bytes, counting from 0
 
   Device Boot Start     End   #cyls    #blocks   Id  System
/dev/sdb1          0       -       0          0    0  Empty
/dev/sdb2          0       -       0          0    0  Empty
/dev/sdb3          0       -       0          0    0  Empty
/dev/sdb4          0       -       0          0    0  Empty
New situation:
Units = sectors of 512 bytes, counting from 0
 
   Device Boot    Start       End   #sectors  Id  System
/dev/sdb1   *      2048    411647     409600  83  Linux
/dev/sdb2        411648   1435647    1024000  82  Linux swap / Solaris
/dev/sdb3       1435648  10485759    9050112  83  Linux
/dev/sdb4             0         -          0   0  Empty
Warning: partition 1 does not end at a cylinder boundary
Successfully wrote the new partition table
 
Re-reading the partition table ...
 
If you created or changed a DOS partition, /dev/foo7, say, then use dd(1)
to zero the first 512 bytes:  dd if=/dev/zero of=/dev/foo7 bs=512 count=1
(See fdisk(8).)
 
# fdisk -l
 
Disk /dev/sda: 5368 MB, 5368709120 bytes
255 heads, 63 sectors/track, 652 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0008f34d
 
   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *           1          26      204800   83  Linux
/dev/sda2              26          90      512000   82  Linux swap / Solaris
/dev/sda3              90         653     4525056   83  Linux
 
Disk /dev/sdb: 5368 MB, 5368709120 bytes
255 heads, 63 sectors/track, 652 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0009b157
 
   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1   *           1          26      204800   83  Linux
/dev/sdb2              26          90      512000   82  Linux swap / Solaris
/dev/sdb3              90         653     4525056   83  Linux
```

Once the partition table has been copied over, change the partition label to “Linux raid autodetect” as shown below. The `partprobe` command can be used to avoid having to reboot, causing the kernel to re-read the partitioning table.

```bash
# fdisk /dev/sdb
 
Command (m for help): p
 
Disk /dev/sdb: 5368 MB, 5368709120 bytes
255 heads, 63 sectors/track, 652 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0009b157
 
   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1   *           1          26      204800   83  Linux
/dev/sdb2              26          90      512000   82  Linux swap / Solaris
/dev/sdb3              90         653     4525056   83  Linux
 
Command (m for help): t
Partition number (1-4): 1
Hex code (type L to list codes): fd
 
Command (m for help): p 
 
Disk /dev/sdb: 5368 MB, 5368709120 bytes
255 heads, 63 sectors/track, 652 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0009b157
 
   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1   *           1          26      204800   fd  Linux raid autodetect
/dev/sdb2              26          90      512000   82  Linux swap / Solaris
/dev/sdb3              90         653     4525056   fd  Linux raid autodetect
 
# partprobe /dev/sdb
```

The next step that you need to do is create the first RAID array with a missing disk, /dev/md0 will be used for the /boot mount point. The command to create the Software RAID with the missing disk is shown below, since we are creating a RAID array for the /boot mount point you need to specify the metadata as 0.90.

```bash
# mdadm --create /dev/md0 --name=0 --metadata=0.90 --level=1 --raid-devices=2 missing /dev/sdb1
mdadm: array /dev/md0 started.
```

Since we are using the mdadm 3.2.3, we need to use the –name argument; otherwise, the RAID array will be started with random values like /dev/md127 etc.  Specifying the name as 1 creates /dev/md1 a name of 2 creates /dev/md2. When you have created the /boot RAID array, create the RAID array for the root filesystem:

```bash
# mdadm --create /dev/md1 --name=1 --level=1 --raid-devices=2 missing /dev/sdb3
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array? yes
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
```

When you have created both RAID arrays, you will need create the /etc/mdadm.conf configuration file and rebuild the initial RAM disk (initrd) image. In order to create the /etc/mdadm.conf configuration file, you can use the `mdadm` command, which automatically generates the configuration file:

```bash
# mdadm --detail --scan > /etc/mdadm.conf
```

When you have generated the /etc/mdadm.conf configuration file, use the `dracut` command to rebuild the initrd image. On Red Hat Enterprise Linux 5 and prior, you would have originally used the `mkinitrd` command but Red Hat have now started shipping dracut.

```bash
# dracut --mdadmconf --force /boot/initramfs-2.6.32-279.el6.x86_64.img $( uname -r )
```

Once the initial RAM disk has been regenerated (initrd), create a filesystem on both your RAID arrays. For a separate /boot partition, I usually use ext2 since I do not need journaling on the /boot mount point. To create an ext3 filesystem, run the `mkfs.ext3` command followed by the RAID device:

```bash
# mkfs.ext3 /dev/md0 
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
Stride=0 blocks, Stripe width=0 blocks
51200 inodes, 204736 blocks
10236 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=67371008
25 block groups
8192 blocks per group, 8192 fragments per group
2048 inodes per group
Superblock backups stored on blocks: 
        8193, 24577, 40961, 57345, 73729
 
Writing inode tables: done                            
Writing superblocks and filesystem accounting information: done
 
This filesystem will be automatically checked every 39 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.
```

When the filesystem has been written, mount the RAID array and copy over the contents from the /boot partition:

```bash
# mount /dev/md0 /mnt
# rsync -av /boot/ /mnt
```

When the files have been copied over using the `rsync` command, update the /etc/fstab file to specify that /dev/md0 will now be the device for the boot mount point. Once you have done this, you should reboot your system to verify that it comes back online. Issue the `df` command or `cat /proc/mounts` and see that /dev/md0 is now the device for the boot mount point. You can go ahead and add the first partition on the first disk to the RAID array:

```bash
# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/sda3             4.3G  1.6G  2.5G  40% /
tmpfs                 654M     0  654M   0% /dev/shm
/dev/md0              194M   39M  145M  22% /boot
 
# mdadm --manage --add /dev/md0 /dev/sda1
```

When you have added the first partition to the RAID array, you can verify this using the mdadm command or by issuing the cat command followed by /proc/mdstat:

```bash
# cat /proc/mdstat 
Personalities : [raid1] 
md1 : inactive sdb3[1]
      4524032 blocks super 1.2
 
md0 : active raid1 sda1[0] sdb1[1]
      204736 blocks [2/2] [UU]
 
unused devices: 
 
# mdadm --detail /dev/md0 
/dev/md0:
        Version : 0.90
  Creation Time : Mon Nov 26 13:14:37 2012
     Raid Level : raid1
     Array Size : 204736 (199.97 MiB 209.65 MB)
  Used Dev Size : 204736 (199.97 MiB 209.65 MB)
   Raid Devices : 2
  Total Devices : 2
Preferred Minor : 0
    Persistence : Superblock is persistent
 
    Update Time : Mon Nov 26 13:33:43 2012
          State : clean 
 Active Devices : 2
Working Devices : 2
 Failed Devices : 0
  Spare Devices : 0
 
           UUID : c22b32b3:c1054365:bfe78010:bc810f04 (local to host localhost.localdomain)
         Events : 0.40
 
    Number   Major   Minor   RaidDevice State
       0       8        1        0      active sync   /dev/sda1
       1       8       17        1      active sync   /dev/sdb1
```

When the device has been verified, you will need update GRUB on the RAID array, which can be done by issuing the grub-install command:

```bash
# grub-install --recheck /dev/md0
grep: /boot/grub/device.map: No such file or directory
mv: cannot stat `/boot/grub/device.map': No such file or directory
Probing devices to guess BIOS drives. This may take a long time.
Installation finished. No error reported.
This is the contents of the device map /boot/grub/device.map.
Check if this is correct or not. If any of the lines is incorrect,
fix it and re-run the script `grub-install'.
 
(fd0)   /dev/fd0
(hd0)   /dev/sda
(hd1)   /dev/sdb
 
# grub-install /dev/md0
Installation finished. No error reported.
This is the contents of the device map /boot/grub/device.map.
Check if this is correct or not. If any of the lines is incorrect,
fix it and re-run the script `grub-install'.
 
(fd0)   /dev/fd0
(hd0)   /dev/sda
(hd1)   /dev/sdb
```

You have now completed the migration from the standard boot partition to the RAID array. The next step is to migrate the root filesystem to the new RAID array. First, install a filesystem on the /dev/md1 device. In the example below, we have gone with the default filesystem on Red Hat Enterprise Linux 6, which is ext4.

```bash
# mkfs.ext4 /dev/md1 
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
282800 inodes, 1131005 blocks
56550 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=1161822208
35 block groups
32768 blocks per group, 32768 fragments per group
8080 inodes per group
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736
 
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
 
This filesystem will be automatically checked every 24 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.
```

Once you have formatted the /dev/md1 device, modify the /etc/fstab file and set the device for the root filesystem. Next, mount the RAID array and copy over the files from the root filesystem to the new RAID array, using the rsync command:

```bash
# rsync -av / --exclude=/dev --exclude=/proc --exclude=/tmp --exclude=/sys --exclude=/var/run --exclude=/mnt /mnt
# mkdir -p /mnt/{dev,proc,tmp,sys,var/run,mnt}
# chmod 555 /mnt/proc
# chmod 1777 /mnt/tmp
```

As you can see from the output above we created several manual directories. These directories are important and the boot process will fail without them. Once all the files have been copied over, you will need to reboot your system. During the boot process, interrupt the GRUB boot loader and edit the boot arguments, you will need to specify the root=/dev/md1 and make sure that rd_NO_MD is not present because this will cause the boot process to fail. If you are using SELinux, set the selinux=0 boot argument; otherwise, you will be unable to login to your system when it comes back online. When your server comes back online, you can add the standard partition to the RAID array:


```bash
# mdadm --manage --add /dev/md1 /dev/sda3
```

After adding the standard partition to the root RAID array, you will need to modify the /boot/grub/menu.lst file to make sure that root=/dev/md1 is persistent. Now you should have successfully migrated from the standard partitioning to the Software RAID.
