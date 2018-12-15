---
title: unison 同步配置详解
date: 2018-01-30 22:16:30
categories: centos6
tags:
  - centos6
  - unison
---

**前提，两文件系统可以  免秘钥ssh登录**
1、两边都安装unison，在一边新建配置文件
```
vim /home/srmweb/.unison/webapp.prf
```

配置参数（附件有）
单项同步，以force目录为基准，force=a， a是什么，b就是什么
反之#force为双向同步，a+b+，a-b-，b+a+，b-a-
​
```
batch = true  #表示全自动模式，接受缺省动作，并执行
fastcheck = true 
#表示同步时使用文件的创建时间来比较两地文件，如果这个选项为false，
#unison则将比较两地文件的内容.建议设置为true
group = false  #保持同步过来的文件组信息,注意，同步目录是否一个组
ignore = Path WEB-INF/aurora.plugin.quartz/*  #忽略文件
log = true 
logfile = /home/srm23/unison/logs/webapp.log #记得创建日志路径
maxthreads = 300
owner = false   #保持同步过来的文件属主,注意，同步目录是否一用户
perms = -1   #保持同步过来的文件读写权限
repeat = 60  #间隔60秒后,开始新的一次同步检查
retry = 3    #失败重试
root = /usr/srm23/webapp/webRoot    #这个是本地的
root = ssh://mascloud@192.168.116.22//home/mascloud/webapp/Web    #这个是远程的
#force = /usr/srm23/webapp/webRoot    # 以本地的这个目录为准
#force是基准目录，单双向同步选择
​
rsync = true
silent = false   #终端中不显示任何信息，除非出现错误
times = true    #同步修改时间
xferbycopying = true    #不变目录，扫描 时可以忽略
```


2、编写运行脚本（附件）

```
新建或编辑
/home/srmweb/unison/scripts/webapp.sh
检测unison状态
#!/bin/bash
result=`ps -ef | grep "/usr/bin/unison webapp" | grep -v grep`;
if [ -z "$result" ];then
/usr/bin/unison webapp;
fi
```

3、设置定时任务

```
crontab -e
​
*/1 * * * * sh /home/srmweb/unison/scripts/webapp.sh >/dev/null 2>&1
```
