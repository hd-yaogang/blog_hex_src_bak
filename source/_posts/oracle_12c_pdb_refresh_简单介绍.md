---
title: oracle 12c pdb refresh 简单介绍
categories: oracle12c
tags:
  - oracle12c
  - refresh
  - pdb
  - clone
---
## oracle 12c pdb refresh 简单介绍



#### 闲话

```
   好久没有写博客，虽然忙但时间还是有的，就是感觉没东西写，学到的东西小的记笔记，大的写成文档，没有那种自己领悟很深的东西来写博客，就写一下pdb refresh凑个篇数 ^_^
   感兴趣的看一下
```

:smiling_imp:

#### 正文

> 环境：
> 
>        oracle12.2单实例clone到12.2 rac，我需要把测试环境pdb复制到正式，但是他们测试系统还没有开发结束就急着上线，后面开发的数据库代码需要技术手动插入正式，听说过pdb刷新，就想能不能就解决这个问题，于是学习了一波...结果搞明白了，但是用处不是太大，哈哈:sob:

---

**克隆**

> 先决条件：
> 
>       1.目标位置具有建库权限
> 
>       2.源pdb启用归档或者read only
> 
>       3.源pdb使用本地undo
> 
>       4.尽量目标库为al32utf8
> 
>       6.有dblink连接
> 
>       7.刷新模式必须为2个不同的cdb容器中的pdb
> 
>          其他不重要...

```sql
--源库开启归档或者read only
SQL> alter pluggable database open read only;

--目标库创建源库的dblink,这里使用的tns：srm_test需要提前创建好
SQL> conn /as sysdba;
SQL> create public database link srmtest_dblink connect to srm identified by xxxx using 'srm_test';  

--进行hot远程克隆,并且制定刷新模式为 手动
SQL> create pluggable database srm_pro from srm_test@srmtest_dblink refresh mode manual;

--克隆完成，打开目标pdb
SQL> alter pluggable database srm_pro open; --默认为open read write模式
```

> 这里会显示修改完成，即目标pdb打开，但是show pdbs一看，还是srm_pro还是mounted状态，再来一次，不报错，但还是死活打不开。--官方说明，必须read only模式打开

```sql
--于是执行,果然打开了
SQL> alter pluggable database srm_pro open read only instances = all;
```

**刷新**

```sql
--于是执行,果然打开了
SQL> alter pluggable database srm_pro open read only instances = all;

--测试刷新，现在源库新建一张表，然后到目标库刷新，再查询目标库是否同步数据
SQL> conn xx/xxx@srm_test;
SQL> create table t1(c1 number);insert into t1 values(123);
SQL> desc t1; select * from t1; --查看新建的表

--目标库进行刷新
SQL> alter pluggable database srm_pro close immediate instances = all;
SQL> alter session set container = srm_pro;
SQL> alter pluggable database refresh;
SQL> alter pluggable database open read only;
SQL> conn xxx/xxxx@srm_pro;
SQL> desc t1;select * from t1; --已经查询到在源库新建的测试数据表t1
```

**总结**

```
刷新模式：3种  --注意，刷新模式可以修改，但是从刷新模式变成不刷新后就不能再改回刷新模式了
-- 手动刷新
SQL> CREATE PLUGGABLE DATABASE pdb5_ro FROM pdb5@clone_link
  REFRESH MODE MANUAL;

-- 60分钟定时刷新
SQL> CREATE PLUGGABLE DATABASE pdb5_ro FROM pdb5@clone_link
  REFRESH MODE EVERY 60 MINUTES;

-- 不刷新，默认创建pdb时不加刷新参数就是不刷新
SQL> CREATE PLUGGABLE DATABASE pdb5_ro FROM pdb5@clone_link
  REFRESH MODE NONE;
SQL> CREATE PLUGGABLE DATABASE pdb5_ro FROM pdb5@clone_link;
```

> 思考：
> 
> 为什么只能read only呢？我想可能是为了保证目标数据于原数据库一致，不会数据冲突吧。

```
修改pdb的刷新模式：
-- 修改自动刷新时间
SQL> ALTER PLUGGABLE DATABASE pdb5_ro REFRESH MODE EVERY 60 MINUTES;
SQL> ALTER PLUGGABLE DATABASE pdb5_ro REFRESH MODE EVERY 120 MINUTES;

-- 修改自动刷新为手动刷新
SQL> ALTER PLUGGABLE DATABASE pdb5_ro REFRESH MODE MANUAL;

-- 修改为不刷新模式，注意：就无法再更改为手动或者自动刷新模式了
SQL> ALTER PLUGGABLE DATABASE CLOSE IMMEDIATE;
SQL> ALTER PLUGGABLE DATABASE pdb5_ro REFRESH MODE NONE;
SQL> ALTER PLUGGABLE DATABASE OPEN;
```

#### 结语

> 本来想着，等测试开发结束，帮他们直接刷新到正式即可，但是不行，因为要保留刷新模式的话必须正式pdb一直处于read only状态，无法测试和预使用，有点小难受。:stuck_out_tongue_closed_eyes:
> 
> oracle官方说明是，刷新模式可以用来做报表系统查询数据，可以用来源pdb克隆。




