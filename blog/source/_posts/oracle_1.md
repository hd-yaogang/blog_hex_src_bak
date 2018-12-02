---
title: oracle 配置 自启动 和 关闭
date: 2018-01-30 22:16:30
categories: centos6
tags:
  - oracle
  - centos6
---

*今天在看oracle自启动脚本，突然有点时间，总结一下！！！*

*第一次写博客，大家随便看看就好，有错误麻烦提醒下，不喜欢别喷，主要是锻炼自己，形成写博客的好习惯。*

*刚毕业，现在还没转正，在干运维和自学dba，零零碎碎学了很多东西，很早就想写博客，但没时间和懂得不够深刻全面，写了很多简单的笔记，所以从今天开始练习写博客，养成思考和坚持学习的好习惯*

---
## 系统服务（rhel6/oracle linux6/centos6 都一样）
```shell
[oracle@srm-yg ~]$ service oracle start        # 将oracle启动加到系统服务
```
##### 这个oracle服务是哪里来的，系统有哪些服务？
```
[oracle@srm-yg ~]$ ls /etc/init.d/    #查看系统服务
```
这下面所有的可执行文件都是系统服务，你可以使用 service xxx start/stop/...  来使用它，但系统启动和关闭时不会开启和关闭所有服务。

** 例子：           添加 oracle 服务 和 tomcat 服务**
```shell
[root@srm-yg yyy]# service tomact start               # 这里还未将tomcat添加为系统服务
tomact: unrecognized service

[root@srm-yg test_service]# ls
tomcat
[root@srm-yg test_service]# cp tomcat/bin/catalina.sh  /etc/init.d/tomcat
[root@srm-yg test_service]# vim /etc/init.d/tomcat               # 在第一行后面添加：
#!/bin/sh
# chkconfig: 12345 80 90
# description: srm-yg haha
#CATALINA_HOME=/tmp/test_service/tomcat
保存退出

[root@srm-yg logs]# service tomcat start
[root@srm-yg logs]# curl -I localhost:8080
HTTP/1.1 200 OK
[root@srm-yg logs]# service tomcat stop
[root@srm-yg logs]# service tomcat run           # 这些命令都是可以的
```
**这里有人会也许会奇怪，一般不是就start、stop、status、restart、reload吗？怎么还有个run？**

-----其实服务名后面跟run，跟什么都是可以的，这就得看你脚本里面怎么指定了，上面我们是把 tomcat/bin/catalina.sh 文件复制为 /etc/init.d/tomcat 文件，而catalina.sh脚本是可以接收run、debug等参数的。

service xxx服务 start，这里的服务都是在 /etc/init.d/目录下面，所以当你想要使用简洁的 service服务命令去启动 某个服务的话，你只需要定义一个脚本，放到 /etc/init.d/ 目录下面，添加执行权限，脚本中再指定接收的位置参数，可以接收 start、stop、reload、hh、xxx 等等自定义参数。

---

**那系统开机和关机执行的服务又是怎么定义呢？**


## 开机 和 关机
```
[root@srm-yg init.d]# ls ..            # 这些是系统启动的服务指定的地方
init.d  rc  rc0.d  rc1.d  rc2.d  rc3.d  rc4.d  rc5.d  rc6.d  rc.local  rc.sysinit

[root@srm-yg init.d]#cat /etc/inittab        #  这是系统默认的运行级别
# Default runlevel. The runlevels used are:
#   0 - halt (Do NOT set initdefault to this)
#   1 - Single user mode
#   2 - Multiuser, without NFS (The same as 3, if you do not have networking)
#   3 - Full multiuser mode
#   4 - unused
#   5 - X11
#   6 - reboot (Do NOT set initdefault to this)
# 
id:3:initdefault:
```
----linux具有7个运行级别：0-6

**0是关机，1是单用户，3是多用户，6是重启，这几个使我们常用的。**

在 /etc/inittab 文件中定义了系统开机时的初始化运行级别，我虚拟机没装X图形服务，所以默认的初始化为3，如果你装了桌面，一般默认是5.

当系统启动的时候会执行rc文件，这个脚本会检测当前的运行级别，然后会执行对应级别的rcX.d目录下面的SXXxx文件，例如：S80tomcat ，当系统关闭时，又会执行KXX，K开头的脚本。rc0.d-rc6.d这7个目录下的文件都是从 /etc/rc.d/init.d 目录下链接过来的，S代表系统启动时执行，K代表系统关闭时执行。

**记得刚刚我们在 /etc/init.d/tomcat 文件中加入的3行命令吗？**
```
# chkconfig: 12345 80 90
# description: srm-yg haha
1个时服务定义，1个时服务注释
#CATALINA_HOME=/tmp/test_service/tomcat
```
**别以为它只是注释，它可以配置 chkconfig 命令来使用， 12345 代表12345运行级别为on，未写明的0和6为 off **

刚刚我们将 tomcat 的启动和停止 变成了一项系统服务，但只能手动运行，并不是开机自动运行的，现在执行下面这个命令：
```
[root@srm-yg init.d]# chkconfig --add tomcat    # 将 tomcat 服务加入运行级别
```
通过# chkconfig: 12345 80 90 的指定，系统会将我们的 /etc/init.d/tomcat 连接到rcX.d的各个目录下面，指定的为on，未指定为off。所以，在 ( rc1.d  rc2.d  rc3.d  rc4.d  rc5.d )下面会有一个从  /etc/rc.d/init.d/tomcat 链接过来的 S80tomcat 文件，在 rc0.d 和 rc6.d 目录下会有一个 K90tomca 文件，80 90分别代表启动和停止的顺序，序号越小越先执行，S代表启动时执行，K代表关闭时执行，这样你在运行12345级别时一开机系统就会按顺序，到80就启动tomcat服务了，然后运行0和6是系统会按顺序停止tomcat服务。
>执行S80tomcat时，系统会调用tomcat服务，并自动传递一个start参数
执行K90tomcat时，系统调用tomcat服务，自动传递一个stop参数


>chkconfig --add xxx服务      
它会安装脚本定义的级别和开关机顺序将服务添加到系统管理

>我们还可以使用 chkconfig  --level 1 tomcat off 命令来更改，这样系统会把rc1.d目录下的S80tomcat，变成K90tomcat

**还有一个文件可以做到开机启动的方法**
````
[root@srm-yg init.d]# vim /etc/rc.local     # 在文件中执行启动tomcat的shell命令，如下：
 /tmp/test_service/tomcat/bin/startup.sh
```

一些不重要的开机自启服务可以放在这里，但它不是系统服务，需要在rcX.d里面所有脚本按顺序执行完成后才会执行这里，而且不会关闭前执行关闭，但优点是简单。

---

## oracle配置 自启动 和 关闭
1. 修改oracle系统配置文件
```
[oracle@srm-yg ~]$ vim /etc/oratab      #  将 N 改为 Y
```

2. 在 /etc/init.d/ 目录下添加 oracle启动脚本（我们公司大佬写的）
```
#!/bin/sh
# chkconfig: 2345 80 10
# description: Oracle auto start-stop script.
#
# Set ORA_HOME to be equivalent to the $ORACLE_HOME
# from which you wish to execute dbstart and dbshut;
#
# Set ORA_OWNER to the user id of the owner of the
# Oracle database in ORA_HOME.
ORA_HOME=/home/oracle/product/12.2.0/db_1
ORA_OWNER=oracle
if [ ! -f $ORA_HOME/bin/dbstart ]
then
    echo "Oracle startup: cannot start"
    exit
fi
case "$1" in
'start')
# Start the Oracle databases:
echo "Starting Oracle Databases ... "
echo "-------------------------------------------------" >> /var/log/oracle
date +" %T %a %D : Starting Oracle Databases as part of system up." >> /var/log/oracle
echo "-------------------------------------------------" >> /var/log/oracle
su - $ORA_OWNER -c "$ORA_HOME/bin/dbstart" >>/var/log/oracle
echo "Done"
# Start the Listener:
echo "Starting Oracle Listeners ... "
echo "-------------------------------------------------" >> /var/log/oracle
date +" %T %a %D : Starting Oracle Listeners as part of system up." >> /var/log/oracle
echo "-------------------------------------------------" >> /var/log/oracle
su - $ORA_OWNER -c "$ORA_HOME/bin/lsnrctl start" >>/var/log/oracle
echo "Done."
echo "-------------------------------------------------" >> /var/log/oracle
date +" %T %a %D : Finished." >> /var/log/oracle
echo "-------------------------------------------------" >> /var/log/oracle
touch /var/lock/subsys/oracle
;;
'stop')
# Stop the Oracle Listener:
echo "Stoping Oracle Listeners ... "
echo "-------------------------------------------------" >> /var/log/oracle
date +" %T %a %D : Stoping Oracle Listener as part of system down." >> /var/log/oracle
echo "-------------------------------------------------" >> /var/log/oracle
su - $ORA_OWNER -c "$ORA_HOME/bin/lsnrctl stop" >>/var/log/oracle
echo "Done."
rm -f /var/lock/subsys/oracle
# Stop the Oracle Database:
echo "Stoping Oracle Databases ... "
echo "-------------------------------------------------" >> /var/log/oracle
date +" %T %a %D : Stoping Oracle Databases as part of system down." >> /var/log/oracle
echo "-------------------------------------------------" >> /var/log/oracle
su - $ORA_OWNER -c "$ORA_HOME/bin/dbshut" >>/var/log/oracle
echo "Done."
echo ""
echo "-------------------------------------------------" >> /var/log/oracle
date +" %T %a %D : Finished." >> /var/log/oracle
echo "-------------------------------------------------" >> /var/log/oracle
;;
'restart')
$0 stop
$0 start
;;
esac
```

3. 更改脚本权限
```
[root@srm-yg init.d]# chmod 755 oracle
```
4. 添加到 系统自启 和 关闭
```
[root@srm-yg init.d]# chkconfig --add oracle
```


***
**完毕，谢谢，其中可以还有些原理解释的不好，需要读者慢慢去揣摩脚本。。**
