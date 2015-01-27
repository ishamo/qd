---
layout: post
title: git里面一些不常用的功能
date:  2014-12-27 12:52:51   
category: git
tags: git
---

git博大精深，准确的说是功能很多，可以满足你的各种奇葩的需求，但是使用起来还是有一些门槛的。我没有系统的买一本书或者找一个在线教程看，只是在这个网站 https://try.github.io/levels/1/challenges/1# 做了一些练习就开始用了几个月，然后发现各种不顺利，现在我不得不挖掘一些新的功能了，以后有机会，一定要找一本书瞅瞅才行~

创建本地孤儿分支。就是创建一个没有主分支的分支，有什么用呢？当我们要用gitpages构建博客时会用到：`git branch --orphan gh-pages`

删除远程分支。首先你需要检查你的远程分支是不是默认分支，如果是的话你需要用别的分支设成默认分支才行，这个很好理解，你不能直接在你的本地删除你远程的默认分支，系统会提示你没有权限。怎么用别的分支设成默认分支呢？找到你项目的setting，修改一下就可以了。

![setting-gh-pages.png](http://shamospace.qiniudn.com/setting-gh-pages.png)

然后执行：`git push origin --delete <branchName>` 或 `git push origin :<branchName>` 注意冒号后面有一个空格，代表用一个空的分支push到远程分支，也相当于是删除了远程分支了。

关联远程分支。	`git remote add origin git@github.com:YourName/YourRepositories.git` origin代表你在本地看到的你的远程仓库听名字（bucket或者形象一点）。

下面这些都比较常用简单了。

删除本地分支。 `git br -d <branchName>` 

切换本地分支。 `git checkout <branchName>` 

查看远程分支。 `git branch -a`

用你的本地分支与远程分支建立关联。 这个系统会提示怎么做的，不记得了，日后再遇到的话补上。




















