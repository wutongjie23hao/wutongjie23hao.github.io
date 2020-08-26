---
layout: post
title: "linux笔记: pyinstall打包python应用"
subtitle: "pyinstall打包python应用，包括相关依赖"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - linux


---

# pyinstall打包应用

## 基本步骤

1. 安装pyinstall:

   ```
   > pip install pyinstall
   ```

2. 打包应用及依赖

   ```
   > pyinstall -F -p /usr/lib/site-packages1:/usr/lib/site-packages2 app.py
   ```

   参数选项说明：

   - ``-F``: 生成一个可执行文件。
   - ``-p``: 打包依赖，如果没有这个参数，在执行可执行文件的时候，可能会报指定模块不存在的错误。

> python2的时候，使用 pip install pyinstaller; pyinstaller -F -p /usr/lib/site-packages1:/usr/lib/site-packages2 app.py
## 其它

| 参数/选项                   | 意义                                                     |
| --------------------------- | -------------------------------------------------------- |
| -F, --onefile               | 产生单个可执行文件                                       |
| -D, --onedir                | 在dist目录下生成一个目录作为可执行程序                   |
| -a, --ascii                 | 不包含Unicode字符集支持                                  |
| -d, --debug                 | debug版本的可执行文件                                    |
| -w, --windowed, --noconsolc | 指定程序运行时不显示命令行窗口(仅windows有效)            |
| -c, --nowindowed, --console | 指定使用命令行窗口运行程序(仅windows有效)                |
| -o DIR, --out=DIR           | 指定可执行文件的生成路径，默认当前目录                   |
| -p DIR, --path=DIR          | 设置python导入模块的路径，可使用系统分隔符来分割多个目录 |
| -n NAME, --name=NAME        | 可执行文件或目录的名字                                   |
| -h, --help                  | 查看该模块的帮助信息                                     |

