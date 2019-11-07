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
$ yum install epel-release
$ yum -y install python36 python36-devel
```

# 2. 源码安装python3.7.5
## 2.1 准备
```bash
$ yum -y install make gcc gcc-c++ zlib* openssl openssl-devel
```
## 2.2 下载安装包并解压
```bash
$ wget https://www.python.org/ftp/python/3.7.5/Python-3.7.5.tgz
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

# 3. centos下设置python3和pip3为系统默认
因为centos的yum使用python2，所以，需要设置yum相关脚本为/usr/bin/python2

脚本如下：

```bash
$ rm -rf /usr/bin/python 
$ ln -s /usr/local/python3/bin/python3 /usr/bin/python 
$ ln -s /usr/local/python3/bin/pip3 /usr/bin/pip 
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
* 文件设置
```bash
$ echo ~/.pip/pip.conf # (没有就创建一个)， 修改 index-url
    [global]
    index-url = https://pypi.tuna.tsinghua.edu.cn/simple 
    trusted-host=pypi.tuna.tsinghua.edu.cn
```
* 命令行设置
```bash
  # 设置pip源
  $ pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U 
  $ pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple 
```

# 6. dockerfile升级centos7的python2到python3.7.5
```Dockerfile
FROM centos:7
ARG PYTHON_URL=https://www.python.org/ftp/python/3.7.5/Python-3.7.5.tgz
RUN yum -y install make gcc gcc-c++ zlib* openssl openssl-devel &&\
    curl -Lo /tmp/tmp.tgz "$PYTHON_URL" &&\
    mkdir -p /tmp/python &&\
    cd /tmp/python &&\
    tar -xzvf /tmp/tmp.tgz --strip-components=1 &&\
    ./configure  &&\
    ./configure --prefix=/usr/local/python3 --enable-optimizations &&\
    make && make install &&\
    rm -rf /tmp/python  &&\
    rm -rf /tmp/tmp.tgz &&\
    ln -s /usr/local/python3/bin/python3 /usr/bin/python3 &&\
    ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3 &&\
    # 设置为默认
    rm -rf /usr/bin/python && \
    ln -s /usr/local/python3/bin/python3 /usr/bin/python && \
    ln -s /usr/local/python3/bin/pip3 /usr/bin/pip && \
    sed -i "s/python/python2/g" /usr/bin/yum && \
    sed -i "s/python/python2/g" /usr/libexec/urlgrabber-ext-down && \
    # 设置pip源
    pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U && \
    pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple && \
    pip config set global.trusted-host pypi.tuna.tsinghua.edu.cn
```
# 7. 问题
1. ``Can't connect to HTTPS URL because the SSL module is not available`` \
  原因：在Python3.7之后的版本，依赖的openssl，必须要是1.1或者1.0.2之后的版本，或者安装了2.6.4之后的libressl \
  解决方法：网上说使用源码升级openssl，亲自实验后，发现未生效。最后解决方法是，删除openssl和openssl-devel，然后重装openssl和openssl-devel，再重新编译安装python3.7.5
# 8. reference
* [清华镜像源](https://mirror.tuna.tsinghua.edu.cn/help/pypi/)
* [python安装包下载地址](https://www.python.org/ftp/python/)
* []()