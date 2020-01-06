### u-boot 
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
 #define CONFIG_BOOTCOMMAND   "sf probe 0 50000000; "                           \
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