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

关于全志的这款 f1c200s，很多资料是通过[WhyCan Forum](https://whycan.cn/f_17.html)和[荔枝派Nano](http://nano.lichee.pro/index.html)获取的。

# 开发环境搭建
既然是嵌入式系统，spiflash启动是一定要的。tf卡的启动比较简单，基本跟树莓派类似，所以就不再操作了。<br>

我会从以下几个过程讲述：<br>
1. 编译u-boot<br>
2. 编译linux内核<br>
3. 编译根文件系统<br>
4. 打包spiflash 16MB bin文件<br>
5. 烧写验证<br>

## 编译linux内核
获取 linux 源码：[linux](https://github.com/Icenowy/linux)，拉取 f1c100s-480272lcd-test 分支：
```
git clone git@github.com:Icenowy/linux.git -b f1c100s-480272lcd-test f1c100s-480272lcd-test
```

从荔枝派官网下载 linux 编译 .config 配置文件：[config](http://dl.sipeed.com/LICHEE/Nano/SDK/config)，自行重命名为 .config

修改 dts 以适配 spi flash，修改内核源码目录下的 ./arch/arm/boot/dts/suniv-f1c100s-licheepi-nano.dts，将原来的&spi0{...}替换为:
```
&spi0 {
    pinctrl-names = "default";
    pinctrl-0 = <&spi0_pins_a>;
    status = "okay";
    spi-max-frequency = <50000000>;
    flash: w25q128@0 {
        #address-cells = <1>;
        #size-cells = <1>;
        compatible = "winbond,w25q128", "jedec,spi-nor";
        reg = <0>;   
        spi-max-frequency = <50000000>;
        partitions {
            compatible = "fixed-partitions";
            #address-cells = <1>;
            #size-cells = <1>;

            partition@0 {
                label = "u-boot";
                reg = <0x000000 0x100000>;
                read-only;
            };

            partition@100000 {
                label = "dtb";
                reg = <0x100000 0x10000>;
                read-only;
            };

            partition@110000 {
                label = "kernel";
                reg = <0x110000 0x400000>;
                read-only;
            };

            partition@510000 {
                label = "rootfs";
                reg = <0x510000 0xAF0000>;
            };
        };
    };
};  
```

编译设备树
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- dtbs -j2
```

修改内核配置：
```
make ARCH=arm menuconfig
```

勾选 File systems ‣ Miscellaneous filesystems ‣ Journalling Flash File System v2 (JFFS2) support

修改 ./drivers/mtd/spi-nor/spi-nor.c
```
注释掉以下一行:
//{ "w25q128", INFO(0xef4018, 0, 64 * 1024, 256, SECT_4K) },
在这一行下面增加一项:
{ "w25q128", INFO(0xef4018, 0, 64 * 1024, 256, 0) },
```

编译：
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- -j2
```

编译完成后得到如下固件：
```
-rwxrwxr-x 1 ted ted 3.8M 1月   9 00:58 arch/arm/boot/zImage
```

## 编译rootfs

### 应用程序编译
需要用 rootfs 的编译器进行编译
```
buildroot-2019.02.8/output/host/bin/arm-linux-gcc
```

## 打包nor-spiflash
16MB spi flash:
```
#!/bin/bash

dd if=/dev/zero of=f1c100s_spiflash_16M.bin bs=1M count=16 &&\
#dd if=Lichee-Pi_u-boot/u-boot-sunxi-with-spl.bin of=f1c100s_spiflash_16M.bin bs=1K conv=notrunc &&\
dd if=u-boot/u-boot-sunxi-with-spl.bin of=f1c100s_spiflash_16M.bin bs=1K conv=notrunc &&\
dd if=linux/arch/arm/boot/dts/suniv-f1c100s-licheepi-nano.dtb of=f1c100s_spiflash_16M.bin bs=1K seek=1024 conv=notrunc &&\
dd if=linux/arch/arm/boot/zImage of=f1c100s_spiflash_16M.bin bs=1K seek=1088 conv=notrunc &&\
mkfs.jffs2 -s 0x100 -e 0x10000 --pad=0xAF0000 -d rootfs/ -o rootfs.jffs2 &&\
dd if=rootfs.jffs2 of=f1c100s_spiflash_16M.bin bs=1k seek=5184 conv=notrunc &&\
sync
```

## 烧写验证
按住 boot 按键，短按一下 reset 按键，松开 boot 按键，进入 fel 下载模式；

在 win10 环境下

烧录整个 spi-flash 16MB 空间：<br>
> .\sunxi-fel.exe -p spiflash-write 0 binfilepath

只烧录设备树：<br>
> .\sunxi-fel.exe -p spiflash-write 0x100000 ${PATH}\linux/arch/arm/boot/dts/suniv-f1c100s-licheepi-nano.dtb

只烧录 kernel 镜像：<br>
> .\sunxi-fel.exe -p spiflash-write 0x110000 ${PATH}\linux\arch\arm\boot\zImage

