---
layout: post
title: "linux笔记: 容器内部署nfs"
subtitle: "容器内安装nfs来共享自己的存储实践"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - docker
  - linux
---

## 1. linux nfs服务器配置和搭建
[https://www.jianshu.com/p/ed4ddac6b0ea](https://www.jianshu.com/p/ed4ddac6b0ea)

### 1.1 安装
NFS的安装是非常简单的，只需要两个软件包即可，而且在通常情况下，是作为系统的默认包安装的，因此你不需要安装。

* nfs-utils-* ：包括基本的NFS命令与监控程序
* rpcbind-* ：支持安全NFS RPC服务的连接

### 1.2 配置
NFS服务的配置文件为： /etc/exports，这个文件是NFS的主要配置文件，不过系统并没有默认值，所以这个文件不一定会存在，可能要使用vim手动建立，然后在文件里面写入配置内容。

``/etc/exports``文件内容格式：
```
<输出目录> [客户端1 选项（访问权限,用户映射,其他）] [客户端2 选项（访问权限,用户映射,其他）]
```

说明:
```
1、输出目录：输出目录是指NFS系统中需要共享给客户机使用的目录；
2、客户端：客户端是指网络中可以访问这个NFS输出目录的计算机
          客户端常用的指定方式：
          指定ip地址的主机：192.168.8.106
          指定子网中的所有主机：192.168.0.0/24或 192.168.0.0/255.255.255.0
          指定域名的主机：wj.bsmart.com
          指定域中的所有主机：*.bsmart.com
          所有主机：*

3、 选项：选项用来设置输出目录的访问权限、用户映射等。

   NFS主要有3类选项：

  1）访问权限选项：

          设置输出目录只读：ro
          设置输出目录读写：rw
          
2）用户映射选项

    all_squash：将远程访问的所有普通用户及所属组都映射为匿名用户或用户组（nfsnobody）；
    no_all_squash：与all_squash取反（默认设置）；
    root_squash：将root用户及所属组都映射为匿名用户或用户组（默认设置）；
    no_root_squash：与rootsquash取反；
    anonuid=xxx：将远程访问的所有用户都映射为匿名用户，并指定该用户为本地用户（UID=xxx）；
    anongid=xxx：将远程访问的所有用户组都映射为匿名用户组账户，并指定该匿名用户组账户为本地用户组账户（GID=xxx）；

3）其它选项

    secure：限制客户端只能从小于1024的tcp/ip端口连接nfs服务器（默认设置）；
    insecure：允许客户端从大于1024的tcp/ip端口连接服务器；
    sync：将数据同步写入内存缓冲区与磁盘中，效率低，但可以保证数据的一致性；
    async：将数据先保存在内存缓冲区中，必要时才写入磁盘；
    wdelay：检查是否有相关的写操作，如果有则将这些写操作一起执行，这样可以提高效率（默认设置）；
    no_wdelay：若有写操作则立即执行，应与sync配合使用；
    subtree：若输出目录是一个子目录，则nfs服务器将检查其父目录的权限(默认设置)；
    no_subtree：即使输出目录是一个子目录，nfs服务器也不检查其父目录的权限，这样可以提高效率；
```

### 1.3 nfs服务器的启动和暂停
为了使NFS服务器能正常工作，需要启动rpcbind和nfs两个服务，并且rpcbind一定要先于nfs启动（不过centos7上是按需启动）。

* rpcbind
* rpc.mountd
* rpc.statd
* nfsidmap
* rpc.gssd
* gssproxy
* rpc.svcgssd 

#### nfs客户端验证
```bash
$ yum install nfs-utils-* -y
$ rpcbind 
$ showmount -e 192.168.0.110
Export list for 192.168.0.110:
/data 192.168.0.0/24

$ mkdir /data
$ mount -t nfs 192.168.0.101:/data /data
```

#### 客户端自动挂载
```
$ sudo vi /etc/fstab
#
# /etc/fstab
# Created by anaconda on Thu May 25 13:11:52 2017
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/cl-root     /                       xfs     defaults        0 0
UUID=414ee961-c1cb-4715-b321-241dbe2e9a32 /boot                   xfs     defaults        0 0
/dev/mapper/cl-home     /home                   xfs     defaults        0 0
/dev/mapper/cl-swap     swap                    swap    defaults        0 0
192.168.0.110:/data     /data                   nfs     defaults        0 0

$ sudo systemctl daemon-reload
$ mount
```

#### 碰到的问题
1. ``nfs touch: cannot touch 'hello2': Permission denied``

    解决： 服务器上给分享文件夹的group和other的读写权限，命令``chmod go+wx /export/app``

    参考： [NFS客户端写入NFS共享文件夹出错：Permission denied](https://www.crifan.com/nfs_client_write_to_nfs_server_share_folder_error_permission_denied/)

2. ``linux mount.nfs: Operation not permitted``
  
  使用docker启动的容器，需要启用privileged权限。

