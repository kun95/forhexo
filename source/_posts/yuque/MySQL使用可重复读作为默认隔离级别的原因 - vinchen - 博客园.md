
---

title: MySQL使用可重复读作为默认隔离级别的原因 - vinchen - 博客园

date: 2019-02-27 14:05:18 +0800

tags: []

---
一般的DBMS系统，默认都会使用读提交（Read-Comitted，RC）作为默认隔离级别，如Oracle、SQL Server等，而MySQL却使用可重复读（Read-Repeatable，RR）。要知道，越高的隔离级别，能解决的数据一致性问题越多，理论上性能损耗更大，可并发性越低。隔离级别依次为

SERIALIZABLE > RR > RC > Read-Uncommited

在SQL标准中，前三种隔离级别分别解决了幻象读、不可重复读和脏读的问题。那么，为什么MySQL使用可重复读作为默认隔离级别呢？

Binlog是MySQL的逻辑操作日志，广泛应用于复制和恢复。MySQL 5.1以前，Statement是Binlog的默认格式，即依次记录系统接受的SQL请求；5.1及以后，MySQL提供了Row和Mixed两个Binlog格式。

从MySQL 5.1开始，如果打开语句级Binlog，就不支持RC和Read-Uncommited隔离级别。要想使用RC隔离级别，必须使用Mixed或Row格式。

mysql> set tx_isolation='read-committed';

Query OK, 0 rows affected (0.00 sec)

mysql> insert into t1 values(1,1);

ERROR 1598 (HY000): Binary logging not possible. Message: Transaction level 'READ-COMMITTED' in InnoDB is not safe for binlog mode 'STATEMENT'

那么，为什么RC隔离级别不支持语句级Binlog呢？我们关闭binlog，做以下测试。

会话1

会话2

use test;

#初始化数据

create table t1(c1 int, c2 int) engine=innodb;

create table t2(c1 int, c2 int) engine=innodb;

insert into t1 values(1,1), (2,2);

insert into t2 values(1,1), (2,2);

#设置隔离级别

set tx_isolation='read-committed';

Query OK, 0 rows affected (0.00 sec)

#连续更新两次

mysql> Begin;

Query OK, 0 rows affected (0.03 sec)

mysql> update t2 set c2 = 3 where c1 in (select c1 from t1);

Query OK, 2 rows affected (0.00 sec)

Rows matched: 2  Changed: 2  Warnings: 0

mysql> update t2 set c2 = 4 where c1 in (select c1 from t1);

Query OK, 1 row affected (0.00 sec)

Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from t2;

+------+------+

| c1   | c2   |

+------+------+

|    1 |    4 |

|    2 |    3 |

+------+------+

2 rows in set (0.00 sec)

mysql> commit;

#设置隔离级别

set tx_isolation='read-committed';

Query OK, 0 rows affected (0.00 sec)

#两次更新之间执行删除

mysql> delete from t1 where c1 = 2;

Query OK, 1 row affected (0.03 sec)

由以上测试知，RC隔离级别下，会话2执行时序在会话1事务的语句之间，并且会话2的操作影响了会话1的结果，这会对Binlog结果造成影响。

由于Binlog中语句的顺序以commit为序，如果语句级Binlog允许，两会话的执行时序是

#会话2

set tx_isolation='read-committed';

delete from t1 where c1 = 2;

commit;

#会话1

set tx_isolation='read-committed';

Begin;

update t2 set c2 = 3 where c1 in (select c1 from t1);

update t2 set c2 = 4 where c1 in (select c1 from t1);

select * from t2;

+------+------+

| c1   | c2   |

+------+------+

|    1 |    4 |

|    2 |    2 |

+------+------+

2 rows in set (0.00 sec)

commit;

由上可知，在MySQL 5.1及以上的RC隔离级别下，语句级Binlog在DR上执行的结果是不正确的！

那么，MySQL 5.0呢？5.0允许RC下语句级Binlog，是不是说很容易产生DB/DR不一致呢？

事实上，在5.0重复上述一个测试，并不存在这个问题，原因是5.0的RC与5.1的RR使用类似的并发和上锁机制，也就是说，MySQL 5.0的RC与5.1及以上的RC可能存在兼容性问题。

下面看看RR是怎么解决这个问题的。

导致RC隔离级别DB/DR不一致的原因是：RC不可重复读，而Binlog要求SQL串行化！

在RR下，重复以上测试

会话1

会话2

use test;

#初始化数据

create table t1(c1 int, c2 int) engine=innodb;

create table t2(c1 int, c2 int) engine=innodb;

insert into t1 values(1,1), (2,2);

insert into t2 values(1,1), (2,2);

#设置隔离级别

set tx_isolation='repeatable-read';

Query OK, 0 rows affected (0.00 sec)

#连续更新两次

mysql> Begin;

Query OK, 0 rows affected (0.03 sec)

mysql> update t2 set c2 = 3 where c1 in (select c1 from t1);

Query OK, 2 rows affected (0.00 sec)

Rows matched: 2  Changed: 2  Warnings: 0

mysql> update t2 set c2 = 4 where c1 in (select c1 from t1);

Query OK, 2 rows affected (0.00 sec)

Rows matched: 2  Changed: 2  Warnings: 0

mysql> select * from t2;

+------+------+

| c1   | c2   |

+------+------+

|    1 |    4 |

|    2 |    4 |

+------+------+

2 rows in set (0.00 sec)

mysql> commit;

#设置隔离级别

set tx_isolation=' repeatable-read';

Query OK, 0 rows affected (0.00 sec)

#两次更新之间执行删除

mysql> delete from t1 where c1 = 2;

--阻塞，直到会话1提交

Query OK, 1 row affected (18.94 sec)

与RC隔离级别不同的是，在RR中，由于保证可重复读，会话2的delete语句会被会话1阻塞，直到会话1提交。

在RR中，会话1语句update t2 set c2 = 3 where c1 in (select c1 from t1)会先在t1的记录上S锁（5.1的RC中不会上这个锁，但5.0的RC会），接着在t2的满足条件的记录上X锁。由于会话1没提交，会话2的delete语句需要等待会话1的S锁释放，于是阻塞。

因此，在RR中，以上测试会话1、会话2的依次执行，与Binlog的顺序一致，从而保证DB/DR一致。

<a name="d5zrtw"></a>
## [](#d5zrtw)幻象读

除了保证可重复读，MySQL的RR还一定程度上避免了幻象读（幻象读是由于插入导致的新记录）。（为什么说一定程度呢？参考第3节可重复读和串行化的区别。）

会话1

会话2

use test;

#初始化数据

create table t1(c1 int primary key, c2 int) engine=innodb;

create table t2(c1 int primary key, c2 int) engine=innodb;

insert into t1 values(1,1), (10,10);

insert into t2 values(1,1), (5,5), (10,10);

#设置隔离级别

set tx_isolation='repeatable-read';

Query OK, 0 rows affected (0.00 sec)

#连续更新两次

mysql> Begin;

Query OK, 0 rows affected (0.03 sec)

mysql> update t2 set c2 = 20 where c1 in (select c1 from t1);

Query OK, 2 rows affected (0.00 sec)

Rows matched: 2  Changed: 2  Warnings: 0

mysql> delete from where c1 in (select c1 from t1);

Query OK, 2 rows affected (0.00 sec)

Rows matched: 2  Changed: 2  Warnings: 0

mysql> select * from t2;

+------+------+

| c1   | c2   |

+------+------+

|    5 |    5 |

+------+------+

2 rows in set (0.00 sec)

mysql> commit;

#设置隔离级别

set tx_isolation=' repeatable-read';

Query OK, 0 rows affected (0.00 sec)

#两次更新之间执行插入

mysql> insert into t1 values(5,5);

--阻塞，直到会话1提交

Query OK, 1 row affected (18.94 sec)

由上述例子知，会话2的插入操作被阻塞了，原因是RR隔离级别中，除了记录锁外，还会上间隙锁(gap锁)。例如，对于表t1，update t2 set c2 = 20 where c1 in (select c1 from t1)以上的锁包括：

(-∞, 1), 1, (1, 10), 10, (10, +∞)

由于对t1做全表扫描，因此，所有记录和间隙都要上锁，其中(x,y)表示间隙锁，数字表示记录锁，全部都是S锁。会话2的insert操作插入5，位于间隙(1,10)，需要获得这个间隙的X锁，因此两操作互斥，会话2阻塞。

SQL标准的RR并不要求避免幻象读，而InnoDB通过gap锁来避免幻象，从而实现SQL的可串行化，保证Binlog的一致性。

要想取消gap lock，可使用参数[innodb_lock_unsafe_for_binlog](#sysvar_innodb_locks_unsafe_for_binlog)=1，默认为0。

InnoDB的RR可以避免不可重复读和幻象读，那么与串行化有什么区别呢？

会话1

会话2

use test;

#初始化数据

create table t3(c1 int primary key, c2 int) engine=innodb;

#设置隔离级别

set tx_isolation='repeatable-read';

Query OK, 0 rows affected (0.00 sec)

mysql> Begin;

Query OK, 0 rows affected (0.03 sec)

mysql> select * from t3 where c1 = 1;

Empty set (0.00 sec)

mysql> select * from t3 where c1 = 1;

Empty set (0.00 sec)

mysql> update t3 set c2 =2 where c1 = 1;

Query OK, 1 row affected (0.00 sec)

Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from t3 where c1 = 1;

+----+------+

| c1 | c2   |

+----+------+

|  1 |    2 |

+----+------+

1 row in set (0.00 sec)

mysql> commit;

#设置隔离级别

set tx_isolation=' repeatable-read';

Query OK, 0 rows affected (0.00 sec)

mysql> insert into t3 values(1,1);

Query OK, 1 row affected (0.05 sec)

由上述会话1中，连续两次读不到数据，但更新却成功，并且更新后的相同读操作就能读到数据了，这算不算幻读呢？

其实，RR隔离级别的防止幻象主要是针对写操作的，即只保证写操作的可串行化，因为只有写操作影响Binlog；而读操作是通过MVCC来保证一致性读（无幻象）。

然而，可串行化隔离级别要求读写可串行化。使用可串行化重做以上测试。

会话1

会话2

use test;

#初始化数据

create table t3(c1 int primary key, c2 int) engine=innodb;

#设置隔离级别

set tx_isolation='SERIALIZABLE';

Query OK, 0 rows affected (0.00 sec)

mysql> Begin;

Query OK, 0 rows affected (0.03 sec)

mysql> select * from t3 where c1 = 1;

Empty set (0.00 sec)

mysql> select * from t3 where c1 = 1;

Empty set (0.00 sec)

mysql> update t3 set c2 =2 where c1 = 1;

Query OK, 0 rows affected (0.00 sec)

Rows matched: 0  Changed: 0  Warnings: 0

mysql> select * from t3 where c1 = 1;

Empty set (0.00 sec)

mysql> commit;

#设置隔离级别

set tx_isolation='SERIALIZABLE';

Query OK, 0 rows affected (0.00 sec)

mysql> insert into t3 values(1,1);

#阻塞，直到会话1提交

Query OK, 1 row affected (48.90 sec)

设置为串行化后，会话2的插入操作被阻塞。由于在串行化下，查询操作不在使用MVCC来保证一致读，而是使用S锁来阻塞其他写操作。因此做到读写可串行化，然而换来就是并发性能的大大降低。

MySQL使用可重复读来作为默认隔离级别的主要原因是语句级的Binlog。RR能提供SQL语句的写可串行化，保证了绝大部分情况（[不安全语句](http://dev.mysql.com/doc/refman/5.1/en/binary-log-mixed.html)除外）的DB/DR一致。

另外，通过这个测试发现MySQL 5.0与5.1在RC下表现是不一样的，可能存在兼容性问题。

[http://dev.mysql.com/doc/refman/5.1/en/binary-log-mixed.html](http://dev.mysql.com/doc/refman/5.1/en/binary-log-mixed.html)

[http://dev.mysql.com/doc/refman/5.1/en/set-transaction.html](http://dev.mysql.com/doc/refman/5.1/en/set-transaction.html)

[http://dev.mysql.com/doc/refman/5.0/en/set-transaction.html](http://dev.mysql.com/doc/refman/5.0/en/set-transaction.html)

[http://dev.mysql.com/doc/refman/5.5/en/innodb-parameters.html#sysvar_innodb_locks_unsafe_for_binlog](http://dev.mysql.com/doc/refman/5.5/en/innodb-parameters.html#sysvar_innodb_locks_unsafe_for_binlog)

[http://blog.bitfly.cn/post/mysql-innodb-phantom-read/](http://blog.bitfly.cn/post/mysql-innodb-phantom-read/)

