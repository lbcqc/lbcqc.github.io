---
layout: post
title: git使用总结
categories: [git]
description: 对git的使用还是不太熟悉，虽然网上很多资料，但还是打算自己总结一下
keywords: git, github
---

专门开一篇博文记录一下有关于git自己走过的坑，主要给自己看的，也是为了总结提升熟练度，同时如果能帮助到其他人也挺好

# 一、拉取远程代码更新

这里在GITHUB.COM上直接修改README.md文件作为示例，修改前README.md文件内容如下所示

![readme.md.1](/data/images/git_use_summary/git_cat_readme1.png)

直接在README.md文件末尾添加了一条语句提交。内容看后面。

## 1. 查看远程和本地的区别

这里以mster分支为例
```
 git diff master origin/master
```
![git_diff_master_origin_master1](/data/images/git_use_summary/git_diff_master_origin_master1.png)

这里有一个坑，git本地有一个origin/master分支，diff比较的是我们是本地的master分支和origin/master，如果我们没有更新origin/master分支到最新状态，得不到最新结果，使用fetch命令更新

## 2. 拉取远程更新到本地origin/master分支

```
git fetch origin
```
![git_fetch_origin](/data/images/git_use_summary/git_fetch_origin.png)

这时候再重新比较就有结果了，如下所示

![git_diff_master_origin_master2](/data/images/git_use_summary/git_diff_master_origin_master2.png)

## 3. 拉取远程origin/master分支到本地master分支

然后我们再拉取这次更新，使用pull命令，将远程master分支更新到本地master分支
```
git pull origin master:master
```
![git_pull_origin](/data/images/git_use_summary/git_pull_origin.png)

再次比较结果如下
```
 git diff master origin/master
```
![git_diff_master_origin_master2](/data/images/git_use_summary/git_diff_master_origin_master1.png)

查看README.md文件内容如下

![readme.md.2](/data/images/git_use_summary/git_cat_readme2.png)


# 二、提交本地更新到远程服务器