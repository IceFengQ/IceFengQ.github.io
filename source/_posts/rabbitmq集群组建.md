---
layout: post
title: "rabbitmq集群组建"
date: 2018-09-20
comments: true
top: true
tags:
	- rabbitmq
---

参考：(https://segmentfault.com/a/1190000010702020)

**要求与考虑**

主要是高可用方案

实际使用rabbitmq是需要做数据持久化的，持久化到本地磁盘，大小可扩展

数据无论如何不能丢，还要提前做好扩容方案，节点扩展是否用弹性扩容

**前提准备**：

三台局域网阿里云ecs实例,slb代理转发

**环境**：

```
CentOS Linux release 7.5.1804
Rabbitmq 3.7.7    
erlang-21.1.1
```
<! --more-->
一，单机准备

程序安装过程：

1,修改hosts并关闭防火墙

```systemctl stop firewalld.service
systemctl stop firewalld.service
```

2,添加repo并安装依赖erlang

To use Erlang 21.x on CentOS 7:

\# In /etc/yum.repos.d/rabbitmq-erlang.repo

```[rabbitmq-erlang]
[rabbitmq-erlang]
name=rabbitmq-erlang
baseurl=https://dl.bintray.com/rabbitmq/rpm/erlang/21/el/7
gpgcheck=1
gpgkey=https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
repo_gpgcheck=0
enabled=1

yum install erlang -y    
#卸载旧版及依赖  rpm -qa|grep erlang|xargs rpm -e    
```

3,安装rabbItmq-server

```
wget <https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.7/rabbitmq-server-3.7.7-1.el6.noarch.rpm>

sudo yum install socat   #安装依赖

sudo rpm -ivh rabbitmq-server-3.7.7-1.el6.noarch.rpm     #安装版本3.7.7
```

后台启动

```
rabbitmq-server -detached
```

查看log 

less  /var/log/rabbitmq/rabbit@node1.log

```
 node           : rabbit@node1
 home dir       : /var/lib/rabbitmq
 config file(s) : (none)
 cookie hash    : 1nnqoRN63vVb/pyFCOhLOw==
 log(s)         : /var/log/rabbitmq/rabbit@test-203.log
                : /var/log/rabbitmq/rabbit@test-203_upgrade.log
 database dir   : /var/lib/rabbitmq/mnesia/rabbit@test-203
```

查看rabbitmq状态 

```
rabbitmqctl status
```

停止服务

```
systemctl status rabbitmq-server.service
```



二，集群环境组建

通过 Erlang 的分布式特性（通过 magic cookie 认证节点）进行 RabbitMQ 集群，各 RabbitMQ 服务为对等节点，即每个节点都提供服务给客户端连接，进行消息发送与接收。

这些节点通过 RabbitMQ HA 队列（镜像队列）进行消息队列结构复制。本方案中搭建 3 个节点，并且都是磁盘节点（所有节点状态保持一致，节点完全对等），只要有任何一个节点能够工作，RabbitMQ 集群对外就能提供服务。

```
scp  /var/lib/rabbitmq/.erlang.cookie  node:/var/lib/rabbitmq/  # 到两台集群机，注意权限问题
```

***注意事项*** 

cookie在所有节点上必须完全一样，同步时一定要注意。
erlang是通过主机名来连接服务，必须保证各个主机名之间可以ping通。可以通过编辑/etc/hosts来手工添加主机名和IP对应关系。如果主机名ping不通，rabbitmq服务启动会失败。

运行各节点

```
$ rabbitmqctl stop
$ rabbitmq-server -detached 
```

分节点加入集群命令（此处node1为主节点）

node2

```
node2 $ rabbitmqctl stop_app            # 停止rabbitmq服务

node2 $ rabbitmqctl join_cluster rabbit@node1    # node2和node1构成集群, node2必须能通过node1的主机名ping通

node2 $ rabbitmqctl start_app            # 开启rabbitmq服务
```

node3

```
node3 $ rabbitmqctl stop_app            # 停止rabbitmq服务

node3 $ rabbitmqctl join_cluster --ram rabbit@node1    # node3和node1构成集群, node2必须能通过node1的主机名ping通

node3 $ rabbitmqctl start_app            # 开启rabbitmq服务
```

设置节点状态

其中`–ram`指的是作为内存节点,要是想做为磁盘节点的话,就不用加`–ram`这个参数了

**加入内存节点集群**,默认是磁盘节点

```
node2 # rabbitmqctl join_cluster --ram rabbit@node1
```

**在RabbitMQ集群里，必须至少有一个磁盘节点存在**

更改节点属性

```
node2 $ rabbitmqctl stop_app  # 停止rabbitmq服务

node2 $ rabbitmqctl change_cluster_node_type ram # 更改节点为内存节点
Turning rabbit@node2 into a ram node

node2 $ rabbitmqctl change_cluster_node_type disc # 更改节点为磁盘节点
Turning rabbit@node2 into a disc node

node2 $ rabbitmqctl start_app # 开启rabbitmq服务
```

**查看集群状态**

执行完之后分别在每台机器上查看节点状态

```
# rabbitmqctl cluster_status
Cluster status of node rabbit@node1
[{nodes,[{disc,[rabbit@node1,rabbit@node2,rabbit@node3]}]},
 {alarms,[{rabbit@node2,[]},{rabbit@node3,[]}]}]
```

第一行是集群中的节点成员，disc表示这些都是磁盘节点

第二行是正在运行的节点成员

**设置镜像队列高可用模式**

镜像队列概念：

镜像队列可以同步queue和message，当主queue挂掉，从queue中会有一个变为主queue来接替工作。

镜像队列是基于普通的集群模式的,所以你还是得先配置普通集群,然后才能设置镜像队列。

镜像队列设置后，会分一个主节点和多个从节点，如果主节点宕机，从节点会有一个选为主节点，原先的主节点起来后会变为从节点。

queue和message虽然会存在所有镜像队列中，但客户端读取时不论物理面连接的主节点还是从节点，都是从主节点读取数据，然后主节点再将queue和message的状态同步给从节点，因此多个客户端连接不同的镜像队列不会产生同一message被多次接受的情况。

**在任意节点执行**

```rabbitmqctl set_policy ha-all '^' '{"ha-mode":"all"}'
rabbitmqctl set_policy ha-all '^' '{"ha-mode":"all"}'
```

**web界面管理**

启动web管理插件，端口15672

```
rabbitmq-plugins enable rabbitmq_management 
```

**集群重启**
集群重启时，最后一个挂掉的节点应该第一个重启，如果因特殊原因（比如同时断电），而不知道哪个节点最后一个挂掉。可用以下方法重启：
先在一个节点上执行,然后在其他节点直接start，查看状态
```
$ rabbitmqctl force_boot
$ service rabbitmq-server start
```


**用户与安全**

处于安全的考虑，guest这个默认的用户只能通过`http://localhost:15672` 来登录，其他的IP无法直接使用这个账号。 这对于服务器上没有安装桌面的情况是无法管理维护的，除非通过在前面添加一层代理向外提供服务，这个又有些麻烦了，这里通过配置文件来实现这个功能

 初始**添加用户**并授予权限

```
rabbitmqctl add_user root 123456

rabbitmqctl set_permissions -p / root ".*" ".*" ".*" 

rabbitmqctl set_user_tags root administrator
```


删除用户

```
$ rabbitmqctl  delete_user ***
Deleting user "***"
```

修改密码

```
rabbitmqctl  change_password  root 654321 #newpasswd
Changing password for user "root"
```

用户授权

```
rabbitmqctl set_permissions -p / root ".*" ".*" ".*" #root用户可管理vhost "/"
```

查看用户列表

```
rabbitmqctl list_users
```

清除权限

```
rabbitmqctl  clear_permissions  -p /  *** #*处为用户
```

*提示*：以上操作均可使用web界面管理

