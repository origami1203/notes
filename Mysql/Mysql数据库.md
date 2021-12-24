# MYSQL数据库

### 登录和退出

>   mysql  -u    -p 									  登录本地mysql
>
>   mysql -h ip地址 -u 账号 -p 密码 				远程登录mysql
>
>   exit/quit

### 通用语法

SQL语句可以行或多行书写，以分号结尾。

可使用空格和缩进来增强语句的可读性。

MySQL数据库的SQL语句不区分大小写，关键字建议使用大写。

##### 注释

- `-- 注释内容` : sql通用单行注释,注意--之后的空格
- `# 注释内容` : MySQL专用单行注释
- `/* 注释内容 */` : 多行注释

### 常用数据类型

##### 整形

- int(n)  
- tinyint 一个字节小整数 -128~127
- smallint  两个字节

int(5)，后面的数据表示要显示的位数

##### 浮点型

- float(总位数,小数位数)：单精度，4个字节
- double(总共位数,小数位数)：8个字节
- decimal(总共位数,小数位数) : 不会损失精度,用于银行等财物方面

##### 字符串

- varchar(允许的字符长度) : 最大65535，长度可变，根据长度自动变化
- char(字符长度): 长度不可变，始终占据指定长度，不够补上，一般用于长度不变的时候，如uuid，性别等

##### 日期

- data : 3字节，格式yyyy-MM-dd

  范围1000-01-01到9999-12-31

- datatime : 8字节，yyyy-MM-dd HH:mm:ss

  范围 1000-01-01 00:00:00到9999-12-31 23:59:59

- datastamp : 时间戳,未赋值或赋值为null,会自动赋值当前时间

  范围 : 970-01-01 00:00:00/2038
  结束时间是第 2147483647 秒，北京时间 2038-1-19 11:14:07，格林尼治时间 2038年1月19日 凌晨 03:14:07

##### 媒体文件

- BLOB

	- 存储大的二进制文件，如图片，电影等

- CLOB

	- 存储大的文本信息，最大可达4g

### SQL语句分类

##### DDL

- 用于操作数据库,数据表

	- 对数据库和表结构的修改
	
- create  drop  alter
	
- 数据库CRUD

	- 创建数据库

		- CREATE DATABASE 数据库名称;

	- 查看数据库

		- SHOW DATABASES;

	- 删除数据库

		- DROP DATABASE 数据库名称;

	- 更改数据库编码

		- ALTER DATABASE 数据库名称 CHARACTER SET 编码名称;

	- 进入/切换数据库

		- USE 数据库名称;

	- 查看正在使用的数据库

		- SELECT DATABASE( );

- 数据表CRUD

	- 查看当前数据库内所有数据表

	```mysql
	SHOW TABLES;
	```

	- 查询表结构

		- DESC 表名;

	- 创建数据表
    
    ```mysql
    CREATE TABLE 表名 (
	    列名1 列数据类型1,
        列名2 列数据类型2,
	    ......
         列名n 列数据类型n
	       )ENGINE=innodb DEFALUT CHARSET= utf8;
	  ```
	>    注意  : 最后一个列不要加逗号
	
- 复制表
	
	- CREATE TABLE 表名 LIKE 被复制的表名;
	
- 删除表
	
	- DROP TABLE 表名;
	
- 修改表
	
	- 修改表名
	
	```mysql
	ALTER TABLE 表名 RENAME TO 新的表名;
	```
	
	- 修改编码
	
		- ALTER TABLE 表名 CHARACTER SET 字符集;
	
	- 在表中添加列
	
			- ALTER TABLE 表名 ADD 列名 数据类型;

		- 修改列名或数据类型
	
		- ALTER TABLE 表名 CHANGE 列名 新列名 新数据类型;
			- ALTER TABLE 表名 MODIFY 列名 新数据类型;
	
		- 删除表中某列
	
			- ALTER TABLE 表名 DROP 列名;

##### ==DML==

- 用于增删改数据表中的数据

	- 对表中数据的操作
	
- iinsect delete update
	
- 添加数据

  - ```mysql
    INSERT INTO 表名(
        列名1,列名2,...列名n
        ) VALUES(
    	值1,值2,...值n
    );
    ```
  
  >   列名与值要一一对应
  >   除数字类型外.其它类型需要使用引号
  
    ```mysql
  # 此种格式要把所有的列表的值都给出
  INSERT INTO 表名 VALUES(值1,值2,...值n);
    ```
  
- 删除数据

	- DELETE FROM 表名 [ WHERE 条件 ]

	  删除明珠条件的数据,当没有where条件时,将删除表中所有数据,不推荐使用

	- TRUNCATE TABLE 表名;

	  删除所有数据,其实是删除了整个表,然后创建了一个名字一样的表

- 修改数据

	- UPDATE 表名 SET 列名1 = 值1, 列名2 = 值2...[WHERE条件];

	  如果不加where条件,则所有数据的相应列值都会改变

##### ==DQL==

- 用于查询数据表中的数据

	- select

- 基础查询

  - 查询表中要找的字段

    - SELECT * FROM 表名; 

    - ```mysql
       SELECT 
       	字段1,
       	字段2
       ```

     	...  
       FROM 
       	表名;

       ```
    
       ```

  - 去除重复

  	- 使用关键字DISTINCT
  	
  - `SELECT DISTINCT 字段 FROM 表名;`
  	
  - 列数据计算

    - ```mysql
        SELECT 
        	字段1*5,
        	字段2-2,
        	字段3,
    	字段2 + 字段3
        FROM
        	表名;
        ```

  - 起别名

  	- ```mysql
        SELECT 
        	字段1 AS 别名,
        	字段2 AS 别名,
        	字段3 AS 别名,
  	    字段2 + 字段3 AS 别名
        FROM
        	表名;
        ```
        
        

- WHERE : 条件查询

	- 使用where进行条件选择，条件语句位于  WHERE  之后
	- >、<、<=、>=、=、!=、<>(不等于)
	- BETWEEN...AND : 位于...之间

	  `SELECT * FROM stu WHERE age BETWEEN 20 AND 30;`
	  查询stu表中年龄字段在20到30之间

	- IN(集合) : 与正则中的或相似

	  `SELECT * FROM stu WHERE id IN (1,2,3)` 表示123中的任一个
	  从stu表中查询id字段为1或2或3的人

	- IS [NOT] NULL : 是否为空
	- and 或 &&
	- or 或 ||
	- not 或 !
	- LIKE : 模糊查询

		- _(下划线) : 表示任意一个字符

		  `SELECT * FROM stu WHERE name LIKE '王_';`
		  从stu表中查询name字段中叫'王x'的人

		- % : 表示任意字符

		  `SELECT * FROM stu WHERE name LIKE '王%';`
		  从stu表中查询name字段中所有以王开头的人

- 排序查询 : ORDER BY

	- SELECT * FROM exam ORDER BY 
        第一排序字段1 排序方式1,
        第二排序字段2 排序方式2..

	  单列直接排序，当有多列时先按照第一排序字段的方式排,若第一字段的值相等,再按第二字段排序

	- 排序方式

		- ASC : 升序(默认)
		- DESC : 降序

- 聚合函数

	- 将一列数据视为一个整体，如计算一列的平均分，查询一列有多少个数据
	- SELECT 函数名(列名) FROM 表名;

	  SELECT COUNT(name) FROM student;
	  计算student表中name列中多有多少个name值

	- COUNT

		- 用于计数

	- MAX

		- 求列中最大值

	- MIN

		- 最小值

	- SUM

		- 总和

	- AVG

		- 平均数

- 分组查询 : GROUP BY

	- SELECT (分组列名|聚合函数) FROM 表名 [WHERE 条件] GROUP BY 分组列名 [HAVING 条件];

	  SELECT sal，count(*) WHERE sal >1000 GROUP BY sal HAVING count > 2;
	  选择sal且sal大于1000的数据进行分组，且分组后count数大于2

	- WHERE : 用于GROUP BY之前，对分组前的数据进行条件选择
	- HAVING : 用于GROUP BY之后，对分组后的数据进行选择

- 分页查询 : LIMIT(方言)

	- SELECT * FROM 表名 LIMIT 本页的起始索引, 一页需要展示的数据条数；
	- 页数计算公式

		- 开始索引 = （查询页数 - 1｝* 每页数据条数

	- 页面分页页面展示规则.png

- 同一语句中语法顺序

	- select
	- from
	- where
	- group by
	- having
	- order by

##### DCL

- 用于访问权限设定和安全访问等

### 约束

##### 非空约束

- 特点

	- 列的值不能为空

- 创建表时添加约束

  - 列名与数据类型后添加NOT NULL

  - ```mysql
    CREATE TABLE stu(
        id INT,
      name VARCHAR(20) NOT NULL
    );
    ```
    
    
    此即为设置name列为非空

- 删除列的非空约束

	- 使用修改ALERT将列属性中的约束NOT NULL删去即可
	- ALTER TABLE 
        stu 
    MODIFY 
        name VARCHAR(20);
    此即为删除name上的非空约束

- 创建表后添加约束

	- 使用修改ALERT将列重新属性,添加约束即可
	- ALTER TABLE 
        stu 
    MODIFY 
        name VARCHAR(20) NOT NULL;
    此即为给name列添加非空约束

##### 唯一约束

- 特点

	- 被修饰列的值不能重复
	
- 多个null值不为重复
	
- 创建表时添加约束

  - 列名与数据类型后添加UNIQUE

  - ```mysql
    CREATE TABLE stu(
        id INT UNIQUE,
      name VARCHAR(20)
    );
    ```
    
    
    此即为设置id列不能重复

- 删除列的约束

	- 使用ALERT和DROP INDEX删除
	- ALTER TABLE 
        stu 
    DROP INDEX 
        id;
    此即为删除id上的唯一约束

- 创建表后添加约束

	- 使用修改ALERT将列重新属性,添加约束即可
	- ALTER TABLE 
        stu 
    MODIFY 
        id INT UNIQUE;
    此即为给name列添加唯一约束

##### 主键约束

- 特点

	- 非空且唯一
	- 一张表只有一个字段为主键
	- 用于作为数据的唯一标识

- 创建表时添加主键

	- ```mysql
    CREATE TABLE stu(
        sid CHAR(6)　PRIMARY KEY,
        sname VARCHAR(20),
        age INT,
    gender VARCHAR(10)   
    );
    
    CREATE TABLE stu(
        sid CHAR(6),
        sname VARCHAR(20),
        age INT,
        gender VARCHAR(10),
        PRIMARY KEY (sid)
    );
    ```

- 删除列的非空约束

	- 使用修改ALERT和DROP PRIMARY删去即可
	- ALTER TABLE 
        stu 
    DROP
        PRIMARY KEY;
    此即为删除stu表中主键约束(因为一个表只有一个主键,所有不用指定列

- 创建表后添加主键约束

	- ALTER TABLE 
        stu 
    MODIFY 
        id INT PRIMARY KEY;
    此即为给id列添加唯一约束
	- ALTER TABLE 
        Stu 
    ADD PRIMARY KEY(id);
    给id列添加主键

##### 自增长

- 关键字 :auto_increment
- 用于int数字列,本列值在上一行的基础上自动添加1
- 自增长的列的值设为NULL即会自增长
- 一般和主键联合使用,写于PRIMARY后面
- 创建表时添加自增长

	- CREATE TABLE stu(
        id INT PRIMARY KEY AUTO_INCREMENT,-
        name VARCHAR(20)
    );

- 创建表后添加自增长

	- ALTER TABLE
        Stu 
    MODIFY 
        id INT AUTO INCREMENT;

- 删除自增长

	- ALTER TABLE 
        stu 
    MODIFY 
         id INT;

##### 外键约束

- 创建表时添加外键

	- CREATE TABLE 表名(
        ......
        外键列,
        CONSTRAINT 外键名称 FOREIGN KEY(本表外键列名称) REFERENCES 主表名称(主表列名称,一般为主键列)
    );

- 删除外键

	- ALTER TABLE 
        表名
    DROP FOREIGNKEY 外键名称；

- 添加外键

	- ALTER TABLE 
        表名
    ADD
    CONSTRAINT  外键名称  FOREIGN KEY (外键字段名称) REFERENCES 主表名称(主表列名称);

- 级联操作

	- 更改主表后,关联的列自动修改和删除
	- 级联更新

		- ALTER TABLE employee ADD CONSTRAINT emp dept fk FOREIGN KEY(dep id)REFERENCES department(id) ON UPDATE CASCADE;

	- 级联删除

		- ALTER TABLE employee ADD CONSTRAINT emp dept fk FOREIGN KEY(dep id)REFERENCES department(id) ON DELETE CASCADE;

	- 子主题 4

		- ALTER TABLE  表名
ADD CONSTRAINT 外键名
FOREIGN KEY(字段名)REFERENCES 主表名(字段名)
ON DELETE CASCADE

### 多表之间的关系

##### 一对一

- 一对一关系实现，可以在任意一方添加唯一外键指向另一方的主键。
- 方法2让其中一张表的主键，即是主键又是外键。

##### 一对多(多对一)

- 在多的一方建立外键，指向一的一方的主键。

##### 多对多

- 多对多关系实现需要借助第三张中间表。
中间表至少包含两个字段，这两个字段作为第三张表的外键，分别指向两张表的主键

### 数据库的备份与还原

##### 命令行

- 备份

	- MYSQLDUMP -u用户名 -p密码 备份数据库名称 > 保存路径  会生成一个sql文件

- 还原

	- 创建数据库: CREATE DATABASE 数据库名;
	- 使用数据库 : USE 数据库名;
	- SOURCE 路径下的sql文件;

##### 图形化工具

- 备份

	- 选择数据库  备份/导出  转存到sql  导出到指定位置

- 还原

	- 执行sql脚本,选择之前的sql文件导入执行

### 多表查询

##### 连接查询

- 内连接

	- SELECT * FROM 表1,表2;(方言)

	  会把所有情况罗列出来排列组合

	- SELECT * FROM 表1 别名1 INNER JOIN 表2 别名2 ON 条件;(标准查询语句)
	- SELECT * FROM 表1 别名1 NATURAL JOIN 表2 别名2;不需要加条件,会自动选择相同的列进行筛选

- 外连接

	- 左外连接

		- SELECT * FROM 表1 别名1 LEFT OUTER JOIN 表2 别名2 ON 条件;(标准查询语句)

	- 右外连接

		- SELECT * FROM 表1 别名1 RIGHT OUTER JOIN 表2 别名2 ON 条件;(标准查询语句)

##### 子查询

- 单行单列

	- 一般作为条件,可以使用运算符去判断(> < >= <=)

- 多行单列

	- 一般作为条件,可以使用ANY ALL等

- 多行多列

	- 作为查询表

### 事务

##### 特点

- 原子性Atomicity) : 事务不可被分割，要么全部成功，要么全部失败
- 一致性(consistency) : 事务执行后，数据库状态与其它业务规则保持一致。如转账业务，无论事务执行成功与否，参与转账的两个账号余额之和应该是不变的。
- 隔离性(1solation):隔离性是指在并发操作中,不同事务之间应该隔离开来,使每个并发中的事务不会相互干扰.
- 持久性(Durability):一旦事务提交成功,事务中所有的数据操作都必须被持久化到数据库中,即使提交事务后,数据库马上崩溃,在数据库重启时,也必须能保证通过某种机制恢复数据.

##### 事物的操作

- start transaction : 开启事务
- commit : 提交事务
- rollback : 回滚事务

##### 事务的隔离级别

- 问题

	- 幻读 : 一个事务操作（DML）数据表中所有记录，另一个事务添加了一条数据。
	- 不可重复读 : 同一事物中,两次读取到的数据不一样
	- 脏读 : 一个事务读取到另一个没有提交的数据

- 隔离级别

	- read uncommit : 读未提交,可能产生上述三个问题
	- read commited : 读已提交,产生不可重复读和幻读,解决脏读
	- repeatable read : 可重复读,产生问题幻读
	- serializable : 串行化,可解决上述三个问题

- 隔离级别的设置(不常用)

	- `set global transaction isolation level 级别字符串`；设置隔离界别
	- `select @@tx_isolation;` 查询隔离级别

> 数据库锁与ORM乐观悲观锁
>
> 默认RR隔离级别下，使用select语句不会加锁，`select... for update`会加锁，update、delete、insert默认会加锁，被锁记录不允许两个事务同时写，先抢到的获取锁，没抢到的阻塞，先抢到锁的事务提交释放锁后，另一个事务才会执行写操作。

|                          事务1                          |                            事务2                             |
| :-----------------------------------------------------: | :----------------------------------------------------------: |
|                        开启事务                         |                           开启事务                           |
| select * from user where id = 1<br />id = 1,money = 100 |                                                              |
|         update user set money = 200 where id =1         | select * from user where id = 1<br />id = 1,money = 100(快照读) |
|                                                         |                                                              |
|                                                         |                                                              |
|                                                         |                                                              |
|                                                         |                                                              |
|                                                         |                                                              |
|                                                         |                                                              |
|                                                         |                                                              |



### 判断语句

IFNULL(可能为NULL的字段, 为NULL时的代替值)

