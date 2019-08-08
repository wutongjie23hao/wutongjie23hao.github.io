---
layout: post
title: "linux笔记：mac安装iterm2和oh my zsh"
subtitle: "mac输入汉字显示乱码/securecrf中vim选中不能复制却变成visual模式"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - linux
  - mac
  - iterm2
  - zsh
---

## 安装iterm

官网[https://www.iterm2.com/](https://www.iterm2.com/),下载压缩包，解压就行。

## 安装 ``oh my zsh`` 

``` bash
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)" or
$ sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
$ cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```
## 切换bash环境到zsh

```bash
$ chsh -s /bin/zsh
#   切换回来
$ chsh -s /bin/bash
```

## 控制台输入汉字显示乱码

``vim ~/.zshrc`` 添加：

```
export LC_ALL=zh_CN.UTF-8
export LANG=zh_CN.UTF-8bb
```

## securecrf中vim选中不能复制却变成visual模式

与securecrt没关系，和vim的配置有关系
1. ``rm -rf ~/.vimrc``
2. ``brew uninstall vim``
3. ``brew install vim``