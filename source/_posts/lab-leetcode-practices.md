---
title: LeetCode小练习
date: 2020-01-24 16:53:32
updated: 2020-01-24 16:53:32
categories:
- 系统与底层
tags:
- embedded system course
- leetcode
- cache
- recursion
- iteration
- performance
---

这是我的[嵌入式笔记](https://blog.joouis.com/2019/watching-notes-revolution-os/)第六篇，原文写于2015年。

几乎所有候选人都问过我，进微软是不是要刷题，而我总是不厌其烦地告诉他们，我没刷题就进来了（在阿里疯狂996，享不尽福报，哪里还有时间刷题😅）。

嵌入式课程的LeetCode作业应该是我到目前为止仅有的“刷题”经验。Jserv的本意也不是让学生刷题型，而是通过几道算法题对比递归和迭代结构关于程序的性能差异，此外每题要求写出递归和迭代版本对思维能力的提升也是显而易见的。在此与大家分享:)

<!-- more -->



## [2015q3 Homework #3](http://wiki.csie.ncku.edu.tw/embedded/2015q3h3)

-  實做 [Week #2 程式題目](http://people.debian.org.tw/~jserv/ncku/2015q3.pdf)，解釋其運作原理（至少 3 題，限定 [C99](https://en.wikipedia.org/wiki/C99) 或以上的規格）
  - Question #2, #3, #4, #5, #6, #27
- 在 GitHub 上 fork [quiz](https://github.com/embedded2015/quiz)，然後逐一修改每個目錄裡面的檔案
  - 對於 Question #2, #3, … #6 都需要實做**遞迴**和**非遞迴**的版本
  - 要一併準備測試資料
- 除了修改程式，也要編輯 Hackpad 下方「[+作業區](https://paper.dropbox.com/doc/RrcZhOoThosgCSGRKwFTM)」，增添開發紀錄和 GitHub 連結
  - 額外要求觀賞電影《[進擊的鼓手](https://zh.wikipedia.org/zh-tw/爆裂鼓手)》，思考這 4 週以來，課程給你的衝擊（若你沒衝擊的話，可以退選了），在自己的 Hackpad 紀錄心得，特別是對於追求卓越這件事
- 應該要有完整的測試程式，並測試各項邊界狀況
  - 執行時間分析
  - 記憶體需求分析
  - cache miss 分析
  - 時間複雜度分析
- 善用 assert
- 提供遞迴與非遞迴的版本
- 截止日期：Oct 17, 2015（含）之前
- remember to use:
```bash
astyle --style=kr --indent=spaces=4 --indent-switches --suffix=none *.[ch] 
```



## 關於Drop Cache

- 之前一直都是用`echo "echo 1 > /proc/sys/vm/drop_caches" | sudo sh`來清理cache
- 剛才做quiz#4時突然想到好像echo的數字可以有`1`、`2`、`3`，於是乎都試了一下。驚奇地發現用`3`所耗費的時間比用`1`耗費的時間要多出25%：

```bash
echo "echo 3 > /proc/sys/vm/drop_caches" | sudo sh
perf stat -r 100 -e cache-misses,cache-references,L1-dcache-load-misses,L1-dcache-store-misses,L1-dcache-prefetch-misses,L1-icache-load-misses ./recursive
Performance counter stats for './recursive' (100 runs):
    44    cache-misses        #   0.490 % of all cache refs    ( +-  8.59% )
    0.000430226 seconds time elapsed                      ( +- 33.97% )
    
echo "echo 1 > /proc/sys/vm/drop_caches" | sudo sh
perf stat -r 100 -e cache-misses,cache-references,L1-dcache-load-misses,L1-dcache-store-misses,L1-dcache-prefetch-misses,L1-icache-load-misses ./recursive
Performance counter stats for './recursive' (100 runs):
    39    cache-misses        #   0.447 % of all cache refs    ( +-  9.38% )
    0.000327662 seconds time elapsed                      ( +-  3.46% )
```

- 於是查了一下：
  - `echo 1 > /proc/sys/vm/drop_caches`：To free pagecache
  - `echo 2 > /proc/sys/vm/drop_caches`：To free dentries and inodes
  - `echo 3 > /proc/sys/vm/drop_caches`：To free pagecache, dentries and inodes
  - 關於上述三個名詞的[解釋](http://stackoverflow.com/questions/29870068/what-are-pagecache-dentries-inodes)

- 所以用3多清除了dentries和inodes，相比1多花費的時間，就是讀當前目錄找到該執行檔所花的時間




## #2

**Question：**

Given a sorted character array and a character, return the smallest character that is strictly larger than the search character. If no such character exists, return the smallest character in the array. For example:

```
Input: ['c', 'f', 'j', 'p', 'v'], 'a'   
Output: 'c'   
 
Input: ['c', 'f', 'j', 'p', 'v'], 'c'   
Output: 'f'   
 
Input: ['c', 'f', 'j', 'p', 'v'], 'z'   
Output: 'c'   
 
Input: ['c', 'c', 'k'], 'f'   
Output: 'k' 
```

請寫出合法的 C 語言程式 char smallest_character(char str[], char c) { … }



### Recursive

- 因為需要將array里的元素由小到大順序和作比較特定字符：
  - 單純的size沒有狀態
  - size-1可從array中較大的元素遍歷到較小的，但不是從小到大
  - 設一個local tmp[size-1]每次從array取[1..size-1]，然後向下傳，但這樣會丟失了原始array的數據，若array里沒有元素大於比較的字符，則無法取得array[0]

```c
#define uchar unsigned char
uchar smallest_char(uchar str[], uchar c, uchar size, uchar count)
{
  if ( 1 == size || 0 == count) {
    return str[0];
  } else if ( str[size - count] > c ) {
    return str[size - count];
  } else {
    count--;
    return smallest_char(str, c, size, count);
  }
}
```

### Iterative

- 對array進行遍歷，如果比字符大就返回，如果沒有就返回array[0]
- smallest_char()函式里多加一個參數size，因為array傳給子函式傳的是pointer而不是完整的array，[參考](http://stackoverflow.com/questions/9413046/how-to-get-the-length-of-array-in-c-is-sizeof-is-one-of-the-solution)

```c
#define uchar unsigned char
uchar smallest_char(uchar str[], uchar c, uchar size)
{
  uchar i;
  for ( i = 0; i < size; i++ ) {
    if ( str[i] > c ) {
    return str[i];
    }
  }
  return str[0];
} 
```

### Performance

- 想使用perf kmem觀察slab行為時發生錯誤：`invalid or unsupported event: 'kmem:kmalloc'`
- 使用perf測量cache misses和time elapsed: 
  `echo "echo 1 > /proc/sys/vm/drop_caches" | sudo sh` 
  `perf stat -r 100 -e cache-misses,cache-references,L1-dcache-load-misses,L1-dcache-store-misses,L1-dcache-prefetch-misses,L1-icache-load-misses ./iterative`：


```
Performance counter stats for './iterative' (100 runs):
  66    cache-misses        #   0.753 % of all cache refs    ( +-  9.27% )
  8810    cache-references                        ( +-  0.62% )
  9600    L1-dcache-load-misses                     ( +-  0.19% )
  4203    L1-dcache-store-misses                     ( +-  0.23% )
  0    L1-dcache-prefetch-misses
  3714    L1-icache-load-misses                     ( +-  0.77% )
  0.000354619 seconds time elapsed                      ( +-  3.88% )
  
Performance counter stats for './recursive' (100 runs):
  66    cache-misses        #   0.768 % of all cache refs    ( +-  8.41% )
  8627    cache-references                        ( +-  0.63% )
  9597    L1-dcache-load-misses                     ( +-  0.24% )
  4227    L1-dcache-store-misses                     ( +-  0.23% )
  0    L1-dcache-prefetch-misses
  3573    L1-icache-load-misses                     ( +-  0.76% )
  0.000402208 seconds time elapsed                      ( +- 32.71% )
 
```

| Quiz#2       | recursive   | iterative   |
| ------------ | ----------- | ----------- |
| 執行時間     | 0.000402208 | 0.000354619 |
| cache-misses | 66          | 66          |
| 時間複雜度   | O(n)        | O(n)        |

 

## #3

### Question

![img](https://itob9g.bn.files.1drv.com/y4mVd0fYruo-LRS3d2fy4LXEAbbUTRy9PhdBjCgfdnF4SfZjDJglUh_PLO5scg1th8jtlkYKUTXATy16FijZWD7GP1Bins39euZa2PnHWXmf4Tr_dAiRA-XrzIOPNj5L5R1ZRyelwS0WR2hUxXixPJAGGdTsSxuUN1zJtgZV_7WUstKWf1mrC9XH23FYAWQVVTIFGVxO87e01SX_s4O21pE4g?width=971&height=698&cropmode=none)

### Recursive

- 先對右結點處理，再對左結點處理
  - 對於右結點，把他們push進stack
  - 對於左結點把他變成父節點的右結點，然後如果stack里有結點則pop出來接上

```c
Node *flatten(Node *node, stack *S)
{
  if ( !node ) {
    return NULL;
  }
  node->right = flatten(node->right, S);
  if ( node->right  ) {
    stack_push(S, (int)node->right);
    node->right = NULL;
  }
  node->left = flatten(node->left, S);
  if ( node->left  ) {
    node->right = node->left;
    node->left = NULL;
    Node *tmp = (Node *)malloc(sizeof(Node));
    tmp = node;
    while(S->size) {
      node = node->right;
      node->right = (Node *)stack_pop(S);
      if ( !S->size  ) {
        return tmp;
      }
    }
  }
  return node;
}
```

### Iterative

- 自上而下的，把右結點push到stack，然後把左結點變為右結點，然後移動到該結點；直到右結點也沒有時，把stack的結點依次pop出來接上

```c
void flatten(Node *node)
{
  stack *S = (stack *)malloc(sizeof(stack));
  stack_init(S);
  while ( node || S->size ) {
    if ( node->right ) {
      stack_push(S, (int)node->right);
    }
    if ( node->left ) {
      node->right = node->left;
      node->left = NULL;
    } else if ( S->size ) {
      node->right = (Node *)stack_pop(S);
    }
    node = node->right;
  }
} 
```

### Performance

- `echo "echo 1 > /proc/sys/vm/drop_caches" | sudo sh `
- `perf stat -r 100 -e cache-misses,cache-references,L1-dcache-load-misses,L1-dcache-store-misses,L1-dcache-prefetch-misses,L1-icache-load-misses ./iterative`：

```bash
 Performance counter stats for './iterative' (100 runs):
        65    cache-misses        #   0.672 % of all cache refs    ( +- 11.50% )
       9735    cache-references                        ( +-  0.53% )
       11348    L1-dcache-load-misses                     ( +-  0.17% )
       4601    L1-dcache-store-misses                     ( +-  0.21% )
         0    L1-dcache-prefetch-misses
       4014    L1-icache-load-misses                     ( +-  0.63% )
 
    0.000489711 seconds time elapsed                      ( +- 37.21% )
  
 Performance counter stats for './recursive' (100 runs):
        65    cache-misses        #   0.672 % of all cache refs    ( +- 10.02% )
       9637    cache-references                        ( +-  0.54% )
       11429    L1-dcache-load-misses                     ( +-  0.18% )
       4610    L1-dcache-store-misses                     ( +-  0.21% )
         0    L1-dcache-prefetch-misses
       3973    L1-icache-load-misses                     ( +-  0.60% )
 
    0.000320526 seconds time elapsed                      ( +-  1.22% )
```

| Quiz#3       | recursive   | iterative   |
| ------------ | ----------- | ----------- |
| 執行時間     | 0.000320526 | 0.000489711 |
| cache-misses | 65          | 65          |
| 時間複雜度   | O(n)        | O(n)        |

 

## #4

### Question

```
Find the contiguous subarray within an array (containing at least one number) which has the largest sum.
For example, given the array [-2, 1, -3, 4, -1, 2, 1, -5, 4], the contiguous subarray [4, -1, 2, 1] has the largest sum 6.
請寫出合法的 C 程式 : int maxSubArray(int A[], int n) { … } 
 
```

### Recursive

- 採用Divide&Conquer方法，把完整陣列分成兩半，最大值的區間分為以下三種情形：
  - 最大值陣列在完整陣列的左半部分：
    - 解法：直接遞歸然後
  - 最大值陣列在完整陣列的右半部分
    - 解法：直接遞歸然後
  - 最大值陣列穿過中點左右陣列都有
    - 解法：以陣列中心點為初始，向左右延伸求和後遞歸
- 然後將這三種情形的值做比較即可

```c
int Across_mid(int a[], int left, int mid, int right)
{
  int sum = 0,
    lsum = a[mid], i;
  for ( i = mid; i >= left; i-- ) {
    sum += a[i];
    lsum = sum>lsum?sum:lsum;
  }
  sum = 0;
  int rsum = a[mid];
  for ( i = mid+1; i <=right; i++ ) {
    sum += a[i];
    rsum = sum>rsum?sum:rsum;
  }
  return lsum+rsum;
}
 
int maxArray(int a[], int left, int right)
{
  if(left == right) {
    return a[left];
  }
  int mid = (left + right)/2;
  return max3(
        maxArray(a, left, mid),
        maxArray(a, mid+1, right),
        Across_mid(a, left, mid, right)
      );
}
 
int maxSubArray(int a[], int n)
{
  return maxArray(a, 0, n-1);
}
 
```

### Iterative

- 從頭開始累加陣列中元素的和，如果之前的和是負數則拋棄之前的和從該元素開始重頭加，記錄下和最大的那個值便是答案

```c
int maxSubArray(int a[], int size)
{
  int max = a[0], sum = a[0], i;
  for ( i = 1; i < size; i++ ) {
    sum = sum + a[i] > a[i] ? sum + a[i]: a[i];
    max = max > sum ? max : sum;
  }
  return max;
}
```

### Performance

- `echo "echo 3 > /proc/sys/vm/drop_caches" | sudo sh `
- `perf stat -r 100 -e cache-misses,cache-references,L1-dcache-load-misses,L1-dcache-store-misses,L1-dcache-prefetch-misses,L1-icache-load-misses ./iterative`：

```bash
 Performance counter stats for './iterative' (100 runs):
        43    cache-misses        #   0.487 % of all cache refs    ( +- 10.54% )
       8817    cache-references                        ( +-  0.63% )
       9657    L1-dcache-load-misses                     ( +-  0.21% )
       4250    L1-dcache-store-misses                     ( +-  0.22% )
         0    L1-dcache-prefetch-misses
       3736    L1-icache-load-misses                     ( +-  0.75% )
    0.000440431 seconds time elapsed                      ( +- 35.40% )
  
 Performance counter stats for './recursive' (100 runs):
        40    cache-misses        #   0.457 % of all cache refs    ( +-  8.53% )
       8819    cache-references                        ( +-  0.76% )
       9586    L1-dcache-load-misses                     ( +-  0.24% )
       4224    L1-dcache-store-misses                     ( +-  0.26% )
         0    L1-dcache-prefetch-misses
       3674    L1-icache-load-misses                     ( +-  0.85% )
    0.000423999 seconds time elapsed                      ( +- 34.31% )
 
```

| Quiz#4       | recursive   | iterative   |
| ------------ | ----------- | ----------- |
| 執行時間     | 0.000423999 | 0.000440431 |
| cache-misses | 40          | 43          |
| 時間複雜度   | O(nlogn)    | O(n)        |

 

## 觀看《進擊的鼓手》

剛看完《進擊的鼓手》，首先，我愛Jazz。

其次，這部影片討論的是超於自我的主題，很壓抑，但這不是音樂的本質，音樂是帶給人快樂、抒發情感的語言，所以聽首經典的《[All of me](https://www.youtube.com/watch?v=NXKYTJHnqvw)》換一下心情~

電影可以討論的真的太多，所以還是切回到Embedded2015這門課吧。

上學期第一次上課時受到極大的衝擊，從知識本身的深度到思考問題的方式，發現自己學的太淺、想的太淺；接著暑期去台北實習，真正面對一堆資深工程師時，發現自己與他們的差距，也是令自己驚訝不已。

回歸到這學期的課程，單純從心態和認知上已經得到改變，其實對上課是一件很期待的事情。無論是學習使用GProf、Perf等工具量化效能（一直以來都是聽定性的東西太多了）還是今天看到一個compiler是如何做的，這都讓人興奮。我喜歡從無到有的這個創造的過程，並且在知道自己和資工背景（無論好壞至少背景課程的基本概念他們都考試學/背過）的同學的差距時，學到這樣的東西會有大補的快感。

綜上，這半年多來，接受了很多衝擊，但和電影不同，我沒那麼玻璃心，老師也沒那麼苛刻，我們一直認可超越自己、有進步就是賺。電影里的男主角後面追求的東西似乎變了質，不是音樂帶來的快樂本身，而是親情被孤立更想得到親人們的關注而想成功。在我看來，來學嵌入式純粹是對電腦科學的這個領域的喜愛，從大學到現在，動力也是來源於此，不然我就去中國石油行業值班爽爽過公務員的生活了。

最後，關於追求卓越，我覺得是超越自我的某個較遠的階段，當我能更好地管理自己的時間、處理自己的各種情緒時，效率會越來越高，到那個比較ready的時候，應該就是會追求卓越了吧！

 

## Reference

- 官網[Leetcode Algorithms of Problemset](https://leetcode.com/problemset/algorithms/)，簡短的[介紹](http://coolshell.cn/articles/12052.html)
- 經典《[Introduction to algorithms](https://github.com/esbanarango/Competitive-Programming/blob/master/·Documentation/Books/Introduction.to.Algorithms.3rd.Edition.Sep.2010.pdf)》
- 一個簡中的題解《[LeetCode题解](https://www.gitbook.com/book/siddontang/leetcode-solution/details)》
- 《[Recursive solutions to array problems: Three examples](http://www.cs.uofs.edu/~mccloske/courses/cmps144/recursion_feb13_01.html)》
- 《[Stack Implementation in C](http://groups.csail.mit.edu/graphics/classes/6.837/F04/cpp_notes/stack1.html)》


