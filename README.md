# 用qemu调试内核

## x86

+ 安装qemu-system-x86_64
+ 下载内核并编译

      make ARCH=x86 x86_64_defconfig
      make ARCH=x86 -j64

+ 下载busybox并编译制作文件系统

      //制作ext4文件
      dd if=/dev/zero of=rootfs.img bs=1M count=10
      mkfs.ext4 rootfs.img
      
      //再将rootfs.img挂载，例如~/rootfs
      mount rootfs.img ~/rootfs
      
      git clone https://github.com/mirror/busybox.git
      cd busybox
      make menuconfig
      make -j64
      make install CONFIG_PREFIX=~/rootfs/
      
      //补一些rootfs下文件
      cd ~/rootfs
      mkdir proc dev etc home sys tmp
      cp -rd /root/busybox/examples/bootfloppy/etc/* etc/
      
      cd /root
      umount rootfs.img

+ 启动

      qemu-system-x86_64 -kernel ./linux-5.8/arch/x86_64/boot/bzImage -hda ./rootfs.img -append "root=/dev/sda console=ttyS0" -nographic
      
  
## arm

+ 安装qemu-system-arm

+ 编译内核

      make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- defconfig
      make -j64
      
+ 下载busybox并编译制作cpio文件系统

      mkdir rootfs_arm
      
      cd busybox
      make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- install CONFIG_PREFIX=../rootfs_arm
      
      cd ../rootfs_arm
      mkdir proc dev etc home sys tmp
      cp -rd /root/busybox/examples/bootfloppy/etc/* etc/
      
      // 编译 rootfs_arm/etc/init.d/rcS 加入如下内容，因为需要用/dev/ram启动
      mount -t proc none /proc
      mount -t sysfs none /sys
      /sbin/mdev -s
      [ ! -h /etc/mtab ]  && ln -s /proc/mounts /etc/mtab
      [ ! -f /etc/resolv.conf ] && cat /proc/net/pnp > /etc/resolv.conf
      
      // 制作cpio
      find . | cpio -o --format=newc > ../rootfs.cpio
      cd ..
      gzip -c rootfs.cpio > rootfs.cpio.gz
      
+ 启动：

      qemu-system-arm -M virt -m 512M -kernel linux-5.8/arch/arm/boot/zImage -nographic -append "root=/dev/ram0 console=ttyAMA0 rdinit=/sbin/init" -initrd ./rootfs.cpio.gz
      
## x86的img、arm的cpio也可直接从此下载。

      
