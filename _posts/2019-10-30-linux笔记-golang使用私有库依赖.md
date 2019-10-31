---
layout: post
title: "golang笔记: 使用go mod下载私有库依赖"
subtitle: "go mod使用"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - golang
  - linux
---

## 设置和下载
1. 在go.mod中添加依赖的git地址和版本（commitid）
  ```
  gitlab.com/groupName/projectName.git commitid
  ```
2. 在自己的代码中手动添加``gitlab.com/groupName/projectName.git``，相关使用
  ```
  import gitlab.com/groupName/projectName
  func main() {
    projectName.Hello("World!")
  }
  ```
3. 设置：
  ```bash
  $ git config --global url."https://username:password@gitlab.com".insteadOf "https://gitlab.com/"
  ```
4. 使用``go mod``下载相关依赖：
  ```
  $ go mod vendor
  ```

## 参考
[https://segmentfault.com/a/1190000017973252](https://segmentfault.com/a/1190000017973252)
