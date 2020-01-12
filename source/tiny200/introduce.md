# 介绍
之前一直从事单片机的工作（混口饭吃），主要做RTOS，会一些Linux应用，但是对Linux内核和驱动这块一直是一个小白。想进一步学习，然而一直停滞在Linux这扇门外，久久踏不进去。<br>
回想起来，总是困惑我，让我无法迈出第一步的几个问题如下：
1. Linux系统镜像制作复杂
2. Linux系统在嵌入式设备上，如何烧录运行
3. Linux的驱动框架庞大
4. Linux内核让人望而生畏
5. 分不清Linux文件系统和内核的关系

直到我看到Widora的硬件，应了Widora老板的一句话，极品，艺术品，小巧精美，我也自认为看过不少PCB，而Widora也符合我的审美观，于是抱着哪怕积灰也要收藏的想法，买了一块把玩。（这里我就不贴TB路径了，大家如果有想法，凭借多年TB技能找到应该不成问题）

着手Tiny200也是因为价格真是出奇的便宜啊，之前树莓派+tf卡+微雪屏，怎么也要500大洋左右，这个竟然才两位数，谁让我们这些嵌入式工程师总是和价格过不去呢，一直走在扣成本，扣资源，扣内存的道路上。于是我开始入坑了。

接下来的内容是我在学习tiny200，更确切的说是学习Linux的总结和记录。一方面梳理自己碎片化的知识，另一方面给同样路过的朋友做个参考。我的很多知识受益于网络，希望我也能帮助到其他人。<br>
因为工作习惯的原因，我比较喜欢用邮箱，我的邮箱:<tednick@163.com>，有需要的朋友可以通过这个地址联系我。

# 开发环境搭建
既然是嵌入式系统，spiflash启动是一定要的。tf卡的启动比较简单，基本跟树莓派类似，所以就不再操作了。<br>

我会从以下几个过程讲述：<br>
1. 编译u-boot<br>
2. 编译linux内核<br>
3. 编译根文件系统<br>
4. 打包spiflash 16MB bin文件<br>
5. 烧写验证<br>

## 编译u-boot
分区规划
下表为分区规划表：
分区序号 分区大小 分区作用 地址空间及分区名
mtd0 1MB (0x100000) spl+uboot 0x0000000-0x0100000 : “uboot”
mtd1 64KB (0x10000) dtb文件 0x0100000-0x0110000 : “dtb”
mtd2 4MB (0x400000) linux内核 0x0110000-0x0510000 : “kernel”
mtd3 剩余 (0xAF0000) 根文件系统 0x0510000-0x1000000 : “rootfs”
uboot 修改
以下是对 uboot 进行适配的流程描述；
bootcmd修改
在uboot源码目录下 进入 ./include/configs/
修改 suniv.h
 #define CONFIG_BOOTCOMMAND   "sf probe 0 50000000; "  \
                             "sf read 0x80C00000 0x100000 0x4000; "  \
                             "sf read 0x80008000 0x110000 0x400000; " \
                             "bootz 0x80008000 - 0x80C00000"

按照行数解释如下：
挂载 spi-flash
读取 spi-flash 1M（0x100000）位置 64KB(0x4000)大小的 dtb 到地址 0x80C00000
读取 spi-flash 1M+64K（0x110000）位置 4MB(0x400000)大小的 zImage 到地址 0x80008000
从 0x80008000 启动内核，从 0x80C00000 读取设备树配置
回到 uboot 源码一级目录，make ARCH=arm menuconfig 进入TUI配置；
取消勾选 [ ] Enable a default value for bootcmd
bootargs修改
勾选 [*] Enable boot arguments；
在下方一项中填入 bootargs 参数:
console=ttyS0,115200 panic=5 rootwait root=/dev/mtdblock3 rw rootfstype=jffs2
(root=/dev/mtdblock3 指的是mtd设备第三分区，分区指定在dts中声明)

回到根目录重新编译
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- -j4

支持RGB屏

## 编译linux内核

### 重新编译设备树
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- dtbs -j2
```

## 编译rootfs

## 打包nor-spiflash

## 烧写验证