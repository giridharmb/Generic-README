#### Generic

Create Partition + File System (ext4) & Mount Device (Data Disk)

List Block Devices

```bash
root@vm-test-vm:~# lsblk
```

```bash
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0  61.9M  1 loop /snap/core20/1405
loop1    7:1    0  63.4M  1 loop /snap/core20/1950
loop2    7:2    0  79.9M  1 loop /snap/lxd/22923
loop3    7:3    0 111.9M  1 loop /snap/lxd/24322
loop4    7:4    0  53.3M  1 loop /snap/snapd/19457
sda      8:0    0   9.9G  0 disk
├─sda1   8:1    0     1M  0 part
└─sda2   8:2    0   9.9G  0 part /
sdb      8:16   0    20G  0 disk
```

`fdisk` on external data disk `/dev/sdb`

```bash
root@vm-test-vm:~# fdisk /dev/sdb
```

Press `m` for help (while inside `fdisk`)

```bash
Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xd0a3d85c.

Command (m for help): m

Help:

  DOS (MBR)
   a   toggle a bootable flag
   b   edit nested BSD disklabel
   c   toggle the dos compatibility flag

  Generic
   d   delete a partition
   F   list free unpartitioned space
   l   list known partition types
   n   add a new partition
   p   print the partition table
   t   change a partition type
   v   verify the partition table
   i   print information about a partition

  Misc
   m   print this menu
   u   change display/entry units
   x   extra functionality (experts only)

  Script
   I   load disk layout from sfdisk script file
   O   dump disk layout to sfdisk script file

  Save & Exit
   w   write table to disk and exit
   q   quit without saving changes

  Create a new label
   g   create a new empty GPT partition table
   G   create a new empty SGI (IRIX) partition table
   o   create a new empty DOS partition table
   s   create a new empty Sun partition table
```

Create new partition,<br/>
that is `/dev/sdb1`, by pressing `n` (while inside `fdisk`)

```bash
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-41943039, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-41943039, default 41943039):

Created a new partition 1 of type 'Linux' and of size 20 GiB.
```

Write and save changes & then quite by pression `w`<br/>
(while inside `fdisk`)

```bash
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Create `ext4` file system on `/dev/sdb1`

```bash
root@vm-test-vm:~# mkfs -t ext4 /dev/sdb1
```

```bash
mke2fs 1.46.5 (30-Dec-2021)
Discarding device blocks: done
Creating filesystem with 5242624 4k blocks and 1310720 inodes
Filesystem UUID: 68cb55d4-fd79-4cea-84b1-b5885d7e1bff
Superblock backups stored on blocks:
    32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
    4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

Fetch Block Device IDs using `blkid`

```bash
root@vm-test-vm:~# blkid
```

```bash
/dev/sda2: UUID="f966a729-eab1-42c1-8b41-a71becfdd0cd" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="8fe9b8cf-0520-4b39-8729-2fc2223a3396"
/dev/loop1: TYPE="squashfs"
/dev/sdb1: UUID="68cb55d4-fd79-4cea-84b1-b5885d7e1bff" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="d0a3d85c-01"
/dev/loop4: TYPE="squashfs"
/dev/loop2: TYPE="squashfs"
/dev/loop0: TYPE="squashfs"
/dev/sda1: PARTUUID="12f6bec0-701d-4bbb-97cb-b1dfcf9cd029"
/dev/loop3: TYPE="squashfs"
```

Make a note of UUID for `/dev/sdb1` in the above command

In our case, UUID for `/dev/sdb1` is `68cb55d4-fd79-4cea-84b1-b5885d7e1bff`

Create a data directory where `/dev/sdb1` will be mounted

```bash
root@vm-test-vm:~# mkdir -p /datadisk && chmod 777 /datadisk
```

Add this UUID for `/dev/sdb1` to `/etc/fstab` 

- BE CAREFILE WHILE MODIFYING `/etc/fstab`
- IF THIS FILE GETS MESSED UP, VM WILL NOT BOOT UP !

Add (`append`) this entry to `/etc/fstab` (i.e. append it to the end of the file)

```bash
UUID=68cb55d4-fd79-4cea-84b1-b5885d7e1bff /datadisk ext4 defaults 0 0
```

Mount the file system

```bash
root@vm-test-vm:~# mount -av
```

```bash
/                        : ignored
/datadisk                : successfully mounted
```

Since we have added the entry to `/etc/fstab`,<br/>
even if we reboot the VM, `/dev/sdb1` will be<br/>
always mounted on the path `/datadisk`.<br/>