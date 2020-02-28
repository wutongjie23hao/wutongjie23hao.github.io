---
layout: post
title: "linux笔记: sshd错误处理"
subtitle: "关于sshd常见错误总结"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - linux
---

# sshd错误处理

## 1. ``/var/empty/sshd`` 文件错误

错误：

```
SSH fails because /var/empty/sshd is not owned by root and is not group-writable or world-writable
```

原因： 

/var/empty/sshd 这个文件owner要是root，且不加入其它组，不能被开放写权限。

处理：

```bash
# chmod 755 /var/empty/sshd;chown root:root /var/empty/sshd;service sshd restart
```


