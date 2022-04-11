### 引擎

#### MyISAM

​	MyISAM 适合需要一些大量查询的应用，但对于一些大量写操作的并不是很好。由于**不支持行锁**，当我们在进行 add、update、delete 操作时，整张表都会被锁起来，就算是读操作都是无法完成，必须等到写操作完成。 **不支持事务，不支持外键**。

#### InnoDB

​	InnoDB 合适需要一些大量写入的应用，但对于一些大量读操作的并不是很好。支持**行锁**，支持**事务**。

> 在实际项目中，一般都是用 InnoDB ，而且 MyISAM 属于 MySQL 老版本的引擎。

### SQL语句

*  尽量避免全表扫描：select * from table 。

* 尽量避免在 where 子句中使用 != 或 <> 操作符，否则将导致引擎放弃使用索引而进行全表扫描。

* 尽量避免在 where 子句中对字段进行 **null** 值判断，否则将导致引擎放弃使用索引而进行全表扫描，如：

  ```mysql
  select id from table where num is null;  
  -- 类似于这种情况，可以将 id 默认值设置为 0，从而确保 id 列上没有 null 值
  select id from table where num = 0;
  ```

* 尽量避免在 where 子句中使用 **or** 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，如：

  ```mysql
  select id from table where id = 1 or id = 2;
  -- 优化
  select id from table where id = 1
  union all
  select id from table where id = 2;
  ```

* 尽量避免在 where 子句中使用 **like** 来连接条件，如：

  ```mysql
  select name from table where name like '%abc%';
  -- 不要在匹配内容的前面加 %
  ```

* 当我们在用 **in** 来匹配 连续数值时，其实可以用 **Bewteen...and...** 来代替

  ```mysql
  -- in
  select id from table where id in (1,2,3,4,5);
  -- bewteen...and...
  select id from table where id bewteen 1 and 5;
  ```

* 尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描，如：

  ```mysql
  select id from table where id / 2 = 100;
  -- 优化
  select id from table where id = 100 * 2;
  ```

* 尽量避免在 where 子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描，如：

  ```mysql
  select name from table where SUBSTRING(name, 1, 3) = 'abc';
  -- 优化
  select name from table where name Like 'abc%';
  ```

* 任何地方都不要使用 select * from t ，用具体的字段列表代替 “*”，不要返回用不到的任何字段。

* 索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有必要。  

### 索引

> MySQL 的索引分为 主索引 和 辅助索引。

#### 主索引

主索引是按照索引字段值进行排序的一个有序文件，通常建立在有序文件的基于主码的排序字段上。

#### 辅助索引

定义在主文件的任意一个或者多个非排序字段上的辅助存储结构。

目前 MySQL 的几种索引为：BTREE，HASH， FULLTEXT，RTREE。

##### FULLTEXT

全文索引，目前只有 MyISAM 引擎支持。

##### HASH

当我们为某一列或某几列建立 hash 索引时（目前就只有 MEMORY 引擎显式地支持这种索引），会在硬盘上生成类似如下的文件：

| Hash 值      | 存储地址         |
| ------------ | ---------------- |
| 1db54bc745a1 | 77#45b5          |
| 4bca452157d4 | 76#4556,77#45cc… |

> Hash 值即为通过特定算法由指定列数据计算出来，磁盘地址即为所在数据行存储在硬盘上的地址（也有可能是其他存储地址，其实 MEMORY 会将 Hash 表导入内存）

例如我们进行 where age = 18 时，会将 18 通过 Hash 算法计算出一个 Hash 值，然后我们就在 Hash 表中找到对应的存储地址，根据存储地址找到对应的数据。在这里我们可以发现我们的每次查询都会遍历 Hash 表，当数据量很大时，Hash 表也会变得庞大从而导致性能不会很好。

##### B+

###### 简介

B-Tree 就是一颗多路平衡查找树，一颗 m 阶B-tree的特性：

* 每个节点最多有 m 个子节点
*  除根节点和叶子节点，其它每个节点至少有 [m/2] （向上取整的意思）个子节点
*  若根节点不是叶子节点，则其至少有2个子节点
* 所有NULL节点到根节点的高度都一样 
* 除根节点外，其它节点都包含 n 个key，其中 [m/2] -1 <= n <= m-1

BTREE 索引就是一种将索引值按一定的算法，存入一个树形的数据结构中。

###### 在 MyISAM 中的应用

leaf node里存放的不是主键的信息，而是指向数据文件里的对应数据行的信息。

###### 在 InnoDB 中的应用

有两种形态：

* primary key形态

  > leaf node 里存放的是数据，而且不仅存放了索引键的数据，还存放了其他字段的数据

* secondary index

  > leaf node和普通的 BTREE 差不多，只是还存放了指向主键的信息

##### RTREE

***这个不懂呀！！！！***

### 事务

> 事务具有四个特征：原子性（ Atomicity ）、一致性（ Consistency ）、隔离性（ Isolation ）和持续性（ Durability ）。这四个特性简称为 ACID 特性。 具体的解释自行百度。

#### 事务的并发问题

##### 脏读

一个事务读取到，另一个事务未提交的数据。

##### 不可重复读

一个数据对同一行数据重复读取两次却得到了不同的结果，不可重复读的重点在于 update 和 delete。

##### 重复读

重复读是指在一个事务中，对于**指定条件的数据视图**始终是一致的，不管其他事务是否 update 或者 delete。

##### 幻读

一个事务对**同一行数据**重复读取两次，但是却得到了不同的结果。例如事务 T1 读取某一数据后，事务 T2 对其做了修改，当事务 T1 再次读该数据时得到与前一次不同的值。幻读的重点在于 insert。

#### MySQL 数据库事务的四种隔离级别

> MySQL 通过 set  tx_isolation = 'xxxx'; 修改隔离级别

##### Serializale（可串行化）

这是最高的隔离级别，它通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁。在这个级别，可能导致大量的超时现象和锁竞争。

##### Repeatable read (可重复读)

这是 MySQL 的默认事务隔离级别，它确保同一事务的多个实例在并发读取数据时，会看到同样的数据行。不过理论上，这会导致另一个棘手的问题：**幻读 （Phantom Read）**。简单的说，**幻读**指当用户读取某一范围的数据行时，另一个事务又在该范围内插入了新行，当用户再读取该范围的数据行时，会发现有新的“幻影” 行。InnoDB 和 Falcon 存储引擎通过多版本并发控制（MVCC，Multiversion Concurrency Control）机制解决了该问题。很通俗的讲就是：可重复读，就是在当前事务中查询到的数据不会因为其他事物的 update 和 delete 发生改变，但是 insert 会发生改变。

##### Read committed (读已提交)

一个事务只能看见已经提交事务所做的改变。这种隔离级别也支持所谓的不可重复读（Nonrepeatable Read），因为同一事务的其他实例在该实例处理其间可能会有新的 commit，所以同一 select 可能返回不同结果。

##### Read uncommitted (读未提交)

在该隔离级别，所有事务都可以看到其他未提交事务的执行结果。本隔离级别很少用于实际应用，因为它的性能也不比其他级别好多少。读取未提交的数据，也被称之为**脏读（Dirty Read）**。

#### Hibernate 事务

> properties:	hibernate.connection.isolation

#### 行锁

​	悲观锁：使用了 select…for update 的方式，这样就通过开启排他锁的方式实现了悲观锁。

```mysql
-- 要使用悲观锁，我们必须关闭mysql数据库的自动提交属性，因为MySQL默认使用autocommit模式，也就是说，当你执行一个更新操作后，MySQL会立刻将结果进行提交。 set autocommit=0;

-- 0.开始事务
begin;/begin work;/start transaction; (三者选一就可以)
-- 1.查询出商品信息
select status from t_goods where id=1 for update;
-- 2.根据商品信息生成订单
insert into t_orders (id,goods_id) values (null,1);
-- 3.修改商品status为2
update t_goods set status=2;
-- 4.提交事务
commit;/commit work;
```



#### 表锁

​	乐观锁：根据时间戳，版本号,update ..set version = version + 1. where version = #{version}

```mysql
-- 1 .查询出商品信息
select (status,status,version) from t_goods where id=#{id}
-- 2 .根据商品信息生成订单
-- 3 .修改商品status为2
update t_goods 
set status=2,version=version+1
where id=#{id} and version=#{version};
```

#### 范式

[自行百度吧](www.baidu.com)

