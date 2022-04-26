### 1.MySQL中有哪几种锁？

* 表级锁

  ​	开销小，加锁快；不会出现死锁；锁定力度大，发送冲突的概率最高，并发度最低。

* 行级锁

  ​	开销大，加锁慢；会出现死锁；锁定力度最小，发生锁冲突的概率最低，并发度最高。

* 页面锁

  ​	开销和加锁时间结余表锁和行锁之间；会出现死锁；锁定力度界于表锁和行锁之间，并发度一般。

### 2.MySQL中有那些不同的表类型？

共7种：BDB、HEAP、ISAM、MERGE、MyISAM、InnoDB、Memeni

### 3.简述在MySQL中MyISAM和InnoDB的区别

#### MyISAM

>不支持事务，但是每次查询都是原子的；
>
>支持表级锁，即每次操作都是对表加锁；
>
>存储表的总行数；
>
>一个MYISAM表有三个文件：索引文件，表结构文件，数据文件；
>
>采用非聚集索引，索引文件的数据域存储指向数据文件的指针。辅索引与主索引基本一致，但是辅索引不用保证唯一性。

#### InnoDB

>支持ACID的事务，支持事务的四种隔离级别；
>
>支持行级锁及外键约束：因此可以支持写并发；
>
>不存储总行数；
>
>一个InnoDB引擎存储在一个文件空间也有可能为多个，搜操作系统文件大小的限制；
>
>主键索引采用聚集索引，辅索引的数据域存储主键的值；
>
>因此从辅索引查找数据，需要先通过辅索引找到主键值，再访问辅索引；
>
>最好使用自增主键，防止插入数据时，为维持B+树结构，文件的大调整。

### 4.MySQL中InnoDB支持的四种事务隔离级别名称，以及逐级之间的区别？

SQL标准定义的四个隔离级别为：

* reead uncommited：读到未提交数据
* read commited：脏读，不可重复读
* repeatable read：可重读
* serializable：串行事务

### 5.CHAR和VARCHAR的区别？

1.CHAR和VARCHAR类型在存储和检索方面有所不同

2.CHAR列长度固定为创建表时声明的长度，长度值范围是1到255

3.当CHAR值被存储时，它们被用空格填充到特定长度，检索CHAR值时需删除尾随空格

### 6.主键和候选键有什么区别

主键是候选键的子集。候选键是超键的子集。

能确定元组的唯一标识就是超键。

在超键中挑选去除本身没有额外属性的键就是候选键。

在候选键中挑选任何一个都是主键。

### 7.myisamchk是用来做什么的？

用来压缩MyISAM表，减少磁盘或内存使用。

### 8.MyISAM Static 和 MyISAM Dynamic有什么区别？

MyISAM Static表上所有的字段有固定的宽度。

MyISAM Dynamic表将具有像TEXT，BLOB等字段，以适应不同长度的数据类型。

MyISAM Static在受损情况下更容易恢复。

### 9.如果一个表有一列定义为TIMESTAMP，将会发生什么？

每当行记录被更改时，时间戳字段将获取当前时间戳。

### 10.列设置为AUTO INCREMENT时，如果在表中达到最大值，会发生什么情况？

被指定的列会停止递增，任何进一步的插入都将产生错误，因为秘钥已经被使用。

注意：自增值类型是int unsigned形式。

### 11.怎样才能找出最后一次插入时分配了哪个自增量？

LAST_INSERT_ID将返回由Auto_increment分配的最后一个值，并且不需要指定表名称。

### 12.你怎么看到为表格定义的所有索引？

> SHOW INDEX FROM <tableName>

### 13.LIKE声明中的％和_是什么意思？

1. `%`和`_`都是LIKE语句中的通配符。
2. `%`：表示零个，一个或多个字符。
3. `_`：表示单个字符。
4. 示例：

```mysql
# 查找以“a”开头的任何值
WHERE column LIKE 'a%'

# 查找以“a”结尾的任何值
WHERE column LIKE '%a'

# 在任何位置查找任何具有“呵呵”的值
WHERE column LIKE '%呵呵%'

# 在第二个位置查找任何具有“r”的值
WHERE column LIKE '_r%'

# 查找以“a”开头且长度至少为3个字符的值
WHERE column LIKE 'a_%_%'

# 找到以"a"开头，以"o"结尾的值
WHERE column LIKE 'a%o'
```

### 14.如何在Unix和MySQL时间戳之间进行转换？

UNIX_TIMESTAMP是从MySQL时间戳转换为Unix时间戳的命令，FROM_UNIXTIME是从Unix时间戳转换为MySQL时间戳的命令。

### 15.列对比运算符是什么？

在SELECT语句的列比较中使用=，<>，<=，<，> =，>，<<，>>，<=>，AND，OR或LIKE运算符。

### 16.BLOB和TEXT有什么区别？

BLOB是一个二进制对象，可以容纳可变数量的数据。

TEXT是一个不区分大小写的BLOB。

BLOB和TEXT类型之间的唯一区别在于对BLOB值进行排序和比较时区分大小写，而对于TEXT值不区分大小写。

### 17.mysql_fetch_array和mysql_fetch_object的区别是什么？

mysql_fetch_array：将结果行作为关联数组或来自数据库的常规数组返回。

mysql_fetch_object：从数据库返回结果行作为对象。

### 18.MYISAM表类型将在哪里存储，并且还提供其存储格式？

每个MYISAM表格以三种格式存储在磁盘上：

“.frm”文件 存储表定义

数据文件具有“.MYD”（MYData）扩展名

索引文件具有“.MYI”（MYIndex）扩展名

注意：[MYISAM的三个文件](#MyISAM)

### 19.MySQL如何优化distinct

DISTINCT在所有列上转换为GROUP BY，并与ORDER BY子句结合使用。

### 20.可以使用多少列创建索引？

任何标准表最多可以创建16个索引列。

### 21.NOW()和CURRENT_DATE()有什么区别？

NOW()命令用于显示当前年份，日期，小时，分钟和秒。

CURRENT_DATE()仅显示当前年份，月份和日期。

示例：

```
SELECT NOW()
2022-04-22 15:09:38
SELECT CURRENT_DATE
2022-04-22
```

### 22.什么是非标准字符串类型？

 TINYTEXT
 TEXT
 MEDIUMTEXT
 LONGTEXT

### 23.什么是通用SQL函数？

```
 CONCAT(A, B) – 连接两个字符串值以创建单个字符串输出。通常用于将两个或多个字段合并为一个字段。
 FORMAT(X, D)- 格式化数字X到D有效数字。
 CURRDATE(), CURRTIME()- 返回当前日期或时间。
 NOW() – 将当前日期和时间作为一个值返回。
 MONTH()，DAY()，YEAR()，WEEK()，WEEKDAY() – 从日期值中提取给定数据。
 HOUR()，MINUTE()，SECOND() – 从时间值中提取给定数据。
 DATEDIFF(A，B) – 确定两个日期之间的差异，通常用于计算年龄
 SUBTIMES(A，B) – 确定两次之间的差异。
 FROMDAYS(INT) – 将整数天数转换为日期值。
```

### 24.MYSQL支持事务吗？

在缺省模式下，MySQL是autocommit模式，所有的数据库更新操作都会即时提交，所以在缺省情况下，mysql是不支持事务的。

但是如果你的MySQL表引擎是InnoDB或者BDB，你的MySQL就可以使用事务处理，使用 set autocommit=0就可以使MySQL允许在非autocommit模式，在非autocommit模式下，你必须使用commit来提交你的更改，或者用rollback来回滚你的更改。

### 25.mysql里记录货币用什么字段类型好

NUMERIC和DECIMAL类型被Mysql实现为同样的类型，这在SQL92标准允许。他们被用于保存值，该值的准确精度是极其重要的值，例如与金钱有关的数据。当声明一个类是这些类型之一时，精度和规模的能被(并且通常是)指定。

### 26.mysql有关权限的表都有哪几个？

Mysql服务器通过权限表来控制用户对数据库的访问，权限表存放在mysql数据库里，由mysql_install_db脚本初始化。这些权限表分别user，db，table_priv，columns_priv和host。

### 27.列的字符串类型？

|    类型    | 大小                  | 用途                            |
| :--------: | --------------------- | ------------------------------- |
|    CHAR    | 0-255 bytes           | 定长字符串                      |
|  VARCHAR   | 0-65535 bytes         | 变长字符串                      |
|  TINYBLOB  | 0-255 bytes           | 不超过 255 个字符的二进制字符串 |
| MEDIUMBLOB | 0-16 777 215 bytes    | 二进制形式的中等长度文本数据    |
|    BLOB    | 0-65 535 bytes        | 二进制形式的长文本数据          |
|  LONGBLOB  | 0-4 294 967 295 bytes | 二进制形式的极大文本数据        |
|  TINYTEXT  | 0-255 bytes           | 短文本字符串                    |
| MEDIUMTEXT | 0-16 777 215 bytes    | 中等长度文本数据                |
|    TEXT    | 0-65 535 bytes        | 长文本数据                      |
|  LONGTEXT  | 0-4 294 967 295 bytes | 极大文本数据                    |

### 28.MySQL数据库作发布系统的存储，一天五万条以上的增量，预计运维三年,怎么优化？

1. 设计良好的数据库结构，允许部分数据冗余，尽量避免join查询，提高效率。
2. 选择合适的表字段数据类型和存储引擎，适当的添加索引。
3. mysql库主从读写分离。
4. 找规律分表，减少单表中的数据量提高查询速度。
5. 添加缓存机制，比如memcached，redis等。
6. 不经常改动的页面，生成静态页面。
7. 书写高效率的SQL。比如 SELECT * FROM TABEL 改为 SELECT field_1, field_2, field_3 FROM TABLE

### 29.锁优化策略

	1. 读写分离
 	2. 分段加锁
 	3. 减少锁持有的时间
 	4. 多个线程尽量以相同的顺序去获取资源

不能将锁的粒度过于细化，不然可能会出现线程的加锁和释放次数过多，反而效率不如一次加一把大锁。

### 30.索引的底层实现原理和优化

B+树！！！！！！！

### 31.什么情况下索引会失效

### 32. 如何优化MySQL

### 33.优化数据库的方法

### 34.简单描述MySQL中，索引，主键，唯一索引，联合索引的区别，对数据库的性能有什么影响(从读写方面)





