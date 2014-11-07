###AM335x U-Boot User's Guide
[Ti原文地址](http://processors.wiki.ti.com/index.php/AM335x_U-Boot_User%27s_Guide)
####U-Boot
对于AM335x，他的ROM code就好比bootloader的第一阶段。第二和第三阶段都是基于U-Boot实现的。在本篇文档的下文中，第二阶段的二进制文件指的是SPL，第三阶段的二进制文件仅代表U-Boot。SPL是一个非交互性的loader，是一个特定版本的U-Boot。当编译U-Boot的时候，它也同时被build。
<br>ROM代码可以从任何以下设备中加载SPL image
* **Memory devices non XIP (NAND/SDMMC)**
<br>image 必须拥有image header.Image header 必须是8个Byte的大小，包括加载地址（入口点）以及要被copy的image的大小信息。RBL会copy image，它的大小在image header中已经给定，在Image header的load address字段定义了设备地址到加载进内部内存地址。
* **外围设备(UART)**
<br>RBL加载image到内存地址0x402f0400并执行它。不提供image header。

####Two stage U-Boot design
这部分简要介绍适用于AM335X的two stage U-Boot 的方法。
AM335X的内部RAM为128KB ，其中结尾的18KB被ROM code使用。此外，开头的1KB(0x402f0000 - 0x402f0400)被保护了，不允许操作。因此U-Boot的binary只能使用有限的109KB，在DRAM初始化前，ROM code可以转移到内部ＲＡＭ中作为初始化堆栈。
在预留一些空间给堆、栈后，要在小于109KB的空间内挤下所有的功能是不可能的，因此two stage方法被采用了。第一阶段仅装载一些必须的启动设备（NAND, MMC, I2C 等等），第二阶段装载所有其他的设备（ethernet, timers, clocks 等等）第一阶生成MLO，第二阶段生成u-boot.img。
```
注意：当使用内存设备(NAND)加载的时候，a header 必须绑定到SPL的binary上，以表明加载地址和image的容量大小。SPI 启动在烧写image的时候，需要额外的字节转换。
当使用外围设备（UART）启动的时候，因为加载地址是固定的，就不需要header了。
```
#### Updated Toolchain
Sitara Linux SDK 6.0的toolchain的地址被改变了，此外对于非Arm 9 的设备一个新的基于Linaro的toolchain就派上用场了。toolchain地址变更的详细信息参看[here](http://processors.wiki.ti.com/index.php/Sitara_Linux_SDK_GCC_Toolchain#Updated.C2.A0Linux-Devkit_Structure),变更Linaro的详细信息在这里可以找到[here](http://processors.wiki.ti.com/index.php/Sitara_Linux_SDK_GCC_Toolchain#Switch_to_Linaro)
AM18x的用户切换到Linaro不会受到影响。因此，任何引用Linaro toolchain的前缀"**arm-linux-gnueabihf-**" 都应该替换为"**arm-arago-linux-gnueabi-**"
#### Building U-Boot
###**Prerequisite**
针对Arm设备的GNU toolchain比较推荐的是来自Arago的。Arago toolchain在SDK[here](http://software-dl.ti.com/dsps/dsps_public_sw/am_bu/sdk/AM335xSDK/latest/index_FDS.html)的linux-devkit 目录下可以找到.
```
以下步骤假定release package在名为$AM335x-PSP-DIR的路径下解压
```
首先，切换到U-Boot的根目录。
    $ cd ./AM335x-LINUX-PSP-MM.mm.pp.bb/src/u-boot/u-boot-MM.mm.pp.bb

