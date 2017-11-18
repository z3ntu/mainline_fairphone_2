# Mainline kernel for the Fairphone 2

**Please note,** that I cannot help everybody and I am quite busy with all the stuff I'm doing without helping other people. So basically I'm probably not in the mood of helping you with your problems :)

I will gladly accept pull requests (or issues) in this repo that improve stuff.

And now, let's start with **mainline**!

## Sources
* Mainline: https://github.com/torvalds/linux
* WIP-tree: https://github.com/z3ntu/linux (in multiple branches)

## Build instructions
### Get the source
* Mainline Linux has support for the Fairphone 2 starting with 4.15-rc1, so that should work too for basic funtions.
* If you decide to use my tree, clone https://github.com/z3ntu/linux and check out your desired branch (as time of this writing, the branch `FP2-wip-2` contains experimental support for display)

### Compile the kernel

Install these packages on Arch Linux: `sudo pacman -S arm-none-eabi-gcc cpio` (similar packages should be available on other distributions too).

I am using `mkbootimg` from https://github.com/efidroid/build/blob/master/tools/mkbootimg but other variants should work too.
```
# export variables
export ARCH=arm
export CROSS_COMPILE=arm-none-eabi-
```
You have multiple options for the defconfig:
* Use qcom_defconfig which should work for basic functions: `make qcom_defconfig`
* Use my config which has stuff like display, usb etc configured correctly: Download https://pastebin.com/raw/KFZnEWqn into the file `.config`
```
# compile it
make -j9
# append the dtb
cat arch/arm/boot/zImage arch/arm/boot/dts/qcom-msm8974-fairphone-fp2.dtb > ../zImage-dtb
```

### Ramdisk
This "guide" covers 2 options for the ramdisk:
* A custom one
* One from postmarketOS (which configures USB networking etc properly and automatically)

#### Create it yourself (example)
```
mkdir -p ramdisk/bin && cd ramdisk
curl https://www.busybox.net/downloads/binaries/1.26.2-defconfig-multiarch/busybox-armv6l -o bin/busybox
ln -s busybox bin/sh
find . | cpio -o -H newc | gzip > ../ramdisk.cpio.gz
cd ..
```

#### Use the postmarketOS ramdisk
You can either get it yourself from postmarketOS or you just download it from https://cloud.z3ntu.xyz/index.php/s/pTic7S6VeFnxS5b

### Create the boot.img
```
mkdir out
mkbootimg --base 0 --pagesize 2048 --kernel_offset 0x00008000 --ramdisk_offset 0x01000000 --second_offset 0x00f00000 --tags_offset 0x00000100 --cmdline 'rdinit=/init ignore_loglevel drm.debug=31 cma=64m' --kernel zImage-dtb --ramdisk ramdisk.cpio.gz -o out/mainline-boot.img
```

## Boot instructions
For `sign_img` follow https://z3ntu.github.io/2017/06/16/Signing-boot-images.html
```
cd out
sign_img mainline-boot.img
fastboot boot mainline-boot.img
```
If you used the defconfig mentioned above, you should see the boot console on screen.
On serial you can see someting like https://pastebin.com/BjbcPZUB.
