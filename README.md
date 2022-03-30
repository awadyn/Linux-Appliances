# Linux-Appliances
Instructions and source code to build Linux appliances: busybox or specialized rootfs + configurable Linux kernel

## General Build
A Linux appliance can be built on your local machine. It will end up looking like a subset of your local machine.

Follow this tutorial to see how:
https://www.usenix.org/publications/loginonline/building-linux-appliances

The main build targets are:
1. Linux kernel --> configured to include only the needed system utilities and drivers
2. root file system (rootfs) --> containing only the binaries, libraries, and devices to be used by the appliance

*NOTE: Build the Linux kernel on a separate, powerful machine with many cores if possible.
This process may take a tediously long time on your local machine.
Hence, you may build it remotely and copy the bzImage, zImage, or Image bootable binary over to your local machine.
You may then build the rootfs locally, after which you may boot the appliance locally as well.*

## aarch64 Build
To build a Linux appliance for running on an ARM 64-bit processor, the same build process is followed,
but every build target is compiled for an aarch64 architecture.
Hence, both the Linux kernel and the rootfs contents need be compiled for the 64-bit ARM architecture.

The build targets remain the same:
1. aarch64-compatible configurable Linux kernel
2. customizable rootfs with aarch64-compatible binaries

#### Linux Kernel for aarch64
- On your chosen build machine, install pre-requisites:
```
sudo apt install gcc-aarch64-linux-gnu binutils-aarch64-linux-gnu
```
- Make a directory for your aarch64 build:
```
mkdir aarch64_build
```
- Download a version of the Linux kernel, *e.g. linux-5.5.1*:
```
wget --no-check-certificate https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.5.1.tar.gz
```
- Untar and cd to linux-5.5.1:
```
tar -xf linux-5.5.1.tar.gz
cd linux-5.5.1
```
- Configure and build Linux kernel:
```
make ARCH=arm64 mrproper
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- O=../aarch64_build/ nconfig
make -j24 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- O=../aarch64_build/
```

You will now find the Linux kernel image *Image* in *aarch64_build/arch/arm64/boot/*.

#### rootfs for aarch64
- On your chosen build machine, create a working directory:
```
mkdir aarch64_appliances
cd aarch64_appliances
```
- Install pre-requisites:
```
sudo apt install binutils-aarch64-linux-gnu gcc-aarch64-linux-gnu g++-aarch64-linux-gnu
```
- Copy over the built Linux kernel image:
```
scp user@remote:<PATH_TO>/aarch64_build/arch/arm64/boot/Image .
```
- Download and untar a version of busybox; this will be used to create the rootfs structure and contents:
```
wget https://busybox.net/downloads/busybox-1.31.1.tar.bz2
tar -xjf busybox-1.31.1.tar.bz2
cd busybox-1.31.1
```
- Configure and build busybox:
```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```
*Here, select Settings -> Build Options -> Build BusyBox as a static binary (no shared libs)*
```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- install
```

You will now find a basic rootfs structure in *busybox-1.31.1/_install*.

#### Boot aarch64 Linux Appliance
- Navigate to your chosen working directory:
```
cd aarch64_appliances
ls
```
- Confirm see the aarch64 Linux image and the busybox build environment:
```
busybox-1.31.1  Image
```
- Create a new directory for your rootfs and copy the file system components previously built:
```
mkdir rootfs
cp -r busybox-1.31.1/_install/* rootfs/
```
- Create an init script that the Linux kernel runs after booting:
```
cd rootfs
vim init
```
*init script:*
```
#!/bin/sh

export HOME=/root
export LOGNAME=root
export TERM=vt100
export PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin
export ENV="HOME=\$HOME LOGNAME=\$LOGNAME TERM=\$TERM PATH=\$PATH"

# setup standard file system view
mount -t proc /proc /proc
mount -t sysfs /sys /sys
mount -t devpts devpts /dev/pts

# Some things don't work properly without /etc/mtab.
ln -sf /proc/mounts /etc/mtab

echo -e "\nSUCCESS\n"
echo -e "\nBoot took $(cut -d' ' -f1 /proc/uptime) seconds\n"

# if we get here then we might as well start a shell :-) 
/bin/sh

# if bash fails, shuts off machine
poweroff -f
EOF
```
- Confirm that the init script is executable:
```
chmod u+x init
```
