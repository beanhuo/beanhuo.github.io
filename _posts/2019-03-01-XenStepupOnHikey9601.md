---
layout: post
title: "Xen stetup on hikey960"
date: 2019-03-01   
tag: Xen
---
### Setting Up Xen on HiKey960

This tutorial explains how to set up Xen on the HiKey960. It is based on the official [Xen wiki tutorial](https://wiki.xenproject.org/wiki/HiKey960). Since there are several points that can be confusing, I have summarized all the steps I followed.

---

## **HiKey960 Versions**
Before proceeding, be aware that there are two versions of the HiKey960:
- The earliest version has **3GB DRAM**.
- The latest version has **4GB DRAM**.

In my experience, enabling Xen is much easier on the later version.

---

## **Requirements**
The Xen boot sequence follows this order:

**Bootloader → Xen → Linux → Xen Domain0 → Xen Control Bench → DomainU**

Since Xen Domain0 is bundled with Debian, I chose **Debian** as my Linux distribution. For the bootloader, I followed the Xen Wiki and selected **UEFI**.

Below are the necessary image files I downloaded:

### **1. UEFI**
[Download UEFI Image](http://snapshots.linaro.org/96boards/reference-platform/components/uefi-staging/57/hikey960/release)

According to the Wiki, you should download the debug version. However, I am unsure of the exact differences between the debug and release versions. When I tried flashing `prm_ptable.img` with `fastboot`, it failed for the debug version, so I used the release version instead.

### **2. Grub Source Code**
[GRUB Source Code](https://git.savannah.gnu.org/git/grub.git)

Since the provided UEFI does not support Xen boot commands, I needed to update the GRUB image.

### **3. Linux Kernel Source Code**
[Linux Kernel Repository](https://github.com/96boards-hikey/linux.git) (Branch: `hikey960-upstream-rebase`)

### **4. Tools Image**
[Tools Image Repository](https://github.com/96boards-hikey/tools-images-hikey960.git) (Branch: `master`)

If your board is already flashed with these tools, you do not need to reflash these images.

### **5. Wi-Fi Firmware (wl18xx-fw-4.bin)**
Clone from the Linux firmware repository:
```bash
git clone git://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git
```
This is the Wi-Fi network card driver.

### **6. Xen Source Code**
[Xen Source Code](git://xenbits.xen.org/xen.git)

I tested both **Xen 4.12** and **Xen 4.8**, and both work on the HiKey960.

### **7. Debian Images**
[Debian Images](http://snapshots.linaro.org/96boards/hikey/linaro/debian/19/)

This includes two files:
- `rootfs`
- `boot.img` (which contains `grub.img`)

---

## **Flashing Debian on HiKey960**
After downloading all the required images and source code, follow the official [Debian installation guide](https://wiki.debian.org/InstallingDebianOn/96Boards/HiKey960) to flash Debian onto the HiKey960.

Initially, flash the required image files to UFS. In the next steps, we will only port Xen and update GRUB. I pre-flashed the boot and rootfs images to ensure that all my downloaded files were valid.

According to the Wiki, you do not need to flash `boot.img` (`boot-0.0...`) or `system.img` (`rpb-console-<something>rootfs.img.gz`), as we will replace the kernel image and `grub.img` later. However, pre-flashing does not cause any issues, so the choice is yours.
