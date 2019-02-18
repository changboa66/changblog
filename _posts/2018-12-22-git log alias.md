---
layout: mypost
title: git 学习
categories: [git]
---

2个小时弄出来一个自认为很好看的git log的显示方式
```
git log --graph --pretty="%C(Yellow)%h %C(Cyan)【%an】 %C(reset)%ad %C(Green)(%cr)%C(reset)  %C(reset)%s" --date=format:"%Y-%m-%d %H:%M:%S" -n 12 --name-status
```

![git log](git-log.png)


将命令保存到 `~/.bash_profile` 文件里
```
// 打开文件
vi ~/.bash_profile
// 将短命令添加到文件末尾
git log --graph --pretty="%C(Yellow)%h %C(Cyan)【%an】 %C(reset)%ad %C(Green)(%cr)%C(reset)  %C(reset)%s" --date=format:"%Y-%m-%d %H:%M:%S" -n 12 --name-status
```
