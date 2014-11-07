###AM335x U-Boot User's Guide
####U-Boot
对于AM335x，他的ROM code就好比bootloader的第一阶段。第二和第三阶段都是基于U-Boot实现的。在本篇文档的下文中，第二阶段的二进制文件指的是SPL，第三阶段的二进制文件仅代表U-Boot。SPL是一个非交互性的loader，是一个特定版本的U-Boot。当编译U-Boot的时候，它也同时被build。
	ROM代码可以从任何以下设备中加载SPL image
	* **Memory devices non XIP (NAND/SDMMC)**
