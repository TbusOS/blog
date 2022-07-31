---
linkTitle: "如何使用qemu启动linux"
author: "廖文雄"
categories: ["Linux Build"]
tags: ["qemu"] 
title: "qemu-lunchlinux"
date: 2022-07-18
weight: 3
description: >


---

# 版本信息

系统版本：Ubuntu Server 20.04 LTS 64bit
编译器：9.4.0
交叉编译器：arm-linux-gnueabi-gcc  (Linaro GCC 7.5-2019.12)  7.5.0
linux内核：linux-5.15.53
busybox：1.35.0
QEMU：7.0.0
ninja-build：1.10.0
# 获取交叉编译链
下载地址：[Linaro Releases](https://releases.linaro.org/components/toolchain/binaries/7.5-2019.12/arm-linux-gnueabi/)
![image-20220718155906055](C:%5CUsers%5Cliulf%5CDesktop%5C%E5%8D%9A%E5%AE%A2%5Cpictures.assets%5Cimage-20220718155906055.png)

``` shell
tar xvf gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabi.tar.tar	#解压
vi ~/.bashrc	#修改环境变量PATH，让系统先去指定目录寻找交叉编译链的可执行文件
```

![image-20220718160818198](https://raw.githubusercontent.com/gitliulf/picture/main/image-20220718160818198.png)
在.bashrc文件加上这行，交叉工具链的目录取决于自己解压的目录
export PATH=/home/ubuntu/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabi/bin:$PATH
``` shell
source ~/.bashrc	#让.bashrc生效
```
安装成功
![image-20220718160830447](https://raw.githubusercontent.com/gitliulf/picture/main/image-20220718160830447.png)

# 编译QEMU
``` shell
sudo apt-get install ninja-build	#Need ninja. Ninja is a small build system with a focus on speed. 
wget https://download.qemu.org/qemu-7.0.0.tar.xz	#下载源码
tar xvf qemu-7.0.0.tar.xz	#解压
mkdir qemu_build && cd qemu_build	# 在下载目录新建文件夹build

# 以下均在/build目录下
../qemu-7.0.0/configure

make && make install	#出去喝杯咖啡，打两把游戏在来看结果
```

编译成功后，qemu-system-arm就生成在当前目录
和上面一样，将编译出的qemu的可执行文件的目录加入PATH环境变量。

``` shell
vi ~/.bashrc	#修改环境变量PATH
```

在.bashrc文件上次修改的基础上，重新修改
export PATH=/home/ubuntu/qemu_build:/home/ubuntu/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabi/bin:$PATH

``` shell
source ~/.bashrc	#让.bashrc生效
```

安装成功
![image-20220718160842204](https://raw.githubusercontent.com/gitliulf/picture/main/image-20220718160842204.png)
# 编译Linux内核
内核代码下载地址：[The Linux Kernel Archives](https://www.kernel.org/)
``` shell
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- vexpress_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- menuconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- -j2
```

ps：Versatile Express系统由ARM Ltd提供，作为CortexA9四核处理器的开发环境，硬件由uATX主板和CoreTile Express A9x4子板组成。
## 验证只启动Linux内核
``` shell
qemu-system-arm -M vexpress-a9 -m 128M -kernel ~/linux-5.15.53/arch/arm/boot/zImage -dtb ~/linux-5.15.53/arch/arm/boot/dts/vexpress-v2p-ca9.dtb -nographic
```

ps：内核和设备树的路径每个人可能不一样，需要自己注意下

![image-20220718160857131](https://raw.githubusercontent.com/gitliulf/picture/main/image-20220718160857131.png)
启动成功，但还没有文件系统，操作系统稍后制作。
输入ctrl+A+x退出qemu

# 编译busybox，制作文件系统
## 编译
``` shell
wget https://busybox.net/downloads/busybox-1.35.0.tar.bz2	#下载源码
tar xvf busybox-1.35.0.tar.bz2	#解压
cd busybox-1.35.0/
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- menuconfig
```
- > Settings —>
  >
  > - [*] Build static binary (no shared libs) #使用静态库

``` shell
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- -j4
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- install
```

- 生成结果位于：当前根目录下的_install文件夹下
- 备注：默认安装到根目录下的_install，若想更改，可以通过menuconfig里面的选项更改

## 制作文件系统镜像

### 根目录构建

``` shell
mkdir -p rootfs/{dev,etc/init.d,lib,proc,sys}   #创建根目录

cp -raf busybox-1.35.0/_install/* rootfs        #拷贝busybox命令到根目录

sudo mknod -m 666 rootfs/dev/tty1 c 4 1			#创建4个tty端终设备
sudo mknod -m 666 rootfs/dev/tty1 c 4 2
sudo mknod -m 666 rootfs/dev/tty2 c 4 2
sudo mknod -m 666 rootfs/dev/tty3 c 4 3
sudo mknod -m 666 rootfs/dev/tty4 c 4 4
sudo mknod -m 666 rootfs/dev/console c 5 1		#创建console字符设备
sudo mknod -m 666 rootfs/dev/null c 1 3			#创建null 字符设备

vim rootfs/etc/init.d/rcS						#输入如下内容
```

``` shell
#!/bin/bash 
mount -t proc proc /proc 
mount -t sysfs sysfs /sys 
/sbin/mdev -s 
echo /sbin/mdev > /proc/sys/kernel/hotplug      #支持热插拔
```

``` shell
sudo chmod +x rootfs/etc/init.d/rcS
cd rootfs
find ./ | cpio -o --format=newc > ./rootfs.img
```

## 启动qemu

``` shell
qemu-system-arm -M vexpress-a9 -m 128M -kernel linux-5.15.53/arch/arm/boot/zImage -dtb linux-5.15.53/arch/arm/boot/dts/vexpress-v2p-ca9.dtb -append "root=/dev/ram rdinit=sbin/init console=ttyAMA0" -nographic -initrd rootfs/rootfs.img
```
启动成功
![image-20220718160906354](https://raw.githubusercontent.com/gitliulf/picture/main/image-20220718160906354.png)
