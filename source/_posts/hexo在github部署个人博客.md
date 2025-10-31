---
title: hexo在github部署个人博客
date: 2025-10-31 11:31:12
tags:
---
1.首先在github部署两个仓库A和B，A用来显示博客内容，B是博客源代码
2.我用wsl的linux作为写作环境，创建两种登陆方式，ssh的登陆方式给hexo，git的登陆方式用token登陆

- ssh登陆方式
``` bash
ssh-keygen -t rsa -C "youremail",
cat ~/.ssh/id_rsa_pub   #复制展示的内容
```
 登陆github,个人头像->settings->SSH and GPG keys->new ssh key ->粘贴上面cat的内容

- token登陆方式
  登陆github,个人头像->settings-> Developer settings -> personal access tokens ->tokens(classic)->generate new token ->generate new token(classic)->生成ghp开头的密钥
``` bash
git remote set-url origin https://ttkk1024:ghp_XXXXXXXXXXXXXXXXXXXXXXXXXXX@github.com/ttkk1024/ttkk1024.github.io-src.git
git push -u origin main
```
切换到blog目录，
``` bash
hexo n "new blog"
code hexo\blog\source\_posts\new blog.md
hexo g
hexo d

git init
git add -A
git commit -m "init"
git push -u origin main
```

这样就能ttkk1024.github.io看到new blog的博客，ttkk1024.github.io-src看到hexo的源代码

为什么创建两个仓库，是因为hexo不能pull只能push，这样就无法获取hexo的原始内容