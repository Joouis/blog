---
title: 五分钟快速搭建一个博客
date: 2016-04-09 01:13:52
updated: 2019-04-09 01:05:00
categories:
- 笔记小抄
tags: 
- blog 
- github 
- hexo
---

作为部落格的第一篇文章，就记录一下它的诞生过程吧：）

<!-- more -->

博客很阳春，使用了 Github 和 Hexo，都是一谷歌就能找许多答案的技术。

## 搭建流程

1. 安装 node.js 和 npm，推荐直接从[官网](https://nodejs.org/en/download/)下载源码编译；
2. 安装 hexo：
```sh
npm install hexo-cli -g
hexo init blog
cd blog
```
3. 安装主题，本博客使用 [NexT](http://theme-next.iissnan.com/) 主题：
```sh
git clone https://github.com/iissnan/hexo-theme-next themes/next
```
4. 启用主题，在根目录下将`_config.yml`文件中的`theme:`改为你所需的主题名；
5. 验证主题：
```sh
hexo s --dedbug
```
6. Git 初始化并同步到 Github 上：
```sh
echo "# blog" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin git@github.com:Joouis/blog.git
git push -u origin master
```
7. 使用 hexo 部署功能，补充根目录`_config.yml`文件中的`deploy`，然后：
```sh
hexo d -g
```
8. 在 source 目录下加入`CNAME`文件，在登陆 DNS 服务商增添 CNAME 记录即可；
9. 參考[主題配置](http://theme-next.iissnan.com/theme-settings.html)、[第三方服务](http://theme-next.iissnan.com/third-party-services.html)一步步客制化自己的博客。



## 心得

五分钟是完全足够搭建起博客的，不过真正费时的，就是一点点把它修改成自己想要的样子吧。 头一次使用 Hexo ，比 Jekyll 更方便，并且使用的 NexT 主题基于 Hexo 增添了很多功能接口，像搜索、评论平台等，只需要设置`_config.yml`一个文档基本上就能解决问题，确实很好用！

博客的用途是用来记录，Markdown 语言的特质就是帮助你专注在记录上，而今天使用的 Hexo 和 NexT 亦是如此。



## 更新

### 2019.04.08

博客三周年，升级了 Hexo 和主题，请看[这里](https://blog.joouis.com/2019/3rd-blog-anniversary/)。