---
layout: post
title:  SQL 精编
author: wilmosfang
categories:  linux mysql 
wc: 
excerpt:  SQL 语句与分类，SQL 常见用法，mysql 一些常用函数，数据分析思路，其它 mysql 管理工具
comments: true
---


# 前言

**SQL** 是结构化查询语言 (Structured Query Language) 的简称

SQL是一套访问和处理数据库的标准和规范，事实上不同数据库对于这套规范的实现各有差异，即便同种数据库不同版本实现出来的也不竟相同 ，但不得不说，因为有了这套规范后，对于数据库的操作变得更容易，绝大部分语句可以不用修改就直接跨数据库(这里指DBMS)执行，也为不同数据库管理系统之间导入导出数据提供了一定的可行性

> 个人感觉 SQL 更像是一种交互规范，或者说是 DBMS 的统一API

这里分享一下工作中会用到的一些操作，不是从基础开始，因为 SQL 基础在网上有很多资料，这里主要分享的是一些实用的小技巧和注意事项，如果实现相同效果有更好的方法，欢迎与我探讨，共同交流进步


> **Tip:** 这篇文章可能会持续更新，因为工作中如果遇到了新的问题，可能会有新的方法，届时就会添加进来，因为这篇的知识很零散并不系统，所以更适合作为字典来查，目前的主要执行环境是 Percona Server 5.6


---


# 概要

* TOC
{:toc}


---

# SQL 语句与分类


* DQL(Data Query Language) : 数据查询语言，主管数据获取
* DML(Data Manipulation Language) : 数据操作语言，主管数据变更
* TPL(Data Transaction Language ) : 事务处理语言，主管事务起止
* DCL(Data Control Language) : 数据控制语言，主管权限分配
* DDL(Data Define Language) : 数据定义语言，主管数据结构与定义
* CCL(Cursor Control Language) : 指针控制语言，主管游标操作

平时用得最多的是 DQL、DML、DDL

---

## alter

~~~
alter table chatter_users alter column ip varchar(50) NULL; 
alter table address modify column city char(30);
alter table address modify column city varchar(50);
alter table user_logins  default charset utf8;
ALTER TABLE `table_name` ADD PRIMARY KEY ( `column` ) 
ALTER TABLE `table_name` ADD UNIQUE ( `column` ) 
ALTER TABLE `table_name` ADD INDEX index_name ( `column` ) 
ALTER TABLE `table_name` ADD FULLTEXT ( `column`) 
ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3` )
alter table comments convert to character set utf8;
alter table comments default charset utf8;
alter table `Wxxxxx` modify column `CONTENT` varchar(30) character set utf8 not null;
pt-online-schema-change --user=root --password=XXX --host=localhost --lock-wait-time=120 --alter="ADD COLUMN domain_id INT" D=test,t=oss_pvinfo2 --execute
set old_alter_table = 1;
ALTER IGNORE TABLE tableA ADD UNIQUE INDEX idx_col1_u (col1) 
~~~

修改表属性、修改列属性，修改默认字符集，添加索引，添加列

---

## replication

~~~
CHANGE MASTER TO MASTER_HOST='192.168.1.123', MASTER_USER='repl',MASTER_PASSWORD='xxxx', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=1;
stop slave;
start slave;
show slave status\G
reset slave all; 
~~~

**reset slave all** 会清除从库的同步复制信息、包括连接信息和二进制文件名、位置, 使用show slave status将不会有输出


---

## create 

~~~
create table MyClass(
> id int(4) not null primary key auto_increment,
> name char(20) not null,
> sex int(4) not null default '0',
> degree double(16,2));
CREATE DATABASE `test`;
~~~

---

## insert

~~~
INSERT INTO tbl_name (col1,col2) VALUES(15,col1*2);
insert into teamstemp select * from teams;
insert into table_a(field_a1,field_a2,field_a3) select field_b1,field_b2,field_b3) from table_b;
~~~

---

## rename table

~~~
rename table teams to teams_ready_to_drop;
~~~

---

## unlock

~~~
show processlist;
kill id;
mysql -u root -p -e "select concat('KILL ',ID,';') from information_schema.processlist where COMMAND='Sleep';"  | cat 
mysql -u root -p -e "select concat('KILL ',ID,';') from information_schema.processlist where COMMAND='Sleep' and time > 259200;"  | cat 

~~~



---

## outfile

~~~
select * from abc_def  into outfile "/tmp/abcdef.sql.925";
select id,the_date,a_name,b_cumsum,c_cumsum,d_spent,e_rate,created_at,updated_at  abc_def  into outfile "/tmp/tmp_xyz.sql.2";
~~~

---

## optimize table

~~~
mysql> select concat('optimize table ',TABLE_SCHEMA,'.',TABLE_NAME,';')  from information_schema.TABLES where (ENGINE='MyISAM' or ENGINE='InnoDB') and TABLE_SCHEMA!='information_schema' and TABLE_SCHEMA!='mysql'  into  outfile  "/tmp/optimize.sql";
Query OK, 365 rows affected (0.09 sec)

mysql>
~~~




---

## import data 

~~~
load  data infile "/tmp/abcdef.sql.925.2"  into table  abc_def;
use xxx;
source fff.sql;
~~~







---


## show


~~~
show charset;
show character set;
show char set;
show character set like '%utf8%';
show collation like "%utf8%";
SHOW TABLE STATUS FROM `xxx_qa` LIKE 'abc'\G
show table status like 'conversations'\G
SHOW CREATE TABLE `xxxx_qa`.`abc`\G
SHOW INDEX FROM `xxxx_qa`.`abc`\G
show variables like "%format%";
show databases;
show tables;
show CREATE DATABASE `xxx_qa`
show grants for 'care'@'192.168.1.%';
~~~


---

## index


~~~
create index profiles_on_user_id on  profiles(user_id);
create index conversations_on_user_id on conversations (user_id);
~~~

---

## grant 


~~~
grant create, CREATE TEMPORARY tables , CREATE VIEW , index , REFERENCES , drop , select , insert , update , delete , lock tables ,show view on xxx.* to 'xxx'@'192.168.1.%' identified by 'xxx';
grant alter on xxx.* to 'xxx'@'192.168.1.%';
GRANT RELOAD, SUPER, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'bkuser'@'localhost' IDENTIFIED BY 'xxx';
flush privileges;
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, REFERENCES, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, CREATE VIEW, SHOW VIEW ON `xxxxxx`.* TO 'xxxxx'@'192.168.1.%'  identified by 'xxxxxx';
~~~

---

## select 

~~~
select user();
select databases();
select version();
select * from information_schema.processlist where Command="sleep" and Time>86400;
~~~




---


## desc

~~~
desc mysql.user
desc mysql.db
~~~






---

# 一些函数


## 时间相关函数

~~~
mysql> select date_sub(now(),interval 30 day);
+---------------------------------+
| date_sub(now(),interval 30 day) |
+---------------------------------+
| 2016-04-17 20:31:42             |
+---------------------------------+
1 row in set (0.00 sec)

mysql> select unix_timestamp(date_sub(now(),interval 30 day));
+-------------------------------------------------+
| unix_timestamp(date_sub(now(),interval 30 day)) |
+-------------------------------------------------+
|                                      1460896325 |
+-------------------------------------------------+
1 row in set (0.00 sec)

mysql> select to_days(date_sub(now(),interval 30 day));
+------------------------------------------+
| to_days(date_sub(now(),interval 30 day)) |
+------------------------------------------+
|                                   736436 |
+------------------------------------------+
1 row in set (0.00 sec)

mysql> select to_days(now());
+----------------+
| to_days(now()) |
+----------------+
|         736466 |
+----------------+
1 row in set (0.00 sec)

mysql> 
~~~


---

# 数据分析



## 明确需求

有时我们会进行数据分析或数据抽取，并且需求是来自于产品经理（或运营小妹或市场推广人员），基于他们的经验差异和对技术的理解程度可能会描述不清楚他们到底需要怎样的数据

这时就需要协助他们，并且当面将需求翻译成简明的描述性语言(以便后面翻译成各种操作)，并且一定要再三确认有无遗漏和调整

>如果这一步偷懒省略掉了，极有可能产生这样的情况，在一个极大的表里你花费了一整天导出来结果，他看到后，他会说 “我不是这个意思，其实当时我说的是balabala......” ，这时就能感受到有一种状态叫蹉跎，有一种情绪叫懊恼~~~
>
>之所以可能花费一天这么久(还可能会更久)，有时是因为某些特征列没有索引，并且数据量真的非常大，而不加索引也是为了考虑业务中写操作的性能，这时一个独立于业务的数据仓库就太有必要了

总之最后要形成如下的一张任务和条件列表：

**task1**

*  注册 1个月~2个月之间：[2 month，1 month)
* 登陆次数 >= 5次 
* 1周内未登陆：[7 days，0 days)
* 留有QQ号

需要： userid ,  QQ号



**task2**

* 最近一个月登陆过
* 初始等级-当前等级 >= 2
* 留有手机号

需要：userid，手机号


有了上面的列表后，就可以最大程度的理解和明确需求，节省时间


---

## 展示 schema 的结构

分析完任务列表后我们要将目标锁定在可以提供数据的表上

最好可以将 **`show create table xxx`** 的结果放在一边，以便随时参阅

> **Tip:** 主要留意其索引列


~~~
test_qa.users  (`id`) (`user_key`) (`user_name`) (`nick_name`) 
cheshi_qa.cks  (`id`)   (`user_id`,`the_date`)   (`the_date`)
~~~

之所以要这么做是为了在生成结果的过程中，尽量提醒自己使用索引来完成，否则大表的数据选取过程会非常难熬


---

##  语句示例

* 注册 1个月~2个月之间：[60 days，30 days)
* 留有QQ号

~~~
create table tmp_1 (select id,qq  from test_qa.users  where unix_timestamp(regsitered_at) >=  unix_timestamp(date_sub(now(),interval 60 day))  and unix_timestamp(regsitered_at) <  unix_timestamp(date_sub(now(),interval 30 day)) and qq is not null);
~~~

> **Tip:** 根据个人对 SQL 的掌控程序，其实很有必要生成一些简单的中间结果来拆解和简化整个过程，这样也可以步步为营，有效避免一个语句失败又得整个重头再来，这些中间结果也最好放在自己创建的临时数据库中，为了一定程度上隔离锁域，尽量不波及无辜

由于 **regsitered_at** 并没有索引，所以可以使用函数对其进行加工，如果有索引，就不要这样使用，否则索引就浪费了

* 登陆次数 >= 5次 

~~~
create table tmp_2 (select distinct(user_id),count(user_id) as ct  from cheshi_qa.cks where the_date  >= '2016-03-16'   group by user_id  having ct >= 5);
~~~

这里是直接计算出来值跟 **the_date** 进行比较，原因是如果依旧使用函数，可能会更通用，但是效能会无法忍

如下，则使用不到 **the_date** 上的索引

~~~
select distinct(user_id),count(*) as ct  from cheshi_qa.cks where to_days(the_date) >=  to_days(date_sub(now(),interval 60 day))  group by user_id  having ct >= 5
~~~
 
* 1周内未登陆：[7 days，0 days)

~~~
create table tmp_3 (select distinct(user_id) from cheshi_qa.cks where the_date >= '2016-05-10' )
~~~

为三张表创建索引

~~~
create index idx_tmp_1 on tmp_1(id);
create index idx_tmp_2 on tmp_2(user_id);
create index idx_tmp_3 on tmp_3(user_id);
~~~

拼接成结果

~~~
create table tmp_rs (select id,qq from tmp_1 where ( EXISTS (select user_id from tmp_2 where user_id = tmp_1.id)) and ( not exists (select user_id from tmp_3 where user_id = tmp_1.id)));
~~~

导出结果

~~~
select id,qq from tmp_rs into outfile '/tmp/task1' fields terminated by  ',' optionally enclosed by '"' escaped by '"' lines terminated by '\r\n';
~~~

* 最近一个月登陆过

~~~
create table tast2_1 (select  distinct(user_id) from cheshi_qa.cks where the_date  >= '2016-04-16'  )
~~~


* 初始等级-当前等级 >= 2
* 留有手机号


~~~
select id,cellphone from  test_qa.users where latest_level-begin_level >= 2  and cellphone is not null
~~~

结合两者，其实第一个去另外再创建一张表包含这一个月以来的所有记录是很没必要的，因为我们只要知道 **存在** 就可以了


~~~
create table tast2_rs1 ( select id,cellphone from  test_qa.users where latest_level-begin_level >= 2  and cellphone is not null and exists ( select user_id from cheshi_qa.cks where  user_id = test_qa.users.id and the_date  >= '2016-04-16' ));
~~~

导出结果成CVS

~~~
select id,cellphone from tast2_rs1 into outfile '/tmp/task2' fields terminated by  ',' optionally enclosed by '"' escaped by '"' lines terminated by '\r\n';
~~~





---


# 其它管理工具


---

## mysqldump

~~~
time mysqldump -u root -p  fake_xx > fake_xx.sql
time mysqldump -u root -p  tab_xx conversations > /tmp/conversations.backup.sql
~~~


---

## mysqladmin 

~~~
/usr/bin/mysqladmin flush-logs ; /etc/init.d/filebeat restart
mysqladmin flush-hosts -h localhost -P 3306 -u root -p
mysqladmin -u root -pPASSWORD flush-hosts flush-tables flush-threads flush-logs flush-privileges flush-status
~~~


---

## mysqlslap

~~~
mysqlslap --no-defaults --debug-info -uroot -p  --number-int-cols=5 --number-char-cols=10 --auto-generate-sql --auto-generate-sql-add-autoincrement --concurrency=100 --number-of-queries=10000 --iterations=3 --engine=innodb

mysqlslap --no-defaults --debug-info -uroot -p  --number-int-cols=5 --number-char-cols=10 --auto-generate-sql --auto-generate-sql-add-autoincrement --concurrency=100 --number-of-queries=10000 --iterations=10 --engine=innodb

mysqlslap --concurrency=100 --iterations=1 --create-schema='test' --query='select * from test;' --number-of-queries=10 --debug-info -uroot  -p


~~~


---

## purge relay log


~~~
purge_relay_logs --user=root --password=xxxxx --workdir=/data/relay_log_dir/  
~~~

/data/relay_log_dir/ must be on the same disk of /var/lib/mysql  

由mha4mysql-node 包提供



---


## pt-table-checksum

~~~
#only sp master
pt-table-checksum --nocheck-replication-filters --replicate=pt.checksums h=h102,u=test,p=test,P=3307

pt-table-checksum --nocheck-replication-filters  --nocheck-binlog-format --replicate=pt.check  h=192.168.100.201,u=test,p=test 

#only sp slave

pt-table-sync --replicate pt.checksums   h=192.168.100.202,u=test,p='test'    --sync-to-master --databases=test   --tables=t2 --print

pt-table-sync --replicate pt.checksums   h=192.168.100.202,u=test,p='test'    --sync-to-master --databases=test   --tables=t1 --print

pt-table-sync --replicate pt.checksums   h=192.168.100.202,u=test,p='test'    --sync-to-master --databases=test   --tables=t1 --execute
~~~


---

## mysqltuner

~~~
./mysqltuner.pl --forcemem 48288 --forceswap 4094
~~~





---

## pt-archiver

~~~
time nohup pt-archiver  --source h=localhost,P=3306,D=xxx,t=xxxtable,u=root,p=xxxxx --dest h=test-target,P=3306,u=root,p=xxxx,D=xpd,t=user_logins  --no-check-charset  --where 'id >= 1 '  --progress 5000  --no-delete --limit=10000 --statistics >> archive.log  2>&1 &
time nohup pt-archiver  --source h=localhost,P=3306,D=xpd,t=checkins,u=root,p=xxxx --dest h=test-target,P=3306,u=root,p=xxxx,D=xxxdb,t=xxxtable  --no-check-charset  --where '1=1 '  --progress 10000  --no-delete --limit=10000 --statistics >> archive_checkins.log  2>&1 &
~~~

