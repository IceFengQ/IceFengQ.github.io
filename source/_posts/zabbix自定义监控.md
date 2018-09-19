---
layout: post
title: "zabbix自定义监控"
date: 2018-09-13
comments: true
top: true
tags:
	- zabbix
---

## 一，进程监控

1，在要监控的agent端编辑/etc/zabbix/zabbix_agent.conf,增加include选项

```
PidFile=/var/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server=10.0.8.10
ServerActive=10.0.8.10
Hostname=Service
Include=/etc/zabbix/zabbix_agentd.d/*.conf
Timeout=8
```

<!--more-->
2，在/etc/zabbix/zabbix_agentd.d/下定义自定义监控配置文件，例：检测某进程是否挂掉

```
UserParameter=check_vp,/home/my/check_vp.sh
```

测试脚本如下：

```
#!/bin/bash
ps -ef|grep vp|wc -l
```

增加脚本执行权限

```
chmod +x /home/my/check_vp.sh
```

3，重启zabbix_agent

```
systemctl restart zabbix-agent
```

4，zabbix_server端执行zabbix_get检测,此处 **-k** 指定键值

```
$ /usr/bin/zabbix_get -s 10.0.8.12 -k check_vp
1
```
值为1表示程序扔在运行，0表示程序已停止

5，配置server web端

配置-主机-监控项

![监控项](/assets/blogImg/monitor.png)

配置触发器，当值为0时触发报警

![触发器](/assets/blogImg/trigger.png)


报警媒介-增加报警邮箱
动作-配置报警时触发动作

## 二，监控80端口连接数

1，增加established.conf

```
UserParameter=established.count,/home/my/estab.sh
```

脚本如下：

```
#!/bin/bash
netstat -ant |grep ':80' |grep -c ESTABLISHED
##netstat -ant是查看当前连接数，grep ':80'是过滤出80端口，grep -c ESTABLISHED是统计ESTABLISHED有多少行
```

2，重启zabbix-agent客户端

```
systemctl restart zabbix-agent
```

3，server端zabbix_get测试

```
$ /usr/bin/zabbix_get -s 10.0.8.12 -k established.count
```



