---
layout: post
title: "How To Libreboot A Thinkpad T440p"
description: Flashing tips, firmware construction and hardware teardown
date: 2026-05-04 00:19:13 +0100
categories: free-software libreboot low-level
image:
  path: /images/libreboot.png
---

If you need to refer to official docs for the t440p please see [here](https://libreboot.org/docs/install/t440p_external.html).

# Why Libreboot?

Libreboot is a firmware designed to replace the BIOS on a number of consumer devices, with the intention of creating a boot software that lives up to free software ideals (open-source, no binaries, free to use, modify and redistribute). One of the most famous features of Libreboot is its disabling of the [Intel Management Engine](https://en.wikipedia.org/wiki/Intel_Management_Engine), whichs runs with most Intel CPUs and some people consider to be a proprietary backdoor due to its complete memory access and unknown functionality.

By using Libreboot with a device that has hardware with only free drivers (such as the Thinkpad T440p once the wifi card is replaced), it is therefore possible to boot a consumer device and run a fully featured Linux system with very little proprietary code executed. This was my end goal with Libreboot, and was finalised by booting the [Free Software Foundation](https://www.fsf.org/)'s [Gnu Guix](https://Guix.gnu.org/), which only allows systems with no proprietary drivers to run it.

<img src="/images/Guix_Logo.png" width="30%" style="display: block; margin-left: auto; margin-right: auto;">

# Preparing The Firmware

To start are two important details:

- You MUST be on a Linux system to build the firmware (I used Debian, but they also support Fedora, Arch etc.)
- You SHOULD be on a matching instruction set to build your firmware (i.e. x86/x64 for most build targets), I originally attempted to use an ARM raspberry pi and was unable to assemble the final firmware

An important realisation is that in most of the docs libreboot will refer to flashing 'Libreboot.bin'. This is actually a stand-in name for the particular bin file you produce for your device motherboard.

To create this bin file go the [Downloads](https://libreboot.org/download.html) page on the Libreboot site, and on the HTTPS mirrors navigate to 'stable/[latest version]/' (as of now this is '26.01rev1/'). Download an unzip the 'libreboot-[version]\_src.tar.xz' file, which contains the build tools, then go into the roms folder and find the correct ROM for your device, in my case this was 'libreboot-[version]\_t440plibremrc_12mb.tar.xz', but do not unzip it!

We must first inject some original vendor files to support certain hardware. To do this you can follow the [guide](https://libreboot.org/docs//install/ivy_has_common.html), producing the final binaries you can inject.

From there simply choose an appropriate binary (likely a SeaGRUB binary with the correct layout for your keyboard). This will be what you flash to the board.

But first, for the t440p, you need to split the binary:

```sh
dd if=libreboot.rom of=spi1_8mb.rom bs=1M count=8
dd if=libreboot.rom of=spi2_4mb.rom bs=1M skip=8
```

Now you can proceed to flashing.

# Tearing Down The Board

I recommend reading the [official disassembly docs](https://libreboot.org/docs/install/t440p_external.html#disassembly) for this part, as they do a very good job describing what to disconnect. Below is a rough paraphrasing of these docs, but please look at the original too.

First remove the screws and slide off the back cover:
![](https://av.libreboot.org/board/t440p/t440p_back.jpg)

Then:

- Unplug the cmos battery
- Unplug and unroute the fan cable
- Unplug and unroute the black LED cable
- Remove all visible screws

![](https://av.libreboot.org/board/t440p/t440p_nocover_orig.jpg)

Now you can pull up around the sides of the bottom assembly to release it. Pull it upwards and lift it open to the front of the machine like a clamshell. Make sure not to break the wires connecting the assembly to the rest of the machine.

![](https://av.libreboot.org/board/t440p/t440p_open_orig.jpg)

You can then spot the two SPI chips we need to flash on the board:
![](https://av.libreboot.org/board/t440p/t440p_chipLocation_orig.jpg)

# Replacing The Wifi Card

At this point you can cleanly remove the wifi card from the device and replace it with a free alternative. I used the [Atheros AR9462 AR5BWB222](https://www.amazon.co.uk/Bluetooth-Adapter-Interface-Connection-Atheros-Default/dp/B0881S4CPN) as it doesn't require binary blobs at all to use and can reach very good speeds for an old laptop.

# Flashing The Chip

To flash I followed the recommended guide of [serprog](https://www.flashrom.org/supported_hw/supported_prog/serprog/overview.html) with a raspberry pi pico.

Initially I wired up an [SOIC-8 clip](https://amazon.co.uk/dp/B012VSGQ0Q), but found that making good contact was difficult (this is common when the chip is soldered to the board). I instead switched to using [cheap test clips](https://amazon.co.uk/dp/B0CGD66VHS) to hook the inidivudual pins and was able to cleanly flash both chips:

<img src="/images/test_clip.jpg" width="60%" style="display: block; margin-left: auto; margin-right: auto;">

This definitely isn't the approved/normal way but it was the most consistent way in my testing.

To flash the chip I wired up the clips to the pico, following this mapping:
<img src="/images/libre_mapping.png" width="100%" style="display: block; margin-left: auto; margin-right: auto;">

Which looked like this for SPI2:
<img src="/images/spi2_flash.jpg" width="70%" style="display: block; margin-left: auto; margin-right: auto;">

Then you have to [build and flash](https://libreboot.org/docs/install/spi.html#download-serprog-firmware-pre-compiled) the pico serprog firmware to the pico.

With the set up you can flash SPI1 by executing:

```bash
# Make a folder to store your firmware to be flashed
mkdir bin
mv spi1_8mb.rom bin/
mv spi2_4mb.rom bin/

# Make a folder for backups and write the SPI1 backup
mkdir backup
sudo ./flashprog -p serprog:dev=/dev/ttyACM0,spispeed=1M -c W25Q64FV -r backup/spi1_8mb_stock.rom

# Then flash SPI1
sudo ./flashprog -p serprog:dev=/dev/ttyACM0,spispeed=1M -c W25Q64FV -w bin/spi1_8mb.rom
```

And SPI2 by executing:

```bash
# Back up SPI2
sudo ./flashprog -p serprog:dev=/dev/ttyACM0,spispeed=1M -c W25Q32FV -r backup/spi2_4mb_stock.rom

# Flash SPI2
sudo ./flashprog -p serprog:dev=/dev/ttyACM0,spispeed=1M -c W25Q32FV -w bin/spi2_4mb.rom
```

Once the flash completes successfully simply reassemble and boot the device, and you should be greeted by the Libreboot BIOS, meaning your install was successful:
<img src="/images/libreboot_seagrub.png" width="70%" style="display: block; margin-left: auto; margin-right: auto;">

# Installing GNU/Guix

Install GNU/Guix is the same as any other distro, assuming your hardware is fully free. Simply flash the .iso to a usb stick, boot into it from Libreboot SeaGRUB, and follow the install instructions to install it to disc.

The .iso can be acquired [here](https://guix.gnu.org/en/download/) (for the full operating system pick 'GNU Guix System' with the correct architecture, likely x86_64).

# Conclusion

Installing Libreboot wasn't as difficult as I expected, with the only major hiccup being getting good contact when flashing the chips. I was surprised to find that booting a fully free system I was able to use every single piece of hardware on the device correctly, and perform normal computer functions including:

- Loading a dvd, and playing it via VLC
- Connecting via ethernet
- Getting high internet speeds over wifi
- Compiling Rust and C code

I chose GNU IceCat as my browser, which disables JS on most websites (if it's minified and therefore proprietary, or uses powerful features like reflection). I found that after whitelisting small js files on most sites, or using extensions to reroute to mirrors of sites, I was able to perform all my daily browsing fine.

Overall I am very happy with Libreboot and GNU/Guix, and I intend to use the laptop for my normal use cases related to personal and portable computing.
