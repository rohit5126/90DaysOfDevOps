# Day 13 – Linux Volume Management (LVM)

## first setp is to go in sudo mode as LVM requires suod permission to perform changes

## Create a virtual disk.
```bash
dd if=/dev/zero of=/tmp/disk1.img bs=1M count=1024   #creating a virtual disk

##Command Breakdown
dd: The Linux command-line utility used to copy and convert files or disk data.
if=/dev/zero: The input file. /dev/zero is a special file that provides a continuous stream of null characters (zeros).
of=/tmp/disk1.img: The output file. This defines the path and name of the file to be created.
bs=1M: The block size is set to 1 Megabyte.
count=1024: The command will write 1024 blocks of the defined block size. 

losetup -fP /tmp/disk1.img

losetup -a
```

## Task 1: Check Current Storage
'''bash

lsblk

df -h

use LVM module for going in depth.

pvs, lvs, vgs.
```

## Task 2: Create Physical Volume
```bash

pvcreate /dev/xvdf

pvs
```

## Task 3: Create Volume Group
```bash

vgcreate cool_grp /dev/xvdf /dev/xvdg 

vgs
```

## Task 4: Create Logical Volume
```bash

lvcreate -L 10G -n cool_lv cool_grp

lvs
```

## Task 5: Format and Mount
```bash
mkfs.ext4 /dev/cool_grp/cool_lv  (for logical volume)
mkfs -t ext4 /dev/xvdh  ( for formatting physical volume)

mkdir /mnt/logical_volume

mount /dev/cool_grp/cool_lv /mnt/logical_volume (for logical volume mounting)

mount /dev/xvdh /mnt/physical_volume ( for physical volume)

df -h
```

## Task 6: Extend the Volume
```bash

lvextend  -L +5G /dev/cool_grp/cool_lv

resize2fs /dev/cool_grp/cool_lv  ( important always run after resizing the volume)

df -h
```

## what did I learn.

**Toady I learned to manage volume in linux. I learned how to create volume in AWs and also a virtual volume in server
and how to attach it and mount it. also I learned to resize volume and to remove volume.it was very helpful.**
