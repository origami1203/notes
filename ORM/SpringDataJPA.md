# Spring Data Jpa

JPA即是Java Persistence API，是JDK 5.0注解或XML描述对象－关系表的映射关系，并将运行期的实体对象持久化到数据库中。它只是一套实现ORM理论的接口，没有实现的代码。

@Enumerated(EnumType.STRING):将注解保存为string类型

Spring Data Jpa是根据JPA的实现类。其底层是对hibernate的封装。

| 注解                                   | 解释                                                         |
| -------------------------------------- | :----------------------------------------------------------- |
| ==**@Entity**==                        | 声明类为实体或表。                                           |
| ==**@Table**==                         | 声明表名。                                                   |
| @Basic                                 | 指定非约束明确的各个字段。                                   |
| @Embeddable                            | 表示可被嵌入，与@Embedded配合使用                            |
| @Embedded                              | 指定类或它的值是一个可嵌入的类的实例的实体的属性，被被标注的类需@Embeddable注解。 |
| @AttributeOverrides/@AttributeOverride | 用于内嵌注解上，指定内嵌类字段与数据库字段的对应。           |
| ==**@Id**==                            | 指定的类的属性，用于识别（一个表中的主键）。                 |
| ==**@GeneratedValue**==                | 指定如何标识属性可以被初始化，例如自动、手动、或从序列表中获得的值。 |
| ==**@Transient**==                     | 指定的属性，它是不持久的，即：该值永远不会存储在数据库中。   |
| **==@Column==**                        | 指定持久属性栏属性。                                         |
| @SequenceGenerator                     | 指定在@GeneratedValue注解中指定的属性的值。它创建了一个序列。 |
| @TableGenerator                        | 指定在@GeneratedValue批注指定属性的值发生器。它创造了的值生成的表。 |
| @AccessType                            | 这种类型的注释用于设置访问类型。如果设置@AccessType（FIELD），则可以直接访问变量并且不需要getter和setter，但必须为public。如果设置@AccessType（PROPERTY），通过getter和setter方法访问Entity的变量。 |
| @JoinColumn                            | 指定一个实体组织或实体的集合。这是用在多对一和一对多关联。   |
| @UniqueConstraint                      | 指定的字段和用于主要或辅助表的唯一约束。                     |
| @ColumnResult                          | 参考使用select子句的SQL查询中的列名。                        |
| @ManyToMany                            | 定义了连接表之间的多对多一对多的关系。                       |
| @ManyToOne                             | 定义了连接表之间的多对一的关系。                             |
| @OneToMany                             | 定义了连接表之间存在一个一对多的关系。                       |
| @OneToOne                              | 定义了连接表之间有一个一对一的关系。                         |
| @NamedQueries                          | 指定命名查询的列表。                                         |
| @NamedQuery                            | 指定使用静态名称的查询。                                     |

### SpringBoot

引入依赖

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

配置application.yaml

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mytest
    type: com.alibaba.druid.pool.DruidDataSource
    username: root
    password: root
    driver-class-name: com.mysql.jdbc.Driver 
    
  # 用于自动生成mysql表
  jpa:
    hibernate:
      ddl-auto: update # entity有变化时，更新
    show-sql: true # 日志中显示sql语句
    # 格式化sql
    properties:
      hibernate:
        format_sql: true
        
        #使用的方言
        dialect: 
        hbm2ddl:
          # 表示 自动执行类路径下表、db下的data.sql语句
          auto: create
          import_files: db/data.sql
```

启动类上添加`@EnableJpaRepositories`注解，否则可能会延迟生成表。

或者

```yaml
spring:
  data:
    jpa:
      repositories:
        bootstrap-mode: default
```

### 常用注解

##### ==**@Entity**== 

标注用于实体类声明语句之前，指出该Java 类为实体类，将映射到指定的数据库表。如声明一个实体类 Customer，它将映射到数据库中的 customer 表上。

##### **@Table**

当实体类与其映射的数据库表名不同名时，需要使用 @Table 标注说明，该标注与 @Entity 标注并列使用，置于实体类声明语句之前，可写于单独语句行，也可与声明语句同行。

*   name：数据库中映射表的名称，若表名与实体类名相同，则name可省略。
*   catalog可选，表示catalog名称，默认为Catalog(“”)。
*   schema可选，表示schema名称，默认为schma(“”)。 

| 数据库     | catalog                           | schema                      |
| :--------- | :-------------------------------- | :-------------------------- |
| oracle     | 不支持                            | Oracle User ID              |
| mysql      | 不支持                            | 数据库名                    |
| SQL Server | 数据库名                          | 对象属主名，2005ban开始有变 |
| DB2        | 指定数据库对象时，catalog部分省略 | Catalog属主名               |

如下代码：

```java
// 映射数据库表名JPA_CUSTOMER，默认情况下可以不写，此时表名与持久化类名相同
@Table(name="JPA_CUSTOMERS")
@Entity //表明这是一个持久化类
public class Customer {}
```

#####  ==**@Id**==

@Id 标注用于声明一个实体类的属性映射为数据库的**主键**。该属性通常置于属性声明语句之前，可与声明语句同行，也可写在单独行上。 @Id标注也可置于属性的**getter方法之前**。

##### **@GeneratedValue**

用于标注主键的生成策略，通过 strategy 属性指定。

generator属性可选，当使用自定义主键生成策略时，指定`@GenericGenerator`的名字。

默认情况下，JPA 自动选择一个最适合底层数据库的主键生成策略：SqlServer 对应 identity，MySQL 对应 auto_increment。 在 javax.persistence.GenerationType 中定义了以下几种可供选择的策略：

*   IDENTITY：采用数据库 ID自增长的方式来自增主键字段，Oracle 不支持这种方式；
*   AUTO： JPA自动选择合适的策略，是默认选项；
*   SEQUENCE：通过序列产生主键，通过 **@SequenceGenerator** 注解指定序列名，MySql 不支持这种方式
*   TABLE：通过表产生主键，框架借由表模拟序列产生主键，使用该策略可以使应用更易于数据库移植。

```java
//定义主键生成策略
@GeneratedValue(strategy=GenerationType.AUTO)
@Id //定义主键
@Column(name="ID")//定义数据库的列名如果与字段名一样可以省略
public Integer getId() {
    return id;
}
```

##### **@GenericGenerator**

与上面的`@GeneratedValue`的配合使用，实现自定义主键生成策略。

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY, generator = "custom-id")
@GenericGenerator(name = "custom-id", strategy = "run.halo.app.model.entity.support.CustomIdGenerator")
private Integer id;
```

##### **@Basic(基本不用)**

@Basic 表示一个简单的属性到数据库表的字段的映射,对于没有任何标注的 getXxxx() 方法,默认即为@Basic。

fetch: 表示该属性的读取策略,有 EAGER 和 LAZY 两种,分别表示主支抓取和延迟加载,默认为 EAGER.

optional:表示该属性是否允许为null, 默认为true

##### ==**@Column**==

**当实体的属性与其映射的数据库表的列不同名时，需要使用@Column 标注说明，**该属性通常置于实体的属性声明语句之前，还可与 @Id 标注一起使用。

*   name：常用，表示数据库表中该字段的名称，默认情形属性与数据库名称一致。

*   Nullable：可选，表示该字段是否允许为null,默认为true。 

*   Unique:可选，表示该字段是否是唯一标识，默认为false。 

*   Length:可选，表示该字段的大小，**仅对String类型的字段有效**，默认值255。 

*   Inserable:可选，表示在ORM框架执行插入操作时，该字段是否应出现INSERT语句中，默认为true。 

*   Updateable:可选，表示在ORM框架执行更新操作时，该字段是否应该出现在UPDATE语句中，默认为true。对于一经创建就不可以更改的字段，改属性非常有用，如对于birthday字段。

*   columnDefinition：属性表示创建表时，该字段创建的SQL语句，一般用于通过Entity生成表定义时使用，如果数据库中表已经建好，该属性没有必要使用

    1.  指定字段类型、长度、是否允许null、是否唯一、默认值

    ```java
    @Column(name = "code",columnDefinition = "Varchar(100) not null default'' unique")
    private String code;
    ```

    2、需要特殊指定字段类型的情况

    ```java
    @Column(name = "remark",columnDefinition="text")
    private String remark;
    
    @Column(name = "salary", columnDefinition = "decimal(5,2)")
    private BigDecimal salary;
    
    @Column(name="birthday",columnDefinition="date")
    private Date birthday;
    
    @Column(name="createTime",columnDefinition="datetime")
    private Date createTime;
    ```


##### **@Transient**

如果一个属性==**并非数据库表的字段映射**==，就务必将其标示为@Transient，表示该属性并非一个到数据库表的字段的映射,ORM框架将忽略该属性。

##### **@Temporal**

在核心的 Java API 中并没有定义 Date 类型的精度(temporal precision). 而在数据库中,表示 Date 类型的数据有 DATE, TIME, 和 TIMESTAMP 三种精度(即单纯的日期,时间,或者两者 兼备).

当我们使用到java.util包中的时间日期类型，则需要此注释来说明转化成java.util包中的类型。

注入数据库的类型有三种：

　　　　TemporalType.DATE（2008-08-08）

　　　　TemporalType.TIME（20:00:00）

　　　　TemporalType.TIMESTAMP（2008-08-08 20:00:00.000000001）

##### **@CreatedDate**、**@CreatedBy**、**@LastModifiedDate**、**@LastModifiedBy**

用于create_time和update_time等

##### **@MappedSuperclass**

　　实现将实体类的多个属性分别封装到不同的非实体类中

　　被注解的类将不是完整的实体类，不会映射到数据库表，但其属性将映射到子类的数据库字段，注解的类不能再标注@Entity或@Table注解，也无需实现序列化接口

　　注解的类继承另一个实体类或标注@MappedSuperclass类，他可使用@AttributeOverride 或 @AttributeOverrides注解重定义其父类属性映射到数据库表中字段。

#####  **@DiscriminatorColumn和@DiscriminatorValue**

DiscriminatorColumn

标注于类上，用于多个实体类对应同一个数据库表

```java
@DiscriminatorColumn(name = "type", discriminatorType = DiscriminatorType.INTEGER, columnDefinition = "int default 0")
public class BaseComment{
    ...
}
```

*   name：指定用于区分两个实体类的列名
*   discriminatorType：区分的类型

DiscriminatorValue

标注于继承了上面标注了DiscriminatorColumn的类上，用于区别多个不同的实体类

```java
@Entity(name = "SheetComment")
@DiscriminatorValue("1")
public class SheetComment extends BaseComment {

}

@Entity(name = "PostComment")
@DiscriminatorValue("0")
public class PostComment extends BaseComment {

}
```

### 核心接口

*   Repository接口
*   CrudRepository接口
*   PagingAndSortingRepository接口
*   JpaRepository接口
*   JPASpecificationExecutor接口

##### Repository接口

最顶层的接口，是一个空接口。它提供了方法名称命名查询方式，也提供了基于@Query注解查询与更新。

###### ==**接口规范方法名查询**==

*Repository支持接口规范方法名查询。**意思是如果在接口中定义的查询方法符合它的命名规则，就可以不用写实现，目前支持的关键字如下。***

*[官方文档](https://docs.spring.io/spring-data/jpa/docs/2.4.2/reference/html/#repository-query-keywords)*

query的主语关键字

| Keyword                                                      | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `find…By`, `read…By`, `get…By`, `query…By`, `search…By`, `stream…By` | 常规查询方法，通常返回存储库类型、Collection或Streamable子类型或结果包装器（如Page、GeoResults或任何其他特定于存储的结果包装器）。可以这样用：findBy…、findMyDomainTypeBy…或与其他关键字组合使用。 |
| `exists…By`                                                  | 存在判断，通常返回布尔。                                     |
| `count…By`                                                   | 计算返回数值结果。                                           |
| `delete…By`, `remove…By`                                     | 删除查询方法，返回void或被删除的数目。                       |
| `…First<number>…`, `…Top<number>…`                           | 将查询结果限制为结果的前\<number>个。此关键字可以在find（和其他关键字）和by之间介于主语的任何位置。 |
| `…Distinct…`                                                 | 使用distinct查询仅返回唯一的结果。请参阅store-specific文档，以确定是否支持该功能。此关键字可以出现在find（和其他关键字）和by之间的主题的任何位置。 |

query的谓语关键字

| Logical keyword       | Keyword expressions                            | **Sample**             | **JPQL snippet**                                            |
| :-------------------- | :--------------------------------------------- | ---------------------- | ----------------------------------------------------------- |
| `AND`                 | `And`                                          | findByNameAndAge       | where x.name = ? and x.age = ?                              |
| `OR`                  | `Or`                                           | findByNameOrAge        | where x.name = ? or x.age = ?                               |
| `AFTER`               | `After`, `IsAfter`                             | findByStartDateAfter   | ... where x.startDate > ?1                                  |
| `BEFORE`              | `Before`, `IsBefore`                           | findByStartDateBefore  | ... where x.startDate < ?1                                  |
| `CONTAINING`          | `Containing`, `IsContaining`, `Contains`       | findByNameContaining   | ...  where x.name like ?1(parameter bound wrapped in %)     |
| `BETWEEN`             | `Between`, `IsBetween`                         | findBtAgeBetween       | ...  where x.age between ?1 and ?2                          |
| `ENDING_WITH`         | `EndingWith`, `IsEndingWith`, `EndsWith`       | findByNameEndingWith   | ...  where x.name like ?1(parameter bound with prepended %) |
| `EXISTS`              | `Exists`                                       |                        |                                                             |
| `FALSE`               | `False`, `IsFalse`                             | findByActiveFalse      | ...  where x.active = false                                 |
| `GREATER_THAN`        | `GreaterThan`, `IsGreaterThan`                 | findByAgeGreaterThan   | ...  where x.age > ?1                                       |
| `GREATER_THAN_EQUALS` | `GreaterThanEqual`, `IsGreaterThanEqual`       |                        |                                                             |
| `IN`                  | `In`, `IsIn`                                   | findByAgeIn            | ...  where x.age in ?1                                      |
| `IS`                  | `Is`, `Equals`, (or no keyword)                |                        |                                                             |
| `IS_EMPTY`            | `IsEmpty`, `Empty`                             |                        |                                                             |
| `IS_NOT_EMPTY`        | `IsNotEmpty`, `NotEmpty`                       |                        |                                                             |
| `IS_NOT_NULL`         | `NotNull`, `IsNotNull`                         | findByAgeNotNull       | ...  where x.age not null                                   |
| `IS_NULL`             | `Null`, `IsNull`                               | findByAgeIsNull        | ...  where x.age is null                                    |
| `LESS_THAN`           | `LessThan`, `IsLessThan`                       | findByAgeLessThan      | ...  where x.age  <  ?1                                     |
| `LESS_THAN_EQUAL`     | `LessThanEqual`, `IsLessThanEqual`             |                        |                                                             |
| `LIKE`                | `Like`, `IsLike`                               | findByNameLike         | ...  where x.name like ?1                                   |
| `NEAR`                | `Near`, `IsNear`                               |                        |                                                             |
| `NOT`                 | `Not`, `IsNot`                                 | findByNameNot          | ...  where x.name <> ?1                                     |
| `NOT_IN`              | `NotIn`, `IsNotIn`                             | findByAgeNotIn         | ...  where x.age not in ?                                   |
| `NOT_LIKE`            | `NotLike`, `IsNotLike`                         | findByNameNotLike      | ...  where x.name not like ?1                               |
| `REGEX`               | `Regex`, `MatchesRegex`, `Matches`             |                        |                                                             |
| `STARTING_WITH`       | `StartingWith`, `IsStartingWith`, `StartsWith` | findByNameStartingWith | ...  where x.name like ?1(parameter bound with appended %)  |
| `TRUE`                | `True`, `IsTrue`                               | findByActiveTrue       | ...  where x.avtive = true                                  |
| `WITHIN`              | `Within`, `IsWithin`                           |                        |                                                             |

谓语修饰关键字

| Keyword                            | Description                                                  |
| :--------------------------------- | :----------------------------------------------------------- |
| `IgnoreCase`, `IgnoringCase`       | 和谓语一块，忽略大小写                                       |
| `AllIgnoreCase`, `AllIgnoringCase` | Ignore case for all suitable properties. Used somewhere in the query method predicate. |
| `OrderBy…`                         | 排序，OrderByFirstnameAscLastnameDesc                        |

==**查找方式**==

```java
List<Person> findByAddressZipCode(ZipCode zipCode);
```

假设`Person`类的`Address`带有邮政编码`ZipCode`。在这种情况下，该方法将创建x.address.zipCode属性遍历。

解析算法首先去除**`findBy`**,然后将剩余部分**`AddressZipCode`**解释为属性，然后在类中检查是否具有该名称的属性（未大写）。如果解析成功，它将使用该属性。

如果不是，算法会从右侧开始，将驼峰部分的源代码拆分为head和tail，并尝试找到相应的属性。在我们的示例中拆分为为**`AddressZip`**和**`Code`**。如果该算法找到了具有该头部的属性**`AddressZip`**，则取出尾部然后从那里开始构建树，按照刚才描述的方式将尾部向上拆分。如果第一个拆分不匹配，则算法将拆分点移到左侧（`Address`，`ZipCode`）并继续。

尽管这在大多数情况下都是可行的，但算法可能会选择错误的属性。假设Person类也有一个addressZip属性。该算法已经在第一轮分割中匹配，选择了错误的属性，然后失败(因为addressZip的类型可能没有代码属性)。
要解决这种歧义，可以在方法名中使用_来手动定义遍历点。所以我们的方法名如下:

```java
List<Person> findByAddress_ZipCode(ZipCode zipCode);
```

因为我们将下划线字符视为保留字符，所以我们强烈建议遵循标准的Java命名约定(即在属性名称中不使用下划线，而是使用驼峰大小写)。

此外，根据方法的参数，还可动态的查询和排序。

```java
Page<User> findByLastname(String lastname, Pageable pageable);

List<User> findByLastname(String lastname, Sort sort);

List<User> findByLastname(String lastname, Pageable pageable);
```

###### @Query

*@Query*注解用来写查询*sql*语句，支持原生sql语句，也支持JPQL 语句。

当更新，或删除操作时，需要添加`@Modifying`，与*@Query*一块使用。

##### CrudRepository接口

继承**Repository**接口，实现了基本的CRUD功能。

```java
// T为查询的实体类型，ID为实体类型的主键类型
CrudRepository<T,Id>;

public interface UsersRepositoryCrudRepository extends CrudRepository<User,Integer> {
}
```

##### PagingAndSortingRepository接口

继承了**CrudRepository**接口，提供了分页与排序的功能。

```java
PagingAndSortingRepository<T,ID>
```

```java
public void testPagingAndSortingRepositorySort() {
    //Order	定义了排序规则
    Sort.Order order=new Sort.Order(Sort.Direction.DESC,"id");
    //Sort对象封装了排序规则
    Sort sort=new Sort(order);
    List<Users> list= (List<Users>) this.usersRepositoryPagingAndSorting
        								.findAll(sort);
    for (Users users:list){
        System.out.println(users);
    }
}

public void testPagingAndSortingRepositoryPaging() {
    //Pageable:封装了分页的参数，当前页，煤业显示的条数。注意：它的当前页是从0开始
    //PageRequest(page,size):page表示当前页，size表示每页显示多少条
    Pageable pageable=new PageRequest(1,2);
    Page<Users> page=this.usersRepositoryPagingAndSorting.findAll(pageable);
    System.out.println("数据的总条数："+page.getTotalElements());
    System.out.println("总页数："+page.getTotalPages());
    List<Users> list=page.getContent();
    for (Users users:list){
        System.out.println(users);
    }
}

public void testPagingAndSortingRepositorySortAndPaging() {
    Sort sort=new Sort(new Sort.Order(Sort.Direction.DESC,"id"));
    Pageable pageable=new PageRequest(0,2,sort);
    Page<Users> page=this.usersRepositoryPagingAndSorting.findAll(pageable);
    System.out.println("数据的总条数："+page.getTotalElements());
    System.out.println("总页数："+page.getTotalPages());
    List<Users> list=page.getContent();
    for (Users users:list){
        System.out.println(users);
    }
}
```

##### JpaRepository接口

继承了PagingAndSortingRepository。对继承的父接口中方法的返回值进行适配。

##### JPASpecificationExecutor接口

JPASpecificationExecutor是独立的，主要是提供了多条件查询的支持，批量功能，并且可以在查询中添加排序与分页。

```java
public void testJpaSpecificationExecutor1() {
		/**
		 * Specification:用于封装查查询条件
		 */
		Specification<Users> spec=new Specification<Users>() {
			//Predicate：封装了单个查询条件

			/**
			 * @param root		对查询对象属性的封装，这里即是Users类
			 * @param criteriaQuery	封装了我们要执行的查询中的各个部分的信息，select from order
			 * @param criteriaBuilder	查询条件的构造器
			 * @return
			 */
			@Override
			public Predicate toPredicate(Root<Users> root, 
                                         CriteriaQuery<?> criteriaQuery, 
                                         CriteriaBuilder criteriaBuilder) {
				//where name="张三"
				/**
				 * 参数一：查询的属性
				 * 参数二：条件的值
				 */
				Predicate predicate=criteriaBuilder.equal(root.get("name"),"张三");
				return predicate;
			}
		};
		List<Users> list=this.userRepositorySpecification.findAll(spec);
		for (Users users:list){
			System.out.println(users);
		}
	}

	/**
	 * JpaSpecificationExecutor		多条件查询方式一
	 */
	@Test
	public void testJpaSpecificationExecutor2() {
		/**
		 * Specification:用于封装查查询条件
		 */
		Specification<Users> spec=new Specification<Users>() {
			//Predicate：封装了单个查询条件

			/**
			 * @param root		对查询对象属性的封装
			 * @param criteriaQuery	封装了我们要执行的查询中的各个部分的信息，select from order
			 * @param criteriaBuilder	查询条件的构造器
			 * @return
			 */
			@Override
			public Predicate toPredicate(Root<Users> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
				//where name="张三" and age=20
				/**
				 * 参数一：查询的属性
				 * 参数二：条件的值
				 */
				List<Predicate> list=new ArrayList<>();
				list.add(criteriaBuilder.equal(root.get("name"),"张三"));
				list.add(criteriaBuilder.equal(root.get("age"),20));
				Predicate[] arr=new Predicate[list.size()];
				return criteriaBuilder.and(list.toArray(arr));
			}
		};
		List<Users> list=this.userRepositorySpecification.findAll(spec);
		for (Users users:list){
			System.out.println(users);
		}
	}

	/**
	 * JpaSpecificationExecutor		多条件查询方式二
	 */
	@Test
	public void testJpaSpecificationExecutor3() {
		/**
		 * Specification:用于封装查查询条件
		 */
		Specification<Users> spec=new Specification<Users>() {
			//Predicate：封装了单个查询条件

			/**
			 * @param root		对查询对象属性的封装
			 * @param criteriaQuery	封装了我们要执行的查询中的各个部分的信息，select from order
			 * @param criteriaBuilder	查询条件的构造器
			 * @return
			 */
			@Override
			public Predicate toPredicate(Root<Users> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
				//where name="张三" and age=20
				/**
				 * 参数一：查询的属性
				 * 参数二：条件的值
				 */
				/*List<Predicate> list=new ArrayList<>();
				list.add(criteriaBuilder.equal(root.get("name"),"张三"));
				list.add(criteriaBuilder.equal(root.get("age"),20));
				Predicate[] arr=new Predicate[list.size()];*/
				//(name='张三' and age=20) or id=2
				return criteriaBuilder
                    .or(criteriaBuilder
                        .and(criteriaBuilder.equal(root.get("name"),"张三"),
                         	 criteriaBuilder.equal(root.get("age"),20)),
                        	 criteriaBuilder.equal(root.get("id"),1));
			}
		};

		Sort sort=new Sort(new Sort.Order(Sort.Direction.DESC,"id"));
		List<Users> list=this.userRepositorySpecification.findAll(spec,sort);
		for (Users users:list){
			System.out.println(users);
		}
	}
```

### Repository

dao层，即mybatis中的mapper。

```java
// 开启Repository的扫描
@EnableJpaRepositories("com.acme.repositories")
```

##### 自定义实现

当查询方法需要不同的行为或不能通过查询派生实现时，您需要提供自定义实现。Spring数据存储库允许自定义存储库代码，并将其与通用的CRUD抽象和查询方法功能集成。

```java
// 自定义repo
interface CustomizedUserRepository {
  void someCustomMethod(User user);
}

// 定义实现类
class CustomizedUserRepositoryImpl implements CustomizedUserRepository {
  public void someCustomMethod(User user) {
    // Your custom implementation
  }
}
```

>   ==**实现类的类名是自定义Repository类名+Impl**==，可通过repository-impl-postfix="xxx"更改自定义后缀

实现本身并不依赖于Spring数据，可以是一个普通的Spring bean。因此，您可以使用标准的依赖项注入行为来将引用注入到其他bean(例如JdbcTemplate)、参与方面，等等。

然后repository继承该实现类。

```java
interface UserRepository extends CrudRepository<User, Long>, CustomizedUserRepository{
  // Declare query methods here
}
```

### 审查

```java
@CreatedBy

@LastModifiedBy

@CreatedDate

@LastModifiedDate
```

```java
class Customer {

  @CreatedBy
  private User user;

  @CreatedDate
  private DateTime createdDate;

  // … further properties omitted
}
```

