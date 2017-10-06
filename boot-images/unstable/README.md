# Kernel Images
This directory contains the builded kernel images that are considered *unstable*. They are added here in case someone wants to try them without building the kernel (and the ramdisk) from scratch.  
The images ending in .unsigned aren't signed: therefore you may only boot them via `fastboot boot image-name.img.unsigned` - flashing them (`fastboot flash boot image-name.img.unsigned`) won't work, because on boot (even when the device has an unlocked bootloader) a signature check is performed.  
To sign the images, you need to use `futility`, and sign the image with ChromeOS developer keys ([available here](https://chromium.googlesource.com/chromiumos/platform/vboot_reference/+/master/tests/devkeys/), or in `/usr/share/vboot/devkeys/` when futility is installed).  
  
If you don't want to mess around with installations and such, run my container, you'll find all the tools needed there:
```
docker run --rm -it -v /absoulte/path/to/kernel/parent-dir/:/kernel dvitali/android-build-tools
```
This will give you a `zsh` shell, with some useful tools like `vbutil_*` and `futility`.


## 20171006_094326
Boots:  
**YES**  

WiFi / BT:  
**NOT WORKING**  
  
Lightbar:  
**NOT WORKING**

GPU:  
**NO ACCELERATION**  
  
[dmesg](https://github.com/denysvitali/linux-on-pixel-c/blob/master/dmesg/4.14-rc4_20171006_094829.md)

Linux:  
`Linux alarm 4.14.0-rc3+ #15 SMP PREEMPT Fri Oct 6 09:41:57 UTC 2017 aarch64 GNU/Linux`  