---
layout: post
title: "golang笔记：mac cgoenabled=1 出现问题"
subtitle: "go交叉编译问题"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - golang
---

## 前言
在golang中使用sqlite3,需要使用``CGO_ENABLED=1``

## 问题
```plain
# runtime/cgo
linux_syscall.c:67:13: error: implicit declaration of function 'setresgid' is invalid in C99 [-Werror,-Wimplicit-function-declaration]
linux_syscall.c:67:13: note: did you mean 'setregid'?
/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/unistd.h:593:6: note: 'setregid' declared here
linux_syscall.c:73:13: error: implicit declaration of function 'setresuid' is invalid in C99 [-Werror,-Wimplicit-function-declaration]
linux_syscall.c:73:13: note: did you mean 'setreuid'?
/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/unistd.h:595:6: note: 'setreuid' declared here
make: *** [build] Error 2
```

## 解决
```plain
1. 安装跨平台编译包
  brew install FiloSottile/musl-cross/musl-cross
2. 修改编译命令为: 
  CGO_ENABLED=1 GOOS=linux  GOARCH=amd64  CC=x86_64-linux-musl-gcc  CXX=x86_64-linux-musl-g++ go build -o BIN_NAME
```

## 参考
* [https://www.baifachuan.com/posts/4862a3b1.html](https://www.baifachuan.com/posts/4862a3b1.html)

## QA
使用上述方法，在构建docker镜像的时候，直接拷贝二进制文件是不行的。这个二进制文件解决方法，在下一篇介绍。
