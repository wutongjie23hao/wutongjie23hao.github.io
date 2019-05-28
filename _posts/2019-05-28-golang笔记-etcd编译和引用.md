---
layout: post
title: "golang笔记：etcd编译和引用中的坑"
subtitle: "cannot use auth.callOpts & unknown field 'CAFile'"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - golang
  - etcd
  - helm
---

## etcd 编译
关于etcd编译，[官方](https://github.com/etcd-io/etcd/blob/master/Documentation/dl_build.md)说不需要配置GOPATH的步骤是这样的：
```
$ git clone https://github.com/etcd-io/etcd.git
$ cd etcd
$ ./build
```

实际上呢，是扯淡，etcd自己的main函数是这样写的：
```
package main

import "go.etcd.io/etcd/v3/etcdmain"

func main() {
	etcdmain.Main()
}
```

问题来了，这个v3从哪里来，在一些旧的依赖上，也有这个问题，这个v3莫名其妙啊。而且v2和v3貌似不兼容。。。

所以，正确的方式是下面这样：

```
$ cd $GOPATH/src/go.etcd.io/etcd
$ git clone https://github.com/etcd-io/etcd.git
$ mv etcd v3
$ ./build
```

v3 就是这么来的，拍了脑瓜就给目录改了。。。

## 依赖etcd的其它库编译
因为etcd库改了位置了，然后产生了一系列的依赖找不到。。。一些不用go vendor的go项目注意了，说的就是你们！

下面以编译chartmuseum来说说中间的坑：
1. ``cannot use auth.callOpts (type []"github.com/coreos/etcd/vendor/google.golang.org/grpc".CallOption) as type []"go.etcd.io/etcd/v3/vendor/google.golang.org/grpc".CallOption in argument to auth.remote.AuthEnable`` :

    俩文件的类型不匹配，网上说，需要更改 "github.com/coreos" 为 “go.etcd.io/etcd“ ，改了以后呢，出现另外错误

    按照上面更改以后，出现啥各种包找不到，

    1. 先是``cannot find package "go.etcd.io/etcd/clientv3" in any of``, 然后按照官方client说的：“go get go.etcd.io/etcd/clientv3"
    2. 然后``cannot find package "go.etcd.io/etcd/clientv3" in any of``, 然后按照搜索所得："go get go.etcd.io/etcd"
    3. 运行3以后，报错``cannot find package "go.etcd.io/etcd/v3/etcdmain" in any of``, 这个是etcd的main函数。。。
2. 绕了一大圈，终极改的方法是，将``github.com/coreos`` 更改为 ``go.etcd.io/etcd/v3``，然后：
   1. rm -rf $GOPATH/src/go.etcd.io/etcd
   2. mkdir $GOPATH/src/go.etcd.io/etcd
   3. cd $GOPATH/src/go.etcd.io/etcd
   4. git clone https://github.com/etcd-io/etcd.git
   5. mv etcd v3
3. 照着1/2更改以后，再重新make build，出错。。。。``unknown field 'CAFile' in struct literal of type "go.etcd.io/etcd/v3/pkg/transport".TLSInfo``
   1. 这个原因是：[etcd把TLSInfo的字段CAFile名字给更改了,改成了TrustedCAFile](https://github.com/projectcalico/libcalico-go/issues/917)，彻底无语了
   2. 解决方法：到使用到的etcd文件中CAFile字段给改成TrustedCAFile.

自此问题解决。

总结来看：还是用vendor吧。。。
   
