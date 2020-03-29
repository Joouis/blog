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

这是我的[嵌入式笔记](https://blog.joouis.com/2019/watching-notes-revolution-os/)第七篇，原文写于2015年。本文除了介绍GDB/CGBD的基础使用方法以及嵌入式开发应用外，还会分享一个解密类经典游戏 `Defusing a binary bomb with GDB` ，非常有趣！

<!-- more -->



## 使用GDB

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



## Using STM32 discovery kits with open source tools

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



## Lab43: Runtime Behavior through defusing a binary bomb

### [Lab43: Runtime Behavior](http://wiki.csie.ncku.edu.tw/embedded/Lab43)

預期目標
- 複習 ARM 處理器和指令集，並深入 stack 和執行時期的行為
- 熟悉 objdump、gdb 等工具，並利用上述工具分析執行檔

### Phase 0, Phase 1: Comparison

- 使用 `b phase_0` 設好斷點後，一個 `c` 來到 `phase0` 函數，在源程式端輸入 `help`，則可見：

  ![](https://g8xeua.bn.files.1drv.com/y4m2NfjgmRex3zQrLCz2ec6NFVRh9RVlYb6UxUVIW5Cp_ERI9Aewx17RtVAc74PA33vZdXPAFsMqMHaKmV2-v_SDvAxa4SEyYr4-WXEyT5QEvDfMBkrjNr70ggOdi7F8irjC_en48kPMwf2HBEiPWzfTYVrCg7LWUmb17hT1ciWMNiR9a9Woxr9HTBiQEVtRbLmfF59Kt7I6Gm9L5d-gahNOg)

- 可以看到程式是用 `r0` 和 `r1` 作比較，`r0` 是我們輸入的值，`r1` 就是我們要猜的值，這裏已經告訴我們是 `help` 了

- 用 `si` 執行到 `0x87ee` ，可以看到 `r0` 和 ` r1` 的值：

  ![](https://g8xgua.bn.files.1drv.com/y4mPoBctTovg6J0yvhXhtffMfAPecqUB0wg5NJ-TYuE67eCPc5pFngsB3YWObYbw43_WCi__xFFjIXMMUfVCTjDQ3LB6WldJeVVdl2GgzHvk4e1NfiWyWz8eGqvKqxMx1Q3DrNTZSsd33h6aalvSdyIOxcohH8GQd7F3Ibi-5khQgNfOWTshN_lITEZAtALGyuedC96KqrLIVX3UkXz3I2kHQ)

- `x/s $r0` `x/s $r1` 直接可以印出 `r0` `r1` 的地址和指向的值：

  ![](https://g8xfua.bn.files.1drv.com/y4m_1q4cb_ll8OFKXEtf82M3NkP2TNsY5Azfo4LAaj05-y7DCWA0U670BtSnsiDXmBxMUSpoGz9DprJ3apqbGyR5MIboLmpWA32V50cpmYdg0310ksBg7LMouIDVvvvAXHuVc3BsB5SIuAVcz8LUgrTyrev5T8nkSifzjYigM5PsfkLBv28c1Ao3kko5shSbA2ExRtj4SUzF0T0lUH-6Frgzg)

- 同理，`continue` 到 `phase_1`，印出 `r1`

  ![](https://g8xlua.bn.files.1drv.com/y4mrKjOr0UFu4U4LD7lJ5pL2tLIW2V6vUrnwlS2dsLfOfuEVWlJzV5pMx7xSbVswnj1Mg-QY5E3QrGgCp3RTtiNHEyDIrtxtUdTYbaEGmazryd_8204Rwn0bXhAtze8hLOyXvsifBbBBylHoGZNUBXjiRiUIBY9JVLCAiw34WfD3j-KKSlnmDNUp1a0m5yZ7bEj0AIrekGdCZt95WOFOS5cbQ)

> **Psalm 23:4**
> Yea, though I walk through the valley of the shadow of death, I will fear no evil; For You are with me; Your rod and Your staff, they comfort me.

### Phase 2: For loop

- 先隨意輸入，看下寄存器 `r1` 的值 `0x8bc8: "%d %d %d %d"`，看起來是要輸入4個整數，重頭再來一次

- 發現用 `ni` 跳過 function 會讀不到指令，還是用 `si` 一步一步來吧

  ![](https://g8xiua.bn.files.1drv.com/y4mqViBPUFsaPQty38x5OBoy_Hs77ynzLJI6UOdjfk4lJmGhtQ4eiaYLFGX5sdKFFwavCyTtx-sZa75RmLRCSyELNwHVzOeoYoWvNhuNkiOwvON5aS47fCuI0LWYpgJNMG-i5AKdiBlsxvqY2mDK3uPWoRPaf5uoIUcNRLIGk0yXFVc-N7RpbfMaktiMDI58yj1MfT_xetol8gQR64f3l4tRg)

- 到 0x885e，如果輸入的數的個數不是4個，就會 explode bomb，若是則跳到 0x8864

- 接下來的代碼中：

  ![](https://g8xhua.bn.files.1drv.com/y4mJJh_4kT5r_QS5Eoq9flUbX7c8Cg0mYiqCXdt7921pMyBN3eDAayILtWmPGZjLEXBty5TADzKfKz7Eh4YTcNxJCO9qq4ENGg_Kni1pOSmrFlLu5IKe2VGUZY6ZqMZQ5N_ypvcEJz-CJiUpdjsROctT39V6tRqWW2fexd78ie9PFxw3SEFH55VstWrwzyYgwyWzeW5Ir2yia0WXOO5rFHFmw)

  - 從 0x886a 到 0x887a 是一個迴圈，完整的 `si` 走了一邊，只是運行完後不能讀到 `r2` `r3` 的值所指向的地址裏的值
  
  - 類似的代碼爲:
  
    ```shell
    a=2;i=0;
    do {
    	a+=1;
    	i++;
    } while (i<=9);
    ```
  
  - 意外但肯定不是巧合地發現，之前輸入的四個數分別存在 `r7+12` `r7+16 `r7+20` `r7+24` 裏（其實使用 `x/s $r7+n` 試出來的）
  - 所以輸入的第二個數（上述代碼中的a）在迴圈後會增加10
  - 而函數最後對比的是 `r7+16` 和 `r7+20`，需要他們相等，故輸入時第三個數減第二個數要等於10
  - 測試果然過關~
  
    ![](https://g8xjua.bn.files.1drv.com/y4mc5EpPCrZB3UjhoesaVAL4k4lRFD6EspybuSYiTw4xBfnC5a4JDC_NgHnhQ7Iqfls1jNxQ2iPxRRgEmStB9oWm0BhhSm9novzSpEsfD0ymcFihk1cGO5gYvvfuE8vmd5bbqYaKHTW4JKMTkbSjFPLyiuf-arPBqnJ1qoz3scVVtNqhgH_4xfE1qLX1zvG1RH0S8LM09fw2f7id47vrCopPQ)

### Phase 3: If condition

- 第一步，設置斷點在 `phase_3` 後隨意輸入進入斷點，印出 `r1` 的值，爲 `"%d %d %d"` ，後面也有判斷輸入個數的代碼，和階段2同理：

  ![](https://gsxjua.bn.files.1drv.com/y4mUqM4HweD1xwIFi5B1A7Bnst_1qzf0FrK2v0xH25U6pZ8P6K76cFWxxHnYTfMwFcYha4mX6Fg-i7FI1qPAdAfx81PntBGquONp82JbgKphAVHnsxDfb9_tCJX3oSmzVFs5wJUgAqY7neLUnLM0YZsWkWprzA-pJSdH_apeFrv1ysTR9qkV8TcnQ5rybRigiyVfh63TJXMzSmKxf2UYVi1ag)

- 接着把斷點設在 0x88be，代碼如下：

  ![](https://gsxkua.bn.files.1drv.com/y4mORvlx72p5_6PDiPtzxSy6__pPrJOV9I_XeiIvDjBtYjCurQUXhRT9cL9aDS5mbRp5QvYZr97fs0nj-iZi2qPHZQjcM-fqoTYnzySfukulgpJjwKwoWkVtf7ccjYof9yes3l7wlgijIsvN9Y0A1Q7Ly9o6-Q1ObfIc99nb1XVTohiBIOZm608wg6p7n8iSsTWyq4SAsPxYh6vjWAHjcdkOw)

- `r3` 會先取出輸入的第一個數，和 40 做對比，這裏我輸入的 1，因此會繼續往下走
- 接着把給輸入的第二個數、第三個數的值賦給 `r2`  `r3`，然後 `r3=r3+r2` ，再把 `r3` 的值存回 `[r7,#16]`
- 然後把 `r2` 的值設爲 `[r7,#16]`，再把 `r3` 的值還原回初始的第三個數的值，對比他們相不相等
  - 即要使得 `r3=r2+r3`，所以在第一個數不爲 40 的情況下，第二個數和第三個數都只能是 0
  - 那如果第一個數等於 40，則 `r2 `r3` 相等即可
- 試驗一下
  - 當第一個數不爲40時，正確

    ![](https://g8xkua.bn.files.1drv.com/y4mNjNKy3ObTQ3OziMqXlvoF7pyUzotNvDaaFyvIY3z_wIogyPhn1NLmB5T4xbph1UQ6Bd9PCPwRQF1onforUpnrNKLiBkYmZvIoXkFYeCPg1hDR2sCv_3P370mUy4ma7qfXu5uuqXixeoKQmiYfQ3cXnndVZ4oeBE9efulUvF2wM-UwjVPSNRgKCQFsVQPctP4SYfxWgwXmW7iUS4D0K091A)
  
  - 當第一個數爲40時，正確
  
    ![](https://g8xmua.bn.files.1drv.com/y4m3Pqfq-DHHyNIvsjgTb7k2BlfrkbLEiwsu3UM3A3iTmqsb0GNqbiOQP_1fgZvrv8KfjwPhNHgXta8PCLrw_19zBsArIwvR_dh_PlcwEoIeo7P1cHSa5TfeBcTz0M9AUBHhEnFbShVas3u4yd6XpNfxTcaMbZdc5p5waVlkUk_gTK_Vyk-qGXfdu5Xzn26_qzgMuTXnJN4ink2FKuq4ZGn9g)
  
### Phase 4: Iteration

- 與階段2、3同理，得知輸入爲一個整數

  ![](https://gsxeua.bn.files.1drv.com/y4ma2y8fO8ZPfARqL6qOz4Fvy00y07cQIIzsEXuwLGcEFMMIJuwh2zqeI_VyBIeel-p_LMbHRmVoGqd0ZuKOg6mdAwBASP1v_YhEn4abeZhkVoENAskxbCV_4WEZMptlxq7VB6R5yVJnkj3h93p8YORHEUxWLiaZoD7o5wE0AA8knib4scjyfZ261UHLugUjSz2wWQ29BNRV_SPZsh1uNZHUw)

- 接下來 `phase_4` 的函數就很短了

  ![](https://g8xnua.bn.files.1drv.com/y4mUvdDoZ8nPQPsuwD8sMK-75pZGwvsiV8sYOTW_qbhWybCYZWFnLgMtiexjznqdNtJzpDqV4B84ajzPPjYzKEkumC24wFXMpJNpgpTak1gylEmJh-dAkWMIAHozf0f3NzJyQx1nOSsPLwajC2Xk0w-q-xFXok0K5kKpdDpLT9vm4TVHuOpGli0Srxe8nEM6Hy3Jd_Cm4w58uyblrkUqeO3Xg)

  把輸入的數，代入fun4後，return的值要等於1024。

- `fun4` 的代碼如下：

  ![](https://gsxiua.bn.files.1drv.com/y4mlxNA_jEBIGKIHZQSqzMTL2a6D2-B8yr-Hhwl39qIZqbA02BYOssQrohJ-8FFgBHwnJotMvff6nTWKUA3Bo-y8cwE92a4CgwNnKujuPR9qqHYZrJLoK2CuEmk6V6cFxXxmcnowE1Sf_d64kZkEJNMsfmYIdhfxXq7aw_PcX31YP8tyVtVR1800adG2tNxzDk7x8ANgNvFD_MN9LVeZgoK_g)

- 當 `r3` 不爲 0 時，會一直減 1 然後調用自身函數；當 `r3` 終於減到 0 後，會在 0x88f2 賦值爲 1，然後跳到 8904，回傳上一層的 `fun` 函數。然後 `r3=r0=1`，開始左移

- 所以相當於輸入的數字 n，可以滿足 2 的 n 次方爲 1024，即是正確答案，真相如下：

  ![](https://gsxhua.bn.files.1drv.com/y4mruackLegsDWFfF6teCuu9RUKNLp1pLHIecfDS5MdIW53DktomQZTUc63fegiDpsIwu46Gj-2u15MvQfPpEJeu9Xy3x2Kbqx3JetweFkeAwW_2AjGgIOTq8FamG56Cw_aQnubjySwTBN6MZxSb_vB66kWfnvaRCbn-E2hyYsTciL8MW1aGxif2yUQSMN6325r40UUBrelZk34NdwAYere7w)

### Phase 5

- 同理，`x/s $r1` 得到 `"%s %d"` 輸入需要是一個字符和一個數字

- 接下來的代碼：

  ![](https://gsxfua.bn.files.1drv.com/y4mcn7nypfliXVNjc8iYFtrA3HxM269gycyXygKaBVRDlVRJZCQPtPV6cpex4OjLIrNIC2uRbHf5xet8SyU3te11VzEbFHdr5SXK0MzGtAHlG0leb-y_2PUjthuhzIFsxzLI5jHKdG4qWABOebzbllMwNInR-txskR8yb-7anjYbL2ijEW40JcGGtgSzmkn6JdkhbPdyr8dqCSYSNDnNCBWCQ)

  - `si` 往下走但是寄存器都沒有改變，證明 0x106e6-0x106ea 都沒有執行
  - ` r3` 爲輸入的字符，要讓 `r3` 的 ASCII 碼等於 120，然後循環 3次使 `r1` 的值加到 3，所以輸入的第一個字符是 x
  - 然後會把 `r2` 的第二位做判斷，所以還需要有字符，不然 `r1` 就只加了一次。跳回 `r3` 與 120 做判斷，是爲了讓 `r1` 再加。因此第一個字符串的答案是“xxx”。

- 再接下來的代碼：

  ![](https://gsxgua.bn.files.1drv.com/y4m-ckOvbP8DY62HbEywdl3moHXPM-hzfokte26XpzqAhngmhlHnBWFW7p6_fNyEj_AFn425x0smmj_4kbsafQ3DnKI9h_G4BzqemuTiZ3qPlpydxvnWRFo2WKSFuutv7yL4XGHRIEwhUmHPQSoV30rURWTzLwDxTe87lax4IdUk3oJE8B6T-6XVNrfiun_16kJ95xbqT4RoGzVXFmepMH-pw)

  - SMULL：*SMULL* *RdLo*, *RdHi*, *Rm*, *Rs*
  - The SMULL instruction interprets the values from *Rm* and *Rs* as two’s complement signed integers. It multiplies these integers and places the least significant 32 bits of the result in *RdLo*, and the most significant 32 bits of the result in *RdHi*.
  - **movw** followed by a **movt** is a common way to load a 32-bit value into a register. It's the equivalent of OR-ing those two immediate values together, with the movt being the upper 16-bit.
  - **ASR**(Arithmetic shift right) by *n* bits moves the left-hand 32-*n* bits of the register *Rm*, to the right by *n* places, into the right-hand 32-*n* bits of the result, and it copies the original bit[31] of the register into the left-hand *n* bits of the result. See [Figure 3.1](http://infocenter.arm.com/help/topic/com.arm.doc.dui0497a/CIHDDCIF.html#CIHFBGJH).
  - cmpls: ls--->lower or same

