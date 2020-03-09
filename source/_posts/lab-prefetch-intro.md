---
title: Prefetcher小练习
date: 2020-01-06 18:20:02
updated: 2020-01-06 18:20:02
categories:
- 系统与底层
tags:
- embedded system course
- prefetch
- avx
- sse
- streaming SIMD extensions
- instruction set
---

这是我的[嵌入式笔记](https://blog.joouis.com/2019/watching-notes-revolution-os/)第六篇，原文写于2015年。

<!-- more -->



# **作業要求** **(B)**

- 閱讀 Week #8 [效能分析: Prefetching](https://embedded2015.hackpad.com/ep/pad/static/7ZSsa98cSKw) 提到的論文: “[When](http://www.cc.gatech.edu/~hyesoon/lee_taco12.pdf)[ Prefetching Works, When It Doesn’t, and Why](http://www.cc.gatech.edu/~hyesoon/lee_taco12.pdf)”，在 Linux/x86_64 (注意，要用 64-bit 系統，不能透過虛擬機器執行) 上編譯並執行 [prefetcher](https://github.com/embedded2015/prefetcher)([source](https://www.facebook.com/notes/champ-yen/the-power-of-prefetching/1214833498543546))
  - 說明 naive_transpose, sse_transpose, sse_prefetch_transpose 之間的效能差異，以及 prefetcher 對 cache 的影響
- 在 github 上 fork [prefetcher](https://github.com/embedded2015/prefetcher)，嘗試用 AVX 進一步提昇效能
  - 修改 Makefile，產生新的執行檔，分別對應於 naive_transpose, sse_transpose, sse_prefetch_transpose (學習 Homework #2 的做法)
  - 用 perf 分析 cache miss/hit
  - 參考 [Performance of SSE and AVX Instruction Sets](http://arxiv.org/pdf/1211.0820.pdf)，用 SSE/AVX intrinsic 來改寫程式碼
  - 詳細描述實驗設計，以及你的觀察
- 建立新的 Hackpad，列於「[+作業區](https://paper.dropbox.com/doc/k9zCX0UmNzKYIPtcXMlnD)」，需要標注「開發紀錄 (B)」



# Learn prefetcher

- 先分析一下原始碼，首先是naive_transpose，其實現最簡單的矩陣轉置想法，從矩陣的左上角第一個元素開始，把舊矩陣中的元素按轉置後的順序存入新的矩陣中

```c
void naive_transpose(int *src, int *dst, int w, int h)
{
    for(int x = 0; x < w; x++){
        for(int y = 0; y < h; y++){
             *(dst + x*h + y) = *(src + y*w + x);
        }
    }
}
```

- 接著sse_prefetch_transpose，使用了Intel處理器SIMD的技術，在[+Week#1](https://paper.dropbox.com/doc/qrEPXoXETqtGUHyPua3tl)有做簡單的整理，簡單說就是一次將4筆資料放入sse暫存器中，執行一條指令就可以完成4筆資料處理。
  具體實現的過程參考了文章[Programming trivia: 4x4 integer matrix transpose in SSE2](https://randombit.net/bitbashing/posts/integer_matrix_transpose_in_sse2.html)。
  - SSE指令的格式：
    ![](https://idoc9g.bn.files.1drv.com/y4m9bN43kHK5gLhfzVqwQeWXHEafixrMU2j8LWuwyHH51gxPNE69jrdp2XJTuwigzllW5jiCb3s1hWj637DFbmBgq_UkKgIHI20cUGonOwWF3e0wKuWVjrWqLxuCNL0ol0u4jkeFgyPYo86njZir6KJWqlPHW9pLM-crLDuZbyxkyL02Y3rjqa9HCVBKXmJFuksBe08HFC1QdgdchSYmVtCxA?width=563&height=253&cropmode=none)
  - [_mm_unpacklo_epi32(I0, I1)](https://msdn.microsoft.com/en-us/library/x8atst9d(v=vs.90).aspx)讀入兩個128位暫存器後會使用他們的2個低32位值，返回[a0, b0, a1, b1]，_mm_unpackhi_epi32同理，而_mm_unpacklo_epi64則是一次取64位
- 這種方法的效能改進在：
  - 一條指令處理4筆數據，要比4筆數據4條指令處理快
  - loop unrolling：
    - 執行loop循環的組合語言代碼執行次數會變少
    - branch prediction miss機率降低
    - Wikipedia還提到如果數據沒有相依性有機會使用並行處理，在這裡SIMD已經實現

```c
void sse_transpose(int *src, int *dst, int w, int h)
{
    for(int x = 0; x < w; x+=4){
        for(int y = 0; y < h; y+=4){
            __m128i I0 = _mm_loadu_si128 ((__m128i*)(src+y*w+x));
            __m128i I1 = _mm_loadu_si128 ((__m128i*)(src+(y+1)*w+x));
            __m128i I2 = _mm_loadu_si128 ((__m128i*)(src+(y+2)*w+x));
            __m128i I3 = _mm_loadu_si128 ((__m128i*)(src+(y+3)*w+x));
            __m128i T0 = _mm_unpacklo_epi32(I0, I1);
            __m128i T1 = _mm_unpacklo_epi32(I2, I3);
            __m128i T2 = _mm_unpackhi_epi32(I0, I1);
            __m128i T3 = _mm_unpackhi_epi32(I2, I3);
            I0 = _mm_unpacklo_epi64(T0, T1);
            I1 = _mm_unpackhi_epi64(T0, T1);
            I2 = _mm_unpacklo_epi64(T2, T3);
            I3 = _mm_unpackhi_epi64(T2, T3);
            _mm_storeu_si128((__m128i*)(dst+(x*h)+y), I0);
            _mm_storeu_si128((__m128i*)(dst+((x+1)*h)+y), I1);
            _mm_storeu_si128((__m128i*)(dst+((x+2)*h)+y), I2);
            _mm_storeu_si128((__m128i*)(dst+((x+3)*h)+y), I3);
        }
    }
}
```

- 最後是sse_prefetch_transpose，相比sse_transpose多使用了4次[_mm_prefetch](https://msdn.microsoft.com/en-us/library/84szxsww(v=vs.90).aspx)指令
  - `void _mm_prefetch(char * p , int i )` 會將地址p的數據加載到cache的一條cache line，int i有_MM_HINT_T0, _MM_HINT_T1, _MM_HINT_T2和_MM_HINT_NTA共4種，表示了不同的prefetch方式：
    - T0 - T2對應了L1 - L3 caches，NTA表示加載數據在L1 cache并標記為首先被替換的
    - 實際將T1替換為T0和T2運行程式，運行的時間區別不大，可能是測量的方式不精准
-  為什麼PFDIST要設為8？實際運行結果是PFDIST＝4比PFDIST＝8平均慢10000us左右

```c
void sse_prefetch_transpose(int *src, int *dst, int w, int h)
{
    for(int x = 0; x < w; x+=4){
        for(int y = 0; y < h; y+=4){
#define PFDIST  8
 
            _mm_prefetch(src+(y+PFDIST)*w+x, _MM_HINT_T1);
            _mm_prefetch(src+(y+PFDIST+1)*w+x, _MM_HINT_T1);
            _mm_prefetch(src+(y+PFDIST+2)*w+x, _MM_HINT_T1);
            _mm_prefetch(src+(y+PFDIST+3)*w+x, _MM_HINT_T1);
 
            __m128i I0 = _mm_loadu_si128 ((__m128i*)(src+y*w+x));
            __m128i I1 = _mm_loadu_si128 ((__m128i*)(src+(y+1)*w+x));
            __m128i I2 = _mm_loadu_si128 ((__m128i*)(src+(y+2)*w+x));
            __m128i I3 = _mm_loadu_si128 ((__m128i*)(src+(y+3)*w+x));
            __m128i T0 = _mm_unpacklo_epi32(I0, I1);
            __m128i T1 = _mm_unpacklo_epi32(I2, I3);
            __m128i T2 = _mm_unpackhi_epi32(I0, I1);
            __m128i T3 = _mm_unpackhi_epi32(I2, I3);
            I0 = _mm_unpacklo_epi64(T0, T1);
            I1 = _mm_unpackhi_epi64(T0, T1);
            I2 = _mm_unpacklo_epi64(T2, T3);
            I3 = _mm_unpackhi_epi64(T2, T3);
            _mm_storeu_si128((__m128i*)(dst+(x*h)+y), I0);
            _mm_storeu_si128((__m128i*)(dst+((x+1)*h)+y), I1);
            _mm_storeu_si128((__m128i*)(dst+((x+2)*h)+y), I2);
            _mm_storeu_si128((__m128i*)(dst+((x+3)*h)+y), I3);
        }
    }
}
```



# Reproduce Prefetcher by using AVX

#### 修改Makefile執行檔

- 在這裡使用了gcc -D來定義了兩個宏，一個"\"$@.h\""用在`#include`時可以找到對應的頭文件，另一個"\"$@\""是在printf時輸出對應的版本名稱

```makefile
CFLAGS = -msse2 --std gnu99 -O0 -Wall
EXEC = naive_transpose sse_transpose sse_prefetch_transpose
all: $(EXEC) format
SRCS_common = main.c
naive_transpose:
        $(CC) $(CFLAGS) -DIMPL="\"$@.h\"" -DSTR="\"$@\"" -o $@ $(SRCS_common) $@.c
sse_transpose:
        $(CC) $(CFLAGS) -DIMPL="\"$@.h\"" -DSTR="\"$@\"" -o $@ $(SRCS_common) $@.c
sse_prefetch_transpose:
        $(CC) $(CFLAGS) -DIMPL="\"$@.h\"" -DSTR="\"$@\"" -o $@ $(SRCS_common) $@.c
clean:
        $(RM) $(EXEC) perf.*
format:
        astyle --style=kr --indent=spaces=4 --indent-switches --suffix=none *.[ch]
run: $(EXEC)
        ./naive_transpose
        ./sse_transpose
        ./sse_prefetch_transpose
        make clean
```

#### 使用Perf分析cache miss/hit

- 在運行Perf之前將main.c中的test part和打印全部刪掉，排除額外的程式碼以增加cache測量的準確率
- `echo "echo 1 > /proc/sys/vm/drop_caches" | sudo sh`
  `perf stat -r 100 -e cache-misses,cache-references,L1-dcache-load-misses,L1-dcache-store-misses,L1-dcache-prefetch-misses,L1-icache-load-misses ./phonebook_orig`

```shell
Performance counter stats for './naive_transpose' (100 runs):
          16857556      cache-misses     # 93.608 % of all cache refs      ( +-  0.06% )
          18008720      cache-references           ( +-  0.01% )
          21069717      L1-dcache-load-misses      ( +-  0.00% )
           4255304      L1-dcache-store-misses     ( +-  0.00% )
                 0      L1-dcache-prefetch-misses                                   
             24305      L1-icache-load-misses      ( +-  0.46% )
       0.352085215 seconds time elapsed            ( +-  0.04% )
 
 Performance counter stats for './sse_transpose' (100 runs):
           4334609      cache-misses              #   79.424 % of all cache refs      ( +-  0.03% )
           5457532      cache-references                                              ( +-  0.02% )
           8516824      L1-dcache-load-misses                                         ( +-  0.01% )
           4292592      L1-dcache-store-misses                                        ( +-  0.03% )
                 0      L1-dcache-prefetch-misses                                   
             27101      L1-icache-load-misses                                         ( +-  0.37% )
       0.242525082 seconds time elapsed                                          ( +-  0.04% )
 
 Performance counter stats for './sse_prefetch_transpose' (100 runs):
           4346864      cache-misses              #   79.615 % of all cache refs      ( +-  0.03% )
           5459859      cache-references                                              ( +-  0.02% )
           8533348      L1-dcache-load-misses                                         ( +-  0.02% )
           4308308      L1-dcache-store-misses                                        ( +-  0.05% )
                 0      L1-dcache-prefetch-misses                                   
             24590      L1-icache-load-misses                                         ( +-  0.29% )
       0.184224127 seconds time elapsed                                          ( +-  0.08% )
```

#### 參考 [Performance of SSE and AVX Instruction Sets](http://arxiv.org/pdf/1211.0820.pdf)，用 SSE/AVX intrinsic 來改寫程式碼

- 兩份參考文件：
  - [Introduction to Intel® Advanced Vector Extensions](https://software.intel.com/sites/default/files/m/d/4/1/d/8/Intro_to_Intel_AVX.pdf) 介紹了基本的AVX指令以及使用範例
  - [Intel® Architecture Instruction Set Extensions Programming Reference](https://software.intel.com/sites/default/files/managed/07/b7/319433-023.pdf)，2015/08最新的文件，介紹了AVX-512指令集，目前支援的CPU僅有Xeon Phi Knights Landing, Xeon Skylake, Cannonlake
  - 關於avx的intrinsic函式指令調用[Intrinsics for Intel® Advanced Vector Extensions](https://software.intel.com/zh-cn/node/582950)

```c
for ( int x = 0; x < w; x += 8 ) {
    for ( int y = 0; y < h; y += 8 ) {
        __m256i I0 = _mm256_loadu_si256((__m256i *)(src + (y + 0) * w + x));
        __m256i I1 = _mm256_loadu_si256((__m256i *)(src + (y + 1) * w + x));
        __m256i I2 = _mm256_loadu_si256((__m256i *)(src + (y + 2) * w + x));
        __m256i I3 = _mm256_loadu_si256((__m256i *)(src + (y + 3) * w + x));
        __m256i I4 = _mm256_loadu_si256((__m256i *)(src + (y + 4) * w + x));
        __m256i I5 = _mm256_loadu_si256((__m256i *)(src + (y + 5) * w + x));
        __m256i I6 = _mm256_loadu_si256((__m256i *)(src + (y + 6) * w + x));
        __m256i I7 = _mm256_loadu_si256((__m256i *)(src + (y + 7) * w + x));

        __m256i T0 = _mm256_unpacklo_epi32(I0, I1);
        __m256i T1 = _mm256_unpackhi_epi32(I0, I1);
        __m256i T2 = _mm256_unpacklo_epi32(I2, I3);
        __m256i T3 = _mm256_unpackhi_epi32(I2, I3);
        __m256i T4 = _mm256_unpacklo_epi32(I4, I5);
        __m256i T5 = _mm256_unpackhi_epi32(I4, I5);
        __m256i T6 = _mm256_unpacklo_epi32(I6, I7);
        __m256i T7 = _mm256_unpackhi_epi32(I6, I7);

        I0 = _mm256_unpacklo_epi64(T0, T2);
        I1 = _mm256_unpackhi_epi64(T0, T2);
        I2 = _mm256_unpacklo_epi64(T1, T3);
        I3 = _mm256_unpackhi_epi64(T1, T3);
        I4 = _mm256_unpacklo_epi64(T4, T6);
        I5 = _mm256_unpackhi_epi64(T4, T6);
        I6 = _mm256_unpacklo_epi64(T5, T7);
        I7 = _mm256_unpackhi_epi64(T5, T7);

        T0 = _mm256_permute2x128_si256(I0, I4, 0x20);
        T1 = _mm256_permute2x128_si256(I1, I5, 0x20);
        T2 = _mm256_permute2x128_si256(I2, I6, 0x20);
        T3 = _mm256_permute2x128_si256(I3, I7, 0x20);
        T4 = _mm256_permute2x128_si256(I0, I4, 0x31);
        T5 = _mm256_permute2x128_si256(I1, I5, 0x31);
        T6 = _mm256_permute2x128_si256(I2, I6, 0x31);
        T7 = _mm256_permute2x128_si256(I3, I7, 0x31);

        _mm256_storeu_si256((__m256i *)(dst + ((x + 0) * h) + y), T0);
        _mm256_storeu_si256((__m256i *)(dst + ((x + 1) * h) + y), T1);
        _mm256_storeu_si256((__m256i *)(dst + ((x + 2) * h) + y), T2);
        _mm256_storeu_si256((__m256i *)(dst + ((x + 3) * h) + y), T3);
        _mm256_storeu_si256((__m256i *)(dst + ((x + 4) * h) + y), T4);
        _mm256_storeu_si256((__m256i *)(dst + ((x + 5) * h) + y), T5);
        _mm256_storeu_si256((__m256i *)(dst + ((x + 6) * h) + y), T6);
        _mm256_storeu_si256((__m256i *)(dst + ((x + 7) * h) + y), T7);
    }
}
```

- AVX指令集是256-bit，所以這裡一次處理8個byte，loop一次加8
- 首先將依次將8行數組的元素載入到暫存器中
- `_mm256_unpacklo/hi_epi32`函式讀入兩個256-bit的數，將低/高128-bit以32-bit為單位交錯排列，舉例：
  `__m256i A = [ A0, A1, A2, A3, A4, A5, A6, A7 ];`
  `__m256i B = [ B0, B1, B2, B3, B4, B5, B6, B7 ];`
  `__m256i C = _mm256_unpacklo_epi32(I0, I1) = [ A0, B0, A1, B1, A2, B2, A3, B3 ];`
- `_mm256_unpacklo_epi64`同理
- 以下對比了sse、avx共4個版本

```shell
Performance counter stats for './sse_transpose' (100 runs):
           4329716      cache-misses      #   79.312 % of all cache refs ( +-  0.03% )
           5459067      cache-references    ( +-  0.02% )
           8514863      L1-dcache-load-misses    ( +-  0.01% )
           4290179      L1-dcache-store-misses    ( +-  0.03% )
                 0      L1-dcache-prefetch-misses       
             26387      L1-icache-load-misses     ( +-  0.46% )
       0.242315241 seconds time elapsed        ( +-  0.04% )
 
 Performance counter stats for './sse_prefetch_transpose' (100 runs):
           4345707      cache-misses    #   79.577 % of all cache refs  ( +-  0.03% )
           5460983      cache-references    ( +-  0.02% )
           8532546      L1-dcache-load-misses      ( +-  0.02% )
           4307174      L1-dcache-store-misses    ( +-  0.04% )
                 0      L1-dcache-prefetch-misses                                   
             24769      L1-icache-load-misses     ( +-  0.46% )
       0.184028053 seconds time elapsed        ( +-  0.06% )
 
 Performance counter stats for './avx_transpose' (100 runs):
           3305343      cache-misses   #   74.670 % of all cache refs  ( +-  0.02% )
           4426624      cache-references     ( +-  0.01% )
           8043457      L1-dcache-load-misses    ( +-  0.02% )
           4859334      L1-dcache-store-misses   ( +-  0.02% )
                 0      L1-dcache-prefetch-misses  
             25696      L1-icache-load-misses      ( +-  0.52% )
       0.188154465 seconds time elapsed     ( +-  0.02% )
 
 Performance counter stats for './avx_prefetch_transpose' (100 runs):
           3320332      cache-misses  #   51.121 % of all cache refs ( +-  0.02% )
           6495085      cache-references       ( +-  0.02% )
           8076035      L1-dcache-load-misses    ( +-  0.02% )
           4886925      L1-dcache-store-misses    ( +-  0.02% )
                 0      L1-dcache-prefetch-misses    
             26493      L1-icache-load-misses     ( +-  0.35% )
       0.187412805 seconds time elapsed        ( +-  0.05% )
```

- 從cache-miss來看avx版本有25%左右的提升，從執行時間上單純avx版比sse版有明顯提升，但輸sse_prefetch；而avx的prefetch版與avx原版幾乎無差別，還輸sse_prefetch
- 關於prefetch版，**PFDIST**取了8/16/32/64，目前暫取16。看來需要去[When Prefetching Works, When It Doesn’t, and Why](http://www.cc.gatech.edu/~hyesoon/lee_taco12.pdf)找答案了



# **Reference**

- [When Prefetching Works, When It Doesn’t, and Why](http://www.cc.gatech.edu/~hyesoon/lee_taco12.pdf)
- [Performance of SSE and AVX Instruction Sets](http://arxiv.org/pdf/1211.0820.pdf)
- [gcc](http://www.rapidtables.com/code/linux/gcc/gcc-d.htm)[ ](http://www.rapidtables.com/code/linux/gcc/gcc-d.htm)[-D](http://www.rapidtables.com/code/linux/gcc/gcc-d.htm)
- [Introduction to Intel® Advanced Vector Extensions](https://software.intel.com/sites/default/files/m/d/4/1/d/8/Intro_to_Intel_AVX.pdf)
- [Intel® Architecture Instruction Set Extensions Programming Reference](https://software.intel.com/sites/default/files/managed/07/b7/319433-023.pdf)
- [Crunching Numbers with AVX and AVX2](http://www.codeproject.com/Articles/874396/Crunching-Numbers-with-AVX-and-AVX)