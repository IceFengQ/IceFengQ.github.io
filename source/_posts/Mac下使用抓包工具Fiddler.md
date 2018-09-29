---
layout: post
title: "Mac下使用抓包工具Fiddler"
date: 2018-09-20
comments: true
top: true
tags:
	- MAC
---

参考：(https://www.jianshu.com/p/57ec761cb5a3)

## 环境安装

### 一、Mono安装

首先，Mac下需要使用.Net编译后的程序，需要用到跨平台的方案Mono(现阶段微软已推出跨平台的方案.Net Core，不过暂时只支持控制台程序)。安装程序可以从<https://www.mono-project.com/download/stable/#download-mac>下载

安装完后，在Terminal里执行以下命令：

```
/Library/Frameworks/Mono.framework/Versions/<Mono Version>/bin/mozroots --import --sync
```

<Mono Version>需替换为本地版本文件夹名称，此步是为了从Mozilla LXR上下载所有受信任的root证书，存于Mono的证书库里。root证书能用于请求https地址。

接下来如果想要运行Fiddler，还需要把Mono加入到环境变量中。
<!--more-->

bash环境，编辑.bash_profile文件：**注意末尾版本号**

```
vi ~/.bash_profile
export MONO_HOME=/Library/Frameworks/Mono.framework/Versions/<Mono Version>
export PATH=$PATH:$MONO_HOME/bin
```

zsh环境，编辑.zshrc文件：

```
vi ~/.zshrc
export MONO_HOME=/Library/Frameworks/Mono.framework/Versions/<Mono Version>
export PATH=$PATH:$MONO_HOME/bin
```
Source配置文件使环境变量生效。

### Fiddler的安装运行

从Fiddler官网[https://www.telerik.com/download/fiddler](https://link.jianshu.com/?t=https://www.telerik.com/download/fiddler)下载**fiddler-mac.zip**的压缩包。解压到非中文字符的路径下。

打开Terminal，进入到刚才解压的Fiddler路径，执行命令运行：

```
sudo mono Fiddler.exe
```

如果出现如下错误：

```
NameError: name 'lldb' is not defined
```

怎追加参数

```
sudo mono --arch=32 Fiddler.exe
```

![fidder](/assets/blogImg/fidder.png)

## 一些问题

现在Fiddler在Mac下还只是Beta1版，所以会有很多问题，比如：

- 界面拉伸或缩小，视图不会自动重新渲染
- 有些HTTPS站点无法访问
- TLS 1.1和1.2无法支持
- SSL/TLS的握手不正常
- 软件无法自动更新
- 只有60天的使用期限，到期后需要重新更新
