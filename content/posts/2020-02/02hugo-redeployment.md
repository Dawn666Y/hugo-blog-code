---
date: 2020-02-27
title: "对博文数据更新再部署"
tags: ["hugo", "leaveit"]
description: "再次部署"
categories: ["博客"]
---

**tips:点击左上角的爱心 ❤️ 可切换夜间模式保护眼睛哦！！**

# 删除 public

删除 public 目录下除`.git`以外的所有数据

# 生成新数据

再次在 blog 文件夹下执行 hugo 的 build 命令重新生成网站数据

```bash
hugo --theme=LeaveIt --baseUrl="https://dawn666y.github.io/" --buildDrafts
```

# 提交部署

在 public 文件夹下依次执行

```bash
git branch newBranch
git checkout newBranch
git add .
git commit -m "第二次提交"
git checkout master
git merge newBranch
git push -u origin master
```

简单解释一下

1. 创建一个新分支 newBranch
2. 切换当前操作分支到新分支
3. 添加数据到新分支
4. 提交数据到新分支
5. 切换当前操作分支到 master
6. 合并新分支到 master
7. 再次 push 部署

> 新的数据就完成了部署，提醒一下 Github 上的资源缓存更新有点慢，在国内可能需要等待一小段时间，然后强制刷新一下就可以看到更新后的博文啦
