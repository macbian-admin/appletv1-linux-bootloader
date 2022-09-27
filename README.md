# Apple TV Linux Bootloader v1.1
This bootloader allows you to run any Linux distribution on the Apple TV (1st Generation). It is forked from the version used in the [OSMC](https://osmc.tv/) media center distribution with some slight modifications:

1. BootLogo.png is updated.
2. boot_loader.c is updated with additional copyright information as well as increased verbosity.
3. Original Makefile is restored since the one used in OSMC does not work.

## Understanding This Repository
I may write a proper guide on how to install Linux on the Apple TV, but for now, I am documenting exactly what the files in this repository do, and what needs to be done in order to install Linux on the Apple TV.

This repository contains all files necessary to compile and run Linux for the Apple TV besides the required kernel binaries. The bootloader works by compiling the **32-bit** kernel file (should be named `vmlinuz`) and initial ramdisk (which must be in gzipped format and named `initrd.gz`) into a fake Darwin/mach executable named `mach_kernel` (necessary because the Apple TV will only boot from Apple official `boot.efi` files). The binaries must be taken from a Linux system (will be in the `/boot` folder) and placed in this directory with the correct names.

*Note: If your initramfs/initrd file is in `.img` or `.lz` format, it must be extracted and re-compressed into `.gz` format (NOT tar.gz).*

This repository also contains the tools for compiling the `mach_kernel` in the `darwin-cross` directory on Linux. That directory must be copied to `/opt` using a command like this:
```
sudo cp -R darwin-cross/ /opt
```

This repository contains 4 other important files: `BootLogo.png`, which will appear on the screen at boot and be changed to anything that you want; `/System/Library/Extensions/KernelMemoryAccess.kext`, which is a dummy kext necessary for tricking the Apple TV into booting a non-Darwin kernel; `boot.efi`, the Apple-official first-stage bootloader that is required to install Linux on the Apple TV; and `com.apple.Boot.plist`, which defines what the kernel parameters for Linux should be.

Once the Linux kernel and initrd are in the same directory as these boot files, and `darwin-cross` is in `/opt`, type `make` at the terminal. If everything goes well, you can copy `mach_kernel`, `boot.efi`, `BootLogo.png`, the contents of the `System` folder, and `com.apple.Boot.plist` onto a GPT-formatted USB device with FAT32 on the first partition and a Linux install on another (I'd recommend using QEMU/KVM and device passthrough to acheive this). Edit the `Kernel Flags` in `com.apple.Boot.plist` to match your needs, and then add the `atvrecv` flag to your USB disk partition using GParted or the `parted` comand line utility. 

```
sudo parted /dev/sdX (what storage device your USB drive is, use lsblk to find this)
(parted) set 1 atvrecv on
(parted) quit
```
Connect the USB flash drive to your Apple TV (a USB hub is recommended to allow for keyboard use) then turn it on/restart it. If you did everything correctly with my admittedly very vague instructions (currently), you should see the boot logo and shortly afterwards Linux should boot on your TV!

## Special thanks to:
- Most of this code comes from the original atv-bootloader team on Google Code (later exported to GitHub). Their archived repository is [here](https://web.archive.org/web/20130525025321/http://code.google.com/p/atv-bootloader/).
- "Edgar (gimli) Hucek" for the 1st boot and original code (mach_linux_boot) and James McKenzie of Mythic-Beasts for mb_boot_tv.
