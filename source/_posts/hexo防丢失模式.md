
---
layout: post
title: "hexo防丢失模式"
date: 2018-09-06
comments: true
top: true
tags: 
	- hexo
---

## 原因
曾经创建的博客因为本地文件丢失，更换电脑后无法继续使用，此方法为备份，并可在任意电脑克隆使用

## 环境准备
本地环境macOS node v10.1.0 git v2.17.0
## 搭建流程
1. 创建仓库，IceFengQ.github.io；
2. 创建两个分支：master 与 hexo；
3. 设置hexo为默认分支（因为我们只需要手动管理这个分支上的Hexo网站文件）；
4. 使用git clone git@github.com:IceFengQ/IceFengQ.github.io.git拷贝仓库；
5. 在本地IceFengQ.github.io文件夹下通过Git bash依次执行npm install hexo、hexo init、npm install 和 npm install hexo-deployer-git（此时当前分支应显示为hexo）;
6. 修改_config.yml中的deploy参数，修改配置文件分支应为master；
7. 依次执行git add .、git commit -m "..."、git push origin hexo提交网站相关的文件；
8. 执行hexo g -d生成网站并部署到GitHub上。
9. 这样一来，在GitHub上的http://IceFengQ.github.io仓库就有两个分支，一个hexo分支用来存放网站的原始文件，一个master分支用来存放生成的静态网页。

## 详细过程
1,在github上创建IceFengQ.github.io仓库,创建分支hexo,并且设为默认分支

2,本地创建进入blog目录

```
mkdir blog
cd  blog
```
3,克隆仓库当本地

```
git clone git@github.com:IceFengQ/IceFengQ.github.io.git
```
4,修改克隆仓库名称为icefeng,并创建新空目录IceFengQ.github.io作为hexo初始化

```
mv IceFengQ.github.io icefeng
mkdir IceFengQ.github.io
```

5,

```
cd IceFengQ.github.io
npm install hero -g
hexo init
npm install
npm install hexo-deployer-git --save
cp -r ../icefeng/.git ./    #拷贝git仓库配置文件
```
注：此时分支应为hexo

6,修改_config.yml中的deploy参数，修改配置文件分支应为master

```
deploy:
  type: git
  repo: git@github.com:IceFengQ/IceFengQ.github.io.git
  branch: master
```

7,将配置文件提交到github远程仓库

```
 git add .
 git commit -m "next1"
 git push origin hexo
```

8,发布
```
hexo g -d
```
成功后浏览器访问<https://IceFengQ.github.io>，主题更换参考<https://hexo.io/themes/>

**注：所有git仓库中操作都在hexo分支下**

## 日常使用流程

在本地对博客进行修改（添加新博文、修改样式等等）后，通过下面的流程进行管理。

1. 依次执行git add .、git commit -m "..."、git push origin hexo指令将改动推送到GitHub（此时当前分支应为hexo）；
2. 然后才执行hexo g -d发布网站到master分支上。


## 本地资料丢失后的流程

当重装电脑之后，或者想在其他电脑上修改博客，可以使用下列步骤：

1. 使用git clone git@github.com:IceFengQ/IceFengQ.github.io.git拷贝仓库（默认分支为hexo）； 2. 在本地新拷贝的IceFengQ.github.io文件夹下通过Git bash依次执行下列指令：npm install hexo、npm install、npm install hexo-deployer-git（记得，不需要hexo init这条指令）。
