---
title: 五分钟回顾我的硕士论文：嵌入式向量图形系统设计
date: 2020-04-01 23:13:31
updated: 2020-04-02 21:30:00
categories:
- 系统与底层
tags:
- embedded system
- vector
- gui
- twin
- window system
- rapid prototyping
- svg
- wearable devices
- µGUI
- stm32
---

研究生时期的嵌入式课程笔记整理告一段落，还剩一点硕士论文相关的笔记有待整理。这几篇笔记记录了论文以外的东西，主要是实验过程中的部分源码分析与问题思考，还能保留至今已是幸事（如前文所叙，同样有不少笔记、文章随着 Dropbox 的自动迁移丢失了😭）！

在开始分享笔记之前，先借本文简单介绍一下鄙人的硕士论文，为后续的笔记做个铺垫。

论文主题非常工程化，不过一点一滴都是自己从头搭建起来的，同时完全拥抱开源，甚至像开发环境也是在 Linux 上写 MAKEFILE、LD Script 等去实现过去 Keil IDE 已经帮开发者做好的事情。因此回顾这个项目时，点滴细节不断在脑海里涌现，品味之下令我获得极大的满足感。

希望也能与你分享这份因技术而生的快乐！

<!-- more -->



<br />

我的论文旨在为低端嵌入式设备设计一个高性能的向量图形系统，名为 EVGUI（Embedded Vector GUI），其最大价值或者说主要应用场景是在 IoT 设备的产品原型阶段帮助研发团队进行概念验证（Proof of Concept, POC），和工业领域的快速成型（Rapid prototyping）是类似的概念。下面我会用通俗易懂的话来解释这个图形库（希望）解决了什么问题。

在嵌入式电子产品的 GUI 开发流程中，通常设计师先基于产品显示设备的尺寸设计 GUI 界面，接着由嵌入式工程师对照设计稿将一个个像素点码出来，然后大家再评估、修正。由于使用底层语言开发的效率不高（相比客户端 APP、网页应用），因此在项目前期如 POC 阶段怎么提升研发效率便成了关键问题，毕竟一寸光阴一寸金呐。

我的论文思路是找到一个共同的表达层，既可以描述设计师的设计，也可以描述嵌入式设备显示的画面。这个表达层需要使用领域特定语言（Domain-specific Language, DSL）实现描述功能，其实没你想的那么复杂，HTML 就是一种 DSL。我也没搞什么发明创造，选择了 SVG 这种向量语言，它除了是一种图形格式外，更重要的一点特性是矢量图，这意味着它天然适配不同分辨率的屏幕设备，所见即所得在快速成型阶段的价值就体现出来了。因此基于这样一个向量图形库，设计师可以直接将设计稿转成SVG格式给嵌入式工程师，后者可以很方便地在基于 ARM Cortex-M4 微控制器等级的硬件上将设计稿渲染出来。

我们日常用的电脑甚至手机设备性能都非常的强劲，例如今年各家骁龙 865 旗舰手机的高配版本内存竟然达到了 12 GB！而我刚提到的 ARM Cortex-M4 微控制器大概是什么水平呢？以我使用的开发套件 ST32F429IDISCOVERY Kit 举例来说，它使用了一枚 STM32F429ZIT6 MCU，主频 180 MHz、2 Mb of Flash memory、256 KB of RAM，是不是有种瞬间回到“石器时代”的感觉😂。所以实验最大的难度就是在这样性能极度匮乏的硬件上，实现一个能成功解析并正确展示 SVG 图片的向量图形库，每一 KB 内存都需要“锱铢必较”！

![Architecture diagram](https://hmxlua.bn.files.1drv.com/y4mgXJcVBrse8QRmGsAgSbN26mPYmCinQuLiI1dHxs1avruyDjjbE_eEl3yi5U3yw4POWK2Yzm0ASNS3G807lK2Trz2zVVVf-LpKDCAaOlW54MgtAIjOBikzFPoMZvL-Td2T88OxOmW-XK8JL7_YqUmqKWpUTvryV2OP3NagIUzEk7OH9hmbDKPU0p2JM7h1cyr-eGCT0urd9cDFtf3K0UR3A?width=1465&height=971&cropmode=none)

如上图所示，EVGUI 图形库包含三大部分：展示层（Presentation Layer）负责 XML、SVG 的解析；渲染层（Render Layer）负责窗口（Window）、插件（Widget）以及 GUI 的一些核心功能如向量库、基础字体、处理模型等；输出层（Output Layer）主要负责内存管理和驱动管理。图中蓝框说明的是面向快速成型的工作流程，也就是前文提到的直接在设备上渲染出 SVG 图片；红框则想解释这样一个图形库除了能用作快速成型外，也兼容传统开发模式，即基于渲染层提供的 API 编写 GUI 业务代码。

具体的实现、使用到的一些开源库可以参考我的 GitHub [仓库](https://github.com/Joouis/EVGUI)，此处不再赘述，在后续（确定会分享）的笔记中会介绍 SVG 的处理、向量图形核心库 TWIN 的渲染流程、简单的触控处理。记得还有一篇把底层驱动库 µGUI 移植到开发套件上的笔记，可惜同样遗失了！在这放个仅存的渲染流程图缅怀一下。

![µGUI rendering process diagram](https://hmxkua.bn.files.1drv.com/y4mFPnTSDrME_OY4iFZHc8gSLkgfs78VAg53NaznJCZGNw8yIzn_8VNcG2CpFezLDRfYYoQfTHbg2eQVjYlF7KHmwn9tunmARvPHn7n2lsRt5IAqRwo_9WQoXaLeTEFjISBuPkXZLhZJa8NkZdO7mAQB422dtcpWRRayyTYu0Y3bwra5fC6kXSwi3TVqbJj5TpX_LAPRplBVf5GXm4CeADlZA?width=1299&height=642&cropmode=none)

值得一提的是，向量图形库 [TWIN](https://keithp.com/~keithp/talks/twin-ols2005/twin-ols2005www/) 是渲染层的核心。作为一个 2005 年还是为 "Sub-PDA" 设计的窗口系统，它只耗费 100KB 左右的内存便可运行起来，代码架构和质量真的非常牛逼。它的作者 Keith Packard 因 X Window 系统而闻名，他在 HP 剑桥研究实验室完成 TWIN 后，又基于 TWIN 研发了 [Xr 向量图形库](https://www.cairographics.org/xr_ols2003/)，随后便是现在著名的 Cairo 图形库。这么介绍似乎有点攀亲带故，主要是 TWIN 相关的资料太少，因此多提一嘴。此外 TWIN 诞生的时代还是 WIMP （Windows, Icons, Menus, Pointer）盛行，而现今的交互方式中触控操作逐渐成为主流，因此集成它时也涉及到了人机交互方式的设计与修改。

既然是一篇论文，当然不能只把工程跑起来证明理论设计是可行的，还需要一些定量测试，例如内存的使用分析和测试。为了这么一点点内存绞尽脑汁，现在回想起来，真的很有意思！

![Memory consumptions of layers diagram](https://hmxjua.bn.files.1drv.com/y4mL7BLZQi1rxMii6tdDKgMLozK3U26xWucyZ1fU0lshEz9CJwoUs6BoWV2Xohz5wBKRZ_7x8qoz0L_lxdE_WqXLE5QeAqKr5aNEO1InDijHml65hsHiTm_-fD-2Cb4YiFdlrbR8Q_2mfLtMUA40a1eF_tHYPfPpGA3bgrGgW89Wm9kndlGvgPy3wuQ5ZssEnuHf9AoSmqJ0qWcNNSq4FaSIA?width=1280&height=720&cropmode=none)

开发板外置的 SDRAM 用来存放 SVG 数据，划分了两个 Layer 1/2 实现双重缓冲（Double buffering）。

![Stack heap analysis diagram](https://hmxiua.bn.files.1drv.com/y4mzByqIpsNZkt7T_L9NGnugQ2wNVuoawtQPRyZrna63cZN0b75r_H7wvqhBax7t581xbYD5m-4A4LFh13hYGISR3PHsJZN6zgpKnReDidlUOuW29JM3JQxhuZ5dr5Ym6bfcYa-51uapn9BXy2LfAMKKo0otjoTR3RDzS_XV3BaNzjf7rdAScjRmQgBVpgVfjbzcSfmFEC9l3QK0Mx2xu-FPA?width=1331&height=870&cropmode=none)

论文套用了当下流行的穿戴式设备话题，因此演示场景也与此相关。至于图二和图四的那个复古时钟，是 TWIN 当年（没错就是 2005 年）的一个演示应用，被我“移花接木”了过来。

![](https://hmxhua.bn.files.1drv.com/y4mgKHoIDX_7IhHNp6523jQNbAUQ6l6HvQmDu90FS1yJT9bA9EoTlZDIUApP6sqkvH8R0R6d6kzIiDyZs_PI7pr_3WbObyrDeXY4FOzbRjmeorNKQmOWcPVdjOrFsvGwhI0JaoahEMsEerO4K2oLqTETK04I1gBdBv6s0QJ29kussj4AQ9wRm9YWMeq20Sps8K4yRQ4Jx14Gl3x6KrWTNSNqg?width=1280&height=622&cropmode=none)

此处也有演示小视频。


{% raw %}
<video width="304" height="540" src="https://cdnhk.blob.core.windows.net/blog/EVGUI Demo.mp4" preload="metadata" controls="" playsinline="" poster="">Sorry, your browser does not support the video tag.</video>

{% endraw %}

<br />



最后需要说明是这篇论文的思路来自我的指导教授 Jserv，在之前的博文中也曾反复提到过这位大牛。他是“大脑”，而我是执行者。现在偶尔仍能回忆起很多找他调试代码到深夜的日子，嵌入式开发的代码量不大，解决一个问题可能只需要编写或修改几行到数十行代码，但这背后的 know how 往往需要花费数日乃至数周的时间。

同时也要感谢我的一直以来都很开明的直系教授同意我做这样一个与他研究方向毫不相干的题目，以及友情帮助设计这个清爽演示 GUI 的设计系学妹 [@Vivian](https://www.zhihu.com/people/vivianvivi) 同学 😊。

![](https://hmxeua.bn.files.1drv.com/y4mb_qFTtVYHbBZlRvN35TJRDKavuir8vDheXQXGiFgXGLA318wiXhyGUOl8QKJx2S5BTEkIagzaduF82j8mpH_Oy5jrs6z8BfE6KBcMVlR_ggdmqBNFYiUAzw21WZobxb3Jxg_1U1_b1wSlQCPKMJgJzffwRRgCBZv6PzlME_Aexp6tB-SkBx0FQgTmQAkqMmWoAkJXQLUyeoh7B3KVL9KHQ?width=2780&height=1542&cropmode=none)