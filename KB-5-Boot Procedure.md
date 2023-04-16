# Boot Procedure
When you turn on your computer, it goes through a process called booting to start up the operating system (OS)  
This process involves two main components: the Basic Input/Output System (BIOS) or Unified Extensible Firmware Interface (UEFI), and the boot loader.  

## BIOS and UEFI

### BIOS

* BIOS is the older, legacy boot method.
* when using BIOS, hardware settings were written to a chip called the Complementary Metal-Oxide-Semiconductor (CMOS) and managed through the BIOS interface.
* a boot loader, such as GRUB2, was then installed in the boot sector of the chosen device.
* the boot loader is responsible for loading the OS and indicating which partition is bootable.
* legacy boot loaders used in BIOS booting include LILO and GRUB.
* in BIOS booting, a 512-byte Master Boot Record (MBR) is used on the disk.
* however, MBR has limited space available for saving the partition table.

### UEFI

* UEFI is the newer boot method that has largely replaced BIOS.
* UEFI works with firmware on the motherboard that corresponds to parts of the OS, enabling more functionality and flexibility.
* UEFI firmware setup can be used to manage hardware settings, and boot order and other boot-related items have moved to the OS.
* the OS registers itself with the UEFI firmware, allowing it to access UEFI settings and use them for booting.
* in UEFI systems, a /boot/efi/ partition with type vfat is used to contain all settings required to boot.
* This partition is separate from the OS partition and stores the boot loader, kernel, and other necessary files.
* UEFI normally works with GUID Partition Table (GPT) partition tables, which allows disks bigger than 2TB.
* by using UEFI, you can take advantage of newer hardware features and enjoy a faster boot process.
* additionally, the flexibility of UEFI allows for more secure booting by verifying digital signatures of bootloaders, kernel, and drivers.

## accessing UEFI from linux

* `efibootmgr`       - makes managing UEFI boot targets possible  
* `efibootmgr -v`    - get more details
* `efibootmgr -n <num>` - force the system to boot into another entry the next time it starts
* active boot entries are marked with an astarisk (*)