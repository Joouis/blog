---
title: 编译器入门小笔记
date: 2020-01-01 16:08:38
updated: 2020-01-01 16:08:38
categories:
- 系统与底层
tags:
- embedded system course
- embedded system
- gnu
- compiler
- jit
---

这是我的[嵌入式笔记](https://blog.joouis.com/2019/watching-notes-revolution-os/)第五篇，原文写于2015年。

<!-- more -->



## Week#5 课程

- 课程共笔[链接](https://paper.dropbox.com/doc/xsUE7O7d02YlsKydh7Xdp)



### [**Steve Jobs ─ Lost interview**](https://www.youtube.com/watch?v=6CJMQHLfqUQ)[ ](https://www.youtube.com/watch?v=6CJMQHLfqUQ)[**(1995)**](https://www.youtube.com/watch?v=6CJMQHLfqUQ)

- "You know, throughout the years in business, **I found something, which I was always ask why you do things, and the answers you inevitably get are** **'oh** **that's just the way it's done', nobody knows why they do, nobody thinks about things very deeply** in business, that's what I found."
- Feeling: Impressive, attractive, passionate



### GNU Toolchain

![](https://hzoe9g.bn.files.1drv.com/y4m9w--0C21r7BamU_dtV96xC7DR5D_011pshYM7_mvvxydtIW1gSdk6mIwKGBVRU90n6rXEmxK136njHYpC3JotX3i3z1OEUJXf9Deimf-SCg5pLRanOImkriiFigfJSJw5PWhrJpknX3DdUkEBYk5rG0lfCMVvRMg-WZg2WpeOYdbyFgqMyBEOxiv6mUhQWX2KFwMeganzREe9ZE9IPVa6A?width=1161&height=203&cropmode=none)

- Procedure: arguments是呼叫函式傳遞的，parameters是被呼叫函式接收到的

- 組語在procedure call如何傳遞參數？[AAPCS](http://infocenter.arm.com/help/topic/com.arm.doc.ihi0042e/IHI0042E_aapcs.pdf)

  - eabi: embedded [abi](https://zh.wikipedia.org/wiki/应用二进制接口)
  - the sizes, layout, and alignment of data types
  - the calling convention, which controls how functions' arguments are passed and return values retrieved; for example, whether all parameters are passed on the stack or some are passed in registers, which registers are used for which function parameters, and whether the first function parameter passed on the stack is pushed first or last onto the stack
  - how an application should make system calls to the operating system and, if the ABI specifies direct system calls rather than procedure calls to system call stubs, the system call numbers and in the case of a complete operating system ABI, the binary format of object files, program libraries and so on.

![](https://hzod9g.bn.files.1drv.com/y4mcrayOGke8OR_LhPHdSuc2gpBSOYUYRmRxs4bmXMcarA6n3snPGX7L1aytHBOvKgjPfE4S7K5mXUZiLIqdKNj5MhlDMvqJUy8EXP3wM5P0CVL9Mn3xNsragTDMgU4KJyBMepumX42EVPOU9aV5UNCNKsxwBevliFCgezVRyATPruRJ3f-jXo2bDsOqUSmes9kXU611ONE48BcATHpbGMk9A?width=1065&height=749&cropmode=none)

- **Accessing operands**

  通常 procedure 存取 operands 透過以下幾種方式:

  - An argument passed on a register : 直接使用暫存器
  - An argument passed on the stack : 使用 stack pointer (R13) 的相對定址 (immediate offset)
  - A constant : PC-relative addressing
  - A local variable : 分配在 stack 上，透過 stack pointer 相對定址方式存取
  - A global variable : 分配在 static area (就是樓上圖片的 static data)，透過 static base (R9) 相對定址存取

- Target triple
  
  - hf: hard float，預設有FPU
  - armeb-linux-gnueabihf-*: [**Linaro**](http://launchpad.net/gcc-linaro) ARMv7 **big-endian Linux GNU EABI HF**



### Compiler concepts

- [Compiler的多元應用](https://paper.dropbox.com/doc/V5PJzVrvEfJ275Wfj1Ilq)
- [Interpreter, Compiler, JIT from scratch](http://www.slideshare.net/jserv/jit-compiler)
  - Turing completeness: a system of data-manipulation rules can be used to simulate any single-taped [Turing machine](https://en.wikipedia.org/wiki/Turing_machine).
  - Interpreter：本身是一個可執行檔案，工作就是翻譯并執行輸入的程式碼
  - compiler: compile source code to native machine code(ISA, ABI)
    - 實作方式是將原始碼翻譯成backend對應的assembly然後打印出來通過stdout存到一個文件(compiler-XX.c)，再用compiler編譯後執行(Makefile實現)
    - GAS(GNU Assembler) program format(AT&T)

```c
const char *const prologue = 
".text\n" 
".global _main\n" 
"main:\n" 
"    pushq %rbp\n" 
"    movq %rsp, %rbp\n" 
"    pushq %r12\n" // store callee saved register 
"    subq $30008, %rsp\n" // allocate 30,008 B on stack, and realign 
"    leaq (%rsp), %rdi\n" // address of beginning of tape 
"    movl $0, %esi\n" // fill with 0's 
"    movq $30000, %rdx\n" // length 30,000 B 
"    call memset\n" // memset 
"    movq %rsp, %r12";

const char *const epilogue =
"    addq $30008, %rsp\n" // clean up tape from stack.
"    popq %r12\n" // restore callee saved register
"    popq %rbp\n"
"    ret\n";
```

- X64 Calling convention

  - [sys/mman.h](http://pubs.opengroup.org/onlinepubs/007904875/basedefs/sys/mman.h.html): memory management declarations

![](https://hzoa9g.bn.files.1drv.com/y4m8DdmNN2dLqOOOtkmEJYhy3Hd4IedX44C-x46ZwdMYH4SYQVv4otM6PDWfxrUqNynl-Q_PzAgE5-cR-S20pGryphoBCIvNfE7YAlj-XS1qz6DY34Pd6Y4lemP1d-4fIA2umx3YQZILsB8YT8INeyo8GxsnGrnr1nbflK6XIosKfpJyWcdjdw7kLK8VP2-ZoDeTJySTdqn0Ag2M57dYHT47Q?width=709&height=940&cropmode=none)

- [Fractals in C](http://blog.linux.org.tw/~jserv/archives/2011/09/_mandelbrot_set.html)

  - [complex.h](https://zh.wikipedia.org/wiki/Complex.h): 提供了複數運算所需的巨集定義和函式申明

  - cimag: get imaginary part of a complex number

  - creal: get real part of a complex number

  - [How to Plot the Mandelbrot Set By Hand](http://www.wikihow.com/Plot-the-Mandelbrot-Set-By-Hand)
- [Virtual Machine Constructions for Dummies](http://www.slideshare.net/jserv/vm-construct)
- [How A Compiler Works: GNU Toolchain](http://www.slideshare.net/jserv/how-a-compiler-works-gnu-toolchain)



### Homework

#### 作業要求 (B)

- 難度：中

- 詳讀 [Virtual Machine Constructions for Dummies](http://www.slideshare.net/jserv/vm-construct)，改善 Brainf*ck 執行效能

  - 改善 JIT compiler，加入若干 optimization techniques

- 在 GitHub 上 fork [jit-construct](https://github.com/embedded2015/jit-construct)

  - 紀錄若干效能最佳化技巧帶來的提昇

- 建立新的 Hackpad，列在「[+作業區](https://paper.dropbox.com/doc/cSnReWUgyLPcwiJ3uUIyo)」

  - 標注「開發紀錄(B)」



#### Homework note

- 根據[Virtual Machine Constructions for Dummies](http://www.slideshare.net/jserv/vm-construct)P43-46中**預先判斷未來指令-減少instructions**的思路，在switch-case時對未來的指令做判斷，如果符合情況就做優化

  - 挑最簡單的Interpreter入手先@@
  - 以Tower of Hanoi程式為例已先貼原始數據

```
origin version：
real    0m30.031s         Towers of Hanoi in Brainf*ck
user    0m30.024stten by Clifford Wolf <http://www.clifford.at/bfcpu/>
sys     0m0.000s
 
optimal version：
real    0m11.575s         Towers of Hanoi in Brainf*ck
user    0m11.572stten by Clifford Wolf <http://www.clifford.at/bfcpu/>
sys     0m0.000s
```

- P47開始介紹了一些最佳化的技巧，并提供了參考的[source code](https://github.com/matslina/bfoptimization)。首先實作最佳化時，使用了IR(Intermediate language)，先將brainfuck程式轉換成IR程式，再將IR程式轉換成C程式

```python
def bf_to_ir(brainfuck):
    """Translates brainfuck to IR."""
 
    simplemap = {'+': Add(1, 0),
                 '-': Sub(1, 0),
                 '>': Right(1),
                 '<': Left(1),
                 ',': In(0),
                 '.': Out(0),
                 '[': Open(),
                 ']': Close()}
 
    return [simplemap[c] for c in brainfuck if c in simplemap]
    def ir_to_c(ir):
    """Translates IR into a C program."""
 
    plain = {Add: 'mem[p] += %(x)d;',
             Sub: 'mem[p] -= %(x)d;',
             Right: 'p += %(x)d;',
             Left: 'p -= %(x)d;',
             Open: 'while (mem[p]) {',
             Close: '}',
             In: 'mem[p] = getchar();',
             Out: 'putchar(mem[p]);',
             Clear: 'mem[p] = 0;',
             Copy: 'mem[p+%(off)d] += mem[p];',
             Mul: 'mem[p+%(off)d] += mem[p] * %(factor)d;',
             ScanLeft: 'p -= (long)((void *)(mem + p) - memrchr(mem, 0, p+1));',
             ScanRight: 'p += (long)(memchr(mem+p, 0, sizeof(mem)) - (void *)(mem+p));'}
 
    woff = {Add: 'mem[p+%(offset)d] += %(x)d;',
            Sub: 'mem[p+%(offset)d] -= %(x)d;',
            In: 'mem[p+%(offset)d] = getchar();',
            Out: 'putchar(mem[p+%(offset)d]);',
            Clear: 'mem[p+%(offset)d] = 0;'}
 
    code = [(woff if getattr(op, 'offset', 0) else plain)[op.__class__] % op._asdict()
            for op in ir]
    code.insert(0, '\n'.join(['#include <stdio.h>',
                              '#include <string.h>',
                              'unsigned char mem[65536];',
                              'int main() {',
                              'int p=0;']))
    code.append('return 0;')
    code.append('}')
```

- 實作IR的缺點是要轉換兩次，但好處更多：
    1. 提供更為抽象的表示方式
    2. 將前端程式語言和後端的機器語言分離
    3. (額外的)過濾原代碼：可以從bf_to_ir看到它只讀bf的6個指令，如果程式碼包含其他的無效代碼，這裡也會先被過濾掉

- 最佳化的方法提到以下幾種：

  - Contraction

  ![](https://hzoz9g.bn.files.1drv.com/y4mluI1HXU2J4HH1xAyJt9U4HJtzwDfjcuk_HbYQbn3EHmggHAt40q_lHZQBirNYpBKVJwz0TwxmYYDc2cD56w1WEZp4oawBwkUhzfVcevITPfl5V2IcHkZrRBHiPRSiiOHVF_cpNEYKHcDfZ6R_AVxUY-EPqv_4O41mpPGu8CB2TenzmhVX-jlubV3KkD-w9OEu4FH1CcuVoF17f6XQ3jXqw?width=542&height=362&cropmode=none)

  - Clear loops

  ![](https://hzoy9g.bn.files.1drv.com/y4mJBRWtSKkkVclaBVPjeo12mbuZ38ASqqQ2e-tTwOxR3bHTR_OSgD80I3Ou1nPI7mwbwvD9MhFeHsLOleHyg8ogxc8Xa4K4uKYwVC8IQW8LwLstRrB7NCGKXSsKwhtxj0H4CfqW16M8mq6X6nBk6dCsQ9B-B0NRZhrRuzUzE96Dquoth4P8EBK_oYZ-I634lWfezuSgS-Z9MtYOOmdelJUGw?width=552&height=423&cropmode=none)

  - Copy loops

  ![](https://hzox9g.bn.files.1drv.com/y4mgL1CuzWuctIeBpU2_i7UObrQ1lWJHQEXqrjLxEXdvA6ZkrYqNqdvmw-LCGkbtEMs2JdjDAD6HR_em9kwWYYqIb0geIEa-pg8v_g0-J8LF34QsdapAbpBL_jQX__Kcee1cGo7tm_kGrRSZ_BuMLGc-hlC9xwMLejCJVgSJxtAfJp9SYfMpTL31TIS2KG2gutcsds7PEoY4cfmU8ikKiZSQQ?width=547&height=451&cropmode=none)
  
  - Multiplication loop
  
  ![](https://hzow9g.bn.files.1drv.com/y4mhPxgVVq8a4LFWupzsrgxTtAEwjfnMw4MiPh7Bhx4Oe8_eXHTG3GfQ_ZUF2PhKPlvVsZA-03O_O5rUXi7oGFiGsTh8WbgkLYosQecMCpFAQDE5caJI4jpxS-6ZOZegREeq7DbLSKmICPIZ1xBIOw09qHC9L8_hpniJAjiSMzQiVcVEoRIAz81U6oSlMSb9OJa8IccUafNagpKLOTXTM6ayA?width=548&height=447&cropmode=none)
  
  - Operation offsets
  
  ![](https://hzov9g.bn.files.1drv.com/y4mWAkUb7LW648AKSnfKgI1uO9KKza9ytYv0uHe0tEbPemOAXYqg1yuA04b874ubZTbmgR0FpcljQBZydrFx7BpFE0wPsBWz2R3zBU3iiCLhXlR4Ry-4HWeNPyAFFjq28vlpa25namyUDidr0d6a5C7bSVr61bONaECrxAT2nq1GMxjtDBRhjEas1winiutKSfso2Povn8bS3DejM60l9JabQ?width=544&height=465&cropmode=none)
  
  - Scan loops
  
  ![](https://hzoc9g.bn.files.1drv.com/y4m2Jh3C2yYNSXvUTHobz1XgSYJAMRSK_97hoepI7ayow50Adi80cxQr8hBpgbHFobAiJS4yj0NoLe99w2rd2cWR7McgbBveiXL29ordnavc0Rp-MlfNaQldHfRAahBiidmMvlrLq_AQJw8dG1eHpMUA9soz3DJ5usgDckzRlYVZLIDFPBUhylq7P9BKsG0E6MIUIZ3ZgNWyxlbDMxIS1w1ZQ?width=545&height=624&cropmode=none)