---
layout: post
title: "Understanding U-Boot Environment Variables and Boot Arguments"
date: 2019-03-01   
tag: uboot 
---

### Understanding U-Boot Environment Variables and Boot Arguments

U-Boot is a popular bootloader used in embedded systems. Understanding its environment variables and boot arguments is crucial for configuring and booting Linux systems effectively. Here’s a detailed explanation of these parameters:

#### Key Environment Variables in U-Boot

- **bootdelay**: Time to wait before auto-booting, in seconds.
- **baudrate**: Baud rate for the serial console.
- **netmask**: Network mask for the Ethernet interface.
- **ethaddr**: MAC address of the Ethernet interface.
- **bootfile**: Default file to download.
- **bootargs**: Kernel boot parameters.
- **bootcmd**: Command executed automatically at boot.
- **serverip**: IP address of the TFTP server.
- **ipaddr**: IP address of the target device.
- **stdin**: Standard input device.
- **stdout**: Standard output device.
- **stderr**: Standard error device.

These variables are crucial for setting up the boot environment and are typically saved in non-volatile memory after using the `saveenv` command.

#### Important U-Boot Variables

- **bootcmd**: Specifies the commands to run automatically at boot. This is typically set to boot the kernel and pass necessary arguments to it.
- **bootargs**: Contains the command-line arguments passed to the kernel. This is essential for setting up the root filesystem, console, and other parameters.

### Common `bootargs` Parameters

1. **root**: Specifies the location of the root filesystem.
   - `root=/dev/ram rw`
   - `root=/dev/mtdblockx rw`
   - `root=/dev/nfs`

2. **rootfstype**: Specifies the type of the root filesystem.
   - Example: `rootfstype=jffs2`

3. **console**: Specifies the console device.
   - `console=ttyS0,115200`

4. **mem**: Specifies the size of the memory.
   - `mem=64M`

5. **ramdisk_size**: Specifies the size of the ramdisk.
   - `ramdisk_size=xxxxx`

6. **initrd**: Specifies the initrd parameters.
   - `initrd=0x32000000,0xa00000`

7. **init**: Specifies the initial script or program to run.
   - `init=/linuxrc`

8. **mtdparts**: Specifies MTD (Memory Technology Device) partitioning.
   - `mtdparts=sa1100:256k(ARMboot)ro,-(root)`

9. **ip**: Specifies the IP address settings.
   - `ip=192.168.0.5:192.168.0.3:192.168.0.3:255.255.255.0::eth0:off`

### Examples of `bootargs` Configurations

1. **Ramdisk in Memory**:
   ```sh
   setenv bootargs 'initrd=0x32000000,0xa00000 root=/dev/ram0 console=ttySAC0 mem=64M init=/linuxrc'
   ```

2. **Ramdisk in Flash**:
   ```sh
   setenv bootargs 'mem=32M console=ttyS0,115200 root=/dev/ram rw init=/linuxrc'
   # Boot command includes ramdisk address
   bootm kernel_addr ramdisk_addr
   ```

3. **JFFS2 Filesystem in Flash**:
   ```sh
   setenv bootargs 'mem=32M console=ttyS0,115200 noinitrd root=/dev/mtdblock2 rw rootfstype=jffs2 init=/linuxrc'
   ```

4. **NFS Root Filesystem**:
   ```sh
   setenv bootargs 'noinitrd mem=64M console=ttySAC0 root=/dev/nfs nfsroot=192.168.0.3:/nfs ip=192.168.0.5:192.168.0.3:192.168.0.3:255.255.255.0::eth0:off'
   ```

### Setting Kernel Command Line in Kernel Configuration

The kernel command line can also be set directly in the kernel configuration file (`.config`):

```sh
CONFIG_CMDLINE="root=/dev/nfs rw nfsroot=XX.XX.XX.XX:/nfs folder ip=board ip console=ttyS0,115200 mem=64M"
```

After the kernel boots, you can check the actual command line passed to the kernel by reading `/proc/cmdline`.

### Conclusion

Understanding and configuring U-Boot environment variables and boot arguments are fundamental for successfully booting and running a Linux system on embedded hardware. These configurations allow for flexible setup and troubleshooting of various boot scenarios, such as booting from different types of filesystems, network booting, and setting up initial scripts and memory parameters.
