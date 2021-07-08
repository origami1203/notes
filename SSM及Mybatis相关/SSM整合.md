# SSM

### 环境准备

##### 1. 创建maven的web项目

##### 2. 导入常用依赖

```xml
<properties>
    <spring.version>5.2.0.RELEASE</spring.version>
    <mybatis.version>3.4.6</mybatis.version>
    <mysql.version>5.1.38</mysql.version>
    <log4j.version>1.2.12</log4j.version>
  </properties>

  <dependencies>
    <!--mybatis-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>${mybatis.version}</version>
    </dependency>
    <!--mybatis整合到spring-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.2</version>
    </dependency>
    <!--数据库连接池-->
    <dependency>
      <groupId>com.mchange</groupId>
      <artifactId>c3p0</artifactId>
      <version>0.9.5.2</version>
    </dependency>
    <!--jackson-->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.11.0.rc1</version>
    </dependency><!--SpringMVC,会把其他依赖自动导入-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <!--spring整合junit-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <!--spring事务-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <!--spring的jdbc-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <!--aop的织入包-->
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.9.2</version>
    </dependency>
    <!--mysql驱动-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>${mysql.version}</version>
    </dependency>
    <!--log4j-->
    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>${log4j.version}</version>
    </dependency>
    <!--单元测试-->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>
    <!--提供编译期依赖的servlet依赖-->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>jsp-api</artifactId>
      <version>2.1</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
      <scope>provided</scope>
    </dependency>
      
      <!--若出现静态资源过滤问题，可以在此处设置-->

  </dependencies>
```

##### 3. 创建数据库

##### 4. 创建包结构

* java下(根据实际情况)
  * pojo
  * dao(mapper)
  * controller
  * service

* resources下

  * SqlSessionConfig.xml(mybatis核心配置文件)

    ```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <!--
    		数据库连接池、事务及mapper映射都交由spring配置，这里不用配置
    		这里只需进行一些设置、别名即可
    	-->
    </configuration>
    ```

  * applicationContext.xml(spring核心配置文件)

    ```xml
    <?xml version="1.e"encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:context="http://www.springframework.org/schema/context"
           xmlns:aop="http://www.springframework.org/schema/aop"
           xmlns:tx="http://www.springframework.org/schema/tx"
      	xsi:schemaLocation="http://www.springframework.org/schema/beans 			http://www.springframework.org/schema/beans/spring-beans.xsd                   												http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
    
    
    
    </beans>
    ```

    

* db.properties(数据库配置文件)

  ```properties
  driver=com.mysql.jdbc.Driver
  # mysql数据库8.0+需要添加时区参数
  url=jdbc:mysql://localhost:3306/mybatis?useSSL=false
  username=root
  password=root
  ```

根据需要

* log4j.properties

根据数据库，编写实体类及实体类Mapper，编写Mapper.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.origami.mapper.BookMapper">
	<!--sql语句-->
</mapper>
```

### spring整合mybatis

##### spring整合dao

[mybatis-spring官方文档](http://mybatis.org/spring/zh/transactions.html)

整合后mybatis不需要再去设置数据库连接池，事务管理器等，这些全部交给spring管理。

将SqlSessionFactory注册到spring的ioc容器中，同时将配置文件交由SqlSessionFactory读取配置。(其实也可以直接将mybatis的全部设置都通过SqlSessionFactory的属性设置，此时mybatis的配置文件可以直接删去)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!--spring整合mybatis-->

    <!--读取配置文件-->
    <context:property-placeholder location="classpath:db.properties"/>

    <!--配置数据库连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
        <property name="driverClassName" value="${driver}"/>
        <property name="url" value="${url}"/>
        <!-- 此处properties
        <property name="username" value="${user}"/>
        <property name="password" value="${password}"/>

        <!--初始化连接数-->
        <property name="initialSize" value="${initialSize}"/>
        <!--最小连接数-->
        <property name="minIdle" value="${minIdle}"/>
        <!--最大连接数-->
        <property name="maxActive" value="${maxActive}"/>
        <!--最长等待时间-->
        <property name="maxWait" value="${maxWait}"/>
    </bean>

    <!--注册sqlSessionFactory-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!--读取Mybatis的配置文件路径-->
        <property name="configLocation" value="classpath:SqlSessionConfig.xml"/>
        <!--读取mapper映射-->
        <property name="mapperLocations" value="classpath:com/origami/mapper/*.xml"/>
    </bean>
    
    
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!--用于指定生成动态代理的mapper的包的位置-->
    <property name="basePackage" value="com.origami.mapper"/>
    
    <!-- 不要使用sqlSessionFactory，因为MapperScannerConfigurer 在启动过程中比 PropertyPlaceholderConfigurer 运行得更早，经常会产生错误。基于这个原因，上述的属性已被废弃，现在建议使用 sqlSessionFactoryBeanName属性。此时使用value属性，而不是ref
	-->
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
</bean>

    

</beans>
```

将datasource和sqlsessionfactory创建后有三种方式实现整合：

1. 在spring容器中注册`sqlsessionTemplate`，它是sqlsession的实现类，可以作为sqlsession使用，编写mapper的实现类，在mapper的实现类中注入sqlsession(即上面的sqlsessiontemplate)，然后使用sqlsession对象执行sql语句。

```xml
<!--注册sqlSession对象-->
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
    <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"/>
</bean>

<!--给实现类注入sqlsession对象-->
<bean id="bookMapper" class="com.origami.mapper.impl.BookMapperImpl">
    <property name="sqlSession" ref="sqlSession"/>
</bean>
```

MapperImpl

```java
public class BookMapperImpl implements BookMapper {

    private SqlSession sqlSession;

    public void setSqlSession(SqlSession sqlSession) {
        this.sqlSession = sqlSession;
    }

    @Override
    public int deleteBookByBid(Integer bid) {
        return getSqlSession().delete("com.origami.mapper.BookMapper.deleteBookByBid", bid);
    }

```

2. 创建MapperIml，并让其继承`SqlSessionDaoSupport`类，此类有一个`getSession()`方法，所以不需要上面的`SqlSessionTemplate`对象，`SqlSessionDaoSupport`需要一个工厂，给MapperImpl注入上面的工厂即可。

```xml
    <bean id="bookMapper" class="com.origami.mapper.impl.BookMapperImpl">
        <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
    </bean>

```

MapperImpl

```java
public class BookMapperImpl extends SqlSessionDaoSupport implements BookMapper {

    @Override
    public int deleteBookByBid(Integer bid) {
        return getSqlSession().delete("com.origami.mapper.BookMapper.deleteBookByBid", bid);
    }
```

3. ==Mapper的动态代理==

```xml
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!--用于指定生成动态代理的mapper的包的位置-->
    <property name="basePackage" value="com.origami.mapper"/>
    
    <!-- 不要使用sqlSessionFactory，因为MapperScannerConfigurer 在启动过程中比 PropertyPlaceholderConfigurer 运行得更早，经常会产生错误。基于这个原因，上述的属性已被废弃，现在建议使用 sqlSessionFactoryBeanName属性。此时使用value属性，而不是ref
	-->
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
</bean>
```

这样spring会动态创建Mapper的实现类，从而不用编写MapperImpl。

##### spring整合service

主要是对事务的管理，看spring.md的事务部分即可。

基于xml的事务配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd
                        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 将想要进行事务管理的类注入ioc -->
    <bean id="fooService" class="x.y.service.DefaultFooService"/>

    <!-- 对该类进行事务管理配置 -->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <!-- 事务配置 -->
        <tx:attributes>
            <!-- 所有以get开头的方法之都 -->
            <tx:method name="get*" read-only="true"/>
            <!-- 其他方法可读写 -->
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>

    <!-- 使用AOP进行事务管理 -->
    <aop:config>
        <aop:pointcut id="fooServiceOperation" expression="execution(* x.y.service.FooService.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceOperation"/>
    </aop:config>
    <!--出现bug，类型不符，为com.sun.proxy.proxy$xx时使用这个
	因为默认使用jdk的动态代理，而有些类没有实现接口
	这是使他使用cglib的基于子类的动态代理-->
    <aop:aspectj-autoproxy  proxy-target-class="true"/>

    <!-- 事务管理器 -->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--使用的是dao配置的dataSource-->
        <property name="dataSource" ref="dataSource"/>
    </bean>


</beans>
```

基于注解的事务配置

```xml
<context:component-scan base-package="com.bluesky" />

<tx:annotation-driven transaction-manager="transactionManager" />

<bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
    <property name="configLocation" value="classpath:hibernate.cfg.xml" />
    <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />
</bean>

<!-- 定义事务管理器（声明式的事务） -->
<bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
    <property name="sessionFactory" ref="sessionFactory" />
</bean>
```

　　此时在DAO上需加上@Transactional注解

### SpringMVC

springMVC是spring的一部分，因此springmvc，不用整合就可以完美融入Spring中，只需要配置好SpringMVC即可。

* 配置web.xml

  * 配置前端控制器DispatcherServlet
  * 配置全局编码过滤器CharacterEncodingFilter

* 配置springmvc.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:mvc="http://www.springframework.org/schema/mvc"
         xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          https://www.springframework.org/schema/context/spring-context.xsd
          http://www.springframework.org/schema/mvc
          https://www.springframework.org/schema/mvc/spring-mvc.xsd">
  
  <!--扫播包，以指定控制层@controller-->
      <context:component-scan base-package="com.origami.controller"/>
      <!--让spring MVC不处理静态资源 .css.js.html.mp3.mp4-->
      <mvc:default-servlet-handler/>
    <!--
      开启mvc的注解支持,这样我们可以使用注解，与@EnableWebMVC功能一致
      而不用去配置DefaultAnnotationHandleMapper和AnnotationMethodHandleAdapter
      -->
      <mvc:annotation-driven/>
  
      <!--配置视图解析器-->
      <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
          <!--设置前缀、后缀 注意最后面有个/ -->
          <property name="prefix" value="/WEB-INF/jsp/"/>
          <property name="suffix" value=".jsp"/>
      </bean>
  </beans>
  ```
  
  