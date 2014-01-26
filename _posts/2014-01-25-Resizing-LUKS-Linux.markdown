---
comments: true
layout: post
title: Resizing encrypted volume(LUKS) on Linux
category: Linux
tags: debian, wheezy, crunchbang, linux, LUKS, resize
year: 2014
month: 1
day: 25
published: true
summary: Resizing encrypted volume(LUKS) on Linux
---
Recently I got into a situation where I installed an application and suddenly started getting login error:   

    failed to execute login command

Turns out it was due to disk full. While it is trivial to add space to VM using Virtual box vboxmanage, it is another thing getting that additional space recognized by Linux when partition is encrypted
<!-- more start -->
Increase VM hardisk with VBoxManage:
    
    VBoxManage modifyhd path/to/OldDisk.vdi –-resize 30000
    --resize argument is in MB

As this post nicely explains LUKS can be viewed as containers inside containers.     
<a href="http://ubuntuforums.org/showthread.php?p=4530641" target="_blank" >http://ubuntuforums.org/showthread.php?p=4530641 </a>

So to get additional space recognized we have to open, enlarge and close each  of these container layers.

Before running any of these commands remember to take backup of any important data. We will be deleting, creating and resizing partitions and while we do so directly modifying partition table. All done correctly there should not be any data loss but it always good knowing that there is backup available for when things do wrong unknowingly.
Also in this case, the additional space that I added was adjacent to existing partition on hard drive.

To start off, boot into your favorite LiveCD which has lvm2 and cryptsetup.
Use fdisk to list and delete partitions(both Extended and Linux):
    
    sudo fdisk -l

This does not show newly added space/partition. If the additional space is already formatted we will need to delete that partition also.
Also note  down starting blocks of partitions we will need these while re-creating new partitions.

Select the disk containing encrypted partition:
    
    sudo fdisk /dev/sda

Delete logical and extended partitions(in my case /dev/sda5 and /dev/sda2):
    
    Command(m for help): d
    Partition number(1-5): 5
    
    Command(m for help): d
    Partition number(1-5): 2


Create new partitions. It is important to start new partitions at same block as old partition to keep existing data intact:
     
    Command (m for help): n
    Command action
    e extended
    p primary partition (1-4): 
    e
    Partition number (1-4): 2
    First cylinder (32-1305, default 32):
    Using default value 32
    Last cylinder or +size or +sizeM or +sizeK (32-1305, default 1305): +12000M
    
    Command (m for help): n
    Command action
    l logical (5 or over)
    p primary partition (1-4):
    l
    First cylinder (32-761, default 32):
    Using default value 32
    Last cylinder or +size or +sizeM or +sizeK (32-761, default 761):
    Using default value 761

Verify partitions blocks(logical block which was encrypted should start at the same block as before re-sizing):

    Command (m for help): p

Write changes:

    Command (m for help): w

Now that we have increased partition sizes, we can make use of additional space in LUKS.
Reboot LiveCD and proceed for enlarging partitions:
    
    sudo cryptsetup luksOpen /dev/sda5 foo
    sudo vgscan --mknodes
    sudo vgchange -ay

Resize Crypt:

    sudo cryptsetup resize foo

Resize LVM Physical volume:

    sudo pvresize /dev/mapper/foo

Resize LVM Logical Volume:

    sudo pvchange -x y /dev/mapper/foo
    lvresize -L +4G /dev/chief/root

Re-lock Physical Volume:

    sudo pvchange -x n /dev/mapper/foo

Finally, make file system aware of new sizes:

    sudo fdisk -l

If fdisk does not show /dev/mapper/user-root, re-read partitions and check again:

    sudo vgscan --mknodes
    sudo vgchange -ay

Proceed with re-size:

    sudo e2fsck -f /dev/mapper/chief-root
    sudo resize2fs -p /dev/mapper/chief-root


Once done reboot the system into main hard drive and enjoy increased space.

<!-- more end -->




