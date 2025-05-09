# Disclaimer #
Automatically exported from code.google.com/p/firmware-mod-kit

Firmware Modification Kit

This kit is a collection of scripts and utilities to extract and rebuild linux based firmware images.

    Extract a firmware image into its component parts
    User makes desired modification to the firmware's file system or web UI (webif)
    Rebuild firmware
    Flash modified firmware onto device and brick it (ha) 

WARNING: You are going to brick your device by using this kit (maybe not, but better to say you will). Brick means to effectively turn into a non-functional 'brick'. Recovery is sometimes possible without hardware modifications. Sometimes it requires hardware mods (e.g. serial or JTAG headers soldered onto the PCB). Sometimes it just isn't feasible, or would cost more in total recovery cost than the unit is worth.

Do NOT use this kit if you are not prepared to have your router bricked!

EULA: By downloading or using this kit, you agree to accept liability for consequences of use or misuse of the Firmware Mod Kit. These include the bricking of your device. The authors of this kit have duly warned you. This kit is only for embedded systems software engineers.


# Firmware Mod Kit #

* [Introduction](#introduction)
* [Prerequisites](#prerequisites)
* [Using The Kit](#using-the-kit)
    * [Kit Excecutables](#kit-executables)
    * [The Firmware Working Directory](#the-firmware-working-directory)
    * [Extracting Firmware](#extracting-firmware)
    * [Re-Building Firmware](#re-building-firmware)
    * [Modifying DD-WRT Web Pages](#modifying-dd-wrt-web-pages)
    * [Reverting to a vendor firmware](#reverting-to-a-vendor-firmware)
    * [Examples](#examples)
    * [Linksys Firmware](#linksys-firmware)
    * [Tools / Utilities](#tools--utilities)
* [Links](#other-links)

## Introduction ##

The Firmware Mod Kit allows for easy deconstruction and reconstruction of firmware images for various embedded devices. While it primarily targets Linux based routers, it should be compatible with most firmware that makes use of common firmware formats and file systems such as TRX/uImage and SquashFS/CramFS.

## Prerequisites ##

In order to use the Firmware Mod Kit, you must have a subversion client, standard Linux development tools (gcc, make, etc), the python-magic module, and the zlib and lzma development packages. If you are running an linux distro that use apt-get, e.g. Ubuntu or Debian, use:

For Ubuntu 18.04 and older:
```sh
sudo apt-get install git build-essential zlib1g-dev liblzma-dev python-magic autoconf
```

For Ubuntu 20.04 and newer:
```sh
sudo apt-get install git build-essential zlib1g-dev liblzma-dev python3-magic autoconf python-is-python3
```

For RedHat:
```sh
yum groupinstall "Development Tools"
yum install git zlib1g-dev xz-devel python-magic zlib-devel util-linux
```

For Arch Linux: 

There is a AUR package for <https://aur.archlinux.org/packages/firmware-mod-kit/>


For other distros, you should install the equivalent packages using your distro's package manager.

```sh
git clone https://github.com/rampageX/firmware-mod-kit.git
cd firmware-mod-kit/src
make
```

Binwalk 2.1.1

```sh
[Back in firmware-mod-kit directory]
cd src/binwalk-2.1.1
sudo python2 setup.py install
```

The Firmware Mod Kit is **only supported on the Linux platform**. With a few small modifications, it should work on other POSIX platforms.

## Using the Kit ##

Git clone the Firmware Modification Kit repository.

### Kit Executables ###

The Firmware Mod Kit is a collection of utilities and shell scripts. The utilities can be used directly, or the shell scripts can be used to automate and combine common firmware operations (e.g. extract and rebuild). The core scripts to facilitate firmware operations are listed below.

Primary scripts:
| **Script** | **Description** |
|:-----------|:----------------|
| extract-firmware.sh | Firmware extraction script |
| build-firmware.sh | Firmware re-building script |

Secondary scripts:
| ddwrt-gui-extract.sh | Extracts Web GUI files from extracted DD-WRT firmware. |
|:---------------------|:-------------------------------------------------------|
| ddwrt-gui-rebuild.sh | Restores modified Web GUI files to extracted DD-WRT firmware. |

### The Firmware Working Directory ###

The Firmware Mod Kit uses a 'hard coded' working directory of 'fmk'. The extraction script extracts to this folder, and the rebuild script rebuilds from this folder. Allowance of alternate working directories is supported for **some** operations, but not all. We'll be expanding that in the future. For now, if you have multiple working directories, we suggest you rename the ones you're not currently operating on.

### Extracting Firmware ###

Automated firmware extraction typically works with most firmware images that employ uImage/TRX firmware headers and use SquashFS or CramFS file systems. Currently, extract-firmware.sh is the preferred method of extraction as it supports more firmware types than the older old-extract.sh script. However, old-extract.sh is still included and works with many firmware formats.

Usage for both `extract-firmware.sh` and `build-firmware.sh` is straight forward:

```sh
./extract-firmware.sh firmware.bin
```

By default, output from extract-firmware.sh will be located in the 'fmk' directory, while old-extract.sh will place extracted data into the specified working directory.

### Re-Building Firmware ###

Which build script to use is dependant on which extraction script was used. If you extracted a firmware image with extract-firmware.sh, then you must use build-firmware.sh to re-build it. Likewise, if old-extract.sh was used, then old-build.sh must be invoked when re-building an image:

```sh
./build-firmware.sh [-nopad] [-min]
```

The new firmware generated by build-firmware.sh will be located at 'fmk/new-firmware.bin', while old-build.sh will generate firmware images in several different formats and save them in the specified output directory.

The optional -nopad switch will instruct build-firmware.sh to NOT pad the firmware up to its original size.

The optional -min switch will use the maximum squashfs block size of 1MB. This will decrease the firmware image size at the cost of additional CPU and RAM resources utilized on the target device. Do not use this switch unless you must. This is a very large block size for embedded systems. The original firmware squashfs block size is preserved on rebuild, and the original block size should be the one used unless you are sure you know what you're doing. Too large a block size may appear to work fine, but runtime performance of the firmware may suffer in all or some loads.

### Modifying DD-WRT Web Pages ###

One very unique feature of the Firmware Mod Kit is its ability to extract and rebuild files from the DD-WRT Web GUI. This is automated by the ddwrt-gui-extract.sh and ddwrt-gui-restore.sh scripts.

Once you have extracted a DD-WRT firmware image using extract-firmwware.sh, you can extract the Web files by running:

```sh
./ddwrt-gui-extract.sh
```

This will create a directory named 'www' and extract the Web files there. You may modify the files any way you like, but you cannot add or delete files.

When you are finished editing, you can rebuild the Web files by running:

```sh
./ddwrt-gui-rebuild.sh
```

### Reverting to a vendor firmware ###

Sometimes you'll enthusiastically flash a third-party firmware like Gargoyle or DD-WRT only to discover it lacks features you need, doesn't perform as well as the vendor firmware, or has functional problems. In this situation, you might find yourself wanting to go back to the vendor firmware, but have no way to do so!

Here's how the Firmware Mod Kit can help you revert to a vendor firmware. The process is this:

  1. Extract vendor firmware. Then rename the 'fmk' directory.
  1. Extract third-party 'upgrade' firmware (e.g. Gargoyle-sysupgrade)
  1. Replace extracted third-party firmware's rootfs and image\_parts with those from the vendor firmware.
  1. Rebuild firmware image
  1. Flash vendor firmware image (now packaged as your third-party firmware expects).
  1. If all succeeded, you're now using the vendor firmware again.

Once you are back to the vendor firmware, then it accepts vendor firmware images again.

### Examples ###

This example demonstrates how to extract a firmware image, replace its existing telnet daemon with a custom built one, and then build a new firmware image:

```sh
./extract-firmware.sh firmware.bin
cp new-telnetd fmk/rootfs/usr/sbin/telnetd
./build-firmware.sh
```

Below is an example of the commands to run in order to extract a DD-WRT firmware image, modify the Web index page, and build a new firmware image:

```sh
./extract-firmware.sh firmware.bin
./ddwrt-gui-extract.sh
echo "HELLO WORLD" > www/index.asp
./ddwrt-gui-rebuild.sh
./build-firmware.sh
```

### Linksys Firmware ###

Linksys has custom footers with Checksum checks, hence this script was written to try and automate the process of calculating the checksum of the image and changing the footer accordingly. New image will be written to modified_checksum.img. Run this after modifying and build-firmware.sh.

```sh
./linksys_footer.sh modified_firmware.img
```

### Tools / Utilities ###

The Firmware Mod Kit consists of a collection of tools useful when working with embedded firmware images. These include those listed below, though there are **MANY MORE** that are not listed here.

| **Tool** | **Description** |
|:---------|:----------------|
| AsusTRX  | An extended version of ASUSTRX that can build both 'normal' TRX files and, optionally, those with an ASUS addver style header appended. It can also, uniquely, force segment offsets in the TRX (with -b switch) for compatibility with Marvell ASUS devices like the WL-530g. This tool replaces both 'normal' trx tool and addver. Current versions included are: 0.90 beta. |
| AddPattern | Utility to pre-pend Linksys style HDR0 header to a TRX. |
| AddVer   | ASUS utility to append a header to a TRX image that contains version information. ASUSTRX includes this capability. Current version: unversioned. |
| Binwalk  | Scans firmware images for known file types (firmware headers, compressed kernels, file systems, etc.)  |
| CramFSCK | CRAMFS file system image checker and extractor. Current versions included are:  2.4x. |
| CramFSSwap | Utility to swap the endianess of a CramFS image |
| CRCalc   | Utility to patch all uImage and TRX headers inside a given firmware image. |
| MkSquashFS | Builds a squashfs file system image. Current versions included are: 2.1-[r2](https://code.google.com/p/firmware-mod-kit/source/detail?r=2), 3.0. |
| MkCramFS | Builds a cramfs file system image. Coming in next version. Current versions included are: 2.4x. |
| MotorolaBin | Utility that prepends 8 byte headers to TRX images for Motorola devices WR850G, WA840G, WE800G. Current version: unversioned. |
| Splitter3 | Utility to scan and extract a firmware image's component parts. |
| Tpl-tool | Utility to manipulate TP-Link vendor format images. |
| UnCramFS | Alternate tool to extract a cramfs file system image. Use cramfsck instead whenever possible as it seems to be more reliable. Current versions included are: 0.7 (for cramfs v2.x). |
| UnCramFS-LZMA | Alternate tool to extract LZMA-compressed cramfs file system images, such as those used by OpenRG. |
| UnSquashFS | Extracts a zlib squashfs file system image. Current versions included are 1.0 for 3.0 images and 1.0 for 2.x images (my own blend). |
| UnSquashFS-LZMA | Extracts an lzma squashfs file system image. Current versions included are 1.0 for 3.0 images and 1.0 for 2.x images (my own blend). Note: Not all squashfs-lzma patches are compatible with one another. I'm working on adding support for all common squashfs-lzma variations. |
| UnTRX    | Splits TRX style firmwares into their component parts. Also supports pre-pended addpattern HDR0 style headers. This was developed exclusively for this kit. Current versions included are: 0.45. |
| WebDecomp | Extracts and restores Web GUI files from DD-WRT firmware images, allowing modifications to the Web pages. |
| WRTVxImgTool | Utility to generate VxWorks compatible firmware images for the WRT54G(S) v5 series. |

## Other Links ##

[Forum](https://bitsum.com/forum/index.php/board,12.html)
