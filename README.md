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
- Install pre-requisites:
```
sudo apt install gcc-aarch64-linux-gnu binutils-aarch64-linux-gnu
```
- Make a directory for your aarch64 build:
```
mkdir aarch64_build
```
- On your chosen machine, download a version of the Linux kernel, *e.g. linux-5.5.1*:
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
make mrproper

```


