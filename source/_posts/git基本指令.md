---
title: git基本指令
top_img: /img/mountain.png
date: 2024-12-24 20:27:49
tags: git
---

# 基本概念
- 工作区：就是你在电脑里能看到的目录。
- 暂存区：英文叫 stage 或 index。一般存放在 .git 目录下的 index 文件（.git/index）中，所以我们把暂存区有时也叫作索引（index）。
- 版本库：工作区有一个隐藏目录 .git，这个不算工作区，而是 Git 的版本库。

# 新建仓库项目推送
 
    echo "# hexo-blog" >> README.md
    git init
    git add README.md
    git commit -m "first commit"
    git branch -M main
    git remote add origin https://github.com/gaocbsh/hexo-blog.git
    git push -u origin main 

