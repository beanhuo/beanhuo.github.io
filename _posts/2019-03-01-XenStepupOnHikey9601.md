---
layout: post
title: "Xen stetup on hikey960"
date: 2019-03-01   
tag: Xen
---
```
This page is a tutorial about how to setup Xen on Hikey960, it is based on the xen
wiki official web tutorial(https://wiki.xenproject.org/wiki/HiKey960).Since there are
several places easily got comfused, I decide to sum up all steps I did.
```
### Hikey960 Version

Before getting down to doing this, should be aware that there are two kinds of different
hikey960, the earliest version hikey960 has 3GB DRAM, and the latest version has updated
its DRAM to 4GB. In my experience, it is munch easier to enable xen on later one.

### Requirements

The Xen boot follows the sequence as bootloader->xen->linux->xen domain0->xen control bench->domainU.
Becuase Xen domain0 is bundled in Debain, so I chose Debain as my Linux.
as for bootloader, I followed Xen WiKi, chose UEFI.

here I downloaded these image files:
#### 1.UEFI:
http://snapshots.linaro.org/96boards/reference-platform/components/uefi-staging/57/hikey960/release

according to WiKi, should download tis debug version. I don't know on earth what's difference between these versions.
I tried debug version, but when I flashed prm_ptable.img with fastboot, it failed. so I used release version.

#### 2. Grub source code
https://git.savannah.gnu.org/git/grub.git

Since previous UEFI doesn't support XEN booting command, so I need to update its grub image.

#### 3. Linux Kernel source code
https://github.com/96boards-hikey/linux.git -b hikey960-upstream-rebase

#### 4. Tools image
https://github.com/96boards-hikey/tools-images-hikey960.git #master

if your board already flashed them, actaully, you don't need these step's images.

#### 5. wl18xx-fw-4.bin
git clone git://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git

this is WIFI network card driver.

#### 6.Xen source code
git clone git://xenbits.xen.org/xen.git

I tried 4.12, and 4.8, both work on Hikey960.

#### 7. Debian images
http://snapshots.linaro.org/96boards/hikey/linaro/debian/19/

it contains two files, one is rootfs, another is boot.img, which includes grub.img.








