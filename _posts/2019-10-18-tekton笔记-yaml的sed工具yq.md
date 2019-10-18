---
layout: post
title: "yaml的sed工具yq介绍"
subtitle: "yq进行yaml文件的更改"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - tekton.build
  - golang
---

## yq介绍

yq的目标是yaml文件的sed工具，就像json的sed工具[jq](https://github.com/stedolan/jq) 一样。

* yq的git地址：[https://github.com/mikefarah/yq](https://github.com/mikefarah/yq)
* yq的教程地址：[yq教程](http://mikefarah.github.io/yq/)

## yq常用命令
```
yq is a lightweight and portable command-line YAML processor. It aims to be the jq or sed of yaml files.

Usage:
  yq [flags]
  yq [command]

Available Commands:
  delete      yq d [--inplace/-i] [--doc/-d index] sample.yaml a.b.c
  help        Help about any command
  merge       yq m [--inplace/-i] [--doc/-d index] [--overwrite/-x] [--append/-a] sample.yaml sample2.yaml
  new         yq n [--script/-s script_file] a.b.c newValue
  prefix      yq p [--inplace/-i] [--doc/-d index] sample.yaml a.b.c
  read        yq r [--doc/-d index] sample.yaml a.b.c
  write       yq w [--inplace/-i] [--script/-s script_file] [--doc/-d index] sample.yaml a.b.c newValue

Flags:
  -h, --help      help for yq
  -t, --trim      trim yaml output (default true)
  -v, --verbose   verbose mode
  -V, --version   Print version information and quit

Use "yq [command] --help" for more information about a command.
```
