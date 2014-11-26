###AM335x U-Boot User's Guide
[Ti原文地址](http://processors.wiki.ti.com/index.php/AM335x_U-Boot_User%27s_Guide)
####U-Boot
-
对于AM335x，他的ROM code就好比bootloader的第一阶段。第二和第三阶段都是基于U-Boot实现的。在本篇文档的下文中，第二阶段的二进制文件指的是SPL，第三阶段的二进制文件仅代表U-Boot。SPL是一个非交互性的loader，是一个特定版本的U-Boot。当编译U-Boot的时候，它也同时被build。
<br>ROM代码可以从任何以下设备中加载SPL image
* **Memory devices non XIP (NAND/SDMMC)**
<br>image 必须拥有image header.Image header 必须是8个Byte的大小，包括加载地址（入口点）以及要被copy的image的大小信息。RBL会copy image，它的大小在image header中已经给定，在Image header的load address字段定义了设备地址到加载进内部内存地址。
* **外围设备(UART)**
<br>RBL加载image到内存地址0x402f0400并执行它。不提供image header。

####Two stage U-Boot design
-
这部分简要介绍适用于AM335X的two stage U-Boot 的方法。
AM335X的内部RAM为128KB ，其中结尾的18KB被ROM code使用。此外，开头的1KB(0x402f0000 - 0x402f0400)被保护了，不允许操作。因此U-Boot的binary只能使用有限的109KB，在DRAM初始化前，ROM code可以转移到内部ＲＡＭ中作为初始化堆栈。
在预留一些空间给堆、栈后，要在小于109KB的空间内挤下所有的功能是不可能的，因此two stage方法被采用了。第一阶段仅装载一些必须的启动设备（NAND, MMC, I2C 等等），第二阶段装载所有其他的设备（ethernet, timers, clocks 等等）第一阶生成MLO，第二阶段生成u-boot.img。
```
注意：当使用内存设备(NAND)加载的时候，a header 必须绑定到SPL的binary上，以表明加载地址和image的容量大小。SPI 启动在烧写image的时候，需要额外的字节转换。
当使用外围设备（UART）启动的时候，因为加载地址是固定的，就不需要header了。
```
#### Updated Toolchain
-
Sitara Linux SDK 6.0的toolchain的地址被改变了，此外对于非Arm 9 的设备一个新的基于Linaro的toolchain就派上用场了。toolchain地址变更的详细信息参看 [here](http://processors.wiki.ti.com/index.php/Sitara_Linux_SDK_GCC_Toolchain#Updated.C2.A0Linux-Devkit_Structure),变更Linaro的详细信息在这里可以找到 [here](http://processors.wiki.ti.com/index.php/Sitara_Linux_SDK_GCC_Toolchain#Switch_to_Linaro)
AM18x的用户切换到Linaro不会受到影响。因此，任何引用Linaro toolchain的前缀"**arm-linux-gnueabihf-**" 都应该替换为"**arm-arago-linux-gnueabi-**"
#### Building U-Boot
-
#####**Prerequisite**
针对Arm设备的GNU toolchain比较推荐的是来自Arago的。Arago toolchain在SDK [here](http://software-dl.ti.com/dsps/dsps_public_sw/am_bu/sdk/AM335xSDK/latest/index_FDS.html)的linux-devkit 目录下可以找到.
```
以下步骤假定release package在名为$AM335x-PSP-DIR的路径下解压
```
首先，切换到U-Boot的根目录。

    $ cd ./AM335x-LINUX-PSP-MM.mm.pp.bb/src/u-boot/u-boot-MM.mm.pp.bb
强烈推荐使用"O=" 参数编译到单独的目录中。
#####**Commands**
```sh
$ [ -d ./am335x ] && rm -rf ./am335x
$ make O=am335x CROSS_COMPILE=arm-linux-gnueabihf- ARCH=arm am335x_evm
```
这会在am335x的目录下生成两个binary，MLO and u-boot.img 以及在某些情况下用到的其他的临时性的binary。
#### Host configuration
-
#####**Serial port configuration**
将串口线连接到EVM 的串口(serial port is next to the power switch)和电脑的COM Port，电脑系统可以是Windows也可以是Linux。
电脑上串口终端软件的配置如下：
```
*Baud rate: 115,200
*Data bits: 8
*Parity: None
*Stop bits: 1
*Flow control: None
```
注意：如果Teraterm 被使用了，确保安装的是最新的软件版本(写作本文时最新版本号为4.67 )，在Teraterm 中Kermit protocol 的实现是不要使用旧版本。最新版Terate从这里下载 [here](http://logmett.com/index.php?/products/teraterm.html)Teraterm 最近的更新会导致用UART传输较慢。因此使用Windows内置的超级终端较好。
####Target configuration
-
#####**Boot Switch Settings**
这项配置仅在Am335x EVM开发板上有效。切换开关SW3选择启动模式。此外，单独的DIP Switch（SW8）也可以选择不同的Profiles。
具体配置图片，参看 [here](http://processors.wiki.ti.com/index.php/AM335x_U-Boot_User%27s_Guide)
####Flashing U-Boot with CCS
-

    注意U-Boot的两个文件都必须烧写到同一个media上。
在PSP release中提供了工具烧写SPL & U-Boot 到NAND flash(for NAND boot) 
参考[AM335x Flashing Tools Guide](http://processors.wiki.ti.com/index.php/AM335x_Flashing_Tools_Guide)了解如何借助NAND flash writer烧写pre-built (or compiled)binary到NAND Flash(or the recompiled one) 中去。
完成两个阶段的烧写后，确定开发板已经上电并且boot mode 选中了NAND。
#### Boot Modes
-
NAND
<br>
注意以下操作都是针对AM335X EVM开发板，关于EVM Switch Settings 的更多信息,你可以参考 [here](http://processors.wiki.ti.com/index.php/AM335x_U-Boot_User%27s_Guide#Boot_Switch_Settings)
以下简要描述U-Boot中对于NAND的支持，此外，还会介绍针对NAND如何保存kernel image，RAMDISK或者是UBIFS filesystem 确保能够无需网络上电就可让内核启动并工作。
##### OverView
Micron NAND parts (page size 2KB, block size 128KB) are supported on AM335XEVM platforms
##### NAND Layout
EVM开发板的NAND按照如下方式配置，这里提及的地址后续NAND指令会用到。
```
+------------+-->0x00000000-> SPL start         (SPL copy on 1st block)
|            |
|            |-->0x0001FFFF-> SPL end 
|            |-->0x00020000-> SPL.backup1 start (SPL copy on 2nd block)
|            |
|            |-->0x0003FFFF-> SPL.backup1 end 
|            |-->0x00040000-> SPL.backup2 start (SPL copy on 3rd block)
|            |
|            |-->0x0005FFFF-> SPL.backup2 end 
|            |-->0x00060000-> SPL.backup3 start (SPL copy on 4th block)
|            |
|            |-->0x0007FFFF-> SPL.backup3 end
|            |-->0x00080000-> U-Boot start
|            |                                    
|            |-->0x002BFFFF-> U-Boot end 
|            |-->0x00260000-> ENV start
|            |
|            |
|            |-->0x0027FFFF-> ENV end
|            |-->0x00280000-> Linux Kernel start
|            |
|            |
|            |
|            |
|            |-->0x0077FFFF-> Linux Kernel end
|            |-->0x00780000-> File system start
|            |
|            |
|            |
|            |
|            |
|            |
|            |
|            |
|            |
|            |
|            |
|            |
+------------+-->0x10000000-> NAND end (Free end)
```
##### **Writing to NAND**
To write len bytes of data from a memory buffer located at addr to the NAND block offset:

    U-Boot# nand write <addr> <offset> <len>
    注意：Offset & len应该对齐到 0x800 (2048) bytes. On writing 3000 (0xbb8) bytes, len field can be aligned to 0x1000 ie 4096 bytes. Offset 字段应该对齐到page start address, multiple of 2048 bytes.
如果在写操作的时候遇到了bad block，会被跳过直到遇到下一个'good' block。举例来说，要将地址在0x80000000 共 0x40000 bytes 写到NAND starting at block 32 (offset 0x400000):

    U-Boot# nand write 0x80000000 0x400000 0x40000
##### **Reading from NAND**
To read len bytes of data from NAND block at a particular offset to the memory buffer in DDR located at addr:

    U-Boot# nand read <addr> <offset> <len>
如果在读操作时候遇到了bad block，他会跳过直到遇上下一个'good' block。例如，要从NAND读取0x40000 bytes-从block 32 (offset 0x400000)开始到内存缓冲区的0x80000000: 

    U-Boot# nand read 0x80000000 0x400000 0x40000
##### **Marking a bad block**
有些NAND Block使用一段时间后会损坏，因此明智做法是将它们标记出来，以免造成写入的image损坏。

    U-Boot# nand markbad <offset>
例如，标记 block 32 (assuming erase block size of 128Kbytes)为bad block。offset = blocknum * 128 * 1024

    U-Boot# nand markbad 0x400000
##### **Viewing bad blocks**
查看bad block的列表

    U-Boot# nand bad
注意：用户标记的bad block **重启**后才能看到。
##### **Erasing NAND**
擦除特定地址范围的NAND blocks 或指定number的block 

    U-Boot# nand erase <start offset addr> <len>
**注意** start offset addr & len fields 应该对齐到 0x20000 (64*2048) bytes, 也即对应 block size 128KB.
这个命令会跳过bad blocks（both factory and user marked）。例如，擦除blocks 32 through 34

    U-Boot# nand erase 0x00400000 0x40000
##### **NAND ECC algorithm selection**
NAND flash memory 尽管便宜，但是会有其他的问题，比如位翻转（bit-flipping）而导致的数据损坏。然而，通过一些错误纠正技术（ error correction coding (ECC)）使得解决这个问题成为了可能。
<br>在AM335xPSP_04.06.00.09-rc2 release版本之前，对于保存在NAND Flash中的数据，UBoot支持以下的NAND ECC schemes:

1. S/W ECC (Hamming code)
2. H/W ECC (Hamming code, BCH8)

从04.06.00.09-rc2 release开始，只支持BCH8，并且这是默认选项。无需用户去主动选择nandecc命令所需要的算法和  所需文件的位置。
##### **BCH Flash OOB Layout**
对于任何一种ECC策略，我们都需要在写NAND 操作时附加额外的数据来检测和纠正（如果有可能）。在BC策略中，有些字节需要用来存储ECC相关的信息。NAND Memory中额外ECC信息存储在称为Out Of Band 或 OOB的区域。
The first 2 bytes are used for Bad block marker – 0xFFFF => Good block
接下来的‘N’ 字节 用作 BCH 信息
N = B * <Number of 512-byte sectors in a page>

B = 8 bytes per 512 byte sector in BCH4
B = 14 bytes per 512 byte sector in BCH8
B = 26 bytes per 512 byte sector in BCH16
所以对于2k page-size 并拥有64-byte OOB size的NAND flash来，我们会使用BCH8，这会消耗掉64 bytes中的2 + (14*4) = 58 bytes。EVM开发板中的NAND Flash没有足够的空间来支持BCH16。

ECC Schemes and their context of usage

| ECC type                   | Usage								 |
|:---------------------------|:-----------------------------------------------------------------| 
| S/W ECC	                 | Not use	                                                         |		
| H/W ECC - Hamming Code     | Should use this scheme only for flashing the U-Boot ENV variables.|
| H/W ECC – BCH8             | Should use this scheme while flashing any image/binary other than the U-Boot ENV variables.|
	 
选择ECC算法应用于NAND Flash输入:
`U-Boot# nandecc [sw | hw <hw_type>]`

用法：
```
sw-Set software ECC for NAND hw <hw_type> - Set hardware ECC for NAND <hw_type> - 0 for Hamming code 1 for bch4 2 for bch8 3 for bch16 Currently we support only Software, Hamming Code and BCH8. We do not support BCH4 and BCH16。
```
ECC schemes usage table

