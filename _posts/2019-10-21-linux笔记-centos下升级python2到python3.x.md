---
layout: post
title: "centos下升级python2.x到python3.x"
subtitle: "centos下升级python到3.6并设置为默认，并配置虚拟环境"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - linux
  - python
---

# 1. yum安装python3.6
```bash
$ yum -y install python36 python36-devel
```

# 2. 源码安装
## 2.1 准备
```bash
$ yum install make gcc gcc-c++ 
```
## 2.2 下载安装包并解压
```bash
$ wget https://www.python.org/ftp/python/3.8.0/Python-3.7.5.tgz
$ tar -zxvf Python-3.7.5.tgz
$ cd Python-3.7.5/
$ ./configure 
$ ./configure --prefix=/usr/local/python3 --enable-optimizations
```
## 2.3 编译和安装
```bash
$ make && make install
```

>> 注意： 经过上面的步骤以后，可以通过``python3``命令来使用python3.x

# 3. centos下设置3.x为系统默认
因为centos的yum使用python2，所以，需要设置yum相关脚本为/usr/bin/python2

脚本如下：

```bash
$ rm -rf /usr/bin/python
$ ln -s /usr/local/bin/python3.7 /usr/bin/python
$ sed -i "s/python/python2/g" /usr/bin/yum
$ sed -i "s/python/python2/g" /usr/libexec/urlgrabber-ext-down 
```

# 4. 设置py3的虚拟环境
```bash
$ python3.6 -m venv /opt/py3
$ source /opt/py3/bin/activate
```

# 5. 设置python的pypi源
## 5.1 临时
```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple requests
```
## 5.2 永久
```bash
$ echo ~/.pip/pip.conf # (没有就创建一个)， 修改 index-url

[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple 
trusted-host=pypi.tuna.tsinghua.edu.cn
```