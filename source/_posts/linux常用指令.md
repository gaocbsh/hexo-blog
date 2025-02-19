---
title: linux常用指令.md
top_img: /img/mountain.png
cover: /img/mountain.png
date: 2024-12-30 13:04:43
tags: linux
---
### 系统源镜像修改
系统源镜像设置文件存储为值在`/etc/apt/source.list` 文件中和`/etc/apt/source.list.d/`目录下。

进入`/etc/apt`目录，编辑`source.list`需要使用sudo指令。执行
`sudo nano source.list`。将不需要的镜像删掉或使用`#`注释掉。

![/img/img.png](img.png)

