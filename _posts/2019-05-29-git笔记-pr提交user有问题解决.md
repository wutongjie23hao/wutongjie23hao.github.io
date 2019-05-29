---
layout: post
title: "git笔记：github提交的pr中user有问题解决"
subtitle: "Why are my commits linked to the wrong user?'"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - git
---

## 前言

github上面提交了一个pull request以后，发现clk过不了，点开，是github的用户不匹配。

官方有来解决方法：
1. 用户是错的：你可能有多个邮箱用户，多个github账号，然后都在本地存着，就会发生问题，就是你第一次的提交和后面几次提交可能用户不一致。针对这个方法，你只能够使用``git config user.name``和``git config user.email``来设置自己的本地github账户，这样在以后的提交中就会保持一致了。但是，这个情况下，已经提交的pr咋整？ 需要reset到正确的提交，然后，错误的重新提交，第二个模块介绍这个。
2. 没有用户：这个貌似可以在浏览器中添加，没有遇到过这种情况。

## github上已经提交pr的用户错误
### 1. 撤回错误的提交

> 注意：备份/备份/备份

1. git log
2. git reset --hard fdb75db7fb128aeef1de90fb9c2d9bfc1f7cafff
3. git commit -m "fix bug"
4. git push -f

上面4个步骤搞了以后，git回滚到fdb75db7fb128aeef1de90fb9c2d9bfc1f7cafff的head上，然后重新进行提交。

### 2. 重新提交

1. git config user.name xxxx
2. git config user.email xxxx@xxx
3. git add -A
4. git commit -m "bugfix"
5. git push

## reference
[http://www.javabin.cn/2018/git_pr.html](http://www.javabin.cn/2018/git_pr.html)
