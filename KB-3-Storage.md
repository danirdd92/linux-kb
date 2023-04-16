# Storage Managment

## linux device handling

* the linux kernel is responsible for device managment
* it uses modules to work with specific device types
* kernel modules are included in the `initramfs`, or can be loaded on demand
* `initramfs` (`initrd` in some distos) is compiled while installing linux 
* automatic on-demend device initialization (PnP) is done by systemd-udevd
* manual device loading can be done using `modprobe`
* to interface with devices, most (but not all) devices have a device node in `/dev`
* hardware related params are stored in `/sys` and its subdirs

## listing devices utils

`lsdev` - information about installed hardware   
`lsusb` - list usb information  
`lspci` - list pci connected device information  
`lsblk` - list block devices  
`lscpu` - display CPU information   
`dmesg` - print or control the kernel-ring buffer

## systemd udevd

* enables plug and play behavior
* when alerted by the kernel systemd-udevd will initialize new devices
* this involves writing device info to `/sys` and creating device nodes in `/dev`
* `udevadm monitor` to trace the procedure of device initialization
* `udevadm info --name=/dev/<node>` to get detailed information about devices

## pseudo filesystems

* the kernel uses pseudo filesystems - meaning they exist in memory only and not on disk
    - `/proc` is for kernel runtime information and tunables
    - `/sys` is for hardware related information
    - `/dev` is used to create device nodes

## managing partitions

* Master Boot Record (MBR) is the older partition kind
    - works wiith BIOS to store information in a 512kb on-disk area, as well as the boostrap loader
    - 4 primary partitions are supported
    - to address more partition, extended and logical partitions are needed
    - a maximum of 2TB can be addressed
    - `fdisk` is the recommended utility to create MBR partitions

* Guid Partition table (GPT) is the newer partition kind used to address the shortcomings of MBR
    - larger are to store up to 128 partitions
    - no need for extended partitions
    - Required on systems that work with UEFI
    - `parted` and `gdisk` are the tools to work with GPT partitionsא

## swap partitions

* swap is used to cache inactive application memory
* more memory available for caching files and other information on behalf of RAM
* if only inactive application memory is swapped, performance won't be effected
* linux servers should have a minimum of swap to alllow for swapping out all inactive application memory
* as it works on contiguous disk space, using a swap device is recomended

* `mkswap`           - set up a swap area  
* `swapon`/`swapoff` - enable/disable devices and files for paging and swapping  
* `free`             - Display amount of free and used memory in the system  


## file systems 

* there are many file systems available on linux
    - Ext3      - journaling file system used by Linux that provides better reliability and data recovery than its predecessor, Ext2
    - Ext4      - successor to Ext3 that adds support for larger file sizes and improved performance, while maintaining compatibility with Ext3
    - XFS       - high-performance journaling file system originally developed by Silicon Graphics for their IRIX operating system, now commonly used on Linux servers and workstations
    - NFS       - network file system protocol used for sharing files over a network between UNIX-based systems
    - SMB/CIFS  - network file system protocol used for sharing files over a network between Windows-based systems
    - FAT       - network file system protocol used for sharing files over a network between Windows-based systems
    - NTFS      - proprietary file system used by modern versions of Windows that supports large file sizes and advanced security features
    - Btrfs     - modern copy-on-write file system designed for Linux that provides features such as snapshots, checksums, and RAID-like functionality


* to use a partition you need to put a filesystem on top
* different file systems exist, but ext4 and xfs are the defaults
* use `mkfs.<fs-kind>` to create a file system
* you can use `-L` to give the file system a label
* use `mount` to connect the file system to a directory as its mount point

### automating mounts 

* `/etc/fstab` is used to automate mounts on between boots
* it is used as an input file for `fstab-generator`, a systemd component that generates systemd mounts
* in `/etc/fstab`, 6 columns are used to automate mounts, last 2 are deprecated
    - device
    - mount point
    - file system type
    - mount options
    - dump option (DEPR)
    - fsck option (DEPR)

### potential issues

* block device names may changed down the road (i.e. extended MBR partition numbers )
* if you're refering to a block device name in `/etc/fstab`, that can create problems
* avoid these problems using UUIDS and labels
* when creating a file system, UUIDS are automatically assigned
* use `blkid` to get the uuid for your filesystems, labels are assigend manually
* to add label after filesystem creation use a specific utility per fs
    - `xfs_admin -L` for XFS
    - `tune2fs -L` for Ext4
* it is useful to check `/etc/fstab` if any device issues are occuring to validate presistent mount between boots


## LVM

* Provides a layer of abstraction between physical storage devices and file systems
* Allows creation of virtual storage volumes that span multiple physical disks
* Enables resizing of volumes on-the-fly without data loss
* Facilitates moving data between physical devices without downtime or data loss
* Can merge logical volumes together for greater flexibility in managing storage resources
* Provides high availability and scalability in enterprise environments

### To use LVM in Linux, follow these basic steps:
* 
    1. Create one or more physical volumes (PVs) by partitioning your storage devices or disks using a tool like fdisk or gdisk. 
        - you can set partition type to be `8e` or `8e00` for which is a low level identifier, it's considered best practice
    2. Add your physical volumes to a volume group (VG) using the `pvcreate` and `vgcreate` commands.
        - `pvcreate /dev/nnn` to mark partition as physical volume
        - `vgcreate vgname /dev/nnn` to create a VG that contains at least 1 partition
    3. Create one or more logical volumes (LVs) within the volume group using the `lvcreate` command.
        - `lvcreate -L <size>G -n lvname vgname`

    4. use pvs, vgs, lvs or pvdisplay, vgdisplay, lvdisplay to varify
    5. Format the logical volumes with the desired file system using a tool like mkfs.
    6. Mount the logical volumes to a mount point of your choice using the `mount` command.  
   



* To resize an existing logical volume, use the `lvresize` command.

* To move data between physical devices, use the `pvmove` command.

* To merge logical volumes, use the `lvmerge` command.

* To remove a logical volume or a volume group, use the `lvremove` or `vgremove` commands, respectively.

* To remove a physical volume, use the `pvremove` command.

## RAID configurations

* Provides data redundancy and/or increased read/write performance by combining multiple physical disks into a single logical unit
* Allows hot-swapping of failed disks without downtime or data loss
* Supports various RAID levels, each with its own trade-offs in terms of performance, redundancy, and capacity utilization
* Can be implemented at the software or hardware level, hardware RAID is more advised
* different RAID types are offered
    - RAID 0: striping
    - RAID 1: mirroring
    - RAID 5: striping with distributed parity
    - RAID 6: striping with dual distributed parity
    - RAID 10: mirrored and striped

- To create a software RAID configuration in Linux, follow these basic steps:
    * Partition the physical disks you want to use for RAID using a tool like fdisk or gdisk.
        - `fdisk /dev/sdb; fdisk /dev/sdc;` use type RAID
    * Create a new RAID array using the 'mdadm' command, specifying the RAID level, the disks to include, and any other relevant options.
        -  `mdadm --create /dev/md0 --level=<raid type> --daid-disks=2 /dev/sdv /dev/sdc` (2 disks in this case)
    * Create a file system on the new RAID array using a tool like mkfs.
        - `mkfs.ext3 /dev/md0`
    * Mount the new RAID array to a mount point of your choice using the 'mount' command.
        - `mkdir /raid`
        - `mount /dev/md0 /raid`
    * To replace a failed disk, hot-swap the disk with a new one of the same size and type, then use the 'mdadm' command to add the new disk to the RAID array and rebuild the data.

## monitoring filesystems and disk usage

### disk usage
`df` - shows the amount of space available on a mounted device  
`du` - shows disk usage per directory

### files ystems
`tune2fs`   - show and optimiz Ext file systems
`dumpe2fs`  - dump Ext file system metadata
`xfs_admin` - monitor XFS properties



