---
title: 观影简记：《Revolution OS（操作系统革命）》
date: 2019-12-09 23:54:55
updated: 2019-12-09 23:54:55
categories:
- 系统与底层
tags:
- os
- free software
- open source
---

这篇笔记是当年参加Embedded System 2015课程的作业之一。Jserv的课程使我受益良多，也帮助我迈入了计算机科学的殿堂。他的课程在成大是出了名的“硬”，信息量巨大，上课基本就是划重点加答疑，课下需要大量时间去自学和实践。同时他也一直鼓励学生参与开源活动，甚至把作业发布到例如Hackpad（现今Dropbox Paper）、HackMD等协同文档平台上，既可以作为自己的网络备份（这也是为什么我还能找到这些“黑历史”😂），也可以供其他人参考、交流。

2016年随着Hackpad被Dropbox收购并推出新的产品Paper，Hackpad上的数据也被自动移植到了Paper上。在这次数据自动迁移中，当年的不少作业笔记被遗失。在2019年的尾声，我会陆续将仅存的少量有意思、有干货的笔记从Dropbox Paper中翻出来放到博客中，也算是对当年热血岁月的一次纪念。

<!-- more -->



## [Revolution OS](https://youtu.be/vWwvh3036Fw)

链接和下方的影片来自YouTube，外网可看~

{% youtube vWwvh3036Fw %}



## 關鍵人物、事件

- Eric Raymond: Author, “The cathedral and the bazaar”

- Linus Torvalds
- Bruce Perens: Author, Open source Definition

- Richard Stallman
  - 哲學觀念：
    - Whoever sitting at the computer, should be able to do whatever he wants. And somebody else who was there yesterday shouldn't be controlling what you do today. 
    - A free software refers not to price but to freedom, so think of free speech not free beer. The freedoms that I'm talking about are the freedoms to make changes if you want to, or hire somebody else to make changes for you if you're using a free software for your business. To redistribute the copies to share with other people, and to make improvements and publish them so other people can get benefit of them too.
    - Copyleft
    - GPL(General Public License)
    - 1984.01 GNU started
    - GNU is recursive acronym XDD
- BSD(12:36終於第一次提到)
  - 關於BSD電影裏提到的太少，詳細補充借用Jserv的《**從 Revolution OS 看作業系統生態變化**》：
    - 作為技術創新者，BSD 在本紀錄片只零星出現四次 (談及 Apache 的成功歸於開發者社群, GNU 計畫源始, 開放原始碼授權, 對 Microsoft 提出反托拉斯訴訟)，幾乎一面倒向 Linux，實在可惜
    - 增添 Berkeley Unix / BSD 發展歷程、和 AT&T 的官司爭議
- Michael Tiemann: Co-Founder, Cygnus Solutions
- Larry Augustin: Co-Founder, VA Linux Systems
- ERL: 啓發GNU與Linux商業應用的誕生地
- Linux: Monolithic
- GNU Hurd: http://zh.wikipedia.org/wiki/GNU_Hurd
- Brian Belhendorf: Apache Web Server，使得Linux第一次在商業市場上大獲成功
- RedHat Software: 首間上市的Linux公司
- Netscape
  - 第一個參與開放原始碼的大公司，爲了對抗IE
  - 後來的Mozilla
- Open Source vs Free Software
- 資料庫軟體供應商的倒戈
- Linux Users Groups
- Linux World Conference



## 思考與疑問

- 昨天看了一遍，感覺很爽，今天再看一遍，把想法都寫出來
- 6:27 Stallman在信息裏說安全機制不過是個玩笑，讓我想到，自由軟體/開源軟體如何確保我們使用的安全性，如果有人利用它的易修改來惡意傳播帶有問題的軟體，會不會是個災難？
- 7:30 Bill Gates提到的軟體知識產權問題，我覺得並沒有錯(當然本片站在開源者的角度肯定是否定態度)。開源和閉源是兩種不同的價值觀，無論是哪種價值觀，從它的出發點去考慮都是合理的，最後也都被證明是可以賺錢、成功的。所以我覺得如今這個多元化的社會，開源和閉源同時存在、相互競爭是非常正常的現象。

- 另外不得不說，微軟這麼多年屹立不倒，也得益與Bill Gates大學念得法律系，面對各種官司都能從容不迫(雖然不是他去打官司)。多元化的社會，如果追求更高的目標，對自己的定位不能僅僅只是懂技術的工程師，有時候還需要更多東西。

- 11:05 Stallman想到自己就是一個OS開發者，決定自己開始幹時，我想到多少次似曾相識，很多成功人士都是這麼說的"XXX不好用，我乾脆自己來做一個"。我們很多時候都在抱怨，卻沒有動力去用自己的專業知識解決問題。往往行動起來的人，就成功了。
- 從1984年開始GNU計劃，到1990年Stallman等人才寫完了UNIX上各種免費的套件包，還沒有寫kernel，工程量的浩大和困難可見一斑
- 14:29 Stallman一開始對釋出源代碼也是存有戒心的，人類的天性吧
- 16:45 Augustin談到自己去找工作時身邊的人用的都是自己寫的程式，這就像Jserv老師說的找工作時給HR或主管秀出自己的Google搜索結果一樣，簡單有效
- 20:33 Stallman分析自由軟體如何能賺錢，邏輯太精彩了，壟斷往往百害而無一利
- 24:11 Linus提出自己開始寫一個OS，完全沒有提到Minix
- 30:47 Linus在被問到"有人建議該寫成GNU/Linux"時認爲只有GNU自己發佈的Linux Distribution才能叫GNU/Linux(或隨他們怎麼稱呼)。但是所有版本的Linux毫無疑問都用到GNU的各種系統軟體，Stallman在意的可能是大家(特別是非IT業的人，但這部分人才是未來更廣闊的目標羣體)只記住了Linux，而忽視了GNU的貢獻吧
- 36:42 Augustin談到自由軟體找風投，那個時候幾乎所有風投都不太信任自由軟體，可見此時自由軟體並未獲得主流的認可
- 48:35 Stallman在談到爲什麼他支持Free Software而不是Open Source時顯得更像一位哲學家，其實也更像一個保持童真的小孩或者說是理想主義者。我還是更支持Perens所說的，自由軟體和非自由軟體可以共存。
- 58:25 電影又開始“攻擊”微軟了，我個人一直認爲微軟是家偉大的公司，推動了平民化電腦市場，使得普通百姓也可以輕鬆使用電腦。不過壟斷確實不是一件好事，Linux等OS的飛速發展可以迫使微軟努力向前走(1:01:17畫面中的“Microsoft Welcome the Linux Community”無論是公關還是誠心的歡迎都是一個很好的舉措，有大企業該有的氣度，至於爭論的話題，個人覺得推給OEM沒問題)
- 1:02:47 “共產主義”這個問題更多是意識形態上的東西，電影裏就可以看出，當這些開源人士與社會主流意見不同時，又因爲行爲上有共產的思想，就被說成是“共產主義”。實際上意識形態這種東西，被統治階級拿來使用，畢然是要求人人信服的(至少在表面上)，不然如何維繫穩定。至於把“共產主義”和Open Source扯在一起，真的是有夠搞笑的(特別是Eric迴應的那副表情)
- VA Linux首次IPO真的讓人吃驚，開盤價30塊，期間達到了320、340塊，最後收盤價299塊，打破了最高開盤交易記錄，what the fxxk!!! 科技創造奇蹟!!!
- 最後關於RedHat和VA Linux的兩個鏡頭是想說黃金時代後的泡沫崩潰嗎？Linux未來發展該何去何從還有待進一步的觀望？



## 字幕勘誤

- 2:11  “它有12萬使用者”---->“它有1200萬使用者”
- 28:58 “(約238萬新臺幣)”--->"(約17萬5千新臺幣)"(91-92年匯率大概爲TWD1=USD 0.04)
- 36:33 “2萬5千行程式碼”--->"25萬行程式碼"