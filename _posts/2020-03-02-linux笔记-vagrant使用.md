---
layout: post
title: "linux笔记: vagrant使用"
subtitle: "关于vagrant和Vagrantfile的使用总结"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - linux
---
# vagrant使用

## 安装virtualbox

软件下载地址： https://www.virtualbox.org/wiki/Downloads 

下载系统对应版本安装即可。

## 安装vagrant

软件下载地址：https://www.vagrantup.com/downloads.html

下载系统对应版本安装即可。

## 下载vagrant box

下载地址：

```
http://www.vagrantbox.es/
https://app.vagrantup.com/boxes/search?provider=virtualbox
```

在上面url中搜索自己想要安装的虚拟机box，然后复制对应的url。

下载对应的box到本地，并重命名。

vagrant box add {title} {url}

这样会在本地下载对应的box，并重命名为title。

> 下载文件默认密码：vagrant

## 使用vagrant创建虚拟机并启动

两种方式，第一种：

```bash
# vagrant box add {title} {url}
# vagrant init {title} # 本地初始化产生Vagrantfile
# vagrant up
```

第二种：

```bash
# vagrant init centos/7 # 直接拉取centos/7的box，初始化产生Vagrntfile
# vagrant up
```

> 添加本地文件：
>
> vagrant add hello2 CentOS-7.1.1503-x86_64-netboot.box
>
> 或者直接下载到文件位置：``~/.vagrant.d/boxes``

## 实践

### 1. 下载box

```bash
# vagrant box add hello2 https://github.com/CommanderK5/packer-centos-template/releases/download/0.7.2/vagrant-centos-7.2.box
```

### 2. 填写Vagrantfile

```Vagrantfile
# cat Vagrantfile
Vagrant.configure("2") do |config|
    (0..1).each do |i|
        config.ssh.username = 'root'
        config.ssh.password = 'vagrant'
        config.ssh.insert_key = 'true'
        config.vm.define "node#{i}" do |node|
            node.vm.box = "hello2"
            node.vm.hostname="node#{i}"
            config.vm.provision "shell", inline: <<-SHELL
                echo root:root123 | chpasswd 
            SHELL
            # 设置虚拟机的IP
            node.vm.network "public_network", bridge: "eno1", ip: "192.168.17.20#{i}"
            # VirtaulBox相关配置
            node.vm.provider "virtualbox" do |v|
                v.name = "node#{i}"
                v.memory = 1024
                v.cpus = 1
            end
        end
    end
end
# vagrant up
# vagrant ssh node0 # 或者vagrant ssh node1
```

## vagrant的命令

1. 修改Vagrantfile后使生效：vagrant reload 或者 vagrant reload --provision
2. Vagrantfile修改后增加主机：vagrant up
3. vagrant文档：https://www.vagrantup.com/docs/


