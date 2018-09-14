
---
layout: post
title: "git 包含git子文件夹怎么push？"
date: 2018-09-06
comments: true
top: true
tags: 
	- git
---


### git 包含git子文件夹怎么push？

情况如下：

当原有git仓库中因需求克隆其他项目时，这时git仓库中就包含了子项目，此时像远程仓库提交需要单独指明git子仓库，然后使用git status查看是否提交。
例如：

```
git status #查看需要项目
git add <子项目>
```

如果此时子项目已经无法add，那么原有可能是该文件夹已经被纳入了版本管理中，
需要先清空该文件夹的本地缓存然后再添加才行。

```
git rm -r --cached path
```