###AM335x U-Boot User's Guide
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
