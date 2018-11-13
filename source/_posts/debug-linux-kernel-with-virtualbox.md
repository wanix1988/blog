---
title: debug_linux_kernel_with_virtualbox
date: 2018-11-13 22:25:52
tags: debug,linux,kernel,virtualbox,kgdb,serial
---
# 使用VirtualBox搭建Linux Kernel调试平台
## 1.安装linux-source-$(uname -r)
    a) 最好在Host上安装/编译，然后将linux-source目录挂载到虚拟机中安装，如果在虚拟机中安装和编译，会导致debug kernel的时候无法访问源码和symbols
    b) 如果/分区够大(free > 30GB)，可以选择放在/usr/src目录下，否则找分区free空间较大的放置，防止空间不够
## 2.解压/usr/src/linux-source-$(uname -r),并修改相关目录权限
## 3.使用make oldconfig创建模板config
## 4.使用make menuconfig客制化选项 ==> 打开debug开关和所需其他功能
## 5.make -j4 && make modules
## 6.make install && make modules_install
## 7.sudo mkinitramfs 4.15.0 -o /boot/initrd-4.15.8
    a) 这步必须要做，否则开机找不到initramfs会无法挂载/root
## 8.sudo update-grub
## 9.设置VirtualBox的串口
    a) com1
    b) Host Pipe
    c) 不要勾选连接已有pipe/socket
    d) Path/Address填写/tmp/vboxsock (随便命名)
## 10.安装minicom
## 11.配置minicom
    a) 串口名称unix#/tmp/vboxsock ==>此处一定要记得加unix#前缀，否则连不上
    b) 115200n8
    c) 配置会写到~/.minirc中
## 12.Kgdb设置
    a) console=ttyS0,115200 kgdboc=ttyS0,115200 kgdbwait
    b) 启动内核后，在串口终端中，输入如下命令：
        i.bash-3.2# echo g > /proc/sysrq-trigger 
            1.SysRq : DEBUG
            2.Entering KGDB
    c) 建议开始时每次在启动时暂停grub，然后在kernel后输入上面command line参数后启动，避免修改grub相关文件导致的无法启动问题
        d) 如果修改/boot/grub/grub.cfg，或者/etc/default/grub，或者/boot/grub/menu.lst等文件导致启动问题无法正常进入系统，可以在光驱中挂载livecd，然后进入后挂载/boot分区后修复即可，无需重装系统。
# 注意事项：
    1.Ubuntu 16.04.5 LTS使用Ubuntu18.04 LTS编译的linux-source-4.15.8执行make modules_install时报错: error while loading shared libraries: libssl.so.1.1 (and libcrypto.so.1.1)
        a) 需要编译libopenssl-1.1.x版本替换1.0.0版本
            ii.下载并解压源码：
                1.wget https://www.openssl.org/source/openssl-1.1.1.tar.gz
                2.tar -zxf openssl-1.1.1.tar.gz
            iii.编译安装：
                1.cd openssl-1.1.1
                2../config
                3.make install
            iv.移除旧版本 Openssl：
                1.mv /usr/bin/openssl /tmp/
                2.ln -s /usr/local/bin/openssl /usr/bin/openssl
            v.复制源码：
                1.如果报错：openssl: error while loading shared libraries: libssl.so.1.1: cannot open shared object
                    a)cp libssl.so.1.1 /lib/x86_64-linux-gnu
                    b)libcrypto.so.1.1 /lib/x86_64-linux-gnu
    2.映射Host文件夹到虚拟机
        a) 虚拟机中安装VirtualBox Guest Additional
            i.sudo mount -t iso9660 /dev/cdrom /mnt/
            ii.sudo /mnt/VBoxAdditional.sh
        b) 在Share Folders中选择Host目录，并自动挂载文件夹
            i.sudo mkdir /mnt/vboxshare
            ii.sudo mount -t vboxsf Host文件夹名 /mnt/vboxshare
                1.上面要使用Host文件夹名，否则使用share会报Protocol Error
