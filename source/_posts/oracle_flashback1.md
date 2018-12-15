---
title: oracle 恢复table删除数据 恢复package（使用闪回）
date: 2018-06-26 22:24:30
categories: oracle
tags:
  - oracle
  - flashback
---

```
	好久没写东西了，今天写一篇凑个数吧，来公司一年多了，感觉自己到了一个小瓶颈期了。 以前每天很多新东西，都是忙着学，感觉没时间写博客总结一下，大部分都是写笔记，现在又是没东西可以写，每天干着95%都是重复的工作，大部分时间在运维，但我内心是把自己当做dba的，毕竟当初老大把我从java开发拉倒系统组^_^
	上次一个技术把表中的数据删除，这次是另一个技术把正确的包给覆盖了，我给恢复了，哈哈哈---有用的话看一下
```
参考博客：https://blog.csdn.net/wyzxg/article/details/6761458
`虽然包恢复原理一样，但是操作差异一点，因为我的数据是12.2c的`

###正文1：表数据删除恢复
```sql
--1、查询删除数据时间点之前的数据
select * from 表名 as of timestamp to_timestamp('2016-08-11 16:12:11','yyyy-mm-dd hh24:mi:ss')； --（若没有数据 ，将时间继续提前）
--2、启用行移动功能
alter table 表名 enable row movement;
--3、恢复数据（激动人心的时刻）
flashback table 表名 to timestamp to_timestamp('2016-08-11 16:12:11','yyyy-mm-dd hh24:mi:ss');
大功告成，数据恢复成功；
```
###正文2：包被覆盖错了，想要闪回package

`注意这里需要使用dba权限，否则查看不到`
```sql
--错误包：XXX_PKG 正确时间：中午12点
SELECT OBJECT_ID
  FROM all_objects 
 WHERE OBJECT_NAME = 'XXX_PKG';
 --如果包直接被删除了，不是被覆盖，那只能先闪回查询对象表
 SELECT OBJECT_ID
  FROM all_objects AS OF TIMESTAMP TO_TIMESTAMP('2018-06-26 12:00:00', 'YYYY-MM-DD HH24:MI:SS')
 WHERE OBJECT_NAME = 'XXX_PKG';
 --查到2个对象id，一个包头，一个包体
93163
94602
--根据对象id查询包头和包体内容
SELECT source
  FROM source$ AS OF TIMESTAMP TO_TIMESTAMP('2018-06-26 12:00:00', 'YYYY-MM-DD HH24:MI:SS')
 where obj# = 93163;
 SELECT source
  FROM source$ AS OF TIMESTAMP TO_TIMESTAMP('2018-06-26 12:00:00', 'YYYY-MM-DD HH24:MI:SS')
 where obj# = 94602;
 --然后确认没问题，用查到的sql内容重新覆盖就好
```

```
注意：一般package body的内容比较多，怎么复制下来呢？
spool  /home/oracle/xxx_pkg.sql

SELECT source
  FROM source$ AS OF TIMESTAMP TO_TIMESTAMP('2018-06-26 12:00:00', 'YYYY-MM-DD HH24:MI:SS')
 where obj# = 93163;

spool off
exit
ll /home/oracle/
```

###知识点
简单总结一下，这里就是利用了oracle的闪回查询功能，非常的简单，一看就懂了，但是oracle闪回有很多种包括删除表和数据库，闪回事务等等。
	最重要的是要理解
	SQL> show parameter undo_retention;

NAME				     TYPE			       VALUE
------------------------------------ --------------------------------- ------------------------------
undo_retention			     integer			       900
这是默认的900，其中undo保留900s，后面undo磁盘不够就可能回收，那样闪回可能会失败，当然也可以强制保留更久，更详细的解释自行百度，下面是我的对闪回的为知笔记的一点点简单记录：http://8840743b.wiz03.com/share/s/28g7gX2-0A_U21N5531o6XTR1JE9nV1SfQc52nk8DX1zamut

#####谢谢阅读，有兴趣交流的留言

