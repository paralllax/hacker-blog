---
title: LVM Lab Setup With VirtualBox
published: true
---

* * *
This guide assumes you have a VM setup on virtualbox with a linux distribution as the host and LVM installed. This guide will show you how to add and expand virtual disks to your VM and examples of how to add them to filesystems with LVM. The purpose of this is for setting up a way to practice with partitioning and LVM in a safe manner without destroying any personal data. Please note if you are not careful, you can erase the data on the VM. If you are worried about such an event, please be certain to take a snapshot prior. 

*If you have not setup a VM yet, feel free to watch my guide* [here](https://www.youtube.com/watch?v=qfBAU86C4qM).

The guide will go as follows:

1. [(Optional) Take a Snapshot](#snapshot)
2. [Create and attach a virtual disk](#attach)
3. [Partition the disk](#partition)
4. [Create the physical volume, volume group, logical volume](#lvm-create)
5. [Create the filesystem, add it to fstab, and mount it](#fs-create)
6. [Resize the virtual Disk](#resize-vd)
7. [Partition and resize filesystem through lvm](#resize-lvm)

If you are unfamiliar with Logical Volume Management (LVM), it is another way to abstract the storage of a device. LVM comes with a lot of benefits, snapshotting, datamigration, resizing, and more. It provides a very flexible way of managing disks online. LVM is built in three layers, physical volumes, volume groups, and logical volumes. 

### [](#header-snapshot)Take a Snapshot<a name="snapshot"></a>

* * *

Let's get right into it. For the lab, in order to attach and detach disks, the VM needs to be powered off. Please be certain your VM is powered off now. First you will need to run powershell as an administrator. Hit the windows key, type in powershell, right click it, and run it as administrator as per below. Note that the path to the binary for VBoxManage will vary depending on the directory you installed Virtual Box. Using tab to autocomplete is very handy, and will work in powershell. 

You can skip this part if needed, but in case you are worried about messing up and want to revert in that event, we'll go ahead and take a snapshot of our VM. Note that if the machine is powered on, you will include the `--live` flag at the end of the command, otherwise, leave it out. With the two commands below, you can see demonstration of this. The second command lists our snapshots. I have 2 in my list, the second in the list, is the one just created. Alternatively, you can create a clone as well, if you plan on expanding disks, it may be easier to work with a clone, as snapshots can get messy from the cli perspective when trying to unattach and delete disks. For the example below, the name of our VM is `tutorial`. The syntax is as follows: `VBoxManage snapshot <VM NAME> take <NAME FOR SNAPSHOT> --description="Any details you want" --live`

```
PS C:\> & 'K:\Program Files (x86)\Virtualbox\VBoxManage' snapshot tutorial take original_predisk_01 --description="The original state of our machine before we begin the lab" --live
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Snapshot taken. UUID: 000af2d3-f63e-44f7-b682-9b69306c3aa4
PS C:\> & 'K:\Program Files (x86)\Virtualbox\VBoxManage' snapshot tutorial list
   Name: snapshot 01 (UUID: a89cffd0-79ec-462a-b75f-d288961574c5)
      Name: original_predisk_01 (UUID: 000af2d3-f63e-44f7-b682-9b69306c3aa4) *
      Description:
The original state of our machine before we begin the lab
PS C:\>
```

If at any point you mess up and need to start over, simply power off the VM, run the following, and power it back on:

```
PS C:\> & 'K:\Program Files (x86)\Virtualbox\VBoxManage' snapshot tutorial restore original_predisk_01
Restoring snapshot 'original_predisk_01' (000af2d3-f63e-44f7-b682-9b69306c3aa4)
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
PS C:\>
```

### [](#header-create)Create & Attach the Virtual Disk<a name="attach"></a>

* * *

With a restore point in hand, we can go ahead and jump into creating the vdisk. In the terminal we will create and attach another disk. This first command will create a 15G virtual disk. The second command will attach the disk to your VM. You may need to adjust the location of where to create the vmdk. You can put it anywhere you like. I would suggest creating a dedicated folder. I've pasted them below along with the output. Note that if came you back here after restoring from a snapshot, you only need to run the second command. 

```
PS C:\> & 'K:\Program Files (x86)\Virtualbox\VBoxManage' createhd --filename K:\VMs\vdisks\vdisk-tutorial-01.vmdk --size 15000 --format VMDK
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Medium created. UUID: c41f2662-a8aa-4d5a-b3af-029c9fa38905
PS C:\> & 'K:\Program Files (x86)\Virtualbox\VBoxManage' storageattach tutorial --storagectl "SATA" --port 1 --device 0 --type hdd --medium K:\VMs\vdisks\vdisk-tutorial-01.vmdk
PS C:\>
```

### [](#header-part)Partition the Virtual Disk<a name="partition"></a>

* * *

1. Power on your device now and launch a terminal. My terminal emulators of choice are terminator or urxvt. Any terminal emulator will suffice. When you login, you should be able to see the new device as /dev/sdb. If multiple disks have been added, this would show up as the next alphabetical letter. You can view this with the command `lsblk` which shows block devices. If you are using a different method of virtualization, and did not need to reboot, but the disk did not show up, you can run the following command: `for host in $(ls /sys/class/scsi_host/*/scan); do echo "- - -" > ${host}; done` - If you are using ASM disks, or mulipathing for SAN/NAS, you may need to scan for the paths individually instead of all of them, as it can cause issues.

    ```
    [terminal_blues@localhost ~]$ lsblk
    NAME                            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sda                               8:0    0   20G  0 disk 
    ├sda1                            8:1    0    1G  0 part /boot
    └sda2                            8:2    0   19G  0 part 
      ├fedora_localhost--live-root 253:0    0   17G  0 lvm  /
      └fedora_localhost--live-swap 253:1    0    2G  0 lvm  [SWAP]
    sdb                               8:16   0 14.7G  0 disk 
    ```

2. Next we are going to hop into the parted cli. parted can be scripted, but for the intents of the guide we will hop into the shell. One thing to note, parted will write changes immediately, instead of after confirmation at the end with tools such as fdisk. Be certain to take care in what you type/paste. As we are using a blank device and using snapshots/clones, there is not too much to worry, but when working with real data, take extra caution. 

    ```
    [terminal_blues@localhost ~]$ sudo parted /dev/sdb
    GNU Parted 3.2.153
    Using /dev/sdb
    Welcome to GNU Parted! Type 'help' to view a list of commands.
    (parted) 
    ```

3. Go ahead and run the print command. You'll see a fair bit of information. Primarily what we are looking at is the error at the top, and to confirm the size and disk are correct. We see we are working with /dev/sdb and that it is 15G as we created. Regarding the error, this is expected. Before continuing we have to create a label. In this case we will create a gpt instead of mbr. gpt (GUID partition table) was meant to replace mbr (master boot record). It has a few benefits such as vastly more primary partitions, and support for much much larger disks. It has CRC (Cyclic redundancy checks), and data recorvery options. There are more pros and cons to both, but I won't get too far in depth with it, in order to stay on track with the topic for this guide. If you need to use mbr, you are welcome to, note that some commands may need to change.

    ```
    (parted) print                                                            
    Error: /dev/sdb: unrecognised disk label
    Model: ATA VBOX HARDDISK (scsi)                                           
    Disk /dev/sdb: 15.7GB
    Sector size (logical/physical): 512B/512B
    Partition Table: unknown
    Disk Flags: 
    (parted)    
    ```

4. Make the label, notice the change in the partition table. 

    ```
    (parted) mklabel gpt                                                      
    (parted) print
    Model: ATA VBOX HARDDISK (scsi)
    Disk /dev/sdb: 15.7GB
    Sector size (logical/physical): 512B/512B
    Partition Table: gpt
    Disk Flags: 
    ```

5. Create our partitions. Please note that for lvm, we really don't need to create multiple partitions. I could have instead created one, then later segmented them out when creating Logical volumes in LVM. I have only created multiple in an effort to show more examples of parted. In this case we will use the entire disk. There is a lot going on in these command so I will break them down.

* We don't need to specify `primary` as this is only meaningful to mbr. Instead, we come up with a unique name (diskb0X). If you would like you can leave it as primary instead. Note that if you are adding these as oracle ASM disks, the names will need to be unique, otherwise non-unique issues will cause issues with udev on reboots. 

* We also don't need to specify the `fs-type`. You can if you would like. Adding it usually adds a byte of data that boot loaders can preview. 

* 2048s (1MiB) - This is the start sector, it will always be 2048s, but only for the first partition.

* 5GiB/10GiB - This is the space in gigabytes that we allocate to/from.

* 100% - This means use the rest of the available space on the disk for the partition

    ```
    (parted) mkpart diskb01 2048s 5GiB
    (parted) mkpart diskb02 5GiB 10GiB                                        
    (parted) mkpart diskb03 10GiB 100%                                        
    (parted) print                                                            
    Model: ATA VBOX HARDDISK (scsi)
    Disk /dev/sdb: 15.7GB
    Sector size (logical/physical): 512B/512B
    Partition Table: gpt
    Disk Flags: 
    
    Number  Start   End     Size    File system  Name     Flags
     1      1049kB  5369MB  5368MB               diskb01
     2      5369MB  10.7GB  5369MB               diskb02
     3      10.7GB  15.7GB  4990MB               diskb03
    ```
    
6. Now we set the lvm bit on and check the partition for alignment. With that, you can then press `Ctrl+d` or type quit to exit the parted cli. 

    ```
    (parted) set 1 lvm on                                                     
    (parted) set 2 lvm on                                                     
    (parted) set 3 lvm on                                                     
    (parted) align-check optimal 1                                            
    1 aligned
    (parted) align-check optimal 2
    2 aligned
    (parted) align-check optimal 3
    3 aligned
    (parted) print                                                            
    Model: ATA VBOX HARDDISK (scsi)
    Disk /dev/sdb: 15.7GB
    Sector size (logical/physical): 512B/512B
    Partition Table: gpt
    Disk Flags: 
    
    Number  Start   End     Size    File system  Name     Flags
     1      1049kB  5369MB  5368MB               diskb01  lvm
     2      5369MB  10.7GB  5369MB               diskb02  lvm
     3      10.7GB  15.7GB  4990MB               diskb03  lvm
    ```

### [](#header-pvcreate)Creating the physical volumes, volume group, and logical volumes<a name="lvm-create"></a>

* * *

1. First, gather information on your existing setup to check if you want to add the space to an existing group, or create new ones. As shown below, we have our original disk, group, and a logical volume for the `/` filesystem and `swap`. The three commands below give a summary of each group, the physical volumes, the volume groups, and logical volumes. We will build new ones with our space we have added. Later I will show you expanding vs creating new. 

    ```
    [terminal_blues@localhost ~]$ sudo pvs
      PV         VG                    Fmt  Attr PSize   PFree 
      /dev/sda2  fedora_localhost-live lvm2 a--  <19.00g     0 
    [terminal_blues@localhost ~]$ sudo vgs
      VG                    #PV #LV #SN Attr   VSize   VFree
      fedora_localhost-live   1   2   0 wz--n- <19.00g    0 
    [terminal_blues@localhost ~]$ sudo lvs
      LV   VG                    Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
      root fedora_localhost-live -wi-ao---- <17.00g                                                    
      swap fedora_localhost-live -wi-ao----   2.00g                                                    
    [terminal_blues@localhost ~]$ 
    ```

2. Here we create the first layer, the physical volumes. These are the partitions we created, and are evetnually added to a volume group. We will use a metadata-size of 250k. These are the raw material in LVM we will use to build our filesystem. You can see they now show up as physical volumes, but are missing a volume group.

    ```
    [terminal_blues@localhost ~]$ sudo pvcreate --metadata-size 250k /dev/sdb1 /dev/sdb2 /dev/sdb3
      Physical volume "/dev/sdb1" successfully created.
      Physical volume "/dev/sdb2" successfully created.
      Physical volume "/dev/sdb3" successfully created.
    [terminal_blues@localhost ~]$ sudo pvs
      PV         VG                    Fmt  Attr PSize   PFree 
      /dev/sda2  fedora_localhost-live lvm2 a--  <19.00g     0 
      /dev/sdb1                        lvm2 a--   <5.00g <5.00g
      /dev/sdb2                        lvm2 a--   <5.00g <5.00g
      /dev/sdb3                        lvm2 a--    4.64g  4.64g
    ```

2. Next we will create a new volume group, it may be easier to think of it of more as a storage pool, partitions and other disks can be stitched together and added here. As a note, you should never mix the underlying datastores in volume groups. In other words, if you have a virtual disk /dev/sdb from one hard drive, and /dev/sdc from another, they should not go into the same volume group as the filesystem will then write to different disks, and may have severe I/O issues. You can add as many disks from the physical volumes as you would like. 

    ```
    [terminal_blues@localhost ~]$ sudo vgcreate vglocal00 /dev/sdb1 /dev/sdb2 /dev/sdb3
      Volume group "vglocal00" successfully created
    [terminal_blues@localhost ~]$ sudo vgs
      VG                    #PV #LV #SN Attr   VSize   VFree  
      fedora_localhost-live   1   2   0 wz--n- <19.00g      0 
      vglocal00               3   0   0 wz--n- <14.64g <14.64g
    ```

3. Finally we will create the logical volumes. These are the final layer, and are the functional equivalent of the partitions we made earlier, however with many more features. These are what we will use to make our filesystem and interact with. 

    ```
    [terminal_blues@localhost ~]$ sudo lvcreate -L 5G -n data00 vglocal00 
      Logical volume "data00" created.
    [terminal_blues@localhost ~]$ sudo lvcreate -L 5G -n terminal00 vglocal00 
      Logical volume "terminal00" created.
    [terminal_blues@localhost ~]$ sudo lvcreate -l 100%FREE -n blues00 vglocal00 
      Logical volume "blues00" created.
    [terminal_blues@localhost ~]$ sudo lvs
      LV         VG                    Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
      root       fedora_localhost-live -wi-ao---- <17.00g                                                    
      swap       fedora_localhost-live -wi-ao----   2.00g                                                    
      blues00    vglocal00             -wi-a-----  <4.64g                                                    
      data00     vglocal00             -wi-a-----   5.00g                                                    
      terminal00 vglocal00             -wi-a-----   5.00g  
    ```

### [](#header-mkfs)Creating the filesystem<a name="fs-create"></a>

* * *

1. Create the mount points for the new filesystems/directories:

    ```
    [terminal_blues@localhost ~]$ sudo mkdir /data /terminal /blues
    ```

2. Create the filesystems. Here we specify the path to the logical volume. I have truncated the output on the last two commands, but it should reflect the same/similar to the first command. I have gone with an ext4 filesystem. If you would like something different, I would reccomend xfs.

    ```
    [terminal_blues@localhost ~]$ sudo mkfs.ext4 /dev/mapper/vglocal00-data00
    mke2fs 1.45.3 (14-Jul-2019)
    Creating filesystem with 1310720 4k blocks and 327680 inodes
    Filesystem UUID: 601fdb3c-8d9d-4e9d-a341-49552ae7ef8c
    Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736
    
    Allocating group tables: done                            
    Writing inode tables: done                            
    Creating journal (16384 blocks): done
    Writing superblocks and filesystem accounting information: done 
    
    [terminal_blues@localhost ~]$ sudo mkfs.ext4 /dev/mapper/vglocal00-terminal00
    mke2fs 1.45.3 (14-Jul-2019)
    Creating filesystem with 1310720 4k blocks and 327680 inodes
    [...]
    [terminal_blues@localhost ~]$ sudo mkfs.ext4 /dev/mapper/vglocal00-blues00
    mke2fs 1.45.3 (14-Jul-2019)
    Creating filesystem with 1215488 4k blocks and 304000 inodes
    [...]
    [terminal_blues@localhost ~]$ 
    ```

3. With your preferred text editor, open and update /etc/fstab with the new filesystems. fstab is read on boot, it tells the system where to mount filesystems. Add the following three lines, then mount. The `-v` flag is for verbose, the `-a` flag is for all. It reads fstab, and attempts to mount any unmounted filesystems. If you have never used vim and would like to, when you enter the file press `i` to insert text. When you are done press the `Esc` key, then type `:wq` and hit enter. `:` lets you type vim commands, `w` means write the file, `q` means quit. 

    ```
    [terminal_blues@localhost ~]$ sudo vim /etc/fstab
    [terminal_blues@localhost ~]$ tail -3 /etc/fstab
    /dev/mapper/vglocal00-data00        /data            ext4    defaults        1 2
    /dev/mapper/vglocal00-terminal00    /terminal        ext4    defaults        1 2
    /dev/mapper/vglocal00-blues00       /blues           ext4    defaults        1 2
    [terminal_blues@localhost ~]$ sudo mount -av
    [sudo] password for terminal_blues: 
    /                        : ignored
    /boot                    : already mounted
    none                     : ignored
    /data                    : successfully mounted
    /terminal                : successfully mounted
    /blues                   : successfully mounted
    ```

With fstab, there are a few columns, I'll break down each:

* This first column is the block device
* The second column is the mount point
* The type (ext4, xfs, swap, nfs, etc...)
* The attributes, for our purposes default is fine. If you use nfs or cifs or other, you may want to tune these.
* Whether the filesystem is dumped or not (1 = yes, 0 = no)
* fsck order (1 for root, 2 for other, 0 for none)

### [](#header-expanding)Expanding a Virtual Disk<a name="resize-vd"><a/>

* * *

For this section, you'll need to pull up powershell as per once before and poweroff your VM. With virtualbox, vmdk's cannot be resized, but vdi's can. So we clone our vmdk into a vdi, resize that, then clone it back into a vmdk. 


1. Clone the vmdk, and specify the out format as a vdi. Note that you may need to change the PATH's of each command. The location can be anywhere you desire. I created a dedicated folder. 

    ``` 
    PS C:\> & 'K:\Program Files (x86)\Virtualbox\VBoxManage' clonehd 'K:\VMs\vdisks\vdisk-tutorial-01.vmdk' K:\VMs\vdisks\vdisk-tutorial-01.vdi --format vdi
    0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
    Clone medium created in format 'vdi'. UUID: 03e9ed3d-7527-44eb-80af-6dc8f592ec6f
    ```

2. Detach the original vmdk by attaching nothing. (Seems silly, I know)

    ```
    PS C:\> & 'K:\Program Files (x86)\Virtualbox\VBoxManage' storageattach tutorial --storagectl "SATA" --port 1 --device 0 --type hdd --medium none
    ```

3. Assign the UUID of the original drive, then remove it. Note that if you are using snapshots, you may encounter some errors, as it creates child disks to track the changes, which make the OS believe it is still attached. Simply skip this step, then remove the disk and snapshots when you are done with them.

    ```
    PS C:\> $tutorial_uuid = & 'K:\Program Files (x86)\Virtualbox\VBoxManage' list hdds | select-string tutorial-01.vmdk -context 5,0 |  findstr /r UUID:.*- | %{ $_.ToString().split(':')[1]; } | %{ $_ -replace '\s',''; }
    
    PS C:\> & 'K:\Program Files (x86)\Virtualbox\VBoxManage' closemedium disk $tutorial_uuid --delete
    0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
    ```

4. Resize the disk to 25G

    ```
    PS C:\> & 'K:\Program Files (x86)\Virtualbox\VBoxManage' modifyhd K:\VMs\vdisks\vdisk-tutorial-01.vdi --resize 25000
    0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
    ```

5. Clone the vdi back into a vmdk

    ```
    PS C:\> & 'K:\Program Files (x86)\Virtualbox\VBoxManage' clonehd 'K:\VMs\vdisks\vdisk-tutorial-01.vdi' K:\VMs\vdisks\vdisk-tutorial_exp-01.vmdk --format vmdk
    0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
    Clone medium created in format 'vmdk'. UUID: 06522c49-cfcb-41b5-b8a1-777d33e8c61b
    ```

6. Attach the new expanded vmdk to the VM

    ```
    PS C:\> & 'K:\Program Files (x86)\Virtualbox\VBoxManage' storageattach tutorial --storagectl "SATA" --port 1 --device 0 --type hdd --medium K:\VMs\vdisks\vdisk-tutorial_exp-01.vmdk
    ```

7. Set the UUID of the vdi to a variable, then delete it. 

    ```
    PS C:\> $tutorial_uuid_vdi = & 'K:\Program Files (x86)\Virtualbox\VBoxManage' list hdds | select-string tutorial-01.vdi -context 5,0 |  findstr /r UUID:.*- | %{ $_.ToString().split(':')[1]; } | %{ $_ -replace '\s',''; }
    
    PS C:\> & 'K:\Program Files (x86)\Virtualbox\VBoxManage' closemedium disk $tutorial_uuid_vdi --delete
    0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
    ```

### [](#header-resize)Partition and resize the filesystem through lvm<a name="resize-lvm"></a>

* * *

1. Power on your VM once more. Verify the new space and drop into a parted shell. It will prompt you to allocate the new space, you will accept. *Note* again - in most situations you will NOT make multiple partitions and add them as separate physical volumes, it is more sensible to break them up on the logical volumes instead. The majority of the time where you would need to break up the partitions on the first layer and add them as separate PV's are when you would expand the disk, as we are doing here. The scheme we have laid out partitioning wise is purely for demonstration purposes. Ideally it would be as contiguous as possible. 

    ```
    [terminal_blues@localhost ~]$ lsblk /dev/sdb
    NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sdb                        8:16   0 24.4G  0 disk 
    ├sdb1                     8:17   0    5G  0 part 
    │ └vglocal00-data00     253:2    0    5G  0 lvm  /data
    ├sdb2                     8:18   0    5G  0 part 
    │ ├vglocal00-data00     253:2    0    5G  0 lvm  /data
    │ └vglocal00-terminal00 253:3    0    5G  0 lvm  /terminal
    └sdb3                     8:19   0  4.7G  0 part 
      ├vglocal00-terminal00 253:3    0    5G  0 lvm  /terminal
      └vglocal00-blues00    253:4    0  4.7G  0 lvm  /blues
    
    ```

2. As mentioned above, since the disk has changed, we need to fix the table to allocate the new space. When you launch the parted shell, it will prompt you to fix, in which case you will. 

    ```
    [terminal_blues@localhost ~]$ sudo parted /dev/sdb
    GNU Parted 3.2.153
    Using /dev/sdb
    Welcome to GNU Parted! Type 'help' to view a list of commands.
    (parted) print                                                            
    Warning: Not all of the space available to /dev/sdb appears to be used, you can fix the GPT to use all of the space (an extra
    20480000 blocks) or continue with the current setting? 
    Fix/Ignore? Fix
    Model: ATA VBOX HARDDISK (scsi)
    Disk /dev/sdb: 26.2GB
    Sector size (logical/physical): 512B/512B
    Partition Table: gpt
    Disk Flags: 
    
    Number  Start   End     Size    File system  Name     Flags
     1      1049kB  5369MB  5368MB               diskb01  lvm
     2      5369MB  10.7GB  5369MB               diskb02  lvm
     3      10.7GB  15.7GB  4990MB               diskb03  lvm
    ```
3. Since we are expanding, I prefer to look at the sectors and partitin from there. Change the unit, then create a partition. This step is the most crucial - the starting point MUST be exactly 1 sector after where the last sector ends. In our case the last sector is on the 3rd partition at `30747951` - therefore, our start sector must be at `30747952`.

    ```
    (parted) unit s
    (parted) print                                                            
    Model: ATA VBOX HARDDISK (scsi)
    Disk /dev/sdb: 51200000s
    Sector size (logical/physical): 512B/512B
    Partition Table: gpt
    Disk Flags: 

    Number  Start      End        Size       File system  Name     Flags
     1      2048s      10485759s  10483712s               diskb01  lvm
     2      10485760s  20971519s  10485760s               diskb02  lvm
     3      20971520s  30717951s  9746432s                diskb03  lvm

    (parted) mkpart diskb04 30717952 100%
    (parted) print                                                            
    Model: ATA VBOX HARDDISK (scsi)
    Disk /dev/sdb: 51200000s
    Sector size (logical/physical): 512B/512B
    Partition Table: gpt
    Disk Flags: 

    Number  Start      End        Size       File system  Name     Flags
     1      2048s      10485759s  10483712s               diskb01  lvm
     2      10485760s  20971519s  10485760s               diskb02  lvm
     3      20971520s  30717951s  9746432s                diskb03  lvm
     4      30717952s  51197951s  20480000s               diskb04
    ```

4. Set the lvm tag on and align the partition, then close out of parted. 

    ```
    (parted) set 4 lvm on
    (parted) align-check optimal 4
    4 aligned
    (parted) print                                                            
    Model: ATA VBOX HARDDISK (scsi)
    Disk /dev/sdb: 51200000s
    Sector size (logical/physical): 512B/512B
    Partition Table: gpt
    Disk Flags: 

    Number  Start      End        Size       File system  Name     Flags
     1      2048s      10485759s  10483712s               diskb01  lvm
     2      10485760s  20971519s  10485760s               diskb02  lvm
     3      20971520s  30717951s  9746432s                diskb03  lvm
     4      30717952s  51197951s  20480000s               diskb04  lvm
    ```

5. Create the physical volume. 

    ```
    [terminal_blues@localhost ~]$ sudo pvcreate --metadata-size 250k /dev/sdb4
      Physical volume "/dev/sdb4" successfully created.
    [terminal_blues@localhost ~]$ sudo pvs
      PV         VG                    Fmt  Attr PSize   PFree
      /dev/sda2  fedora_localhost-live lvm2 a--  <19.00g    0 
      /dev/sdb1  vglocal00             lvm2 a--   <5.00g    0 
      /dev/sdb2  vglocal00             lvm2 a--   <5.00g    0 
      /dev/sdb3  vglocal00             lvm2 a--    4.64g    0 
      /dev/sdb4  vglocal00             lvm2 a--    9.76g 9.76g
    ```

6. Add the physical volume to our volume group. 

    ```
    [terminal_blues@localhost ~]$ sudo vgextend vglocal00 /dev/sdb4
      Volume group "vglocal00" successfully extended
    [terminal_blues@localhost ~]$ sudo vgs
      VG                    #PV #LV #SN Attr   VSize   VFree
      fedora_localhost-live   1   2   0 wz--n- <19.00g    0 
      vglocal00               4   3   0 wz--n- <24.40g 9.76g
    ```

7. Extend the logical volume.

    ```
    [terminal_blues@localhost ~]$ sudo lvextend -l +100%FREE /dev/mapper/vglocal00-data00
      Size of logical volume vglocal00/data00 changed from 5.00 GiB (1280 extents) to 14.76 GiB (3779 extents).
      Logical volume vglocal00/data00 successfully resized.
    [terminal_blues@localhost ~]$ sudo lvs
      LV         VG                    Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
      root       fedora_localhost-live -wi-ao---- <17.00g                                                    
      swap       fedora_localhost-live -wi-ao----   2.00g                                                    
      blues00    vglocal00             -wi-ao----  <4.64g                                                    
      data00     vglocal00             -wi-ao----  14.76g                                                    
      terminal00 vglocal00             -wi-ao----   5.00g   
    ```

8. Now we resize the filesystem online in the OS. Otherwise, the system will not recognize the new space. Note that with larger filesystems that have a high number of inodes, the operation could take a while. For instance, a 15Tb filesysem, with 80% inodes could show an increase in I/O and take anywhere from 5 minutes, to 30 minutes depending on your speeds. In other words, if it's large, and this is a production machine, there is lots of I/O activity already, and the operation can wait, do it off hours. In most cases, it won't matter, but it never hurts to be safe than sorry. If you are using something other than ext4, such as xfs, you may need to use another command, such as `xfs_growfs`. 

    ```
    [terminal_blues@localhost ~]$ sudo resize2fs /dev/mapper/vglocal00-data00
    resize2fs 1.45.3 (14-Jul-2019)
    Filesystem at /dev/mapper/vglocal00-data00 is mounted on /data; on-line resizing required
    old_desc_blocks = 1, new_desc_blocks = 2
    The filesystem on /dev/mapper/vglocal00-data00 is now 3869696 (4k) blocks long.

    [terminal_blues@localhost ~]$ df -hP /data
    Filesystem                    Size  Used Avail Use% Mounted on
    /dev/mapper/vglocal00-data00   15G   25M   14G   1% /data
    ```

As you can see, our filesystem is now 15G from 5G. Odds are, in a dedicated enterprise environemnt, you will use ESXi or some other hypervisor, which typically has the capability to hot add drives and expansions. This benefit is enhanced with LVM as the expansion and resizing can be done while the server is online. 

If you're read this far, this concludes the tutorial. If you have any questions feel free to hit me up on one of my socials linked in the about page. Thanks
