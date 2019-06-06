---
layout: post
title: "linux笔记：criu记录"
subtitle: "centos下criu的安装和体验"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - linux
  - docker
---

> [criu](https://github.com/checkpoint-restore/criu) , checkpoint/rebuild in userspace.

## install (centos)
* yum install -y wget
* 安装c编译工具gcc和make：yum install -y gcc make
* 安装google的protobuf库：yum install -y protobuf protobuf-c protobuf-c-devel protobuf-compiler protobuf-devel protobuf-python
* 安装其它的：yum install -y pkg-config python-ipaddress libbsd-devel iproute2 libcap-devel libnet-devel libaio-devel python2-future 
* 还有依赖： yum install -y protobuf-c protobuf-c-devel protobuf-compiler protobuf-devel protobuf-python libnl3-devel libcap-devel
* wget "http://download.openvz.org/criu/criu-3.4.tar.bz2"
* yum install bzip2 -y
* tar -xjf criu-3.4.tar.bz2 
* cd criu-3.4 
* make
* cp criu/criu /usr/sbin//criu

### reference
* [install](https://criu.org/Installation)
* [criu for mac](https://github.com/boucher/criu-for-mac) : 没实验成功，可能比较古老。

## 体验
### 1. 测试脚本

```sh
$ cat > test.sh <<-EOF
#!/bin/sh
while :; do
    sleep 1
    date
done
EOF
$ chmod +x test.sh
```

### 2. C/R 一个前台shell
#### 2.1 启动
```sh
$ ./test.sh
$ ps -C test.sh
  PID TTY          TIME CMD
 2621 pts/1    00:00:00 test.sh
```

#### 2.2 dump
```sh
# criu dump -vvvv -o dump.log -t 2621 --shell-job && echo OK
OK
```

#### 2.3 restore
```
#  criu restore -vvvv -o restore.log --shell-job
Fri Apr 12 12:41:09 MSK 2013
Fri Apr 12 12:41:10 MSK 2013
```

#### 2.4 注意
在dump的时候，原先的shell job被kill掉。

#### 2.5 reference
[https://criu.org/Simple_loop](https://criu.org/Simple_loop)




