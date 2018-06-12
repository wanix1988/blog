---
title: Running Arm64 Linux Kernel in Qemu
date: 2018-06-12 21:44:16
tags: linux, kernel, arm, qemu
categories: linux, kernel, arm, qemu
---
## 原由
看了一下《奔跑吧Linux内核》这本书，决定好好研究一下ARM64平台，包括汇编和Linux Kernel相关的知识。网络上搜索到了之前他人搭建成功的帖子，自己照着做一遍却发现qemu启动后没有任何输出，查看进程占用率却非常高，感觉就是qemu cpu跑飞了。仔细思考后觉得问题一定出在编译器上面，所以下载了linaro提供的最新编译器，果然解决问题。下面把整个流程记录一下，方便后来人。

## 软件环境
1. Ubuntu 18.04 LTS
2. Qemu 2.11.1
3. [Linaro aarch64 linux toolchain](https://releases.linaro.org/components/toolchain/binaries/latest/aarch64-linux-gnu/gcc-linaro-7.2.1-2017.11-x86_64_aarch64-linux-gnu.tar.xz)
4. [Linux Kernel Torvalds 4.7 or 4.17](git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git) 

## 软件环境配置说明
1. Ubuntu 16.04或者18.04应该都没问题，但是12.04或者14.04可能需要源码编译qemu（未验证）
2. Qemu我尝试了source code build和直接通过
`sudo apt install qemu-system-aarch64`来安装，都没有问题。source code build的时候都指明需要添加`--target-list=aarch64-softmmu`，我也有添加，但是没有不添加的测试结果。后面如果哪天不小心试验了就把结果补上。
3. 交叉编译器千万不要使用Ubuntu提供的，到上面linaro链接中下载最新的版本即可。 
4. Linux Kernel版本我有试过4.7和4.17（今天的master branch，2018/06/12），都OK
5. 如果Ubuntu系统较新，编译时需要安装下面的包：
- libncurses5-dev -- 用于`make menuconfig`时显示终端图形菜单，否则报错;
- libssl-dev -- kernel中的scripts/extract-cert.c中的openssl头文件会无法找到，报错。
6. 为了试验方便，暂未做initramfs。之前有download一份busybox，目前来看也需换编译器再重新build一遍为好。

## Qemu运行命令
```
qemu-system-aarch64 \
         -machine virt \
         -cpu cortex-a57 \
         -machine type=virt \
         -nographic \
         -m 2048 \
         -smp 2 \ 
         -kernel arch/arm64/boot/Image \
         --append "rdinit=/linuxrc console=ttyAMA0"
```

## 运行结果
```
[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 4.9.0 (wenix@wenix-ThinkPad-X230-Tablet) (gcc version 7.2.1 20171011 (Linaro GCC 7.2-2017.11) ) #1 SMP PREEMPT Tue Jun 12 21:31:21 CST 2018
[    0.000000] Boot CPU: AArch64 Processor [411fd070]
[    0.000000] efi: Getting EFI parameters from FDT:
[    0.000000] efi: UEFI not found.
[    0.000000] cma: Reserved 16 MiB at 0x00000000bf000000
[    0.000000] psci: probing for conduit method from DT.
[    0.000000] psci: PSCIv0.2 detected in firmware.
[    0.000000] psci: Using standard PSCI v0.2 function IDs
[    0.000000] psci: Trusted OS migration not required
[    0.000000] percpu: Embedded 21 pages/cpu @ffff80007efbf000 s47896 r8192 d29928 u86016
[    0.000000] Detected PIPT I-cache on CPU0
[    0.000000] CPU features: enabling workaround for ARM erratum 832075
[    0.000000] CPU features: enabling workaround for ARM erratum 834220
[    0.000000] Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 516096
[    0.000000] Kernel command line: rdinit=/linuxrc console=ttyAMA0
[    0.000000] PID hash table entries: 4096 (order: 3, 32768 bytes)
[    0.000000] Dentry cache hash table entries: 262144 (order: 9, 2097152 bytes)
[    0.000000] Inode-cache hash table entries: 131072 (order: 8, 1048576 bytes)
[    0.000000] Memory: 2032644K/2097152K available (7740K kernel code, 494K rwdata, 2744K rodata, 576K init, 271K bss, 48124K reserved, 16384K cma-reserved)
[    0.000000] Virtual kernel memory layout:
[    0.000000]     modules : 0xffff000000000000 - 0xffff000008000000   (   128 MB)
[    0.000000]     vmalloc : 0xffff000008000000 - 0xffff7dffbfff0000   (129022 GB)
[    0.000000]       .text : 0xffff000008080000 - 0xffff000008810000   (  7744 KB)
[    0.000000]     .rodata : 0xffff000008810000 - 0xffff000008ad0000   (  2816 KB)
[    0.000000]       .init : 0xffff000008ad0000 - 0xffff000008b60000   (   576 KB)
[    0.000000]       .data : 0xffff000008b60000 - 0xffff000008bdba00   (   495 KB)
[    0.000000]        .bss : 0xffff000008bdba00 - 0xffff000008c1f734   (   272 KB)
[    0.000000]     fixed   : 0xffff7dfffe7fd000 - 0xffff7dfffec00000   (  4108 KB)
[    0.000000]     PCI I/O : 0xffff7dfffee00000 - 0xffff7dffffe00000   (    16 MB)
[    0.000000]     vmemmap : 0xffff7e0000000000 - 0xffff800000000000   (  2048 GB maximum)
[    0.000000]               0xffff7e0000000000 - 0xffff7e0002000000   (    32 MB actual)
[    0.000000]     memory  : 0xffff800000000000 - 0xffff800080000000   (  2048 MB)
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=2, Nodes=1
[    0.000000] Preemptible hierarchical RCU implementation.
[    0.000000] 	Build-time adjustment of leaf fanout to 64.
[    0.000000] 	RCU restricting CPUs from NR_CPUS=64 to nr_cpu_ids=2.
[    0.000000] RCU: Adjusting geometry for rcu_fanout_leaf=64, nr_cpu_ids=2
[    0.000000] NR_IRQS:64 nr_irqs:64 0
[    0.000000] GICv2m: range[mem 0x08020000-0x08020fff], SPI[80:143]
[    0.000000] arm_arch_timer: Architected cp15 timer(s) running at 62.50MHz (virt).
[    0.000000] clocksource: arch_sys_counter: mask: 0xffffffffffffff max_cycles: 0x1cd42e208c, max_idle_ns: 881590405314 ns
[    0.000154] sched_clock: 56 bits at 62MHz, resolution 16ns, wraps every 4398046511096ns
[    0.006170] Console: colour dummy device 80x25
[    0.008012] Calibrating delay loop (skipped), value calculated using timer frequency.. 125.00 BogoMIPS (lpj=250000)
[    0.008295] pid_max: default: 32768 minimum: 301
[    0.009475] Security Framework initialized
[    0.009960] Mount-cache hash table entries: 4096 (order: 3, 32768 bytes)
[    0.009999] Mountpoint-cache hash table entries: 4096 (order: 3, 32768 bytes)
[    0.033277] ASID allocator initialised with 65536 entries
[    0.066997] EFI services will not be available.
[    0.120844] Detected PIPT I-cache on CPU1
[    0.122607] CPU1: Booted secondary processor [411fd070]
[    0.131049] Brought up 2 CPUs
[    0.131190] SMP: Total of 2 processors activated.
[    0.131388] CPU features: detected feature: 32-bit EL0 Support
[    0.133999] CPU: All CPU(s) started at EL1
[    0.135370] alternatives: patching kernel code
[    0.154940] devtmpfs: initialized
[    0.172605] DMI not present or invalid.
[    0.176170] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
[    0.221075] NET: Registered protocol family 16
[    0.260407] cpuidle: using governor menu
[    0.261303] vdso: 2 pages (1 code @ ffff000008817000, 1 data @ ffff000008b64000)
[    0.261679] hw-breakpoint: found 6 breakpoint and 4 watchpoint registers.
[    0.271773] DMA: preallocated 256 KiB pool for atomic allocations
[    0.273033] Serial: AMBA PL011 UART driver
[    0.335144] 9000000.pl011: ttyAMA0 at MMIO 0x9000000 (irq = 39, base_baud = 0) is a PL011 rev1
[    0.349070] console [ttyAMA0] enabled
[    0.567544] HugeTLB registered 2 MB page size, pre-allocated 0 pages
[    0.582343] ACPI: Interpreter disabled.
[    0.592095] vgaarb: loaded
[    0.597022] SCSI subsystem initialized
[    0.606885] usbcore: registered new interface driver usbfs
[    0.610791] usbcore: registered new interface driver hub
[    0.612937] usbcore: registered new device driver usb
[    0.618796] pps_core: LinuxPPS API ver. 1 registered
[    0.619127] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
[    0.620351] PTP clock support registered
[    0.627272] dmi: Firmware registration failed.
[    0.631976] Advanced Linux Sound Architecture Driver Initialized.
[    0.645917] clocksource: Switched to clocksource arch_sys_counter
[    0.651591] VFS: Disk quotas dquot_6.6.0
[    0.652217] VFS: Dquot-cache hash table entries: 512 (order 0, 4096 bytes)
[    0.658270] pnp: PnP ACPI: disabled
[    0.767641] NET: Registered protocol family 2
[    0.776560] TCP established hash table entries: 16384 (order: 5, 131072 bytes)
[    0.777303] TCP bind hash table entries: 16384 (order: 6, 262144 bytes)
[    0.778647] TCP: Hash tables configured (established 16384 bind 16384)
[    0.780489] UDP hash table entries: 1024 (order: 3, 32768 bytes)
[    0.781007] UDP-Lite hash table entries: 1024 (order: 3, 32768 bytes)
[    0.785364] NET: Registered protocol family 1
[    0.799807] RPC: Registered named UNIX socket transport module.
[    0.800128] RPC: Registered udp transport module.
[    0.800353] RPC: Registered tcp transport module.
[    0.800551] RPC: Registered tcp NFSv4.1 backchannel transport module.
[    0.811081] hw perfevents: enabled with armv8_pmuv3 PMU driver, 1 counters available
[    0.813225] kvm [1]: HYP mode not available
[    0.836878] futex hash table entries: 512 (order: 4, 65536 bytes)
[    0.838616] audit: initializing netlink subsys (disabled)
[    0.839971] audit: type=2000 audit(0.788:1): initialized
[    0.847568] workingset: timestamp_bits=46 max_order=19 bucket_order=0
[    0.969082] squashfs: version 4.0 (2009/01/31) Phillip Lougher
[    0.979427] NFS: Registering the id_resolver key type
[    0.980928] Key type id_resolver registered
[    0.981684] Key type id_legacy registered
[    0.982464] nfs4filelayout_init: NFSv4 File Layout Driver Registering...
[    0.986105] 9p: Installing v9fs 9p2000 file system support
[    1.007251] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 248)
[    1.008184] io scheduler noop registered
[    1.012464] io scheduler cfq registered (default)
[    1.028667] pl061_gpio 9030000.pl061: PL061 GPIO chip @0x0000000009030000 registered
[    1.034893] OF: PCI: host bridge /pcie@10000000 ranges:
[    1.036393] OF: PCI:    IO 0x3eff0000..0x3effffff -> 0x00000000
[    1.037551] OF: PCI:   MEM 0x10000000..0x3efeffff -> 0x10000000
[    1.038668] OF: PCI:   MEM 0x8000000000..0xffffffffff -> 0x8000000000
[    1.040862] pci-host-generic 3f000000.pcie: ECAM at [mem 0x3f000000-0x3fffffff] for [bus 00-0f]
[    1.043966] pci-host-generic 3f000000.pcie: PCI host bridge to bus 0000:00
[    1.044717] pci_bus 0000:00: root bus resource [bus 00-0f]
[    1.046480] pci_bus 0000:00: root bus resource [io  0x0000-0xffff]
[    1.047091] pci_bus 0000:00: root bus resource [mem 0x10000000-0x3efeffff]
[    1.047328] pci_bus 0000:00: root bus resource [mem 0x8000000000-0xffffffffff]
[    1.060800] pci 0000:00:01.0: BAR 6: assigned [mem 0x10000000-0x1007ffff pref]
[    1.061606] pci 0000:00:01.0: BAR 4: assigned [mem 0x8000000000-0x8000003fff 64bit pref]
[    1.063233] pci 0000:00:01.0: BAR 1: assigned [mem 0x10080000-0x10080fff]
[    1.063500] pci 0000:00:01.0: BAR 0: assigned [io  0x1000-0x101f]
[    1.093103] virtio-pci 0000:00:01.0: enabling device (0000 -> 0003)
[    1.099730] xenfs: not registering filesystem on non-xen platform
[    1.156174] Serial: 8250/16550 driver, 4 ports, IRQ sharing disabled
[    1.175144] Unable to detect cache hierarchy from DT for CPU 0
[    1.225263] loop: module loaded
[    1.232529] hisi_sas: driver version v1.6
[    1.249489] libphy: Fixed MDIO Bus: probed
[    1.256540] tun: Universal TUN/TAP device driver, 1.6
[    1.256836] tun: (C) 1999-2004 Max Krasnyansky <maxk@qualcomm.com>
[    1.281267] e1000e: Intel(R) PRO/1000 Network Driver - 3.2.6-k
[    1.282430] e1000e: Copyright(c) 1999 - 2015 Intel Corporation.
[    1.284398] igb: Intel(R) Gigabit Ethernet Network Driver - version 5.4.0-k
[    1.284919] igb: Copyright (c) 2007-2014 Intel Corporation.
[    1.287458] igbvf: Intel(R) Gigabit Virtual Function Network Driver - version 2.4.0-k
[    1.287846] igbvf: Copyright (c) 2009 - 2012 Intel Corporation.
[    1.289391] sky2: driver version 1.30
[    1.296439] VFIO - User Level meta-driver version: 0.3
[    1.311934] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
[    1.312653] ehci-pci: EHCI PCI platform driver
[    1.313626] ehci-platform: EHCI generic platform driver
[    1.316055] ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
[    1.316631] ohci-pci: OHCI PCI platform driver
[    1.319698] ohci-platform: OHCI generic platform driver
[    1.324997] usbcore: registered new interface driver usb-storage
[    1.339686] mousedev: PS/2 mouse device common for all mice
[    1.352956] rtc-pl031 9010000.pl031: rtc core: registered pl031 as rtc0
[    1.356558] i2c /dev entries driver
[    1.372128] sdhci: Secure Digital Host Controller Interface driver
[    1.372477] sdhci: Copyright(c) Pierre Ossman
[    1.374906] Synopsys Designware Multimedia Card Interface Driver
[    1.379711] sdhci-pltfm: SDHCI platform and OF driver helper
[    1.383049] ledtrig-cpu: registered to indicate activity on CPUs
[    1.395135] usbcore: registered new interface driver usbhid
[    1.395452] usbhid: USB HID core driver
[    1.408814] NET: Registered protocol family 17
[    1.412004] 9pnet: Installing 9P2000 support
[    1.413164] Key type dns_resolver registered
[    1.420484] registered taskstats version 1
[    1.431403] input: gpio-keys as /devices/platform/gpio-keys/input/input0
[    1.437461] rtc-pl031 9010000.pl031: setting system clock to 2018-06-12 14:49:36 UTC (1528814976)
[    1.440137] ALSA device list:
[    1.440420]   No soundcards found.
[    1.447909] uart-pl011 9000000.pl011: no DMA platform data
[    1.454875] VFS: Cannot open root device "(null)" or unknown-block(0,0): error -6
[    1.455145] Please append a correct "root=" boot option; here are the available partitions:
[    1.455914] Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
[    1.457050] CPU: 1 PID: 1 Comm: swapper/0 Not tainted 4.9.0 #1
[    1.457405] Hardware name: linux,dummy-virt (DT)
[    1.457952] Call trace:
[    1.459510] [<ffff0000080884b0>] dump_backtrace+0x0/0x1b0
[    1.459943] [<ffff000008088674>] show_stack+0x14/0x20
[    1.460185] [<ffff00000837b218>] dump_stack+0x94/0xb4
[    1.460405] [<ffff00000816630c>] panic+0x118/0x288
[    1.460599] [<ffff000008ad1144>] mount_block_root+0x19c/0x270
[    1.460797] [<ffff000008ad1334>] mount_root+0x11c/0x134
[    1.460980] [<ffff000008ad1484>] prepare_namespace+0x138/0x180
[    1.461186] [<ffff000008ad0d68>] kernel_init_freeable+0x22c/0x250
[    1.461406] [<ffff0000087f9800>] kernel_init+0x10/0x100
[    1.461596] [<ffff000008082e80>] ret_from_fork+0x10/0x50
[    1.462511] SMP: stopping secondary CPUs
[    1.463121] Kernel Offset: disabled
[    1.463866] Memory Limit: none
[    1.464458] ---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)


C-a h    print this help

C-a x    exit emulator

C-a s    save disk data back to file (if -snapshot)

C-a t    toggle console timestamps

C-a b    send break (magic sysrq)

C-a c    switch between console and monitor

C-a C-a  sends C-a

QEMU: Terminated

```