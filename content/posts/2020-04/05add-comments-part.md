---
date: 2020-04-27
title: "为博客添加评论系统"
description: "添加gitalk用作评论系统"
tags: ["leaveit"]
categories: ["博客"]
---

**tips:点击左上角的爱心 ❤️ 可切换夜间模式保护眼睛哦！！**

# 准备功课

> 了解了一下静态博客添加评论应该怎样实现，网上的大多数教程都是调用第三方，常见的有[Valine](https://valine.js.org/quickstart.html)、[gitalk](https://gitalk.github.io/)、[gitment](https://imsun.github.io/gitment/)

### Valine

这个试了一下，要支付宝实名认证，还要绑定手机，比较麻烦，我还是更希望自己的互联网身份是匿名的，不考虑这个方案了

需要了解或者选择使用 Valine 的同学移步[姑苏流白--Hugo Comments](https://blog.hgtweb.com/2019/hugo-comments/)

### gitalk

和博主的 github 绑在一起，使用也比较简单，参考这个博主的文章[Mogeko--为 Hugo 添加谈笑风生区 (Gitalk)](https://mogeko.me/2018/024/)

### gitment

这个很久没维护了，而且据说有 github 账户安全问题，也不考虑

# 添加步骤

参考博文：[Megeko--为 Hugo 添加谈笑风生区 (Gitalk)](https://mogeko.me/2018/024/)

在下方就可以看到评论区

# 遇到的问题

我的评论区上方出现一行报错

![image-20200427160657067](https://raw.githubusercontent.com/Dawn666Y/myPicGo/master/blog/image-20200427160657067.png)

百度之后才知道是我的页面链接名过长导致的--_gitalk 限制域名长度不超过 50，原因未知_，因为包含了中文

![image-20200427161110338](https://raw.githubusercontent.com/Dawn666Y/myPicGo/master/blog/image-20200427161110338.png)

而这个链接是被 gitalk 当作 id 写入的，上面我们在配置 config.toml 时填写的 id 那一栏就是指定 gitalk 加载的 id

我的博客会产生这样的链接，是因为我的 markdown 格式的博文是采用中文命名的，hugo 在编译页面的时候会使用文件名作为跳转到博文的链接，修改所有的本地 markdown 博文为非中文名再编译部署就好了

此外，gitalk 需要拥有者初始化每一篇需要搭建评论区的博文，初始化激活的步骤很简单，博主用自己的 github 账户登录一下这篇博文的评论区就可以了。~~尽管这意味着今后每发表一篇需要评论区的博文就要执行一遍初始化的流程~~

# 最终效果

测试评论能否成功，输入以下内容

````tex
测试一下刚刚添加的gitalk评论区

测试是否支持markdown

​```java
// 这是代码
public class Hello{
}
​```

这是链接：[本博客主页](https://dawn666y.github.io/)

这是图片：![表情包](https://raw.githubusercontent.com/Dawn666Y/myPicGo/master/blog/image-20200418134722115.png)
````

编辑模式：

![image-20200427153715094](https://raw.githubusercontent.com/Dawn666Y/myPicGo/master/blog/image-20200427153715094.png)

预览模式：

![image-20200427153814364](https://raw.githubusercontent.com/Dawn666Y/myPicGo/master/blog/image-20200427153814364.png)

可以点击右下角按钮切换两个模式

评论效果：

![image-20200427160526047](https://raw.githubusercontent.com/Dawn666Y/myPicGo/master/blog/image-20200427160526047.png)

评论区就这样搭建完成了
