---
layout: post
title: "linux笔记: ubuntu下passwd和chpasswd问题"
subtitle: "chpasswd: (user admin) pam_chauthtok() failed, error/passwd: Module is unknown"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - docker
  - linux
---

## chpasswd和passwd出错
* ``chpasswd: (user admin) pam_chauthtok() failed, error``
* ``passwd: Module is unknown``

## 出错原因
缺少pam的某一个库

## 解决方案
```
:~# apt-cache search pam | grep crack
libpam-cracklib – PAM module to enable cracklib support
:~# apt-get install libpam-cracklib
```

## 参考
[https://goinggnu.wordpress.com/2015/06/04/solution-for-passwd-module-is-unknown-issue-in-ubuntu/](https://goinggnu.wordpress.com/2015/06/04/solution-for-passwd-module-is-unknown-issue-in-ubuntu/)
