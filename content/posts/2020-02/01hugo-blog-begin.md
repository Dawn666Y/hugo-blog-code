---
date: 2020-02-27
title: "使用hugo搭建个人博客"
tags: ["hugo", "leaveit"]
description: "折腾博客了我又"
categories: ["博客"]
---

**tips:点击左上角的爱心 ❤️ 可切换夜间模式保护眼睛哦！！**

<!-- TOC -->

- [安装 hugo](#安装-hugo)
- [创建博客](#创建博客)
- [网站部署](#网站部署)

<!-- /TOC -->

> 众所周知受疫情影响，全国大多数同胞宅在家陷入无所事事的状态，本人本着蛋疼找事做的原则尝试搭建一个**个人博客**以供长期使用

**废话少说！让我们开始吧！**

# 安装 hugo

> 安装之前需要知道 **hugo** 是以 golang 编写的一个静态博客搭建框架，所以在开始安装前必须保证电脑上拥有 go 语言的环境，Go 语言安装下载地址[Go 官网](https://golang.google.cn/)，版本至少**1.11**以上，另外部署项目到 **Github** 上也需要安装 [git](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)，最后再安装[hugo](https://github.com/gohugoio/hugo/releases)

hugo 下载压缩包之后解压目录如下：

![解压缩后](https://i.loli.net/2020/02/27/Kt4w1rmahejGMsi.png)

- 新建一个资源文件夹

位置嘛随便，今后的博文也计划保存在这个文件夹中，再建立一个子文件夹 bin 存放解压出的`hugo.exe`

![image.png](https://i.loli.net/2020/02/27/epKORzVt3MkU5ua.png)

- 在环境变量中添加 `hugo.exe`所在目录

  ![image.png](https://i.loli.net/2020/02/27/NnrFwUJG5lEShdV.png)

  要注意分号“;”是英文的

- 打开命令行测试 hugo 是否配置成功

输入命令

```text
> hugo version
```

像下面这样弹出版本信息那就是 hugo 安装成功啦！恭喜！

![image.png](https://i.loli.net/2020/02/27/vm2UxPljEpMJ95K.png)

# 创建博客

在 hugo 的安装目录下--我的是`D:\hugo`，**按住 shift 点鼠标右键** 选择在此处打开命令行窗口

输入命令(名字随便起，我输的是 blog)：

```bash
hugo new site blog
```

然后就可以在文件夹里看到 blog，点进去看看结构
(你应该除了 public 都有，那是后话，暂时忽略)

![image.png](https://i.loli.net/2020/02/27/oQNjitIqXd7R9SH.png)

> 然后去下载我们的网站需要的主题，[**Hugo 官方主题库**](https://themes.gohugo.io/)，挑个喜欢的，我用的是[**LeaveIt**](https://themes.gohugo.io/leaveit/)，比较简洁，也就是你现在看到的样子，下载主题可以使用`git clone`，我选择直接在 **github** 官网下载压缩包：[**LeaveIt on Github**](https://github.com/liuzc/LeaveIt)

![image.png](https://i.loli.net/2020/02/27/7fzScGUwNKIpATn.png)

**重要：解压缩之后把文件夹名字里的`-master`去掉！**

然后我们把文件夹放在 blog/themes 中

![image.png](https://i.loli.net/2020/02/27/8ukLYynHv5xB6Ed.png)

打开 LeaveIt 文件夹可以看到 exampleSite 文件夹，这个文件夹是网站主题的示例，我们把里面的内容全选复制

![image.png](https://i.loli.net/2020/02/27/BfGADpq5zv2TXVm.png)

粘贴到 blog 下，会提示替换，选择全部替换

![image.png](https://i.loli.net/2020/02/27/CZnQyeS9fmuH8Gc.png)

在 blog 文件夹里**按住 shift 点鼠标右键** 选择在此处打开命令行窗口

输入命令

```bash
hugo server
```

可以看到服务已经启动

![image.png](https://i.loli.net/2020/02/27/2EkdjH4arY8oN59.png)

红线部分就是本地访问的地址
http://localhost:1313，在浏览器打开就可以访问啦！

![image.png](https://i.loli.net/2020/02/27/fvI42RLTVJPpHOM.png)

开始你对这个网站主题的探索吧！！每个按钮每个细节都不要放过！

至于网站的数据应该如何填写可以在[**LeaveIt on Github**](https://github.com/liuzc/LeaveIt)下查看

# 网站部署

可以选择[github](https://github.com/)或者国内的 [gitee](https://gitee.com/)，gitee 码云在国内访问倒是不错，但是我选择全球最大的开源网站(同性交友网站) github 上部署项目

github 注册账户、ssh 连接、git 基本命令就不用我说了吧，不会的不准下课

- 新建一个仓库

要注意仓库的命名

![image.png](https://i.loli.net/2020/02/27/vc7wDjOkiIm9348.png)

一定是前面的 owner 名**全小写**加上`.github.io`，我这里建过了所以提示错误

在 blog 里打开命令行

输入

```bash
hugo --theme=LeaveIt --baseUrl="https://dawn666y.github.io/" --buildDrafts
```

![image.png](https://i.loli.net/2020/02/27/WmlrgweKC7I3ZxG.png)

hugo 会在 blog 文件夹下生成 public 文件夹，里面都是发布部署需要的文件

![image.png](https://i.loli.net/2020/02/27/Us4g1bowkFqyE85.png)

我们在刚才的命令行中继续输入

```bash
cd public
```

进入 public 文件夹，依次执行下面的命令：

**注意：！上面说的 ssh 连接没有完成的话是没法儿部署的**

```bash
git init
git add .
git commit -m "first commit"
git remote add origin https://github.com/Dawn666Y/dawn666y.github.io.git
git push -u origin master
```

如果有报错的话，请自行百度解决

这样的我们的个人博客的搭建部署就全部完成啦！通过自己定义的仓库链接 https://dawn666y.github.io/ 就可以访问了，给坚持到这里的自己鼓鼓掌吧！

> 第一篇博客就是这样了，希望在今后的成长中也能像这样多多记录分享，大家一起勉励加油吧！ ❤️❤️❤️
