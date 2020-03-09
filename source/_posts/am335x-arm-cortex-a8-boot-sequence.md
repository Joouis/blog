---
title: 解析AM335x Arm Cortex-A8处理器的启动流程
date: 2019-12-15 21:53:32
updated: 2019-12-15 21:53:32
categories:
- 系统与底层
tags:
- embedded system course
- embedded system
- arm
- cortex-a8
- rom
- ram
- u-boot
- boot
- os
---

这是我的[嵌入式笔记](https://blog.joouis.com/2019/watching-notes-revolution-os/)第三篇，原文写于2015年。

在真实世界中，一个操作系统是如何被加载的？对于单片机、SoC以及我们常见的计算机，由于硬件设备和软件系统的不同，启动流程肯定会有差异。不过对于大型操作系统，启动流程有不少共通之处，本文将简单解析一个消费类电子产品常用的系统（基于TI的AM335x处理器）是如何被加载启动的。

这篇笔记其实是在组队项目时，由一位队友Adrian Huang所写。相比我们这些“愣头青”，Adrian已经有八九年的Linux底层工作经验，先后在HP、联想、MTK工作过，是十足的大佬。

<!-- more -->



## Boot Sequence

圖一為AM335x開機流程，其包含ROM Code、MLO、U-Boot與OS Image，底下將說明ROM Code、MLO與U-Boot。

![Figure 1: High-level Overview to AM335x Boot Sequence](https://ijrexq.bn.files.1drv.com/y4mX9xkUQI0Bru5Z650NymAW4M7z8au24r95TnCHHsV9itCLjqrqM8EctRKllEp08oUSsmfvy4RI-gVh77wYaOU7yyFDZ7jr7N7I5D6Wu_lGPZh7s3S0ne9hJ9cEeW8hsvWqozUOdC2wqCzCH-Q95SvRBftpgfMOJJHSR7TCyItWknIU8nzSEC2Vw36MOtuMBJzZyfHRmQH5OZC6I7R93R36w?width=999&height=683&cropmode=none)

### ROM Code

ROM Code主要有幾項任務:

- Stack Setup
- Watchdog timer 1  configuration (set to three minutes)
- System clock configuration
- Search bootable devices (must be the FAT 12/16/32 partition) for a valid booting image (the image name must be MLO)
- Load the content of the file "MLO" from a bootable device to internal RAM (the 128KB on-chip memory)
- Execute the file "MLO" stored in internal RAM



圖二為ROM Code架構，由"Public ROM Code drivers"可知ROM code支援如下裝置:

- MMCSD (MultiMediaCard SD)
- NAND
- XIP (eXecute In Place)
- SPI
- USB UART
- EMAC (Ethernet Media Access Control)



也就是說，系統一上電，ROM Code會掃描上述裝置，以便找到Bootable device。由於ROM Code只支援FAT檔案系統格式，所以Bootable device一定要是FAT檔案系統 (FAT12/16/32檔案系統都可以)。

注意: 該架構的On-chip boot ROM大小為176 KB。

![Figure 2: ROM Code Architecture](https://ijrgxq.bn.files.1drv.com/y4m91R7T1buLeODZG4ugSeHctLOIr7Pw3uVLGKvtPfWhS9GB7vFonpSTwASXEE7YmsbEKMoknKPgRjleSnwgRqoY-jLlag1dp3aiEbEFK2e3ckbi0smrslhehDMRAIUqolxSHQzVVyRoVUxTdxp_dkZiCujhrxLDin7ETfdoVPxcfxdIhql1UutK4J5Sv6z4qhh3v0PBQt2z0Uxm0WSGV6U5g?width=919&height=573&cropmode=none)



圖三為ROM Memory Map:

- ROM Exception Vectors (0x20000-0x2001F): 該區段定義Exception Handler的位址。譬如: 0x20000存放Reset Handler的位址，也就是板子一上電，第一個執行的地方，課程第七周有詳盡的說明，可參考此[文件](https://docs.google.com/document/d/1Ygl6cEGPXUffhTJE0K6B8zEtGmIuIdCjlZBkFlijUaE/edit)。詳盡的ROM Exception Vectors如表一所示。

![Chart 1: ROM Exception Vectors](https://ijrfxq.bn.files.1drv.com/y4mr5vbOMVqr2C2tsjR3qeH3JyeU4Y8x_zAtbshrWvRdC2EPswXAo9Hc_LwNojcmmkbW6Xs0dizUxHeDAxsecG0BfLfwQlqGFyOulBFIPuxAwIuag2tHF9ROM816OpZWgcxOEEmm4kRRDcUzHA5FGMr1k6HdHEmOIpwFqoVsP8uBQMVKw_BiTh64EKF0AyD79fvQl9DoB1ZFXUdYa40MW95Rw?width=921&height=242&cropmode=none)

- Public ROM Code CRC (0x20020): 由0x20000-0x2BFFF計算得出的四個位元組CRC值。
- Dead loops (0x20080-0x200FF): 該區段定義預設的exception handlers，其預設handlers都是執行while(1)迴圈，程式設計者可以定義相同名字的exception handler，如此便能覆蓋 (override)對應之預設exception handlers。可參考[mini-arm-os](https://github.com/jserv/mini-arm-os/blob/master/05-TimerInterrupt/startup.c#L63)與[freertos](https://github.com/embedded2015/freertos-basic/blob/master/freertos/libraries/CMSIS/CM3/DeviceSupport/ST/STM32F10x/startup/gcc_ride7/startup_stm32f10x_md.s#L124)程式碼，以便了解其設計概念。
- Code (started from 0x20100): ROM程式碼
- ROM Version (0x2BFFC-0x2BFFF): ROM Code Version

![Figure 3: ROM Memory Map](https://ijrdxq.bn.files.1drv.com/y4m1lZgiY0kQlxtKuuClWoEmBLOZYFp0mDWdCMnDla_RSbZC161f532Dwy1EjSxNjCOIKWkDgTN65FLfoE8NLWCgFRY029LQp5BZph_RbFh3VwThpt0Z0WgoIeHqezLJm-v4_6-YVGLzBqUHwizl0wxQ5-EODj0rt0kbpFeKq8bzkRKcxY5PSak9qKmJG6HKxHWDFruFFbQRPP4v92r6QDP6g?width=278&height=299&cropmode=none)



### Two Stage U-Boot Design - MLO (MMC Loader) & U-boot

#### MLO (File Name: `MLO`)

AM335x ARM Cortex-A8處理器的On-chip memory( internal SRAM)大小為128KB (詳見AM3358 Data Sheet)，其Memory Map如圖四所示:

- 1KB Secure Area: 此區域無法存取
- 109KB downloaded image (0x402F0400-0x4030B7FF): 此區域最多可以容納109KB二進制檔案，此檔案可以存放在儲存裝置，或透過UART/USB傳輸，進而ROM Code載入至位址0x402F0400。MLO其實就是小型的U-Boot Boot loader (其大小必須小於109KB)。所以，其簡化版U-Boot (MLO)主要任務如下:
  - 初始化DDR (512MB)
  - 一旦初始化DDR成功後，MLO將`u-boot.img`載入至DDR，並交由U-Boot接管。
- 18KB area (0x4030B800-0x4030FFFF): ROM Code使用該區域。

![Figure 4: Memory Map for 128KB on-chip memory](https://ijrbxq.bn.files.1drv.com/y4m4Moeghhw0ihVSV10wtEpfV51hlm3BlYa0MaWEVLa_ekCytzWsPBV6pRNWlq-huq-RPbjHe8MnlMNQutmLoUlVKmC6Pnu_8ye8PZTUrNFIDGewxKRvUJmDmyGLHQquJdjwUKF_mLgmAky_ttOjUaQ84do7KpIys2FbGmsSIHA3UEPYdhVSRA4GNv7Yp7QJ1Xt7adi1CPaKicmHUoswQK17g?width=914&height=637&cropmode=none)

#### U-boot (File name: `u-boot.img`)

此完整版U-Boot主要做額外platform初始化 (如: 初始化網卡等)，並載入OS Kernel。

**為何需要two-stage U-boot？**

由於On-chip memory只能提供109KB給Boot loader。然而，對於一個完整版的U-Boot，其大小約350KB (參考[ARMhf](http://www.armhf.com/download/)編好的[u-boot](http://s3.armhf.com/dist/bone/bone-uboot.tar.xz))。為因應此限制，便需要一個小型的U-boot (MLO)，用以初始化DDR並將完整版U-boot載入至DDR，再將控制權交給完整版U-boot。下圖在BeagleBoard Black成功載入MLO與U-Boot終端機輸出。

![](https://ijrcxq.bn.files.1drv.com/y4mb1ihQSrUuU4o5Gm9YglL6mLkaTvLOgqORGOXEkFkrvwveoOW5wYAf7CoOOlOgWCfswMZNgmQW1hEJr8tWyFr-7WTgkeItjQ7-FsNk0qBSYdenUNzuHddMLOQ9VKiD-RtiSTjf-Hq-Hx5yhI15hclmtEDcYFKok6R6h9OMBOugYBPd_P4-GlfjFPLdL4J7q0ZMQtmSQDT6iS29e7H0117DQ?width=875&height=661&cropmode=none)