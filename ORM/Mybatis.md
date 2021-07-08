# Mybatis

### 1. 简介

##### 1.1 什么是mybatis

[MyBatis](https://mybatis.org/mybatis-3/zh/index.html) 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

##### 1.2 为什么要使用mybatis

* 简单易学
* 灵活
* sql和代码的分离，提高了可维护性。
* 提供映射标签，支持对象与数据库的orm字段关系映射提供对象关系映射标签，支持对象
* 系组建维护提供xml标签，支持编写动态sql。

### 2. 使用案例

##### 2.1 搭建环境

1. 搭建数据库

``` mysql
CREATE DATABASE `mybatis`;
USE mybatis;
CREATE TABLE `user`(
	uid INT(20) PRIMARY KEY,
	username VARCHAR(20),
	`password` VARCHAR(16)
);
INSERT INTO `user` VALUES(1,'张三','123456');
INSERT INTO `user` VALUES(2,'李四','123456');
INSERT INTO `user` VALUES(3,'王五','123456');
```

2. 创建MAVEN项目
3. 修改pom.xml,建立依赖

##### 2.2 代码编写

1. SqlMapConfig.xml配置文件


```xml
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--核心配置文件-->
<configuration>
    
    <!--默认配置环境-->
    <environments default="development">
        <environment id="development">
            
            <!--事务管理,使用JDBC-->
            <transactionManager type="JDBC"/>
            <!-- 数据池 -->
            <dataSource type="POOLED">
                
                <!--mysql四大参数-->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
    
     <!--注册对应mapper到config文件中-->
    <mappers>
        <!--resources为对应的mapper.xml文件的位置-->
        <mapper resource="com/origami/mybatis/dao/UserMapper.xml"/>
    </mappers>

</configuration>
```

* 文件一般放于resources目录下,可以编写一个工具类,通过读取SqlMapConfig.xml文件来获取**SqlSessionFactory**对象,MyBatis 包含一个名叫 Resources 的工具类，它包含一些实用方法，使得从类路径或其它位置加载资源文件更加容易。

  * SqlSessionFactory对象是用来创建SqlSession的工厂
  * SqlSession即是用来执行sql语句的主要对象,就像是原生jdbc中的connection和dbutils中的QueryRunner

  ```java
  String resource = "/SqlMapConfig.xml";
  InputStream inputStream = Resources.getResourceAsStream(resource);
  SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
  ```

2. **domain编写**

```java
public class User {
    private int uid;
    private String username;
    private String password;

    ...
}
```

3. mapper类编写(dao接口)

```java
public interface UserMapper {
    List<User> findAllUser();
}
```
>   dao接口,而在mybatis中,接口通常被称作**Mapper**

4. mapper配置文件(dao实现)

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--UserMapper对应的映射文件-->

<!--namespace属性即为想要实现的mapper(dao)接口对应的全限定类名-->
<mapper namespace="com.origami.mybatis.dao.UserMapper">

    <!-- select标签代表是查询语句 -->
    <!--id对应Mapper中的方法名-->
    <!--resultType对应返回的bean类型的全限定名-->
    <select id="findAllUser" resultType="com.origami.mybatis.domain.User">
        <!-- sql语句 -->
        select * from user
    </select>

</mapper>
```

* 原来我们需要编写dao的实现类来实现接口,现在则变为编写对应的mapper.xml文件来实现

5. 使用

```java
public class UserMapperTest {
    @Test
    public void findAllUserTest() {
        
        //获取sqlSession对象,自定义获取sqlSession的工具类
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        //获取mapper的实现
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

        //查询结果
        List<User> userList = userMapper.findAllUser();

        for (User user : userList) {
            System.out.println(user);
        }

        //关闭资源
        sqlSession.close();

    }
}
```

### 3. Mapper.xml

##### 3.1 Mapper配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--UserMapper对应的映射文件-->

<!--namespace属性即为想要实现的mapper(dao)接口对应的全限定类名-->
<mapper namespace="com.origami.mybatis.dao.UserMapper">

    <!-- select标签代表是查询语句 -->
    <!--id对应Mapper中的方法名-->
    <!--resultType对应返回的bean类型的全限定名-->
    <select id="findAllUser" resultType="com.origami.mybatis.domain.User">
        <!-- sql语句 -->
        select * from user
    </select>

</mapper>
```



##### `3.2 <select>`标签

* **id** 属性 : 根据要调用的id执行相应的SQL语句,对应Mapper接口中的方法名
* **parameterType **属性 : 该方法传递的参数的数据类型(**常用类型列表在后面**)
* 封装结果类型
    * **resultType **属性 : 返回值的数据类型(domain要写全限定类名,集合写集合的泛型)
    * [**resultMap**](#5. 结果集映射) 属性 : 返回指定的map集合(后面详细讲)
* sql语句 : 需要传递参数的语句使用 **==#{ 传递参数的名称 }==**来占位

> Mapper类

```java
public interface UserMapper {
    //根据id查询用户
    User findUserById(int id);
}
```

> Mapper.xml文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.origami.mybatis.dao.UserMapper">

    <select id="findUserById" parameterType="int" resultType="com.origami.mybatis.domain.User">
        select * from user where uid = #{id}
    </select>

</mapper>
```

> 使用

```java
@Test
public void findUserById() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();

    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    User user = mapper.findUserById(2);

    System.out.println(user);

    sqlSession.close();
}
```

##### `3.3 <insert>` `<update>` `<delete>`

* `<insert>` `<update>` `<delete>`标签与`<select>`用法类似
  * **<font color = red>注意</font>** : 增删改方法需要**<font color = red>提交事务</font>**才能执行成功
  * 传递参数为对象时,可以直接使用 **==#{对象属性}==** 来表示对象中的值
  * 当传递参数为多个值时,传递一个map集合,然后parameterType="map"
* **parameterType **对应的的类型

|    别名    | 映射的类型 |
| :--------: | :--------: |
|   _byte    |    byte    |
|   _long    |    long    |
|   _short   |   short    |
|    _int    |    int     |
|  _integer  |    int     |
|  _double   |   double   |
|   _float   |   float    |
|  _boolean  |  boolean   |
|   string   |   String   |
|    byte    |    Byte    |
|    long    |    Long    |
|   short    |   Short    |
|    int     |  Integer   |
|  integer   |  Integer   |
|   double   |   Double   |
|   float    |   Float    |
|  boolean   |  Boolean   |
|    date    |    Date    |
|  decimal   | BigDecimal |
| bigdecimal | BigDecimal |
|   object   |   Object   |
|    map     |    Map     |
|  hashmap   |  HashMap   |
|    list    |    List    |
| arraylist  | ArrayList  |
| collection | Collection |
|  iterator  |  Iterator  |

##### 3.4 `<sql>`

​	`<sql>`元素可以用来定义可重用的 SQL 代码片段，以便在其它语句中使用。 

* 定义`<sql>`代码块

```xml
<!-- id用于让通过include调用 -->
<sql id="userColumns"> 
    ${alias}.id,${alias}.username,${alias}.password 
</sql>
```

* `<include>`引用代码块

```xml
<include refid="userColumns">
    <property name="alias" value="t2"/>
</include>
```

* 注意 : 
    * 最好基于单表来定义SQL片段！
    * 不要存在`<where>`标签

### 4. 配置文件解析

##### 4.1 配置文件结构

​	MyBatis 的配置文件SqlMapConfig.xml包含了会深深影响 MyBatis 行为的设置和属性信息。 配置文档的顶层结构如下：

* configuration（配置）
  * properties（属性）
  * settings（设置）
  * typeAliases（类型别名）
  * typeHandlers（类型处理器）
  * objectFactory（对象工厂）
  * plugins（插件）
  * environments（环境配置）
    * environment（环境变量）
      * transactionManager（事务管理器）
      * dataSource（数据源）
  * databaseIdProvider（数据库厂商标识）
  * mappers（映射器）
  
  ==配置标签按照如上**顺序**从上到下配置,否则会报错。下面介绍几个重要的配置==

##### 4.2 environments（环境配置）

```xml
<!-- default属性显示要加载的环境的id,可有多个development标签，指定id即可更换环境 -->
<environments default="development">
    
    <!-- id属性定义此环境的id,通过id切换环境 -->
  <environment id="development">
      
      <!-- 事务管理的type有两个取值,都会选择JDBC,基本上不会用到MANAGED -->
    <transactionManager type="JDBC">
    </transactionManager>
      
      <!-- 连接池,默认POOLED type="[UNPOOLED|POOLED|JNDI]",一般无需更改
		POOLED mybatis的数据库连接池
		UNPOOLED 不使用连接池
		JNDI 采用服务器提供的JNDI技术实现，来获取DataSource对象，不同的服务器所能拿到的连接池不同,注意：如果不是web或者maven的war工程，是不能使用的。我们课程中使用的是tomcat服务器，采用连接池就是dbcp连接池。
		-->
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
    
    <!--测试环境-->
    <environment id="test">
    <transactionManager type="JDBC">
      <property name="..." value="..."/>
    </transactionManager>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```

##### 4.3 属性（properties）

  * 用于读取properties文件,如上面的环境变量中数据库中的4个参数全部被写死了,我们可以通过引入外部prop文件来配置这些属性

  * 该属性也可以在内部设置`<property>`属性,若内部外部均有配置,读取外部properties文件

    ```properties
    # properties文件
    driver=com.mysql.jdbc.Driver
    url=jdbc:mysql://localhost:3306/mydb3
    username=root
    password=1234
    ```
```xml
<!-- 引入db.properties文件 -->
<property resource="db.properties"/>

<!-- 使用${key}来配置文件 -->
<dataSource type="POOLED">
    <property name="driver" value="${driver}"/>
    <property name="url" value="${url}"/>
    <property name="username" value="${username}"/>
    <property name="password" value="${password}"/>
</dataSource>
```

##### 4.4 类型别名（typeAliases）

类型别名是为Java类型设置一个短的名字。存在的意义仅在于用来减少类完全限定名的冗余。

以下常用的方式,推荐**方式一**

* ==方式一==

```xml
<typeAliases>
    <!-- 给com.origami.mybatis.domain.User起别名为User -->
    <typeAlias type="com.origami.mybatis.domain.User" alias="User"/>
    ...
    ...
</typeAliases>


<!-- 别名前 -->
<select id="findAllUser" resultType="com.origami.mybatis.domain.User">
       select * from user
</select>

<!-- 别名后 -->
<select id="findAllUser" resultType="User">
       select * from user
</select>
```

​	也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，比如：

* 方式二

```xml
<typeAliases>
  <package name="com.origami.mybatis.domain"/>
</typeAliases>
```

​	则`com.origami.mybatis.domain`路径下JavaBean,没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名。 比如 `com.origami.mybatis.domain.User的别名为user`；若有注解，则别名为其注解值。见下面的例子：

* 方式三

```java
@Alias("user")
public class User {
    ...
}
```

##### 4.5 设置（settings）

这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。

下面是可能用到的几个设置。

|          设置名          | 描述                                                         |                            有效值                            | 默认值 |
| :----------------------: | :----------------------------------------------------------- | :----------------------------------------------------------: | :----: |
|       cacheEnabled       | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。     |                        true \| false                         |  true  |
|    lazyLoadingEnabled    | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。 |                        true \| false                         | false  |
| mapUnderscoreToCamelCase | 是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。 |                        true \| false                         | False  |
|         logImpl          | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。        | SLF4J \| LOG4J \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置 |

```xml
<!-- 标签内可设置很多setting -->
<settings>
    <!-- 每个setting的name和value必须完全拼写正确,区分大小写 -->
    <setting name="cacheEnabled" value="true"/>
    <setting name="logImpl" value="LOG4J"/>
</settings>
```

##### 4.6 映射器（mappers）

MyBatis 的行为已经由上述元素配置完了，我们现在就要来定义 SQL 映射语句了。 但首先，我们需要告诉 MyBatis 到哪里去找到这些语句。 在自动查找资源方面，Java 并没有提供一个很好的解决方案，所以最好的办法是直接告诉 MyBatis 到哪里去找映射文件。有四种方式,常用如下两种。


> 方式一**[<font color = red >推荐使用</font>]**
>
> > 1. xml文件可放置在任意路径,通过路径可以找到
> > 2. xml文件可以和mapper类名不同

```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
```

> 方式二
   >> 1. xml文件需要与类名相同
   >> 2. xml文件需要与mapper在同一文件夹下

```xml
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
```

### <font color=red>5. 结果集映射(重点)</font>

我们编写时需要使我们的实体类属性与数据库中的列名一致,这样mybatis才能正确的讲两者映射

当我们的实体类属性与数据库列名不一致时该怎么办呢

|   属性   |   描述   |
| :------: | :------: |
|  `uid`   |    id    |
| username | username |
| password | password |

* 起别名(不推荐)

```xml
<!-- 将id起别名为uid,以保持与实体类一致 -->
<select id="findAllUser" resultType="User">
       select * from user where id as uid = #{uid}
</select>
```

另一种方法就是前面我们说过`<select>`的属性 **resulrMap** .

定义`<resultMap>`标签,完成数据库与实体类的映射

```xml
<!-- id为该resultMap的唯一表示,使用这一id来调用此map -->
<!-- type为想要此map最后转换为的类型 -->
<resultMap id="userMap" type="com.origami.domain.User">
    <!-- property对应实体类中的属性 column对应数据库列名 -->
    <result property="uid" column="id"/>
    <result property="username" column="username"/>
    <result property="password" column="password"/>
</resultMap>

<!-- resultMap引用标签的id -->
<select id="findAllUser" resultMap="userMap">
       select * from user where id = #{uid}
</select>
```

多表查询

```java
//老师类,一个老师有多个学生
public class Teacher {
    private int tid;
    private String name;
    private List<Student> students;
  	...
}
//学生类,每个学生有一个老师
public class Student {
    private int sid;
    private String name;
    private String gender;
    private Teacher teacher;
	...
}
```

##### 多对一(多个学生有同一个老师)

```xml
<resultMap id="student_teacher" type="Student">
    <!-- 前三个属性:学生类对应学生信息 -->
    <result property="sid" column="sid"/>
    <result property="name" column="sname"/>
    <result property="gender" column="gender"/>

    <!--
  后面的属性其实对应的老师类 我们需要将老师类与老师表映射
  <association>映射一个类时使用,property属性的值对应学生类的属性teacher
  javaType表示这个属性实际对应老师类
  <association>中的<result>实际就像上面的学生类的映射一样,将类属性与数据库表的列名对应
  -->
    <association property="teacher" javaType="Teacher">
        <result property="name" column="tname"/>
    </association>
</resultMap>

<select id="getStudentByTeacher" resultMap="student_teacher" parameterType="Teacher">
    select s.sid,s.name sname,s.gender,t.name tname
    from student s,teacher t
    where s.tid = t.tid
</select>
```

##### 一对多(一个老师有多个学生)

```xml
<select id="findTeacherById" parameterType="_int" resultMap="teacher_student">
    select t.tid,t.name tname,s.sid,s.name sname
    from teacher t,student s
    where t.tid = s.tid and t.tid = #{tid}
</select>


<resultMap id="teacher_student" type="Teacher">
	<!--
	与上面多对一类似,前面两个属性对应Teacher类和teacher数据库的列
	-->
    <result property="tid" column="tid"/>
    <result property="name" column="tname"/>

	<!--
	<collection>映射一个泛型集合时使用,此时不能使用javaType来指定集合的泛型类型
	而应该使用ofType属性来指定集合的泛型对象,其他与<association>类似
	-->
    <collection property="students" ofType="Student">
        <result property="sid" column="sid"/>
        <result property="name" column="sname"/>
    </collection>
</resultMap>
```

* 小结
    1. 关联-association【多对一】
    2. 集合-collection【一对多】
    3. javaType & ofType
        1. JavaType用来指定实体类中属性的类型
        * `private Teacher teacher`
        * 此时使用`javaType="Teacher"`
        3. ofType用来指定映射到List或者集合中的pojo类型，泛型中的约束类型！
        * `private List<Student> students`
        * 此时使用`javaType="arraylist" ofType="Student"`



### 6. 日志

​	通过指定SqlMapConfig.xml文件中`<settings>`标签中的`<setting>`标签来指定日志实现

​	这里只介绍log4j

* log4j.properties

```properties
#log4j.rootLogger = [level], appenderName1, appenderName2, …
# ERROR > WARN > INFO > DEBUG
# 指定默认日志级别及要输出日志的appender的名字
log4j.rootLogger = DEBUG,console,logfile

# 自定义变量配置
log.directory=E://logs

### 输出控制抬的日志 ###
# console为log4j.rootLogger中设置的appenderName，必须大小写一致
log4j.appender.console=org.apache.log4j.ConsoleAppender
# 默认日志级别,若不设置,则跟随rootLogger的设置
log4j.appender.console.Threshold=INFO
log4j.appender.console.Target=System.out
log4j.appender.console.layout=org.apache.log4j.PatternLayout
# 指定输出到控制台日志的输出格式,详细百度
log4j.appender.console.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} [%p] [%t] %l >>> %m%n

### 输出logFile的日志 ###
# logfile为log4j.rootLogger中设置的appenderName，必须大小写一致
log4j.appender.logfile=org.apache.log4j.RollingFileAppender
# 日志文件的最大数据大小
log4j.appender.logfile.MaxFileSize=10MB
# 最大日志文件数
log4j.appender.logfile.MaxBackupIndex=50
log4j.appender.logfile.Threshold=INFO
# 追加模式,将日志追加到最后,为false则覆盖原日志
log4j.appender.logfile.Append=true
# 指定日志存放位置
log4j.appender.logfile.File=${log.directory}/test.log

log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
# 日志文件的格式
log4j.appender.logfile.layout.ConversionPattern=%d [%p] [%t] %l >>> %m%n

### 自定义日志模块定义 ###
# 将某个模块输出独立的日志，并将这些日志从原有的日志分离出来
log4j.logger.com.pwx.test.ZkClientDemo=INFO,zkDemo
log4j.appender.zkDemo=org.apache.log4j.RollingFileAppender
# 设置不在父级别logger里面输出
log4j.additivity.zkDemo=false
log4j.appender.zkDemo.MaxFileSize=10MB
log4j.appender.zkDemo.MaxBackupIndex=50
log4j.appender.zkDemo.Threshold=INFO
log4j.appender.zkDemo.File=${log.directory}/zkDemo.log
log4j.appender.zkDemo.layout=org.apache.log4j.PatternLayout
log4j.appender.zkDemo.layout.ConversionPattern=%d [%p] [%t] %l >>> %m%n
```

### 7. 动态SQL

​	动态SQL即根据不同的条件生成不同的SQL语句,mybatis提供了以下语法用于生成动态SQL

​	语法与JSTL类似

主要是以下几个 : 
* if
* choose (when, otherwise)
* trim (where, set)
* foreach

##### if

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```

##### choose、when、otherwise

​	与java中的switch类似

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

##### where,set,trim

* `<where>`用于解决SQL拼接时`where`与`and`的问题

    ​	可用`where 1 = 1`解决

* `<set>`与`<where>`类似,解决`update`时的问题

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG
  <where>
    <if test="state != null">
         state = #{state}
    </if>
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
```

* `<trim>`用于自定义代码,解决类似`where`和`set`问题
    * `prefix`定义类似`where`和`set`的关键字
    * `prefixOverrides`用于定义可能因条件造成拼接错误的关键字
        * *prefixOverrides* 属性会忽略通过分隔符的文本序列（注意此例中的空格是必要的）。

```xml
<!--用trim定制<where>,实际作用与<where>一致-->
<trim prefix="WHERE" prefixOverrides="AND |OR ">
  ...
</trim>
```

##### foreach

​	与jstl的`<c:forEach>`类似

```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

### 8. 缓存(了解)

##### 8.1 缓存[Cache]

1. 什么是缓存

	* 存在内存中的临时数据。
	
	* 将用户经常查询的数据放在缓存（内存）中，用户去查询数据就不用从磁盘上（关系型数据库数据文件）查询，从缓存中查询，从而提高查询效率，解决了高并发系统的性能问题。

2. 为什么使用缓存？

	* 减少和数据库的交互次数，减少系统开销，提高系统效率。

3. 什么样的数据能使用缓存？

  * 经常查询并且不经常改变的数据。(像redis一样)

##### 8.2 Mybatis缓存

MyBatis系统中默认定义了两级缓存：**一级缓存**和**二级缓存**。

* 默认情况下，只有一级缓存开启。（SqlSession级别的缓存，也称为本地缓存）
* 二级缓存需要手动开启和配置，他是基于namespace级别的缓存。
* 为了提高扩展性，MyBatis定义了缓存接口Cache。我们可以通过实现Cache接口来自定义二级缓存。

**一级缓存**

​	一级缓存默认开启,在SqlSession获取后有效,SqlSession关闭后失效

```java
@Test
public void findTeacherByIdTest() {
    //缓存从此处开启
    SqlSession sqlSession = MybatisUtils.getSqlSession();

    TeacherMapper mapper = sqlSession.getMapper(TeacherMapper.class);

    Teacher teacher = mapper.findTeacherById(1);
    
    

    System.out.println(teacher);

    //到此处失效
    sqlSession.close();
}
```

基本上就是这样。这个简单语句的效果如下:

* 映射语句文件中的所有 select 语句的结果将会被缓存。

* 映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。
* 缓存会使用最近最少使用算法（LRU, Least Recently Used）算法来清除不需要的缓存。
* 缓存不会定时进行刷新（也就是说，没有刷新间隔）。
* 缓存会保存列表或对象（无论查询方法返回哪种）的 1024 个引用。
* 缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。

因此,一级缓存的作用域极小,只有当两次查询同一个对象时有效,基本上没有什么作用,一般只有在用户不断刷新一个页面时使用.

**二级缓存**

* 二级缓存也叫全局缓存，一级缓存作用域太低了，所以诞生了二级缓存

* 基于namespace级别的缓存，一个名称空间，对应一个二级缓存；

* 工作机制

  * 一个会话查询一条数据，这个数据就会被放在当前会话的一级缓存中；
  * 如果当前会话关闭了，这个会话对应的一级缓存就没了；但是我们想要的是，会话关闭了，一级缓存中的数据被保存到二级缓存中；
  * 新的会话查询信息，就可以从二级缓存中获取内容；
  * 不同的mapper查出的数据会放在自己对应的缓存（map）中；

* 使用步骤

  1. 在`SqlMapConfig.xml`中的`<settings>`中开启缓存,虽然mybatis默认时开启缓存的,但是一般我们仍会显示地开启缓存.

  ```xml
  <!-- 显示的开启全局缓存 -->
  <setting name="cacheEnabled" value="true"/>
  ```
2. 在Mapper.xml中添加`<cache/>`标签即可
  
   ```xml
     <cache/>
     ```
  
   这些属性可以通过 cache 元素的属性来修改。比如：
  
   ```xml
     <cache
            eviction="FIFO"
            flushInterval="60000"
            size="512"
            readOnly="true"/>
     ```
  
   这个更高级的配置创建了一个 FIFO 缓存，每隔 60 秒刷新，最多可以存储结果对象或列表的 512 个引用，而且返回的对象被认为是只读的，因此对它们进行修改可能会在不同线程中的调用者产生冲突。
  
   可用的清除策略有：
     
     *  `LRU` – 最近最少使用：移除最长时间不被使用的对象。
     *  `FIFO` – 先进先出：按对象进入缓存的顺序来移除它们。` 
     *  `SOFT` – 软引用：基于垃圾回收器状态和软引用规则移除对象。
     *  `WEAK` – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象。
     
     默认的清除策略是 `LRU`。
     
     **提示** 二级缓存是事务性的。这意味着，当 SqlSession 完成并提交时，或是完成并回滚，但没有执行 flushCache=true 的 insert/delete/update 语句时，缓存会获得更新。
  
* 自定义缓存

  ```xml
  <cache type="com.domain.something.MyCustomCache"/>
  ```

  可以导入外部jar包,详情自己了解,一般不会使用

  一般使用`Redis`

### 9. 分页

##### pagehelper

使用pagehelper

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.5</version>
</dependency>
```

配置

```yaml
# pagehelper   
pagehelper:
    helperDialect: mysql	# mysql的语法
    reasonable: true	# 合理化，当页数<1时设置为1
    supportMethodsArguments: true # 支持通过 Mapper 接口参数来传递分页参数
    params: count=countSql
```

代码

```java
@GetMapping("/getAllPerson")
public String getAllPerson(@RequestParam(defaultValue = "1",value = "pageNum") Integer pageNum){
    PageHelper.startPage(pageNum,5); // 第几页，每页数据
    List<Person> list = personService.getAllPerson();
    PageInfo<Person> pageInfo = new PageInfo<Person>(list); // 将list传给pageinfo即可
    model.addAttribute("pageInfo",pageInfo);
    return "list";
}
```

##### mybatis-plus

集成了分页插件，可以注册进ioc使用

# Mybatis-Plus

[官方文档](https://mp.baomidou.com/guide/)。用于对mybatis进行增强。

### 快速开始

1.  ==**引用mybatis-plus的依赖，不需要再引用mybatis和mybatis-spring，防止依赖冲突。**==

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.3.1</version>
</dependency>
```

2.  在properties或yaml文件中配置数据库配置。

3.  配置mybatis-plus，application.yml中

    ```yaml
    mybatis-plus:
      ......
      mapperLocations: # mapper路径
      typeAliasesPackage: # 别名
      configuration: # 原生 MyBatis 所支持的配置
        ......
        db-config:
    ```

    >   ==**注意：typeAliasesPackage的后面的包路径不能加·`classpath:`**==，否则报错。

4.  mapper继承`BaseMapper<T>`

    mybatis-plus中有一个`BaseMapper<T>`，继承该类即可实现最基本的crud操作。

    *   selectByld：根据id查询
    *   selectBatchIds：批量id查询

```java
public interface UserMapper extends BaseMapper<User> {
    // 父类有基本的crud方法
}
```

5.  主启动类上添加添加`MapperScan("xxx")`，也可以使用@Mapper注解在mapper类上。


开启sql日志信息

```yaml
mybatis-plus: 
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

### 常用注解

##### @TableName

-   描述：表名注解

|       属性       |   类型   | 必须指定 | 默认值 | 描述                                                         |
| :--------------: | :------: | :------: | :----: | ------------------------------------------------------------ |
|      value       |  String  |    否    |   ""   | 表名                                                         |
|      schema      |  String  |    否    |   ""   | schema                                                       |
| keepGlobalPrefix | boolean  |    否    | false  | 是否保持使用全局的 tablePrefix 的值(如果设置了全局 tablePrefix 且自行设置了 value 的值) |
|    resultMap     |  String  |    否    |   ""   | xml 中 resultMap 的 id                                       |
|  autoResultMap   | boolean  |    否    | false  | 是否自动构建 resultMap 并使用(如果设置 resultMap 则不会进行 resultMap 的自动构建并注入) |
| excludeProperty  | String[] |    否    |   {}   | 需要排除的属性名(@since 3.3.1)                               |

关于`autoResultMap`的说明:

mp会自动构建一个`ResultMap`并注入到mybatis里(一般用不上).下面讲两句: 因为mp底层是mybatis,所以一些mybatis的常识你要知道,mp只是帮你注入了常用crud到mybatis里 注入之前可以说是动态的(根据你entity的字段以及注解变化而变化),但是注入之后是静态的(等于你写在xml的东西) 而对于直接指定`typeHandler`,mybatis只支持你写在2个地方:

1.  定义在resultMap里,只作用于select查询的返回结果封装
2.  定义在`insert`和`update`sql的`#{property}`里的`property`后面(例:`#{property,typehandler=xxx.xxx.xxx}`),只作用于`设置值` 而除了这两种直接指定`typeHandler`,mybatis有一个全局的扫描你自己的`typeHandler`包的配置,这是根据你的`property`的类型去找`typeHandler`并使用.

##### @TableId

-   描述：主键注解

| 属性  |  类型  | 必须指定 |   默认值    |    描述    |
| :---: | :----: | :------: | :---------: | :--------: |
| value | String |    否    |     ""      | 主键字段名 |
| type  |  Enum  |    否    | IdType.NONE |  主键类型  |

IdType属性

|         值          |                             描述                             |
| :-----------------: | :----------------------------------------------------------: |
|        AUTO         |                         数据库ID自增                         |
|        NONE         | 无状态,该类型为未设置主键类型(注解里等于跟随全局,全局里约等于 INPUT) |
|        INPUT        |                    insert前自行set主键值                     |
|      ASSIGN_ID      | 分配ID(主键类型为Number(Long和Integer)或String)(since 3.3.0),使用接口`IdentifierGenerator`的方法`nextId`(默认实现类为`DefaultIdentifierGenerator`雪花算法) |
|     ASSIGN_UUID     | 分配UUID,主键类型为String(since 3.3.0),使用接口`IdentifierGenerator`的方法`nextUUID`(默认default方法) |
|   ID_WORKER(过时)   |     分布式全局唯一ID 长整型类型(please use `ASSIGN_ID`)      |
|    UUID（过时）     |           32位UUID字符串(please use `ASSIGN_UUID`)           |
| ID_WORKER_STR(过时) |     分布式全局唯一ID 字符串类型(please use `ASSIGN_ID`)      |

##### @TableField

-   描述：字段注解(非主键)

|       属性       |             类型             | 必须指定 |          默认值          |                             描述                             |
| :--------------: | :--------------------------: | :------: | :----------------------: | :----------------------------------------------------------: |
|      value       |            String            |    否    |            ""            |                         数据库字段名                         |
|        el        |            String            |    否    |            ""            | 映射为原生 `#{ ... }` 逻辑,相当于写在 xml 里的 `#{ ... }` 部分 |
|      exist       |           boolean            |    否    |           true           |                      是否为数据库表字段                      |
|    condition     |            String            |    否    |            ""            | 字段 `where` 实体查询比较条件,有值设置则按设置的值为准,没有则为默认全局的 `%s=#{%s}`,[参考](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/SqlCondition.java) |
|      update      |            String            |    否    |            ""            | 字段 `update set` 部分注入, 例如：update="%s+1"：表示更新时会set version=version+1(该属性优先级高于 `el` 属性) |
|  insertStrategy  |             Enum             |    N     |         DEFAULT          | 举例：NOT_NULL: `insert into table_a(<if test="columnProperty != null">column</if>) values (<if test="columnProperty != null">#{columnProperty}</if>)` |
|  updateStrategy  |             Enum             |    N     |         DEFAULT          | 举例：IGNORED: `update table_a set column=#{columnProperty}` |
|  whereStrategy   |             Enum             |    N     |         DEFAULT          | 举例：NOT_EMPTY: `where <if test="columnProperty != null and columnProperty!=''">column=#{columnProperty}</if>` |
|       fill       |             Enum             |    否    |    FieldFill.DEFAULT     |                       字段自动填充策略                       |
|      select      |           boolean            |    否    |           true           |                     是否进行 select 查询                     |
| keepGlobalFormat |           boolean            |    否    |          false           |              是否保持使用全局的 format 进行处理              |
|     jdbcType     |           JdbcType           |    否    |    JdbcType.UNDEFINED    |           JDBC类型 (该默认值不代表会按照该值生效)            |
|   typeHandler    | Class<? extends TypeHandler> |    否    | UnknownTypeHandler.class |          类型处理器 (该默认值不代表会按照该值生效)           |
|   numericScale   |            String            |    否    |            ""            |                    指定小数点后保留的位数                    |

关于`jdbcType`和`typeHandler`以及`numericScale`的说明:

`numericScale`只生效于 update 的sql. `jdbcType`和`typeHandler`如果不配合`@TableName#autoResultMap = true`一起使用,也只生效于 update 的sql. 对于`typeHandler`如果你的字段类型和set进去的类型为`equals`关系,则只需要让你的`typeHandler`让Mybatis加载到即可,不需要使用注解

**FieldStrategy**

|    值     |                           描述                            |
| :-------: | :-------------------------------------------------------: |
|  IGNORED  |                         忽略判断                          |
| NOT_NULL  |                        非NULL判断                         |
| NOT_EMPTY | 非空判断(只对字符串类型字段,其他类型字段依然为非NULL判断) |
|  DEFAULT  |                       追随全局配置                        |

**FieldFill**

|      值       |         描述         |
| :-----------: | :------------------: |
|    DEFAULT    |      默认不处理      |
|    INSERT     |    插入时填充字段    |
|    UPDATE     |    更新时填充字段    |
| INSERT_UPDATE | 插入和更新时填充字段 |

##### @Version

-   描述：乐观锁注解、标记 `@Verison` 在字段上

##### @EnumValue

-   描述：通枚举类注解(注解在枚举字段上)

##### @TableLogic

-   描述：表字段逻辑处理注解（逻辑删除）

|  属性  |  类型  | 必须指定 | 默认值 |     描述     |
| :----: | :----: | :------: | :----: | :----------: |
| value  | String |    否    |   ""   | 逻辑未删除值 |
| delval | String |    否    |   ""   |  逻辑删除值  |

##### @SqlParser

-   描述：租户注解,支持method上以及mapper接口上

|  属性  |  类型   | 必须指定 | 默认值 |                             描述                             |
| :----: | :-----: | :------: | :----: | :----------------------------------------------------------: |
| filter | boolean |    否    | false  | true: 表示过滤SQL解析，即不会进入ISqlParser解析链，否则会进解析链并追加例如tenant_id等条件 |

##### @KeySequence

-   属性：value、resultMap

| 属性  |  类型  | 必须指定 |   默认值   |                             描述                             |
| :---: | :----: | :------: | :--------: | :----------------------------------------------------------: |
| value | String |    否    |     ""     |                            序列名                            |
| clazz | Class  |    否    | Long.class | id的类型, 可以指定String.class，这样返回的Sequence值是字符串"1" |

### 代码生成器

自动生成代码

依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.0</version>
</dependency>

<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.2</version>
</dependency>
```

[代码生成器的一些配置](https://mp.baomidou.com/config/generator-config.html#基本配置)

```java
public class CodeGenerator {

    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        // 获取当前项目路径
        String projectPath = System.getProperty("user.dir");
        // 设置生成文件的输出路径
        gc.setOutputDir(projectPath + "/src/main/java");
        // 作者
        gc.setAuthor("jobob");
        // 重新生成文件时是否覆盖原文件
        gc.setFileOverride(false)；
        // 自动打开
        gc.setOpen(false);
        // mapper的名称模式%sEntity，同理mapper，service，xml文件，control等
        // 例如User类生成UserMapper，可以更改为%sDao,生成UserDao
        gc.mapperName("%sMapper")
        // gc.setSwagger2(true); 实体属性 Swagger2 注解
        // 主键类型
        gc.idType("IDTYPE.UUID")
        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc,setDbType(DbType.MYSQL)
        dsc.setUrl("jdbc:mysql://localhost:3306/ant?useUnicode=true&useSSL=false&characterEncoding=utf8");
        // dsc.setSchemaName("public");
        dsc.setDriverName("com.mysql.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("密码");
        mpg.setDataSource(dsc);

        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setParent("com.origami");
        pc.setModuleName(scanner("mall"));
        
        pc.setController("controller");
        pc.setEntity("pojo");
        pc.setService("service");
        pc.setMapper("mapper");       
        mpg.setPackageInfo(pc);
        

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        // 下划线转驼峰
        strategy.setNaming(NamingStrategy.underline_to_camel);
        // 列名下划线转驼峰
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        strategy.setSuperEntityClass("你自己的父类实体,没有就不用设置!");
        strategy.setEntityLombokModel(true);
        // rest风格，@RestController
        strategy.setRestControllerStyle(true);
        // 公共父类
        strategy.setSuperControllerClass("你自己的父类控制器,没有就不用设置!");
        // 写于父类中的公共字段
        strategy.setSuperEntityColumns("id");
        // 哪些表需要逆向生成
        strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
        strategy.setControllerMappingHyphenStyle(true);
        // 生成实体时去除前缀
        strategy.setTablePrefix(pc.getModuleName() + "_");
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }

}
```

### 主键策略

mp支持不同的主键策略。[详细说明](https://www.cnblogs.com/haoxinyue/p/5208136.html)

> 主键自增长

AUTO INCREMENT，策略简单，但是分库分表操作困难

> UUID

32位随机字符串，简单，但是占用空间且无法排序

> redis生成id

> 雪花算法

使用不同的主键策略，在实体类的主键上加上注解

```java
@TableId(type=IdType.XXX)
```

AUTO：主键自增

INPUT：自己添加主键

NONE：无主键

ID_WORKER：mp自带策略，Long类型的19位

ID_WORKER_STR：mp自带，19位字符串类型

UUID：随机唯一值

### 自动填充

一般表中都会有添加时间`create_time`和修改时间`update_time`，自动填充添加时间，更改时间。

在实体类的添加时间和更改时间上添加注解

```java
// 添加时填充
@TableField(.. fill = FieldFill.INSERT)
private String createTime;

// 第一次添加或是修改时填充
@TableField(.. fill = FieldFill.INSERT_UPDATE)
private String updateTime;
```

编写自定义填充类，实现MetaObjectHandler方法

```java
@Slf4j
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    
	// 创建时会将create和update都填充
    @Override
    public void insertFill(MetaObject metaObject) {
        log.info("start insert fill ....");
        this.strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now()); 
        this.strictInsertFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now()); 
       
    }

    // 修改时之填充update
    @Override
    public void updateFill(MetaObject metaObject) {
        log.info("start update fill ....");
        this.strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now()); 
       
    }
}
```

### 乐观锁

**丢失修改问题：**事务A访问该数据，事务B也访问该数据，事务A修改了该数据，事务B也修改了该数据，这样导致事务A的修改被丢失，因此称为丢失修改。

对于此问题有两种解决方案：**悲观锁**和**乐观锁**。

悲观锁：认为每次都会发生这种事，当一个线程访问数据时加锁，另一个线程不允许访问此数据，效率低。

乐观锁：假设每次都不会有这种情况发生，修改前先看是否

设置版本号，线程A、B读取数据时会读取到当前版本号n，事务A修改数据后会将版本号n+1，此时线程B进行修改会发现版本号与自己读取到的不同，此时不能进行修改。

例如抢票过程，谁先付款其他人会付款失败。

乐观锁实现方式：

* 取出记录时，获取当前version
* 更新时，带上这个version
* 执行更新时， set version = newVersion where version = oldVersion
* 如果version不对，就更新失败

配置乐观锁插件

```java
@Bean
public OptimisticLockerInterceptor optimisticLockerInterceptor() {
    return new OptimisticLockerInterceptor();
}
```

实体类添加version字段和`@Version`注解

```java
@Version
private Integer version;
```

### 分页

配置分页插件

```java
@Bean
public PaginationInterceptor paginationInterceptor() {
    PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
    // 设置请求的页面大于最大页后操作， true调回到首页，false 继续请求  默认false
    // paginationInterceptor.setOverflow(false);
    // 设置最大单页限制数量，默认 500 条，-1 不受限制
    // paginationInterceptor.setLimit(500);
    // 开启 count 的 join 优化,只针对部分 left join
    paginationInterceptor.setCountSqlParser(new JsqlParserCountOptimize(true));
    return paginationInterceptor;
}
```

```java
public void selectPage() {
    QueryWrapper<User> wrapper = new QueryWrapper<>();
    wrapper.ge("age",26);
    Page<User> page = new Page<>(1, 5);	// 当前页，每页数据
    IPage<User> userIPage = userMapper.selectPage(page, wrapper);
    System.out.println("总条数"+userIPage.getTotal());
    System.out.println("总页数"+userIPage.getPages());

}

```

```java
public interface UserMapper {//可以继承或者不继承BaseMapper
    /**
     *
     * @param page 分页对象,xml中可以从里面进行取值,传递参数 Page 即自动分页,必须放在第一位(你可以继承Page实现自己的分页对象)
     * @param state 状态
     * @return 分页对象
     */
    IPage<User> selectPageVo(Page<?> page, Integer state);
}

public IPage<User> selectUserPage(Page<User> page, Integer state) {
    // 不进行 count sql 优化，解决 MP 无法自动优化 SQL 问题，这时候你需要自己查询 count 部分
    // page.setOptimizeCountSql(false);
    // 当 total 为小于 0 或者设置 setSearchCount(false) 分页插件不会进行 count 查询
    // 要点!! 分页返回的对象与传入的对象是同一个
    return userMapper.selectPageVo(page, state);
}
```

### 逻辑删除

添加逻辑删除字段

使用@TableLogic注解添加到`deleted`字段上

```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: flag  #全局逻辑删除字段值 3.3.0开始支持，详情看下面。
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
```

### CRUD接口

##### mapper层crud

mapper层实现`BaseMapper`接口

``` java
// Insert------------------------------------------------

// 插入一条记录
int insert(T entity);


// delete------------------------------------------------

// 根据 entity 条件，删除记录
int delete(@Param(Constants.WRAPPER) Wrapper<T> wrapper);
// 删除（根据ID 批量删除）
int deleteBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
// 根据 ID 删除
int deleteById(Serializable id);
// 根据 columnMap 条件，删除记录
int deleteByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);


// update-----------------------------------------------

// 根据 whereEntity 条件，更新记录
int update(@Param(Constants.ENTITY) T entity, @Param(Constants.WRAPPER) Wrapper<T> updateWrapper);
// 根据 ID 修改
int updateById(@Param(Constants.ENTITY) T entity);


// select------------------------------------------------

// 根据 ID 查询
T selectById(Serializable id);
// 根据 entity 条件，查询一条记录
T selectOne(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

// 查询（根据ID 批量查询）
List<T> selectBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
// 根据 entity 条件，查询全部记录
List<T> selectList(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 查询（根据 columnMap 条件）
List<T> selectByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
// 根据 Wrapper 条件，查询全部记录
List<Map<String, Object>> selectMaps(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询全部记录。注意： 只返回第一个字段的值
List<Object> selectObjs(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

// 根据 entity 条件，查询全部记录（并翻页）
IPage<T> selectPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询全部记录（并翻页）
IPage<Map<String, Object>> selectMapsPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询总记录数
Integer selectCount(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

##### service层crud

```java
// save-----------------------------------------------

// 插入一条记录（选择字段，策略插入）
boolean save(T entity);
// 插入（批量）
boolean saveBatch(Collection<T> entityList);
// 插入（批量）
boolean saveBatch(Collection<T> entityList, int batchSize);

// SaveOrUpdate------------------------------------------

// TableId 注解存在更新记录，否插入一条记录
boolean saveOrUpdate(T entity);
// 根据updateWrapper尝试更新，否继续执行saveOrUpdate(T)方法
boolean saveOrUpdate(T entity, Wrapper<T> updateWrapper);
// 批量修改插入
boolean saveOrUpdateBatch(Collection<T> entityList);
// 批量修改插入
boolean saveOrUpdateBatch(Collection<T> entityList, int batchSize);


// Remove--------------------------------------------------

// 根据 entity 条件，删除记录
boolean remove(Wrapper<T> queryWrapper);
// 根据 ID 删除
boolean removeById(Serializable id);
// 根据 columnMap 条件，删除记录
boolean removeByMap(Map<String, Object> columnMap);
// 删除（根据ID 批量删除）
boolean removeByIds(Collection<? extends Serializable> idList);


// Update-----------------------------------------------------

// 根据 UpdateWrapper 条件，更新记录 需要设置sqlset
boolean update(Wrapper<T> updateWrapper);
// 根据 whereEntity 条件，更新记录
boolean update(T entity, Wrapper<T> updateWrapper);
// 根据 ID 选择修改
boolean updateById(T entity);
// 根据ID 批量更新
boolean updateBatchById(Collection<T> entityList);
// 根据ID 批量更新
boolean updateBatchById(Collection<T> entityList, int batchSize);


// get-----------------------------------------------------------

// 根据 ID 查询
T getById(Serializable id);
// 根据 Wrapper，查询一条记录。结果集，如果是多个会抛出异常，随机取一条加上限制条件 wrapper.last("LIMIT 1")
T getOne(Wrapper<T> queryWrapper);
// 根据 Wrapper，查询一条记录
T getOne(Wrapper<T> queryWrapper, boolean throwEx);
// 根据 Wrapper，查询一条记录
Map<String, Object> getMap(Wrapper<T> queryWrapper);
// 根据 Wrapper，查询一条记录
<V> V getObj(Wrapper<T> queryWrapper, Function<? super Object, V> mapper);


// list---------------------------------------------------------

// 查询所有
List<T> list();
// 查询列表
List<T> list(Wrapper<T> queryWrapper);
// 查询（根据ID 批量查询）
Collection<T> listByIds(Collection<? extends Serializable> idList);
// 查询（根据 columnMap 条件）
Collection<T> listByMap(Map<String, Object> columnMap);
// 查询所有列表
List<Map<String, Object>> listMaps();
// 查询列表
List<Map<String, Object>> listMaps(Wrapper<T> queryWrapper);
// 查询全部记录
List<Object> listObjs();
// 查询全部记录
<V> List<V> listObjs(Function<? super Object, V> mapper);
// 根据 Wrapper 条件，查询全部记录
List<Object> listObjs(Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询全部记录
<V> List<V> listObjs(Wrapper<T> queryWrapper, Function<? super Object, V> mapper);


// page---------------------------------------------------

// 无条件分页查询
IPage<T> page(IPage<T> page);
// 条件分页查询
IPage<T> page(IPage<T> page, Wrapper<T> queryWrapper);
// 无条件分页查询
IPage<Map<String, Object>> pageMaps(IPage<T> page);
// 条件分页查询
IPage<Map<String, Object>> pageMaps(IPage<T> page, Wrapper<T> queryWrapper);


// count------------------------------------------------

// 查询总记录数
int count();
// 根据 Wrapper 条件，查询总记录数
int count(Wrapper<T> queryWrapper);
```

### 条件查询

##### AbstractWrapper

QueryWrapper(LambdaQueryWrapper) 和 UpdateWrapper(LambdaUpdateWrapper) 的父类
用于生成 sql 的 where 条件

常用条件

`allEq eq ne gt ge It le between notBetween like notLike likeLeft likeRight isNull isNotNull in notln insql notlnSql groupBy orderByAsc orderByDesc orderBy having func or and nested apply last exists notExists`

```java
QueryWrapper wrapper = new QueryWrapper();
wrapper.ge("age",30);
List<User> users = userMapper.selectList(wrapper);
```

between，notNetween：...与...之间

like：模糊查询

orderByDesc：排序

last()：在sql语句最后添加指定的sql语句

select("id","name")：查询指定的列的数据

# 通用mappker

与mybatis-plus类似，继承**Mapper**接口

>   注意：pojo内的属性要使用包装类，而不要使用基本类型，否则会出错
>
>   pojo的主键上要使用@Id注解，否则使用selectByPrimaryKey会有问题。

条件查询

```java
Example example = new Example(Goods.class);
Example.Criteria criteria = example.createCriteria();
userMapper.selectByExample(example);
```

分页

使用pageHelper