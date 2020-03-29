---
title: GDB/CGDB入门与拆弹游戏
date: 2020-03-29 16:22:19
updated: 2020-03-29 16:22:19
categories:
- 系统与底层
tags:
- gdb
- cgdb
- stm32
- disassembly
- binary
- defusing binary bomb
---

这是我的[嵌入式笔记](https://blog.joouis.com/2019/watching-notes-revolution-os/)第七篇，原文写于2015年。本文除了介绍GDB/CGBD的基本使用方法以及嵌入式开发的应用外，还有一个 `Defusing a binary bomb with GDB` 的经典游戏分享，非常有趣！

<!-- more -->



# 使用GDB

#### GDB

- b <function name> *<address>：設置斷點
- d <breakpoint num>：刪除斷點
- c：繼續執行，到下一個斷點處（或運行結束）
- n：單步跟蹤程序，當遇到函數調用時，也不會進入函數內（針對src）
- s：單步調試，如果有函數調用，則進入函數（針對src）
- si：同s（針對asm）
- ni：同n（針對asm）
- p /x $r0：以16進制格式打印寄存器r0的值
- x/s <address>：以字串方式打印出地址所指向的內容
- layout asm/src：會顯示出當前正在執行的asm/src代碼段（這樣就不用看objdump -D啦）
- layout split：同時顯示當前執行的原始碼和組語
- layout regs：會以10進制和16進制方式顯示寄存器的值，包含r0-r12、sp、lr、pc、cpsr（這樣就可以不用p了）
- winheight name +/- count：調整 TUI 窗口大小，例如 winheight src -5 代表 TUI 窗口減少 5 行代碼
- info stack/variables/...
- info files: 超好用的一招
- REF:
  - http://linuxtools-rst.readthedocs.org/zh_CN/latest/tool/gdb.html
  - TUI-specific Commands](https://sourceware.org/gdb/onlinedocs/gdb/TUI-Commands.html)

### CGDB

- **安裝**：`sudo apt-get install cgdb`
- 官方手冊：[CGDB Manual 0.6.6](http://cgdb.sourceforge.net/docs/cgdb-no-split.html) （请查看最新的手册：[CGDB Manual 0.7.1](https://cgdb.github.io/docs/cgdb.html)）
- 相比 GDB 無需用 layout 調出 TUI，CGDB 會自動顯示出來，source window 和 GDB command window 切換方式和 VI 相同，**使用 i 和 Esc**。
- 新增及移除 break point 方法，直接在 source window 裏移動到想要新增/移除的位置，使用 space 鍵就可以了。
- **使用 cross compiler 方法**：`cgdb -d arm-none-eabi-gdb -x "gdb.script"`
- Source window 類似 VI 界面，常見的冒號設定、/ 查詢等都可以使用，具體參考 [CGDB configuration commands](http://cgdb.sourceforge.net/docs/cgdb.html/Configuring-CGDB.html#Configuring-CGDB)
- 使用 o 可以選擇要顯示的檔案源碼，超潮der
- **注意**：
  - 當前代碼行的行數會是粗體，不過不太明顯，用 space 敲敲看就知道在哪一行了
  - 相比 GDB，記得在 CGDB 裏不要 layout 指令！會產生一堆亂碼死掉@@

### 可視化除錯工具

- [DDD](https://www.gnu.org/software/ddd/)
- [XXGDB](http://manpages.ubuntu.com/manpages/hardy/man1/xxgdb.1.html)



# Using STM32 discovery kits with open source tools

- 原 PDF 链接已失效，原文请自行 Google
- Written by STLINK development team

### Installing GNU toolchain

- 请自行 Google

### Installing STLINK

- Dependencies
  - libusb-1.0
  - pkg-config
  - autotools
- Install stlink

```shell
git clone https://github.com/texane/stlink stlink.git
cd stlink.git
./autogen.sh
./configure
make
make install
```

In includes:

- A communication library (stlink.git/libstlink.a).
- A GDB server (stlink.git/st-util).
- A flash manipulation tool (stlink.git/st-flash).

### Using the GDB server

- A GDB server must be started to interact with the STM32. Depending on the discovery kit you are using, you must run one of the 2 commands:
  - `st-util --stlinkv1`: STM32VL discoverykit (onboard ST−link)
  - `st-util`: STM32L or STM32F4 discoverykit (onboard ST−link /V2)
- Then, GDB can be used to interact with the kit:
  - `arm-none-eabi-gdb XXX.elf`
- From GDB, connect to the server using:
  - `(gdb) target extended localhost : 4242`
  - `(gdb) load`
  - `(gdb) continue`

### Building and flashing a program

- If you want to simply flash binary files to arbitrary sections of memory, or read arbitary addresses of memory out to a binary file, use the st-flash tool, as shown below:
  - `st-flash read v1 out.bin 0x8000000 4096`: stlink-v1 command to read 4096 from flash into out.bin
  - `st-flash read out.bin 0x8000000 4096`: stlink-v2 command
  - `st-flash write v1 in.bin 0x8000000`: stlink-v1 command to write the file in.bin into flash
  - `st-flash write in.bin 0x8000000`: stlink-v1 command

### Notes

1. Disassembling THUMB code in GDB
   By default, the disassemble command in GDB operates in ARM mode. The programs running on CORTEX-M3 are compiled in THUMB mode. To correctly disassemble them under GDB, uses an odd address. For instance, if you want to disassemble the code at 0x20000000, use `(gdb) disassemble 0x20000001`

2. 每次都輸入一堆必備的指令很麻煩，可以寫入一個 `gdb.script` 檔案裏，例如：

```
target remote:4242
file stm32f429-example.elf
layout src
```

然後使用 arm-none-eabi-gdb -x gdb.script 進入後，單步執行即可。
- 記得先打開 st-util；
- 個人習慣把arm-none-eabi-gdb -x gdb.script寫進 `Makefile` 裏。



## BOMB

### Phase0, Phase1（比較）

- 在b phase_0設好斷點後，一個c來到phase0函數，在源程式端輸入“help”，則可見：

