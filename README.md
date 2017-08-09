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

**WARNING:** you need dm-verity and encryption OFF, or a newly formatted `/data` partion. Otherwise the system won't be able to boot into Linux.

### Prepare the filesystem

1. Boot in TWRP
2. `adb push filesystem.img /data/`
3. `cd /data`
4. `mkdir Arch`
5. `mount /data/filesystem.img /mnt`
6. `cp -rp /mnt/* /data/Arch`

### From prebuilt images
Prebuilt boot.img images aren't available yet, but you can still boot the system by putting the Pixel C in fastboot mode and doing
```
wget https://ded1.denv.it/pixel-c/Image.fit -O ~/kernel/Image.fit
wget https://ded1.denv.it/pixel-c/ramdisk.gz -O ~/kernel/ramdisk.gz
fastboot boot ~/kernel/Image.fit ~/kernel/ramdisk.gz
```

#### Ramdisk
Ramdisk is available at

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
