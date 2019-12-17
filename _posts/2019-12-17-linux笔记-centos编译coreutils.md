---
layout: post
title: "linux笔记: centos下编译coreutils"
subtitle: "glibc1.12(centos6)和glibc2.17(centos7)下编译coreutils"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - docker
  - linux
---

>> 源码地址： [https://github.com/coreutils/coreutils](https://github.com/coreutils/coreutils)

## centos7 + coreutils v8.22 及以上 + glibc 2.17及以上
```bash
# docker run --rm -it centos:7 bash
# useradd admin
# yum install sudo -y
# echo "admin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
# su - admin
$ cd /tmp
$ sudo yum install git -y
$ git clone https://github.com/coreutils/coreutils.git
$ cd coreutils/
$ git checkout v8.22
$ sudo yum install autoconf automake autopoint bison gettext gperf makeinfo patch gettext-devel texinfo make -y
$ sudo yum install gcc -y
$ cd /tmp && curl -Lo texinfo6.1.tar.gz 'https://mirrors.tuna.tsinghua.edu.cn/gnu/texinfo/texinfo-6.1.tar.gz'
$ tar -xzvf texinfo6.1.tar.gz
$ cd texinfo-6.1 && ./configure
$ make 
$ sudo make install
$ sudo yum install wget -y
$ ./bootstrap
$ ./configure --disable-gcc-warnings
$ make
$ make src/mv
```

## centos6 + coreutils v8.4 + glibc2.12
```bash
# docker run --rm -it centos:6 bash
# useradd admin
# yum install sudo -y
# echo "admin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
# su - admin
$ cd /tmp
$ sudo yum install git -y
$ git clone https://github.com/coreutils/coreutils.git
$ cd coreutils/
$ git checkout v8.4
$ sudo yum install autoconf automake autopoint bison gettext gperf makeinfo patch gettext-devel texinfo make -y
$ sudo yum install gcc -y
$ sudo yum install wget -y
$ ./bootstrap
$ ./configure --disable-gcc-warnings
$ make
```

## 问题：
1. ./bootstrap: Error: 'makeinfo' version == 5.1 is too old

	解决：源码安装texinfo6.1

2. error: function might be candidate for attribute 'const' [-Werror=suggest-attribute=const]

解决： ./configure --disable-gcc-warnings

参考： https://bug-coreutils.gnu.narkive.com/q14Ima4F/bug-32762-bug-at-coreutils-compile

3. 编译某一个文件：

解决： make src/cp src/mv

参考： https://www.icode9.com/content-3-264498.html

4. date: /lib64/libc.so.6: version `GLIBC_2.14' not found (required by date); 
	date: /lib64/libc.so.6: version `GLIBC_2.17' not found (required by date)

解决： 转到centos6下编译


