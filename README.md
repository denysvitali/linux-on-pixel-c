# Linux on Pixel C
This repo aims at documenting how to run GNU/Linux on a [Google Pixel C](https://en.wikipedia.org/wiki/Pixel_C) (2015) device.

![Pixel C running Arch Linux ARM](/images/intro.jpg)

## Introduction
### What is the Google Pixel C?
The Pixel C is a 10.2 inch Android tablet, made by Google, which was released on December 8, 2015.
It didn't had much success apparently, but its hardware is definitively still a flagship killer.

Unfortunately it runs Android, which doesn't seem to be really a productive / development oriented operating system, therefore we decided to port a Linux distro to it, to make it more productive and fast.

### What distro are currently supported?
At the moment, we've got [Arch Linux Arm](http://archlinuxarm.org/) to work on it. We haven't tried other distros yet, but PR are always welcome.

### What's the development status?
We are still in an early pre-alpha stage, the device boots with the latest kernel (4.13-rc4) but has still many issues that makes it hard to use as a daily driver.

### What are the issues right now?

 - Wi-Fi / Bluetooth chip doesn't work (Broadcom 4354 / bcm4354)
 - Graphics Acceleration doesn't work, everything is rendered by the CPU apparently
 - Sound doesn't work
 - There is no output until the device is booted
 - (Lightbar control isn't exposed on 4.13+, may be a config issue)
 - GNOME doesn't seem to work with Arch Linux ARM for aarch64 (but works with Alarm armv7)
 - Pixel C Keyboard doesn't work (need BT LE, which is provided by bcm4354)
 - Physical buttons don't work (detected, but the events aren't properly coded in libevent)
 - [Random green bars](/issues/green-bars.md)

You can follow the issues [here](https://github.com/denysvitali/linux-on-pixel-c/issues): if you happen to know how to solve one of these problems, please help us! Even the smallest comment may bring us one step forward.

### What works?
 - Boots!
 - Display
 - Touch (hid-over-i2c)
 - SSH
 - Network (Ethernet) via an USB dongle (Pretty much USB-Ethernet dongle)
 - Network (Wi-Fi) via an USB dongle (Mediatek 7601u)
 - Lightbar (with 4.12-rc2)

## Prerequisites

 - Google Pixel C (2015)
 - USB Type-C HUB (I'm personally using [this](https://www.amazon.com/HooToo-Adapter-Charging-MacBook-Chromebook/dp/B01K7C53K2))
 - USB Wi-Fi Adapter (only a few are supported, I personally use [this](http://www.ebay.com/itm/150Mbps-Mini-USB-WiFi-Wireless-Adapter-Dongle-Network-LAN-Card-802-11n-g-b-PC-/252404008603) one) or:
 - USB Ethernet Adapter (any model should work)

### Using an USB Wi-FI adapter
Create an AP named 'Pixel C', and use 'WiFiPixelC' as a password. Then boot the Pixel C and wait till it connects to your Wi-Fi network.

### Using an USB Ethernet adapter
Plug the ethernet cable, then plug the USB Ethernet adapter to your Pixel C. As soon as the adapter is detected, the Pixel C (if it booted correctly) will get an IP address via DHCP.
Use the IP to log-in

## Running Linux on your Pixel C

**WARNING:** You may void your warranty. I am not responsible for bricked devices, dead SD cards, or thermonuclear wars. Seriously though, if after a kernel modification you've made, the device gets too hot: you probably messed up with the thermal sensor and you may need to consider a force shutdown (by pressing Power + Vol- until the device powers itself off) before any damage may appear. Whilst I played around with my Pixel C on my own and never had any problem whatsoever, you may not be as lucky as I was.
If you are not sure about a modification you've made, keep the device monitored.
Don't put yourself in danger, we still need you.

**WARNING:** you need dm-verity and encryption OFF, or a newly formatted `/data` partion. Otherwise the system won't be able to boot into Linux.


### Prepare the filesystem

[Download here](https://ded1.denv.it/pixel-c/arch-xfce-lightdm.tar.gz) my custom-made filesystem (3GB), or set up your own using the [official ALARM rootfs](http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz).

*Be aware that the official ALARM rootfs requires an USB Ethernet adapter since the Wireless connection to a "PixelC" hotspot isn't preconfigured. You will also need to configure your Desktop Manager and your Sessions, all without seeing anything on the screen.*

#### Default login for my custom-made FS:

  | Username | Password   |
  |----------| ---------- |
  | alarm    | pixelcalarm |

#### Default login for the original ALARM rootfs:
  | Username | Password   |
  |----------| ---------- |
  | alarm    | alarm |

### Extract the filesystem

*NOTE: You may need this [busybox binary](https://busybox.net/downloads/binaries/1.26.2-defconfig-multiarch/busybox-armv6l), personally I had some troubles when extracting `tar.gz` files with TWRP included busybox (returns "Killed" after some files extracted / has many segfaults). Therefore you may need to `adb push busybox-armv6l /cache; chmod u+x /cache/busybox-armv6l` and then call `/cache/busybox-armv6l tar -xvf ...` instead*

1. Boot in TWRP
2. `adb push arch-xfce-lightdm.tar.gz /data/`
3. `cd /data`
4. `tar -xvf arch-xfce-lightdm.tar.gz`

### From prebuilt images
~~Prebuilt boot.img images aren't available yet, but you can still boot the system by putting the Pixel C in fastboot mode and doing~~

Prebuilt images are available [here](https://github.com/denysvitali/linux-smaug/releases), just flash the latest boot.img with the following command
```
fastboot flash boot boot.img
fastboot boot boot.img
```

### From sources

#### Requirements
 - Docker

#### Preparation
Pull the [dvitali/android-build-tools](https://hub.docker.com/r/dvitali/android-build-tools/) image with `docker pull dvitali/android-build-tools` or build it yourself from [this repository](https://github.com/denysvitali/docker-android-build-tools).

Run the container with:
`docker run -v kernel:/kernel -it dvitali/android-build-tools:latest`

This will give you a shell (zsh) from where you can compile the kernel - all the necessary tools are there.

If you need another shell, just do `docker exec -it containerName zsh` (where `containerName` is the name of the container, which can be found with `docker ps`)

#### Compilation

Run the following commands in the docker container:
```
cd /kernel
git clone https://github.com/denysvitali/linux-smaug/ -b linux-4.13-rc4 linux-smaug
cd linux-smaug/
make -j$(nproc)
./build-image.sh
wget https://ded1.denv.it/pixel-c/ramdisk.gz -O /kernel/ramdisk.gz
```

For the last command, you can use your own ramdisk, or compile it from my [source]().

### Mounting the kernel dir
Mount the `kernel` dir in your home:
```
mkdir ~/kernel
sudo mount -o bind /var/lib/docker/volumes/kernel/_data ~/kernel
```

### Flashing the image
```
sudo fastboot boot ~/kernel/linux-smaug/Image.fit ~/kernel/ramdisk.gz
```

## Boot
If your screen looks like this after booting the image, wonderful! It means that it is booting (don't be scared about the static).

*I honestly was really scared at first, but don't worry, if my Pixel C didn't die after all I did, this won't be harmful*

![Static showing on Pixel C](/images/static.jpg)

# Resources
[Arch Linux Forums: How to compile the Pixel C Kernel](https://bbs.archlinux.org/viewtopic.php?pid=1594095#p1594095)  
[Thierry's Blog: Booting a Pixel C tablet with a custom kernel](http://tescande.blogspot.ch/2016/12/booting-pixel-c-tablet-with-custom.html)  
[Git at Google: Kernel for Tegra](https://android.googlesource.com/kernel/tegra/+/android-o-preview-4_r0.1)  
[GitHub: Mathieu's Kernel for Smaug (4.11-RC1)](https://github.com/Samt43/linux/tree/Smaug_kernel_RC1)  
[GitHub: Mathieu's Ramdisk](https://github.com/Samt43/Smaug)  
[GitHub: My Kernel for Smaug (4.13-RC4)](https://github.com/denysvitali/linux-smaug)  
[GitHub: My initramfs](https://github.com/denysvitali/smaug-custom-initram)   

# Contributors
 - [Samt43](https://github.com/Samt43), original author of the project

# Support the project
## Donate to the major contributors of this project
 - [Donate to Mathieu Tournier, aka Samt434](http://paypal.me/MathieuTournier)
 - [Donate to Denys Vitali, aka denvit](http://paypal.me/denvit)

## Don't like to donate money to strangers?
Donate some money to the [Linux Foundation](https://www.linuxfoundation.org/about/linux-donate) or the [EFF](https://supporters.eff.org/donate).

## Don't have money to spend?
Give us some help by solving [some of the issues](https://github.com/denysvitali/linux-on-pixel-c/issues)! The more we are working on this project, the faster we could all enjoy the Linux experience on our tablets!

### Don't have a Pixel C?
You can still help us! We've got you covered: in the [dmesg](/dmesg/) folder we've put some dmesg outputs to help us find / spot some errors and fix some bugs. Take a look!
