
---

title: 实例分析MySQL下的四种事务隔离级别 - 后端 - 掘金

date: 2019-03-15 09:37:09 +0800

tags: []

---
数据库事务有四种隔离级别：

- 未提交读(Read Uncommitted)：允许脏读，也就是可能读取到其他会话中未提交事务修改的数据。
- 提交读(Read Committed)：只能读取到已经提交的数据，Oracle等多数数据库默认都是该级别。
- 可重复读(Repeated Read)：可重复读。在同一个事务内的查询都是事务开始时刻一致的，InnoDB默认级别。在SQL标准中，该隔离级别消除了不可重复读，但是还存在幻读。
- 串行读(Serializable)：完全串行化的读，每次读都需要获得表级共享锁，读写相互都会阻塞。

上面这样的教科书式定义第一次接触事务隔离概念的朋友看了可能会一脸懵逼，下面我们就通过具体的实例来解释四个隔离级别。

首先我们创建一个user表：

```
CREATE TABLE user (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `name` varchar(255) NOT NULL,
    PRIMARY KEY (`id`),
    UNIQUE `uniq_name` USING BTREE (name)
) ENGINE=`InnoDB` AUTO_INCREMENT=10 DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```

<a name="815c458e"></a>
## 读未提交隔离级别

我们先将事务的隔离级别设置为`read uncommitted`：

```
mysql> set session transaction isolation level read uncommitted;
Query OK, 0 rows affected (0.00 sec)

mysql> select @@session.tx_isolation;
+------------------------+
| @@session.tx_isolation |
+------------------------+
| READ-UNCOMMITTED       |
+------------------------+
1 row in set (0.00 sec)
```

在下面我们开了两个终端分别用来模拟事务一和事务二，p.s: 操作一和操作二的意思是按照时间顺序来执行的。

**事务1**

```
mysql> start transaction; # 操作1
Query OK, 0 rows affected (0.00 sec)

mysql> insert into user(name) values('ziwenxie'); # 操作3
Query OK, 1 row affected (0.05 sec)
```

**事务2**

```
mysql> start transaction; # 操作2
Query OK, 0 rows affected (0.00 sec)

mysql> select * from user; # 操作4
+----+----------+
| id | name     |
+----+----------+
| 10 | ziwenxie |
+----+----------+
1 row in set (0.00 sec)
```

从上面的执行结果可以和清晰的看出来，在read uncommited级别下面我们在事务一中可能会读取到事务二中没有commit的数据，这就是脏读。

<a name="3fdc691d"></a>
## 读提交隔离级别

通过设置隔离级别为`committed`可以解决上面的脏读问题。

```
mysql> set session transaction isolation level read committed;
```

**事务一**

```
mysql> start transaction; # 操作一
Query OK, 0 rows affected (0.00 sec)


mysql> select * from user; # 操作三
+----+----------+
| id | name     |
+----+----------+
| 10 | ziwenxie |
+----+----------+
1 row in set (0.00 sec)

mysql> select * from user; # 操作五，操作四的修改并没有影响到事务一
+----+----------+
| id | name     |
+----+----------+
| 10 | ziwenxie |
+----+----------+
1 row in set (0.00 sec)

mysql> select * from user; # 操作七

+----+------+
| id | name |
+----+------+
| 10 | lisi |
+----+------+
1 row in set (0.00 sec)

mysql> commit; # 操作八
Query OK, 0 rows affected (0.00 sec)
```

**事务二**

```
mysql> start transaction; # 操作二
Query OK, 0 rows affected (0.00 sec)

mysql> update user set name='lisi' where id=10; # 操作四
Query OK, 1 row affected (0.06 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> commit; # 操作六
Query OK, 0 rows affected (0.08 sec)
```

虽然脏读的问题解决了，但是注意在事务一的操作七中，事务二在操作六commit后会造成事务一在同一个transaction中两次读取到的数据不同，这就是不可重复读问题，使用第三个事务隔离级别repeatable read可以解决这个问题。

<a name="4716d740"></a>
## 可重复读隔离级别

MySQL的Innodb存储引擎默认的事务隔离级别就是可重复读隔离级别，所以我们不用进行多余的设置。

**事务一**

```
mysql> start tansactoin; # 操作一

mysql> select * from user; # 操作五
+----+----------+
| id | name     |
+----+----------+
| 10 | ziwenxie |
+----+----------+
1 row in set (0.00 sec)

mysql> commit; # 操作六
Query OK, 0 rows affected (0.00 sec)

mysql> select * from user; # 操作七
+----+------+
| id | name |
+----+------+
| 10 | lisi |
+----+------+
1 row in set (0.00 sec)
```

**事务二**

```
mysql> start tansactoin; # 操作二

mysql> update user set name='lisi' where id=10; # 操作三
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> commit; # 操作四
```

在事务一的操作五中我们并没有读取到事务二在操作三中的update，只有在commit之后才能读到更新后的数据。

<a name="924b55a7"></a>
### Innodb解决了幻读么

实际上RR级别是可能产生幻读，InnoDB引擎官方称中利用MVCC多版本并发控制解决了这个问题，下面我们验证一下Innodb真的解决了幻读了么？

为了方便展示，我修改了一下上面的user表：

```
mysql> alter table user add salary int(11);
Query OK, 0 rows affected (0.51 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> delete from user;
Query OK, 1 rows affected (0.07 sec)

mysql> insert into user(name, salary) value('ziwenxie', 88888888);
Query OK, 1 row affected (0.07 sec)

mysql> select * from user;
+----+----------+----------+
| id | name     | salary   |
+----+----------+----------+
| 10 | ziwenxie | 88888888 |
+----+----------+----------+
1 row in set (0.00 sec)
```

**事务一**

```
mysql> start transaction;  # 操作一
Query OK, 0 rows affected (0.00 sec)

mysql> select * from user; # 操作三
+----+----------+----------+
| id | name     | salary   |
+----+----------+----------+
| 10 | ziwenxie | 88888888 |
+----+----------+----------+
1 row in set (0.00 sec)

mysql> update user set salary='4444'; # 操作六，竟然影响了两行，不是说解决了幻读么？
Query OK, 2 rows affected (0.00 sec)
Rows matched: 2  Changed: 2  Warnings: 0

mysql> select * from user; # 操作七， Innodb并没有完全解决幻读
+----+----------+--------+
| id | name     | salary |
+----+----------+--------+
| 10 | ziwenxie |   4444 |
| 11 | zhangsan |   4444 |
+----+----------+--------+
2 rows in set (0.00 sec)

mysql> commit; # 操作八
Query OK, 0 rows affected (0.04 sec)
```

**事务二**

```
mysql> start transaction; # 操作二
Query OK, 0 rows affected (0.00 sec)

mysql> insert into user(name, salary) value('zhangsan', '666666'); # 操作四
Query OK, 1 row affected (0.00 sec)

mysql> commit; # 操作五
Query OK, 0 rows affected (0.04 sec)
```

从上面的例子可以看出，Innodb并没有如官方所说解决幻读，不过上面这样的场景中也不是很常见不用过多的担心。

<a name="76a4df16"></a>
## 串行化隔离级别

所有事务串行执行，最高隔离级别，不会出现幻读性能会很差，实际开发中很少使用到。

<a name="2432b575"></a>
### 备注

- 脏读：读到了其他会话还未提交的更新
- 幻读：读取到其他会话的新增或者删除，侧重于数量的改变
- 不可重复读：同一个会话中两次执行相同的语句得到的结果不同

<a name="Contact"></a>
## Contact

GitHub: [github.com/ziwenxie](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fziwenxie)<br />
Blog: [www.ziwenxie.site](https://link.juejin.im/?target=https%3A%2F%2Fwww.ziwenxie.site)

> 本文同步发于我的[个人博客](https://link.juejin.im/?target=https%3A%2F%2Fwww.ziwenxie.site%2F2017%2F08%2F08%2Fmysql-transaction-isolation%2F)，转载请声明博客出处 ![](https://gw.alipayobjects.com/os/lib/twemoji/11.2.0/2/svg/1f603.svg#align=left&display=inline&height=18&originHeight=150&originWidth=150&status=done&width=18)


