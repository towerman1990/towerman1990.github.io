---
title: MySQL 事务隔离级别和锁
date: 2020-12-09 09:43:39
tags:
---

### 事务概念

数据库事务（简称:事务）是数据库管理系统执行过程中的一个逻辑单位，由一个有限的数据库操作序列构成。事务主要用于处理操作量大，复杂度高而的数据。比如在支付过程中，涉及到账户金额的变更、用户商品关系的建立和订单状态修改等操作，此时我们需要保证整个支付过程中所有数据的变更操作全部执行成功，或者完全恢复到支付前的状态，此时，我们就需要使用事务。

> 注：
> 在 MySQL 中只有使用了 Innodb 数据库引擎的数据库或表才支持事务。
> 事务处理可以用来维护数据库的完整性，保证成批的 SQL 语句要么全部执行，要么全部不执行。
> 事务用来管理 insert,update,delete 语句。

<!-- more -->

一般来说，事务是必须满足4个条件（ACID）：原子性（Atomicity，或称不可分割性）、一致性（Consistency）、隔离性（Isolation，又称独立性）、持久性（Durability）。

- 原子性：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。

- 一致性：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。

- 隔离性：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读已提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。

> 注：MySQL 通过锁机制来保证事务的隔离性。

- 持久性：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

> 注：MySQL 使用 redo log 来保证事务的持久性。

### 事务控制语句

- BEGIN 或 START TRANSACTION 显式地开启一个事务；

- COMMIT 也可以使用 COMMIT WORK，不过二者是等价的。COMMIT 会提交事务，并使已对数据库进行的所有修改成为永久性的；

- ROLLBACK 也可以使用 ROLLBACK WORK，不过二者是等价的。回滚会结束用户的事务，并撤销正在进行的所有未提交的修改；

- SAVEPOINT identifier，SAVEPOINT 允许在事务中创建一个保存点，一个事务中可以有多个 SAVEPOINT；

- RELEASE SAVEPOINT identifier 删除一个事务的保存点，当没有指定的保存点时，执行该语句会抛出一个异常；

- ROLLBACK TO identifier 把事务回滚到标记点；

- SET TRANSACTION 用来设置事务的隔离级别。InnoDB 存储引擎提供事务的隔离级别有READ UNCOMMITTED、READ COMMITTED、REPEATABLE READ 和 SERIALIZABLE。

### 事务处理方法

1、用 BEGIN, ROLLBACK, COMMIT来实现

- BEGIN 开始一个事务
- ROLLBACK 事务回滚
- COMMIT 事务确认

2、直接用 SET 来改变 MySQL 的自动提交模式:

SET AUTOCOMMIT=0 禁止自动提交
SET AUTOCOMMIT=1 开启自动提交

> 注：在 MySQL 命令行的默认设置下，事务都是自动提交的，即执行 SQL 语句后就会马上执行 COMMIT 操作。因此要显式地开启一个事务务须使用命令 BEGIN 或 START TRANSACTION，或者执行命令 SET AUTOCOMMIT=0，用来禁止使用当前会话的自动提交。

### 事务隔离级别

SQL 标准定义了四种隔离级别，MySQL 全都支持。这四种隔离级别分别是：

1. 读未提交（READ UNCOMMITTED）
2. 读已提交 （READ COMMITTED）
3. 可重复读 （REPEATABLE READ）
4. 串行化 （SERIALIZABLE）

从上往下，隔离强度逐渐增强，性能逐渐变差。采用哪种隔离级别要根据系统需求权衡决定，其中，可重复读是 MySQL 的默认级别。

之所以要有隔离级别是为了解决数据库操作过程中可能出现的问题，下面说明一下几种可能出现问题的原因：

* 脏读（Dirty Read）

脏读指的是读到了其他事务未提交的数据，未提交意味着这些数据可能会回滚，也就是可能最终不会存到数据库中，也就是不存在的数据。读到了并一定最终存在的数据，这就是脏读。

* 不可重复读（Non-Repeatable Read）

不可重复读指的是在同一事务内，不同的时刻读到的同一批数据可能是不一样的，可能会受到其他事务的影响，比如其他事务改了这批数据并提交了。通常针对数据更新（UPDATE）操作。

* 幻读（Phantom）

幻读是针对数据插入（INSERT）操作来说的。假设事务A对某些行的内容作了更改，但是还未提交，此时事务B插入了与事务A更改前的记录相同的记录行，并且在事务A提交之前先提交了，而这时，在事务A中查询，会发现好像刚刚的更改对于某些数据未起作用，但其实是事务B刚插入进来的，让用户感觉很魔幻，感觉出现了幻觉，这就叫幻读。


### 隔离级别演示

下面我就使用实际操作演示一下各种级别隔离级别及出现的问题。

首先准备两个 mysql 客户端 t1（执行事务A）与 t2（执行事务B），一张测试表 `city`，写入一条测试数据并调整隔离级别为 `READ COMMITTED`：

```sql
CREATE TABLE `city` (
	`id` INT(11) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(50) NOT NULL,
	PRIMARY KEY (`id`)
)
COLLATE='utf8mb4_general_ci'
ENGINE=InnoDB
;
INSERT INTO `city` (`name`) VALUES ('北京');
```

#### 读未提交（READ UNCOMMITTED）

t1（事务A）:
``` sql
mysql> SET @@session.transaction_isolation = 'READ-UNCOMMITTED' /* 这种设置方式对当前会话的后续的事务有效，后面会详解各种设置方式 */;
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> UPDATE `city` SET name = 'shanghai'
WHERE id = 1;
Query OK, 0 rows affected (0.00 sec)
Rows matched: 1  Changed: 0  Warnings: 0

mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)

```

t2（事务B）:
``` sql
mysql> SET @@session.transaction_isolation = 'READ-UNCOMMITTED';
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT *
    -> FROM `city`
    -> WHERE id = 1; /* 此时事务A已经执行更新操作（UPDATE）但尚未提交（COMMIT） */
+----+--------+
| id | name   |
+----+--------+
|  1 | 上海   |
+----+--------+
1 row in set (0.00 sec)

mysql> SELECT *
    -> FROM `city`
    -> WHERE id = 1; /* 此时事务A已经提交（COMMIT） */
+----+--------+
| id | name   |
+----+--------+
|  1 | 上海   |
+----+--------+
1 row in set (0.00 sec)

mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)

```

可以看到，在读未提交隔离级别下，事务A可以读取到事务B修改过但未提交的数据，即产生了 `脏读` 。大部分业务场景都不允许脏读出现，一般很少使用此隔离级别，但是此隔离级别下数据库的并发能力是最好的。

#### 读已提交（READ COMMITTED）

t1（事务A）:
``` sql
mysql> SET @@session.transaction_isolation = 'READ-COMMITTED';
Query OK, 0 rows affected (0.00 sec)

mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> UPDATE city SET name = '深圳'
    -> WHERE id = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)
```
t2（事务B）:
``` sql
mysql> SET @@session.transaction_isolation = 'READ-COMMITTED';
Query OK, 0 rows affected (0.00 sec)

mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT *
    -> FROM `city`
    -> WHERE id = 1; /* 此时事务A已经执行更新操作（UPDATE）但尚未提交（COMMIT） */
+----+--------+
| id | name   |
+----+--------+
|  1 | 上海   |
+----+--------+
1 row in set (0.00 sec)

mysql> SELECT *
    -> FROM `city`
    -> WHERE id = 1; /* 此时事务A已经提交（COMMIT） */
+----+--------+
| id | name   |
+----+--------+
|  1 | 深圳   |
+----+--------+
1 row in set (0.00 sec)

mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)
```

可以看到，在读已提交隔离级别下，事务B只能在事务A修改过并且已提交后才能读取到事务B修改的数据。所以读已提交隔离级别解决了脏读的问题，但可能发生不可重复读和幻读问题。该隔离级别是 Oracle 和 SQL Server 的默认隔离级别。

#### 可重复读（REPEATABLE READ）

t1（事务A）:
``` sql
mysql> SET @@session.transaction_isolation = 'REPEATABLE-READ';
Query OK, 0 rows affected (0.00 sec)

mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> UPDATE city SET name = '广州'
    -> WHERE id = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)
```

t2（事务B）:
``` sql
mysql> SET @@session.transaction_isolation = 'REPEATABLE-READ';
Query OK, 0 rows affected (0.00 sec)

mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT *
    -> FROM `city`
    -> WHERE id = 1; /* 此时事务A已经执行更新操作（UPDATE）但尚未提交（COMMIT） */
+----+--------+
| id | name   |
+----+--------+
|  1 | 深圳   |
+----+--------+
1 row in set (0.00 sec)

mysql> SELECT *
    -> FROM `city`
    -> WHERE id = 1; /* 此时事务A已经提交（COMMIT） */
+----+--------+
| id | name   |
+----+--------+
|  1 | 深圳   |
+----+--------+
1 row in set (0.00 sec)

mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT *
    -> FROM `city`
    -> WHERE id = 1;/* 此时事务B也已经提交（COMMIT） */
+----+--------+
| id | name   |
+----+--------+
|  1 | 广州   |
+----+--------+
1 row in set (0.00 sec)
```

可以看到，在可重复读隔离级别下，事务B只能在事务A修改过数据并提交后，自己也提交事务后，才能读取到事务B修改的数据。可重复读隔离级别解决了脏读和不可重复读的问题，但可能发生幻读问题。该隔离级别是 MySQL 默认的隔离级别。

#### 可串行化（SERIALIZABLE）

可串行化可能会面临4种情况：

读读操作：
t1（事务A）:
``` sql
mysql> SET @@session.transaction_isolation = 'SERIALIZABLE';
Query OK, 0 rows affected (0.00 sec)

mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT *
    -> FROM `city`
    -> WHERE id = 1;
+----+--------+
| id | name   |
+----+--------+
|  1 | 广州   |
+----+--------+
1 row in set (0.00 sec)

mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)
```

t2（事务B）:
mysql> SET @@session.transaction_isolation = 'SERIALIZABLE';
Query OK, 0 rows affected (0.00 sec)

``` sql
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT *
    -> FROM `city`
    -> WHERE id = 1; /* 此时事务A已经执行查询操作（SELECT）但尚未提交（COMMIT） */
+----+--------+
| id | name   |
+----+--------+
|  1 | 广州   |
+----+--------+
1 row in set (0.00 sec)

mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)
```

可以看到，事务A的读操作并未阻塞事务B的读操作，两个事务之间未发生影响。可见在可串行化隔离级别下读读操作不会发生阻塞。

读写操作：
t1（事务A）:
``` sql
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT *
    -> FROM `city`
    -> WHERE id = 1;
+----+--------+
| id | name   |
+----+--------+
|  1 | 广州   |
+----+--------+
1 row in set (0.00 sec)

mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)
```

t2（事务B）:
``` sql
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> UPDATE city SET name = '杭州'
    WHERE id = 1; /* 此时事务A已经执行查询操作（SELECT）但尚未提交（COMMIT） */

/* 发生阻塞 */

Query OK, 1 row affected (5.19 sec) /* 此时事务A已经提交（COMMIT） */
Rows matched: 1  Changed: 1  Warnings: 0
mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)
```

可以看到，事务A的读操作阻塞了事务B的写操作，只有当事务A提交（COMMIT）之后，事务B的写操作才执行完成。可见在可串行化隔离级别下读写操作会发生阻塞。

写读操作：
t1（事务A）:
``` sql
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> UPDATE city SET name = '南京'
    WHERE id = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)

```

t2（事务B）:
``` sql
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT *
    -> FROM `city`
    -> WHERE id = 1; /* 此时事务A已经执行更新操作（UPDATE）但尚未提交（COMMIT） */

/* 发生阻塞 */

+----+--------+
| id | name   |
+----+--------+
|  1 | 南京   |
+----+--------+
1 row in set (7.44 sec) /* 此时事务A已经提交（COMMIT） */

mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)
```

可以看到，事务A的写（更新）操作阻塞了事务B的读操作，只有当事务A提交（COMMIT）之后，事务B的读操作才执行完成。可见在可串行化隔离级别下写读操作会发生阻塞。

写写操作：
t1（事务A）:
``` sql
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> UPDATE city SET name = '成都'
    -> WHERE id = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT *
    -> FROM `city`
    -> WHERE id = 1; /* 此时事务B尚未提交（COMMIT） */
+----+--------+
| id | name   |
+----+--------+
|  1 | 成都   |
+----+--------+
1 row in set (0.00 sec)
```

t2（事务B）:
``` sql
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> UPDATE city SET name = '重庆'
    -> WHERE id = 1; /* 此时事务A已经执行更新操作（UPDATE）但尚未提交（COMMIT） */

/* 发生阻塞 */

Query OK, 1 row affected (11.71 sec) /* 此时事务A已经提交（COMMIT） */
Rows matched: 1  Changed: 1  Warnings: 0

mysql> SELECT *
    -> FROM `city`
    -> WHERE id = 1;
+----+--------+
| id | name   |
+----+--------+
|  1 | 重庆   |
+----+--------+
1 row in set (0.00 sec)

mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)
```

可以看到，事务A的写（更新）操作阻塞了事务B的写操作，只有当事务A提交（COMMIT）之后，事务B的写操作才执行完成。可见在可串行化隔离级别下写写操作会发生阻塞。

可串行化隔离级别的阻塞区别：

| 可串行化隔离级别 | 事务A读操作  | 事务A写操作 |
| ----- | -----  | -----  |
| 事务B读操作 | 不阻塞 | 阻塞 |
| 事务B写操作  | 阻塞 | 阻塞 |

4种隔离级别的比较：

| 隔离级别 | 脏读  | 不可重复读 | 幻读 |
| ----- | -----  | -----  | -----  |
| 读未提交 | 可能 | 可能 | 可能 |
| 读已提交  | 不可能 | 可能 | 可能 |
| 可重复读  | 不可能 | 不可能 | 可能 |
| 幻读  | 不可能 | 不可能 | 不可能 |

### 查看当前会话隔离级别

方式1
命令：`SHOW VARIABLES LIKE 'transaction_isolation'`;

``` sql
mysql> SHOW VARIABLES like 'transaction_isolation';
+-----------------------+--------------+
| Variable_name         | Value        |
+-----------------------+--------------+
| transaction_isolation | SERIALIZABLE |
+-----------------------+--------------+
1 row in set (0.00 sec)
```

方式2
命令：`SELECT @@transaction_isolation;`

``` sql
mysql> SELECT @@transaction_isolation;
+-------------------------+
| @@transaction_isolation |
+-------------------------+
| SERIALIZABLE            |
+-------------------------+
1 row in set (0.00 sec)
```

### 设置隔离级别

方式1：通过set命令 `SET [GLOBAL|SESSION] TRANSACTION ISOLATION LEVEL level`;

其中level有4种值：
```
level: {
     REPEATABLE READ
   | READ COMMITTED
   | READ UNCOMMITTED
   | SERIALIZABLE
}
```

关键词：GLOBAL
`SET GLOBAL TRANSACTION ISOLATION LEVEL level`;

* 只对执行完该语句之后产生的会话起作用
* 当前已经存在的会话无效

关键词：SESSION
`SET SESSION TRANSACTION ISOLATION LEVEL level`;

* 对当前会话的所有后续的事务有效。
* 该语句可以在已经开启的事务中间执行，但不会影响当前正在执行的事务。
* 如果在事务之间执行，则对后续的事务有效。

无关键词
`SET TRANSACTION ISOLATION LEVEL level`;

* 只对当前会话中下一个即将开启的事务有效。
* 下一个事务执行完后，后续事务将恢复到之前的隔离级别。
* 该语句不能在已经开启的事务中间执行，会报错的。

方式2：通过服务启动项命令

可以修改启动参数transaction-isolation的值。
比方说我们在启动服务器时指定了`--transaction-isolation=READ UNCOMMITTED`，那么事务的默认隔离级别就从原来的`REPEATABLE READ`变成了`READ UNCOMMITTED`。