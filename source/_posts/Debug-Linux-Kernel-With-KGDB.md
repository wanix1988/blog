---
title: Debug Linux Kernel With KGDB
date: 2018-12-04 11:30:00
tags: debug,linux,kernel,kgdb,kdb
---
# 使用KGDB调试Linux Kernel(20181114)
1. Grub启动命令行中添加kgdboc=ttyS0,115200 kgdbwait nokaslr   
   a) rodata=off   
   > 这条好像没什么用？待验证  

   b)必须添加nokaslr，否则vmlinux符号会对不上，无法正常显示debug信息和下断点
2. 使用socat做转发   
   a) sudo socat -d -d /tmp/vboxsock PTY
    > 1. 记下打印出的转发设备，比如/dev/pts/4  
    > 2. 这个操作要在串口连接虚拟机前做，不然就会直接退出，连接不上了  
    > 3. 控制台不要关闭
    > 4. 如果没有权限连接/dev/pts/4，可以在gdb中执行下面的命令开启权限
    >> sudo chmod 777 /dev/pts/4
3. 不要在start_kernel处下断点，因为kgdb在start_kernel执行后才初始化，所以不会在start_kernel断下来
4. References   
   * https://kgdb.wiki.kernel.org/index.php/Main_Page
   * https://kernelnewbies.org/KernelHacking-HOWTO/Debugging_Kernel
   * https://www.kernel.org/doc/htmldocs/kgdb/index.html
   * https://stackoverflow.com/questions/48539434/kgdb-and-gdb-over-serial-cable-cannot-set-breakpoints
