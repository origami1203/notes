# Mysql

### 安装

1.  卸载自带mariadb

    ```shell
    [root@centos7 src] rpm -qa|grep mariadb
    mariadb-libs-5.5.60-1.el7_5.x86_64
    [root@centos7 src] rpm -e --nodeps  mariadb-libs-5.5.60-1.el7_5.x86_64
    ```

2.  删除/etc下my.cnf配置文件（如果存在的话）、检查mysql是否存在

```
[root@centos7 src]# rm -rf /etc/my.cnf
[root@centos7 src]# rpm -qa | grep mysql
```

3.  检查mysql用户组、用户是否存在，不存在则创建

```
[root@centos7 src]# cat /etc/group | grep mysql 
[root@centos7 src]# cat /etc/passwd | grep mysql
[root@centos7 src]# groupadd mysql
[root@centos7 src]# useradd -g mysql mysql
```

4.  解压mysql到/usr/local/mysql目录，并创建data目录

5.  更改文件夹权限

```shell
chown -R mysql:mysql ./
chgrp -R mysql:mysql ./
```

6.  配置参数

```
./bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
```

注意记录临时密码

7.  配置文件`vim /etc/my.cnf`

```properties
[mysqld]

#需要修改mysql地址
basedir =basedir=/usr/local/mysql
#需要修改mysql日志地址
datadir = basedir=/usr/local/mysql/data
port = 3306
#mysqld.sock生成地址（不用修改）
socket = /tmp/mysqld.sock
character-set-server=utf8
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

[mysql]
# mysql默认编码
default-character-set = utf8mb4

[client]
# mysql客户端默认编码
default-character-set = utf8mb4
#mysqld.sock生成地址（不用修改）
socket=/tmp/mysqld.sock
```

8.  修改/etc/init.d/mysqld文件

```
cp ./support-files/mysql.server /etc/init.d/mysqld

vim /etc/init.d/mysqld
# 配置如下
basedir=basedir=/usr/local/mysql 
datadir=basedir=/usr/local/mysql/data
```

9.  启动，修改密码,远程登录

```shell
service mysqld start
```

```mysql
set password=password('root');
# 刷新
flush privileges;

use mysql;
update user set user.Host='%' where user.User='root';
```

### 索引

MySQL官方对索引的定义为：索引(Index)是帮助MySQL高效获取数据的==**数据结构**==。是一种排好序的快速数据。 例如以b树的形式存储数据，查找时通过索引快速查找。

```mysql
SELECT DISTINCT
	<select_list>
FROM
	<left_table> <join_type >
JOIN < right_table>
ON< join_condition >
WHERE
	<where_condition >
GROUP BY
	<group_by_list>
HAVING
	<having_condition >
ORDER BY
	order_by_condition>
LIMIT<limit number
```

执行顺序

```mysql
FROM <left_table>
ON <join_condition>
<join_type>JOIN <right_table>
WHERE <where_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
SELECT
DISTINCT <select_list>
ORDER BY <order_by_condition>
LIMIT <limit_number>
```

==**优势**==：

​	提高检索效率，降低io成本

​	通过索引排序，降低排序成本，降低了 CPU消耗

==**劣势**==：

 	索引结构占用空间，存储了主键和索引字段，指向表数据

​	索引虽然增加了查询速度，但降低了增删改数据的速度，对表进行了修改以后，索引数据也要进行更改

##### 索引创建、删除

单值索引：即一个索引只包含单个列，一个表可以有多个单列索引

唯一索引：索引列的值必须唯一，但允许有空值

复合索引：即一个索引包含多个列

==**索引创建**==

​	`create [unique] index idx_user_name on user(name)`

​	`ALTER mytable ADD [UNIQUE] INDEX [indexName] ON (columnname(length))`

​	若`on`后面跟了多个字段，则为复合索引

==**删除**==

​	`DROP INDEX [indexName] ON table`

==**查看**==

`SHOW INDDEX FROM table`

==**更改索引**==

`ALTER TABLE tbl_name ADD PRIMARY KEY (column_list)`：该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL。

`ALTER TABLE tbl_name ADD UNIQUE index_name (column_list)`:这条语句创建索引的值必须是唯一的(除了NULL外  NULL可能会出现多次)

`ALTER TABLE tblname ADD INDEX index_name(column_list)`添加普通索引，索引值可出现多次。

`ALTERTABLE tb_name ADD FULLTEXT index_name (Ccolumn_list)`该语句指定了索引为 FULLTEXT，用于全文索引。

##### 索引结构

索引的结构一般都是b树或b+树，mysql使用b+树作为索引结构，使用b+树可以快速定位到想要的数据所处的位置，从而取得想要的数据。

对于联合索引，先按第一个字段进行排序，第一个字段相同再按第二个字段，进行排序，形成b+树结构。因此使用联合索引时，只使用第一个 字段可以使用索引，而只使用二或三不能。

索引还可以分为聚集索引和普通索引。

*   **聚集索引**的叶子节点存储行记录。聚集索引为主键或唯一索引，若两个都没有。mysql会维护一个隐藏行作为聚集索引。(innodb必须有聚集索引)
*   **普通索引**的叶子节点存储主键值。

![image-20200629164212829](E:%5CJava%5C%E8%B5%84%E6%96%99%5C%E8%B5%84%E6%96%99%5Cmysql%E9%AB%98%E7%BA%A7.assets%5Cimage-20200629164212829.png)

回表查询：普通索引无法获取想要的数据，要根据获得的主键值再次去查询聚集索引来获得数据，这称为回表查询。

### explain

==**explain sql**==：展示语句的解析过程，用于分析sql

```mysql
CREATE TABLE `employee` (
  `rec_id` int(11) NOT NULL AUTO_INCREMENT,
  `no` varchar(10) NOT NULL,
  `name` varchar(20) NOT NULL,
  `position` varchar(20) NOT NULL,
  `age` varchar(2) NOT NULL,
  PRIMARY KEY (`rec_id`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8 
```

```mysql
EXPLAIN SECLECT * FROM `employee`
+----+-------------+----------+------+---------------+------+---------+------+------+-------------+
 |  id  |  select_type | t   able    |  type | possible_keys |  key  |  key_len |   ref   | rows  |     Extra       |
+----+-------------+----------+------+---------------+------+---------+------+------+-------------+
```

*   id : 读取表的顺序，id越大，优先级越高，越先执行，id相同时，从上向下依次执行。

*   select_type： 查询类型

	|       类型       |                                                              |
| :--------------: | ------------------------------------------------------------ |
|      SIMPLE      | 简单的select查询，查询中不包含子查询或者UNION                |
|     PRIMARY      | 查询中若包含任何复杂的子部分，最外层查询则被标记为primary    |
| SUBQUERY(子查询) | 在SELECT或WHERE列表中包含了子查询，此子查询称为SUBQUERY      |
|  DERIUED(衍生)   | 在FROM列表中包含的子查询被标记为DERIVED(衍生)<br/>MySQL会递归执行这些子查询,把结果放在临时表里。 |
|      UNION       | 若第二个SELECT出现在UNION之后,则被标记为UNION;<br/>若UNION包含在FROM子句的子查询中,外层SELECT将被记为: DERIVED |
|   UNION RESULT   | 从UNION表获取结果的SELECT                                    |

*   table：对应的表名

*   type：访问类型

	|  类型  |                                                              |
| :----: | ------------------------------------------------------------ |
|  ALL   | 全表查询，对于前表的每一行(row)，后表都要被全表扫描。        |
| index  | 根据索引进行全表扫描                                         |
| range  | 范围内查询，如>,<,between等                                  |
|  ref   | 对于前表的每一行(row)，后表数据可能有多行被扫描。            |
| eq_ref | 对于前表的每一行(row)，后表数据只有一行被扫描。              |
| const  | 常量连接，==**表最多只有一行匹配**==，通用用于**主键**或者**唯一索引**比较时，且条件为常量。如将主键置于where列表中(**where id = 1**) |
| system | 表只有一行记录，这是const类型的特列,平时不会出现,可以忽略    |
|  NULL  | 表示不用查询数据库即可获得想要的数据                         |

从上到下，性能越来越差，一般来说，得保证查询至少达到range级别，最好能达到ref。

​		**`system/const`**：当查询最多匹配一行时，常出现于where条件是＝的情况。system是const的一种特殊情况，既表本身只有一行数据的情况。

```mysql
select * from student where id = 1;
```

​		但是要注意的是，并不是所有的where＝都是const，只有＝的右边是常量的时候才会走const。比如：

```mysql
select * from class where id = grade;
```

​	**`eq_ref`**：eq_ref扫描的条件为，对于前表的每一行(row)，后表只有一行被扫描。

　（1）join查询；

　（2）命中主键(primary key)或者非空唯一(unique not null)索引；

　（3）等值连接；

```mysql
select * from class c
join student s
on c.id = s.class_id
```

**`ref`**：当**const**的where条件改为**非主键或唯一索引**时，连接查询也由const降级为了ref，因为可能有多于一行的数据被扫描。也有可能是**eq_ref**连接时，条件不是主键或非空索引。ref扫描，可能出现在join里，也可能出现在单表普通索引里，每一次匹配可能有多行数据返回，它比eq_ref要慢。

```mysql
// name字段需要创建普通索引
explain select * from user where name = 'tom' 

explain select * from user,
```

**`range`**：范围查询

```mysql
explain select * from user where id in(1,2,3);
explain select * from user where id > 3;
```

**`index`**：根据索引进行全表扫描

**`all`**：全表扫描，速度最慢，未建立索引时就是使用这种方式。

*   possible_key : 显示可能应用在这张表中的索引,一个或多个。查询涉及到的字段上若存在索引,则该索引将被列出,但不一定被查询实际使用
*   key : 实际用到的索引，若为null，表示没有使用索引

*   ref：表的哪些字段被使用
*   rows：大致估计每张表需要读取多少行
*   extra：显示一些其他额外的信息
    *   Using filesort：没有办法利用现有索引进行排序，需要额外排序，建议：根据排序需要，创建相应合适的索引
    *   Using temporary：需要用临时表存储结果集，通常是因为group by的列列上没有索引。也有可能是因为同时有group by和order by，但group by和order by的列又不一样
    *   **出现以上两个表示需要优化**
    *   Using index：使用==**覆盖索引**==，速度更快

关于覆盖索引，想要查询的数据根据索引就可全部获得不用回表查询，这就是覆盖索引。

```mysql
create index idx_user_name on user(name);

# 此时根据name索引可以获得name和id(因为非聚集索引存储聚集索引的值)，此时即为覆盖索引
select id,name from user where name = 'lisi'

# 此时无法覆盖索引，因为无法获取gender的值，若想覆盖索引需要添加name-gender联合索引
select id,name,gender from user where name = 'lisi'
```

### 存储引擎

MyISAM

frm(表结构文件)、MYD(存储数据)和MYI(存储索引的数据结构)

InnoDB

frm(存储表结构)、ibd(本身就是一个包含索引和数据的b+树文件)

innodb必须要有主键(推荐整型，自增)，若未设置会选择某一列为主键，没有合适的主键会添加一列数据用于维护主键(mysql维护，看不到)

改写成 union 查询

|         | MylSAM                                                       | InnoDB                                                       |
| ------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 主键    | 允许没有任何索引和主键的表存在， myisam的索引都是保存行的地址。 | 如果没有设定主键或者非空唯一索引，就会自动生成个6字节的主键（用户不可见）, innodb的数据是主索引的一部分，其他索引保存的是主索引的值。 |
| 事务    | MyISAM类型的表强调的是性能，其执 行数度比InnoDB类型更快，但是不提供 事务支持、不支持外键 | InnoDB提供事务支持事务，外部键（foreign key）等高级数据库功能 |
| DML操作 | 如果执行大量的SELECT,MyISAM是 更好的选择                     | 1.如果你的数据执行大量的INSERTUPDATE，出于性能方面的考虑，应该使用InnoDB表 <br />2.DELETE FROM table时, InnoDB不会重新建立表，而是一行一行的删除。 |
| 自增    | myisam引擎的自动增长列必须是索 引，如果是组合索引，自动增长可以不 是第一列，他可以根据前面几列进行排 序后递增。 | innodb引擎的自动增长必须是索引，如果是组合索引也必须是组合索引的第一列。 |
| count() | 函数 myisam保存有表的总行数，如果select count（）from table;会直接取出出该值 | innodb没有保存表的总行数，如果使用select count（）from table;就会遍历整个表，消耗相 当大，但是在加了wehre 条件后，myisam和innodb处理的方式都一样。 |
| 锁      | 表锁                                                         | 提供行锁，另外，InnoDB表的行锁也不是绝对的，如果在执行—个SQL语句时MySQL不能 确定要扫描的范围,InnoDB表同样会锁全表，例如update table set num=1 where name like "%aaa%" |

### 索引失效及优化

*   最佳左前缀法则
    *   创建符合索引后，若要使用索引，需要遵守最佳做前缀法则，不能跳过某个索引。

```mysql
create index idx_user_nameAgeGender on user(name,age,gender);
# 其左前缀是name、name-age，name-age-gender，这样才能够使用索引

# 以下三个会使用索引
select * from user where name='zhangsan';
select * from user where name='zhangsan' and age=18;
select * from user where name='zhangsan' and age=18 and gender=1;

# 以下不会使用索引
select * from user where age=18;
select * from user where gender=1;
select * from user where age=18 and gender=1;

# 以下只会使用name索引
select * from user where name='zhangsan' and gender=1;
```

*   不在索引列上做任何操作（计算、函数、(自动or手动)类型转换)，会导致索引失效而转向全表扫描

```mysql
# 此句表示，name字段前四位为aaaa的用户
# 这会导致索引失效
select * from user where left(name,4)='aaaa';
```

*   存储引擎不能使用索引中范围条件右边的列

```mysql
create index idx_user_nameAgeGender on user(name,age,gender);

# age为范围条件，age及age后的所有索引失效like,>,between
select * from user where name='zhangsan' and age>18 and gender=1;
```

*   尽量使用覆盖索引(只访问索引的查询(索引列和查询列一致))，减少`select *`
*   mysql 在使用==**不等于**==的时候无法使用索引会导致全表扫描
*   is null ,is not null 也无法使用索引
*   like==**以通配符开头(’%abc...‘)**==mysql索引失效会变成全表扫描的操作
*   字符串==**不加单引号**==索引失效
*   ==**避免使用or**==,用它来连接时会索引失效，==**改写成 union 查询**==

```mysql
select id from t where num=10 or num=20    
# 可改写为    
select id from t where num=10    
union all    
select id from t where num=20    
```

##### 优化

首先应考虑在 where 及 order by 涉及的列上建立索引

尽可能减少Join语句中的NestedLoop的循环总次数; ==**“永远用小结果集驱动大的结果集”**==。

```mysql
select * from stu left join class on stu.class_id=class.id

select * from class left join stu on class.id=stu.class_id

# 班级的个数比学生的个数少，应该用class去join stu，这样可以减少循环次数
```

优先优化最内层循环

用 exists 代替 in 是一个好的选择

```mysql
select num from a where num in(select num from b)    
# 用下面的语句替换：    
select num from a where exists(select 1 from b where num=a.num) 
```

### 慢查询

查询sql响应时间超过设置的时间的SQL语句

默认情况下，MySQL数据库没有开启慢查询日志，需要我们手动来设置这个参数。 当然，如果不是调优需要的话，一般不建议启动该参数，因为开启慢查询日志会或多或少带来一定的性能影响。慢 查询日志支持将日志记录写入文件

```mysql
# 查询是否开启慢查询，及慢SQL日志存储位置
SHOW VARIABLES LIKE '%oslow query log%';

# 开启慢查询，只对当前数据库有效，且重启失效
set global slow_query_log=1

# 查询慢SQL的最长查询时间，单位为秒
SHOW VARIABLES LIKE "long_query_time%";

# 更改最长查询时间，此命令需要重新打开一个会话层可以看到修改
set global long_query_time=3
# 无需新开会话 即可看到修改
show global variables like long_-query_time
```

### 主从复制

在一般的应用中，总是读多写少，读的操作会占用大量资源，可以使用==**主从复制**==，从库同步主库的数据，而应用从主库写入数据，从从库读取数据，从而实现==**读写分离**==。

主从复制，主库会将所进行的写操作写入到 binlog日志中，从库将主库的日志文件读取到relaylog中，通过日志实现对主库数据的复制。

*   主机配置

     修改mysql的配置文件：`vim /etc/my.cnf `

```properties
# 主服务器唯一ID，习惯上设置主库为1
server-id=1 
# 启用二进制日志 
log-bin=master-bin 
# 设置不要复制的数据库(可设置多个)”
# 例如系统库mysql不需要复制
binlog-ignore-db=mysql 
binlog-ignore-db=information_schema 
# 设置需要复制的数据库 
binlog-do-db=需要复制的主数据库名字 
# 设置logbin格式 
binlog_format=STATEMENT
```

​	binlog日志的三种格式：

|   格式    | 方式                                                         | 优缺点                                                       |
| :-------: | ------------------------------------------------------------ | ------------------------------------------------------------ |
| STATEMENT | 基于语句的复制，日志文件记录为sql语句，从库执行相应的sql语句完成数据库的复制。 | 优点：binlog文件较小，节约I/O，性能较高。<br>缺点：不是所有的数据更改都会写入binlog文件中，尤其是使用MySQL中的一些特殊函数（如LOAD_FILE()、UUID()等）和一些不确定的语句操作，或是更改时间(now)等，从而导致主从数据无法复制的问题。 |
|    Row    | 基于行的复制，记录每一行数据的变化，而不是sql语句            | 优点：不会由于使用一些特殊函数或其他情况导致不能复制的问题。<br>缺点：由于row格式记录了每一行数据的更改细节，会产生大量的binlog日志内容，性能不佳，并且会增大主从同步延迟出现的几率。 |
|   MIXED   | 混合复制，一般情况使用statement，特殊情况使用row方式         |                                                              |



*   从机配置

    修改配置文件:`vim /etc/my.cnf `

```properties
#从服务器唯一ID
server-id=2 
#启用中继日志 
relay-log=mysql-relay
```

*   授权账户

    在主库执行以下命令

```mysql
# 创建一个名为2333，密码为123456的用户，允许它通过192.168.126.*ip进行连接，可以直接用%代替ip
grant replication slave on *.*  to '2333'@'192.168.126.%'  identified by '123456';

# 刷新权限
FLUSH PRIVILEGES;

# 显示主库的信息，主要记录file和position
show master status;
# 重置master
reset master;
```

​		在从库执行以下命令

```mysql
CHANGE MASTER TO
MASTER_HOST='主库ip地址',
MASTER_USER='主库授权的用户',
MASTER_PASSWORD='授权密码',
MASTER_LOG_FILE='查询到的file',
MASTER_LOG_POs=查询到的position;

# 开启从服务器
start slave;
```

```mysql
# 停止主从复制
stop slave;

# 展示从库状态
show slave status;
# 主要看以下两个都是yes成功
Slave IO Running:Yes 
Slave SQL Running: Yes

# 重新配置主从复制
reset master;
```





