---
title: linux常用指令.md
top_img: /img/background.png
cover: /img/unhappy.jpg
date: 2024-12-30 13:04:43
tags: linux
---
### 系统源镜像修改
系统源镜像设置文件存储为值在`/etc/apt/source.list` 文件中和`/etc/apt/source.list.d/`目录下。

进入`/etc/apt`目录，编辑`source.list`需要使用sudo指令。执行
`sudo nano source.list`。将不需要的镜像删掉或使用`#`注释掉。

![/img/img.png](../img/img.png)

### dpkg
安装软件：`sudo dpkg -i `

卸载软件：
- 首先使用`sudo dpkg -l 软件名*`查找已安装或未安装完成的相关组件
- 然后使用`sudo dpkg -P 组建名`卸载已安装组件

