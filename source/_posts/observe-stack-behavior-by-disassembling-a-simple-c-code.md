---
title: 小练习：通过反汇编一段C代码观察栈行为
date: 2019-12-11 16:34:21
updated: 2019-12-11 16:34:21
categories:
- 系统与底层
tags:
- embedded system course
- stack
- assembly
- disassemble
- c
- gcc
---

本文的来源请参考[这篇博文](https://blog.joouis.com/2019/12/09/watching-notes-revolution-os/)的前言。

深入了解计算机的底层行为是软件工程师的内功之一，有助于理解程序代码是如何被执行的。本文的练习小而完整，不需要太多背景知识，因此摘选出来与大家分享。

<!-- more -->



## 目的

- 在學習 Linux kernel 分析的基礎課程時有分析到 C 代碼執行時 stack 的行爲，那麼這裏做一個最簡單的實驗，通過把 C 代碼反彙編成 ASM 代碼，來看堆棧的操作情況。
- 備註：
  - 直接用的 **64-bit** 環境，`gcc -S -o main.s main.c`（因爲網路大多都是32bit的分析，照做沒新意）
  - 也可以用 GDB 來看，覺得有點殺雞用牛刀XDD



## 源代碼和編譯出來的代碼

- 源代碼：

![](https://idrgxq.bn.files.1drv.com/y4m55mxQAjM25AHJzomnT_XrvcXvro540HjYgQ9-YWE09y8fx95IvWhoSZEX-fCihgqF2WPT0aP2usavJ-9JVHiK0Df1hpuHSKoc32nIMQhv0EOqu24I6tLSwlGORK0-f5cgy8h9TOug5Ny9BMstlLz-2or9oi5tnWJZ-tK9BjBXIvMKaozi_FdacFAB3OZ7WPZ39lQvXV3OnIu-XP1Tveo-A?width=351&height=441&cropmode=none)

- 反編譯出來的代碼（64-bit）：

![](https://idrfxq.bn.files.1drv.com/y4m-JuD2XCVCaoQUn5lYvubcjojAM7X83VaMjQpRsDvGq0sVUAxDgqKZADgrFo4snyZjgSPvwX-bMDsE-GXhWhOUmnRiW2MyrMjYcssqbV3SVgMf9YAZyWO0p7gTRD26R-yP91fd0qQ6DH_1Q1trPyvE-f40Qjqzf2M_0dS1hL_FOLQazrCK3-AuhxggUb9gkhqNmsVxKdJRQvrYv8uY7_ukQ?width=464&height=720&cropmode=none)



## 分析

- 首先，可以看到一件事是：編譯出來的組合語言是 32-bit 和 64-bit 混合的，應該是針對寄存器的 bit 不同指令做的相應的使用
- 指令從第14行開始也就是 main 函數第一條指令開始執行，首先是把`%rbp`寄存器的值壓入棧內，保存當前工作的棧基指
- 接着把`rsp`的地址賦給`rbp`，含義是初始化一個新的 stack
- 第16行是預設的空間，GCC下`-O1`指令似乎可以優化
- 第17、18行將`a(1,2)`中的兩個參數分別存到`esi`和`edi`寄存器中，這個地方需要說明的是 x86-32 中會使用 push 將實參存入 stack 內，如下圖（原`main.c`反編譯成 32-bit ASM 代碼）：
  - ![](https://idrexq.bn.files.1drv.com/y4m9pIm6sx0UjxAkcwghzHWLAOVrOAVclzqC0meFhZR9_airLB3cMJEPbWgCUtmmUPC0EV-PK3PMllvlkGVXeVzad49Uxj3ne2fiDiy6tnZlKqIaFqBwlH5idcSgefHpeOEzvOrnqmQOf8LRqxmzergtUnSeziVS0S70Lf2mqhEyK2Cqj2IMbw-LyqZgf3ugc1oUUC2GUQgg3nUgMl_KGRJag?width=213&height=54&cropmode=none)
  - 而在 x86-64 中，如果參數在6個以內，GCC 就可以利用寄存器來存儲參數值；超過6個的參數，還是通過上述操作實現，這裏給我們了至少兩個啓發：
    - 儘量使用6個以下的參數列表
    - 傳大的數值時儘量使用指針或引用，因爲寄存器只有64位並且只能存整形數值
- 然後就呼叫 a 函數了，call其實也分爲兩個動作：
  - 第一個是把當前`rip`的值保存
  - 把 a 函數的指令初始地址給`rip`
- a 函數也是同理，先保存當前的`rbp`然後初始化新的 stack
- 第4、5行，就將保存在`esi`和`edi`內的值按順序放入`-24(%ebp)`和`-20(%ebp)`內
- 這可以說明，**在X86-64中，實參入棧和賦值給形參都是在子函數代碼段完成的**
- 第6到8行就是從 stack 中取出參數到通用寄存器完成加法運算
- 第9、10行是算出的值會先從`eax`存到`-4(%rbp)`，即給 tmp 變量賦值
- 然後再將`tmp`的值給`eax`寄存器，函數的返回值默認使用`eax`寄存器返回給上一級函數
- 第11行將`rbp`出棧，然後第12行ret就是把之前保存的`rip`出棧
- 回到第20行指令，將`eax`的值給`-4(%rbp)`，也就是給變量 i
- 爲了`return 0`把0給`eax`
- 因爲是main函數，所以最後是leave（leave: Releases the local stack storage created by the previous ENTER instruction.）




## 流程圖

![](https://idrcxq.bn.files.1drv.com/y4mmFoqM4ecZs3QlssVTjcuH8z6o-e_krbOlw-16ePd5rX0CK_RUA1T-_-OiwFiYzJJGdaRBY17Ho75BE-3SPqpq7IHFceINb4r5LSXGAu2Y3Qhxq0nrnJalDmHKeXIP9qrPYS8tJUxtZMqHpNVSbyy7zgDg7HbfBq5Su1oYFgtSDn7cFjv3LmeT9opN-qF4c4iXS-Rh02InCUeXh-JlhbLtA?width=1502&height=719&cropmode=none)



## Ref

- http://www.searchtb.com/2013/03/x86-64_register_and_function_frame.html
- http://en.wikipedia.org/wiki/X86_instruction_listings