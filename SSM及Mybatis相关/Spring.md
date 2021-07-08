# Spring

### 高内聚，低耦合

耦合既是程序间的依赖关系，若是程序间存在依赖关系，你想要删除或修改一个类，对其有依赖关系的类就可能会出问题，因此，我们要尽量降低程序间的耦合。以jdbc为例，为什么我们使用Class.forName("com.mysql.jdbc.Driver"),而不是DriverManger.registe(new com.mysql.jdbc.Driver)，因为此时会产生依赖关系，若我们此时删除jdbc的jar包，那么程序会连编译都过不了就会报错，而使用Class.forName方法注册，我们是可以编译通过的，只有实际运行时才会报错。我们设计的原则应该是==**编译期不依赖，运行期才依赖**==。

三层架构中，controller依赖service，service依赖dao，其也有很强的依赖关系。

### 配置文件

* beans.xml（applicationContext.xml)

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
     		http://www.springframework.org/schema/beans/spring-beans.xsd">
  
  	 <!-- id为此对象的唯一标识，class为此对象的全限定类名-->
      <bean id="..." class="...">
          <!-- 详细配置 -->
          ...
      </bean>
  
      <!-- 配置更多的对象 -->
  
  </beans>
  ```

* `<context:component-scan base-package="com.origami.pojo"/>`
  
  * 扫描指定包下的component，使ioc容器接管
* `<context:property-placeholder location="classpath:db.properties"/>`
  
  * 读取指定路径下的配置，以便后面使用SpEl表达式获取(配置连接池时使用)

### IoC容器

`org.springframework.beans`和`org.springframework.context`包是 Spring Framework 的 IoC 容器的基础。 BeanFactory接口提供了一种高级 configuration 机制，能够管理任何类型的 object。 ApplicationContext是`BeanFactory`的 子接口。![image-20200408142456626](C:\Users\28455\AppData\Roaming\Typora\typora-user-images\image-20200408142456626.png)

<div align=center>继承实现图</div>

`BeanFactory`提供 configuration framework 基本功能，`ApplicationContext`添加更多企业应用功能。`ApplicationContext`是`BeanFactory`的完整超集。

`org.springframework.context.ApplicationContext`接口表示 Spring IoC 容器，负责实例化，配置和组装 beans。

Spring 提供了`ApplicationContext`接口的几个实例。当我们使用xml配置文件时，一般使用`ClassPathXmlApplicationContext`类（==常用==） 或 `FileSystemXmlApplicationContext`实例。使用注解开发时，使用`AnnotationConfigApplicationContext`实例。


```java
//实例化ApplicationContext对象，使用ClassPathXmlApplicationContext读取classpath路径下的配置文件
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");

//使用FileSystemXmlApplicationContext读取任意位置的文件
ApplicationContext context = new FileSystemXmlApplicationContext("路径")；
```

通过`ApplicationContext`的`getBean`方法，即可得到`bean`对象。

在默认作用域(singleton,单例模式)下，在`ApplicationContext`对象被创建后，bean对象已经创建，但是只有在实际得到这些bean的时候才会给他的属性赋值。可以通过将 bean 定义 `lazy-initialized`属性 来阻止 singleton bean 的预加载。 lazy-initialized bean 告诉 IoC 容器在第一次请求时创建 bean 实例，而不是在启动时创建 bean 实例。

但是，当懒加载的bean是另一个未懒加载的单例bean的依赖项时，ApplicationContext会在启动时创建懒加载的bean，因为它必须满足单例的依赖关系。

除此之外的作用域，只有第一次请求时时，bean对象才会被创建。

### Bean对象

Spring IoC 容器管理一个或多个 beans。这些 beans 是使用您提供给容器的configuration数据（xml文件`<bean>`或是注解）创建的 。

根据`<bean>`标签的属性构建出相应的`<bean>`对象。

 ##### Bean对象的命名

  * `<id>` `<name>`属性

    * `<id>`为该`<bean>`的唯一标识
    * `<name>`可以为该`<bean>`起别名，可以多个，由(,)开始(;)结束
    * 约定成俗，命名一般使用驼峰式。

  * `<alias/>`标签

    * 用于`<bean>`标签之外起别名

      ```xml
      # 为名为fromName的bean添加一个名为toName的别名
      <alias name="fromName" alias="toName"/>
	  ```

 ##### 实例化Bean对象

  * **使用`class`属性实例化**

    * 指定要实例化的bean对象的全限定类名

    * 若没有指定其他属性，通过反射调用其无参构造函数完成创建

    * 如果在`com.example`包中有一个名为`SomeThing`的类，并且`SomeThing`类有一个`static`嵌套类，名为`OtherThing`，则 `OtherThing`类的class属性为`com.example.SomeThing$OtherThing`

      注意在 name 中使用`$`字符将嵌套的 class name 与外部 class name 分开。

  * **工厂的实例化**

    * 普通工厂的实例化
    
    	```java
    	//若在com.example包下有一个工厂UserFactory用于构建User
    	public class UserFactory {
    	public User getUser() {
    	    return new User;
    	}
    	}
    	```
    
	配置`<bean>`
    	
    	```xml
    <bean id='user' class="com.example.UserFactory"/>
    	```
    	
    	
    	此时，创建的bean并不是`User`，而是`UserFactory`的bean
    	
    	我们想要得到`User`的bean，需要使用`factory-bean`和`factory-mothod`属性
    	
    	```xml
    	# 工厂的bean对象
    	<bean id="userFactory" class="com.example.UserFactory"/>
    	
    	# factory-bean指向工厂的id，factory-mothod指向工厂的方法
    	<bean id="user" factory-bean="userFactory" factory-mothod="getUser"/>
    	```
    
* **静态工厂的实例化**

  ```java
  //若在com.example包下有一个静态工厂UserFactory用于构建User
  public class UserFactory {
    public static User getUser() {
        return new User;
    }
  }
  ```

  我们应该使用`class`和`factory-mothod`属性

  ```xml
  <bean id="user" class="com.example.UserFactory" factory-mothod="getUser"/>
  ```

##### Bean的作用范围

通过`<bean>`标签内的 `scope`属性来对bean的作用范围。

官方文档上没有`global-session`范围。

|      范围      |                             描述                             |
| :------------: | :----------------------------------------------------------: |
|   singleton    | (默认)IOC容器仅创建一个Bean实例，IOC容器每次返回的是同一个Bean实例。(如配置dao和service) |
|   prototype    | IOC容器可以创建多个Bean实例，每次请求会返回新的bean对象(配置实体类时) |
|    request     | 将单个 bean 定义范围限定为单个 HTTP 请求的生命周期。也就是说，每个 HTTP 请求创建一个新的 bean 实例。仅在支持web的Spring `ApplicationContext`中有效。 |
|    session     | 将单个 bean 定义范围限定为一个会话范围的生命周期。每次会话创建一个新的bean实例。仅在支持web的Spring`ApplicationContext`中有效。 |
|  application   | 将单个 bean 定义范围限定为`ServletContext`(即application)的生命周期。仅在支持web的Spring`ApplicationContext`中有效。 |
|   WebSocket    | 将单个 bean 定义范围限定为`WebSocket`的生命周期。仅在支持web的Spring`ApplicationContext`中有效。 |
| global-session | 作用于集群环境内的一个session范围，。仅在支持web的Spring`ApplicationContext`中有效。 |

##### 依赖注入

注入方式

* 基于构造函数的依赖注入
* 基于Setter的依赖注入
* 基于注解的依赖注入

==**基于构造函数的依赖注入**==

使用标签`<constructor-arg/>`

```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

属性配置

* `value` ：表示给参数要注入的值(只能是基本数据类型)

* `type` : 根据构造函数参数的数据类型注入

  ```xml
  <bean id="exampleBean" class="examples.ExampleBean">
      <constructor-arg type="int" value="7500000"/>
      <constructor-arg type="java.lang.String" value="42"/>
  </bean>
  ```

  此种方法，当参数类型有重复时，无法区分

* `index` : 给指定索引参数，设置指定值

  ```xml
  <bean id="exampleBean" class="examples.ExampleBean">
      <constructor-arg index="0" value="7500000"/>
      <constructor-arg index="1" value="42"/>
  </bean>
  ```

  > 索引从0开始

* `name` : 通过构造函数的参数名称注入(==**推荐**==)

  ```xml
  <bean id="exampleBean" class="examples.ExampleBean">
      <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
  </bean>
  ```
  
  `ref` : 注入指定的bean数据类型
  
  ```java
  package x.y;
  public class User {
      ...
          
      private Date birthday;
      
      public User(..., Date birthday){
          ...
              
          this.birthday=birthday;
      }
  }
  ```
  
  此时若想给`birthday`注入`Date`类型，就需要`ref`属性
  
  ```xml
  <bean id="date" class="java.util.Date"/>
  
  <bean id="user" class="x.y.User">
      ...
      <constructor-arg name="birthday" ref="date"/>
  </bean>
  ```

==**基于setter的依赖注入**==

使用标签`<property>`

可用属性 ：

* `name` : 注入时调用的setter方法的名称
* `value` ：给name属性注入指定的基本数据类型
* `ref` ：注入指定bean数据类型的引用

**关于集合类型数据的注入**

上面我们知道，`name`属性可以为基本数据类型注入值，`ref`可以为bean数据类型注入值，但是集合类型的数据类型的注入需要新的标签。

在基于构造函数的依赖注入方式中可以在`<constructor-arg/>`标签内部使用`<list>`、`<map>`、`<set>`、

`<array>`、`<props>`标签来进行注入

在基于setter方法的依赖注入方式中可以在`<property/>`标签内使用上述标签。

其中，`<list>`、`<set>`、`<array>`标签可以互相替换，`<map>`和`<props>`标签可以互相替换。

**选择基于构造函数的依赖注入还是基于setter的依赖注入？**

由于可以混合使用基于构造函数的DI和基于setter的DI，因此，将构造函数用于强制性依赖项，将setter方法或配置方法用于可选性依赖项是一个很好的法则。请注意，可以在setter方法上使用==@Required注解==，以使该属性成为必需的依赖项。但是，最好使用带有参数的构造函数注入。且通过构造函数可以用`<null>`标签给属性注入`null`值。

**p命名空间和c命名空间**

用于简化代码，需要在约束文件中导入相应约束。

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  	# p命名空间约束
    xmlns:p="http://www.springframework.org/schema/p"
	# 命名空间约束
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    ...
    
</beans>
```

使用

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="thingOne" class="x.y.ThingTwo"/>
    <bean id="thingTwo" class="x.y.ThingThree"/>

    <!-- 使用c标签 -->
    <bean id="thingOne" class="x.y.ThingOne" c:thingTwo-ref="thingTwo" c:thingThree-ref="thingThree" c:email="[emailprotected]"/>
    <!-- 使用p标签 -->
    <bean name="john-modern" class="com.example.Person" p:name="John Doe"
        p:age="18"/>

</beans>
```

##### 自动装配

Spring容器可以自动装配bean之间的关系。您可以通过检查ApplicationContext的内容，让Spring为您的bean自动解决依赖。自动装配具有以下优点：

* 自动装配可以大大减少指定属性或构造函数参数的需要。
* 随着对象的改变，自动装配可以更新配置。例如，如果您需要向类中添加依赖项，则无需修改配置即可自动满足该依赖项。

**配置文件实现自动装配**

使用基于XML的配置文件时，可以使用`<bean/>`标签的`autowire`属性为bean定义指定自动装配模式。自动装配具有四种模式。您可以为每个bean指定自动装配，因此选择哪些配置需要自动装配。下表描述了四种自动装配模式：

|     Mode      | Explanation                                                  |
| :-----------: | :----------------------------------------------------------- |
|     `no`      | （默认）无自动装配。Bean引用必须由ref元素定义。对于较大的部署，建议不要更改默认设置，因为明确指定协作者可以提供更好的控制和清晰度。 |
|   `byName`    | 按id的名字自动装配。spring会自动在容器上下文中查找，和自己对象set方法后面的值对应的 bean的id！ |
|   `byType`    | 按bean需要的class类型查找。如果容器中恰好存在一个需要的class type，则使该属性自动装配。如果存在多个相同class，则将引发异常，这表明您不能对该bean使用byType自动装配。如果没有匹配的bean，则什么都不会发生（未设置该属性）。 |
| `constructor` | 与byType类似，但适用于构造函数参数。如果容器中不存在构造函数参数类型的一个bean，则将引发错误。 |

 **注解自动装配`@autowired`**

要使注解生效，必须要导入`http://www.springframework.org/schema/context`约束，并在配置文件中添加`<context:annotation-config/>`标签

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

`@autowired`注解可以直接添加在构造方法上(基于构造方法的DI)，也可以添加在的类的属性或setter方法上(基于setter方法的DI)，此时会在IOC容器中，会按照Bytype模式进行自动装配。

当注入出现多个相同类型的bean时，spring不知道该注入哪个bean，此时会报错，这是可以再添加`@Qualifier("指定id")`注解，给其指定注入的bean。

> `@autowired`注解上有一个`required `属性，默认为true，它表示是否将该属性设置为必须注入项，若不需要可以将其值设置为false

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

> 在`javax.annotation`包下，也有一个自动装配注解 `@Resource`(先按Byname模式装配)，用法与`@autowired`相似，`@Resource(value="xxx")`可以实现 `@autowired`+`@Qualifier("指定id")`的效果。

##### Bean的生命周期

**单例对象(singleton)的生命周期**

* 出生(初始化)：当IOC容器创建时对象出生
* 活着：只要容器还在，对象一直活着
* 死亡(销毁)：IOC容器销毁，对象消亡

> 总结：单例对象的生命周期和容器相同

**原型对象(prototype)的生命周期**

* 出生：当我们使用对象时spring框架为我们创建

* 活着：对象只要是在使用过程中就一直活着。

* 死亡：当对象长时间不用，且没有别的对象引用时，由Java的垃圾回收器回收

1.  在`<bean>`标签内，`init-method`属性调用对象的初始化方法，`destory-method`属性调用对象的销毁方法。

2.  使用`@Bean(init-method=xxx,destory-mothod=xxx)`进行管理

3.  `@PreDestroy`，`@PostConstruct`在方法上进行管理
4.  `BeanPostProcessor`实现该后置处理器接口，实现postProcessAfterInitialization和postProcessBeforeInitialization方法，用于在**初始化之前和初始化之后对所有bean进行处理**

### 使用注解开发

==**@component[(value="xxx")]**==

* 标注于类上
* 表示此类被spring的IOC容器接管。(可以用于替换`<bean>`标签)
* value属性可选,表示此bean的id。当省略时，默认为该类的类名的首字母小写形式。

> 注意 ：要想使注解生效，需要导入`xmlns:context="http://www.springframework.org/schema/context"`约束，并设置要扫描的包的路径。
>
> ```xml
> <?xml version="1.0" encoding="UTF-8"?>
> <beans xmlns="http://www.springframework.org/schema/beans"
>        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>        # 导入此约束
>        xmlns:context="http://www.springframework.org/schema/context"
>        xsi:schemaLocation="http://www.springframework.org/schema/beans
>         https://www.springframework.org/schema/beans/spring-beans.xsd
>         http://www.springframework.org/schema/context
>         https://www.springframework.org/schema/context/spring-context.xsd">
> 
>     <!-- IOC容器创建时要扫描的包 -->
>     <context:component-scan base-package="com.origami.pojo"/>
> 
> 
> </beans>
> ```

```java
import ori.gami.pojo;
@Component("user")
public class User {
    private int uid;

    private String name;

    private List friends;

    ...
}
```

此时，使用`@component`与如下配置文件具有相同的作用。

```xml
<bean id="user" class="com.origami.pojo.User"/>
```

**==基于@component注解的衍生注解==**

* `@controller` ： 一般用于表现层
* `@Service` ： 一般用于业务层
* `@Repository` ： 一般用于持久层

以上三个注解的作用与 `@Component` 的作用和用法相同，只是为了我们能够明确区分每层的作用，而`@Component` 一般用于除此之外的情况。

==**@Autowired**==

* 自动按照类型注入(byType方式)。
  * 只要容器中有唯一的一个bean对象类型和要注入的变量类型匹配，就可以注入成功。
  * 若有多个bean对象类型符合，再按byName方式从多个bean对象中查找id与setXxx方法的xxx相同的那个bean对象(若是在setter方法上注入)，或寻找id与变量名相同的bean对象(在字段上注入变量)，若找不到，报错。
* 可以用变量上，也可以是setter方法上，也可以用在构造函数上。用于替换 `<property>` 和 `<constructor-arg>` 标签。
  * 细节：使用注解注入时，set方法可以不用写。

**==@Qualifier(value="xxx")==**

* 用于`@Autowired`下方使用，也可以用于方法的参数前
* 用于解决找到多个类型匹配，id不匹配时的注入问题
* `value` 指定想要匹配的bean对象的id

**==@Resource[(name="xxx")]==**

* javax的原生注解，不是spring的注解
* 相当于@Autowried加上@Qualifier注解的
* byName方式注入，按照id注入
* name属性为想要注入的bean的id

> ==**注意**== ： 以上三个注入都只能注入bean类型的数据，而基本类型和string类型无法使用上述注解实现。另外，集合类型的注入只能通过XML来实现。

==**@Value(value="xxx")**==

* 用于注入基本类型的数据
* value属性表示要注入的数据的值
* 它可以使用spring中SpEl(spring中的el表达式)，用来注入变量。`${el表达式}`

==**@scope[(value="xxx")]**==

* 与`<bean>`标签中的scope属性功能一致
* value属性可选，用于指定域。不写默认为singleton。

==**@PreDestroy，@PostConstruct**==(了解即可)

* 作用：用于指定销毁方法和用于指定初始化方法
* 在`<bean>`标签中使用`init-method`和`destroy-method`的作用是一样的

前面我们虽然学习了spring的注解，但是在使用的过程中，还是需要使用applicationContext.xml文件，来进行一些设置，spring提供了一种新的方式，在Java代码中使用注解来代替xml文件。

##### @Configuration

* 用于类上，指定当前类是一个配置类，**==用来代替xml文件==**的。

##### **@ComponentScan**(basePackages | values = "com.acme")

* 用于类上，位于@Configuration下，指定创建ioc容器时要扫描的包的范围，用来代替配置文件中的 `<context:component-scan base-package="com.origami.pojo"/>`

* values和basePackages作用一样，都是表示包的范围。

##### @Bean[value="xxx"]

* 用于方法上，将方法的返回值作为bean对象存入到spring的ioc容器中。
* 属性可选，将value的值作为bean的id。若不写，默认方法名为该bean的id。
* initMethod="xxx",destroyMethod="xxx"，与xml文件一致，初始化方法(对象创建完成后调用)与销毁方法(单实例容器关闭或者多实例不销毁)

> @Bean与@Component的区别
>
> > 两者都是将bean对象注入到Ioc容器中，但是@Component的使用有限制，必须是自己定义的类，或是有.java文件的类才能够使用，而如果你只有.class文件，或是只有jar包，那么你就没有地方可以使用@Component注解，而@Bean的使用 更加广泛，任意的类都可以添加到ioc容器中。

若在Configuration类中有一个bean配置带有参数，那么 spring会在configuration中查找参数类型的bean对象，查找方式与@Autowried一致。

##### @Import(value="xxx.class")

**@Import注解可以实现导入第三方的包的bean到容器的功能，配合注解Configuration一起使用， 可以实现一个注解就可以注入第三方bean的能力**

我们可能会有很多的configuration类，将相同类型的bean对象放置到同一个configuration类中，此时要加载很多的配置类，我们可以在主配置类下使用@Import注解，将其他的配置类导入到主配置类，这样只需要加载主配置类就可以了。

* 用于主配置类上，便于同时加载多个子配置类。
* 用于==@Configuration类==上，将指定类加入ioc容器，类似于@Bean，spring4.2以前只可导入配置类，之后可以导入普通类
* **==导入`ImportSelect`或`ImportBeanDefinitionRegistrar`的实现类==**
  
    * 实现ImportSelector接口可以个性化加载类
    
        ```java
        public class MyImportSelector implements ImportSelect{
            //返回值，就是到导入到容器中的组件全类名
            // AnnotationMetadata :当前标注@Import注解的类的所有注解信息
            @Override
            public String[] selectImports(AnnotationMetadata Metadata){
                // 要加入到ioc容器的类的全限定类名数组，可对元数据进行筛选等
                return xxx;
            }
        }
        ```
    
        
    
* `ImportBeanDefinitionRegistrar`与上面类似，用于对bean进行自定义

##### **==@propertySource("classpath:xxx/xxx/xxx.properties")==**

* 用于类上，用于读取指定路径下的properties文件
* 可以使用`classpath`关键字，指定类路径，填写以类路径为根目录的文件的位置
* ==在类中配合`SpEl`表达式${key}来读取props文件中的内容==

### AOP(面向切面编程)

##### 简介

AOP称作面向切面编程。通过预编译方式和运行期**动态代理**实现在**不修改源代码**的情况下给程序动态统一添加功能的一种技术。

与OOP（Object Oriented Programming 面向对象）对比，传统的OOP开发中的代码逻辑是自上而下在这些之上而下的过程中会产生写横切性的问题。而这些横切性的问题又与我们主业务逻辑关系不大。会散落在代码的各个地方，造成难以维护。

AOP的编程思想就是把这些横切性的问题和主业务逻辑进行分离，从而起到解耦的目的。

* 日志记录，性能统计，安全控制，事务处理，异常处理等。

动态代理分为两种方法：

- 基于接口的动态代理(proxy)
  * 要求：被代理类最少实现一个接口
  * 提供者：JDK官方
  * 涉及的类：Proxy创建代理对象的方法：newProxyInstance(ClassLoader，Class[],InvocationHandler)
    * 参数的含义：
    * ClassLoader：类加载器。和被代理对象使用相同的类加载器。一般都是固定写法。
    * Class[]：字节码数组。被代理类实现的接口。（要求代理对象和被代理对象具有相同的行为）。一般都是固定写法。
    * InvocationHandler：它是一个接口，就是用于我们提供增强代码的。我们一般都是些一个该接口的实现类。实现类可以是匿名内部类。
- 基于子类的动态代理(cglib)
  - 要求：被代理类不能是最终类。不能被final修饰
  - 提供者：第三方CGLib
  - 涉及的类：Enhancer创建代理对象的方法：create(Class，Callback)
    - 参数的含义：
    - Class：被代理对象的字节码
    - Callback：如何代理。它和InvocationHandler的作用是一样的。它也是一个接口，我们一般使用该接口的子接口MethodInterceptor在使用时我们也是创建该接口的匿名内部类。

若类有实现了接口，则spring默认使用基于接口的动态代理，否则使用基于子类的动态代理。

##### 相关术语

**Joinpoint（连接点）**

所谓连接点是指那些被拦截到的点。在spring中，这些点指的是方法，因为spring只支持方法类型的连接点。我们要在增强的方法所在类的所有方法都是连接点。

**Pointcut（切入点）**

所谓切入点是指我们要对哪些Joinpoint进行拦截的定义。即是我们要在哪些方法前后进行一些操作，这个方法即为切入点。

**Advice（通知/增强）**

所谓通知是指拦截到Joinpoint之后所要做的事情就是通知。

通知的类型：

* 前置通知(在想要增强的方法前面的方法)
* 后置通知(在想要增强的方法后面的方法)
* 异常通知(关于异常处理catch内的方法)
* 最终通知(finally中的方法)
* 环绕通知(在Java代码中可以一次实现上述四个通知，即为环绕)。

**Proxy**

代理对象

**Target**

被代理的对象(被增强的那个类)

**Weaving（织入)**

把增强应用到目标对象来创建新的代理对象的过程。

**Aspect（切面）**

是切入点和通知（引介）的结合。就是用于增强的那个类。

##### 基于xml的AOP

需要导入aspectj依赖，用于织入execution表达式(在spring的aop中即为excutu方法)。

```xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.2</version>
</dependency>
```

需要在xml文件中增添aop约束

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       # 导入此约束
       xmlns:context="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop
		https://www.springframework.org/schema/aop/spring-aop.xsd
		http://www.springframework.org/schema/context
		https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- IOC容器创建时要扫描的包，可用@ComponentScan注解代替-->
    <context:component-scan base-package="com.origami.pojo"/>
    
    <!-- 开启注解aop支持，可被@EnableAspectJAutoPxoxy注解代替，此注解用于配置类上-->
    <aop:aspectj-autoproxy/>


</beans>
```

创建增强类，类内的方法用于增强代理对象

```java
public class Log {
    public void before() {
        System.out.println("前置方法");
    }

    public void after() {
        System.out.println("后置方法");
    }
}
```

创建接口和实现类

```java
public interface UserMapper {
    public void addUser(int id);

    public int update();

    public String find(int id);
}

public class UserMapperImpl implements UserMapper {

    @Override
    public void addUser(int id) {
        System.out.println("成功添加用户");
    }

    public int update() {
        System.out.println("成功更新用户");
        return 0;
    }

    public String find(int id) {
        System.out.println("成功查找用户");
        return "ok";
    }
}
```

配置xml文件

```xml
<!-- 将增强对象注入ioc容器-->
<bean id="userDao" class="com.origami.dao.impl.UserMapperImpl"/>

<!-- 将切面对象注入ioc容器-->
<bean id="log" class="com.origami.log.Log"/>

<!-- 设置aop配置-->
<aop:config>
    <!-- 切面，id为唯一标识，ref指向切面对象 -->
    <aop:aspect id="logAdvice" ref="log">
        <!-- 前置通知-->
        <!-- pointcut为切入点，为要增强的对象、方法-->
        <aop:before method="before"
                    pointcut="execution( * com.origami.dao.impl.*.*(..))"/>
        <!-- 后置通知-->
        <aop:after-returning method="after" pointcut="execution( * *..*(..))"/>
    </aop:aspect>
</aop:config>
```

xml详细配置

1. 把通知Bean交给spring来管理
2. 使用`<aop:config>`标签表明开始AOP的配置
3. 使用`<aop:aspect>`标签表明配置切面

	* id属性：是给切面提供一个唯一标识
* ref属性：是指定通知(增强)类bean的Id。

1. 在`<aop:aspect>`切面标签的内部使用对应标签来配置通知的类型
   * `<aop:before>`：表示配置前置通知
   * `<aop:after-returning>`：表示配置后置通知
   * `<aop:after-throwing>`：异常通知
   * `<aop:after>` : 最终通知
   * `<aop:around>` : 环绕通知
     * `method`属性：用于指定用增强类中哪个方法进行增强
  * `pointcut`属性：用于指定为哪个类哪些方法进行增强。格式为切入点表达式。切入点表达式的写法：
    
* 关键字：execution(表达式)
  
* 织入表达式
  
  * 访问修饰符 返回值类型 包名.包名.包名..类名.方法名(参数列表)
  
* 标准的表达式写法：
  
  `public void com.origami.dao.impl.UserMapperImpl.addUser(int)`
  
  * 权限修饰符可以省略
  * 通配符`*`可以代替返回值，表示任意返回值类型或void
  * 包名可以用通配符 `*` 表示，包名后跟`..`可以表示此包下任意位置
  * 类名可以用通配符 `*`表示任意类名
  * 方法名可以用通配符 `*`表示任意方法
  * 基本参数类型直接表示，其他参数类型全限定包名。`*`表示任意类型参数，`..`表示任意类型也可以没有参数。
  
* 开发中常用如下写法：

  * `execution( * com.origami.dao.impl.*.*(..))`
  * 表示指定包下(如dao或service包下)的任意类的任意方法，可以任意返回值，可以任意参数

若有多个通知要给同一个包的方法进行增强，每次都写`execution( * com.origami.dao.impl.*.*(..))`这样一长串很麻烦，可以使用`poinntcut-ref`属性和`<aop:pointcut>`标签进行重用。

```xml
<aop:pointcut id="pt1" expression="execution( * com.origami.dao.impl.*.*(..))"/>

<aop:before method="x" pointcut-ref="pt1"/>
<aop:after-returning method="xx" pointcut-ref="pt1"/>
<aop:after-throwing method="xxx" pointcut-ref="pt1"/>
<aop:after method="xxxx" pointcut-ref="pt1"/>
```

`<aop:pointcut>`标签若定义在标签切面标签内部，则只在该切面内有效，若定义在切面标签外部(必须定义在所有切面标签之前)，则对所有切面有效。

环绕通知

* 编写环绕通知方法

  ProceedingJoinPoint为spring提供的一个接口，

```java
//环绕通知方法
public Object around(ProceedingJoinPoint pjt) {
        Object result = null;
        Object[] args = pjt.getArgs();
        try {
              //此处若有增强代码则为前置通知
            
            // 调用原方法
            result = pjt.proceed(args);
              //此处若有增强代码则为后置通知
            return result;
        } catch (Throwable throwable) {
              //此处若有增强代码则为异常通知
            throw new RuntimeException(throwable);
        }finally{
            //此处若有增强代码则为最终通知
        }
    }
```

此时可以在任意地方对方法进行增强，若增强在proceed()方法前，则为前置通知，若增强在proceed()方法后，则为后置通知，若在catch中，则为异常通知，若在finally中，则为最终通知。

##### 基于注解的AOP

==@EnableAspectJAutoPxoxy==

开启基于注解的AOP功能，用于配置类上。

其中有个参数`proxyTargetClass` 赋值为 `true`，如果我们不写参数，默认为 false，若织入表达式不是一个接口，会报错ClassCaseException。这跟Spring AOP 动态代理的机制有关，这个 `proxyTargetClass` 参数决定了代理的机制，==当这个参数为 false 时，通过jdk的基于接口的方式进行织入==，这时候代理生成的是一个接口对象，将这个接口对象强制转换为实现该接口的一个类，自然就抛出了上述类型转换异常。
反之，`proxyTargetClass` 为 `true`，则会使用 cglib 的动态代理方式，这种方式的缺点是拓展类的方法被`final`修饰时，无法进行织入。

==@Aspect==

* 用于切面类上(用于增强的类)，表示当前类是一个切面类

>   注意 ：此类上应该添加@component，使ioc容器接管此类。

==@Before("xxx")、@AfterTurning("xxx")、@AfterThrowing("xxx")、@After("xxx")、@Around("xxx")==

* 用于切面类中的方法上，即增强方法
* 括号内为切入点表达式
* **注意不要导错包，是aspcetj的包，junit也有@after注解**

==@Pointcut("xxx")==

* 代表一个切入点，方便其他通知注解引用

* 用于自定义的成员变量上

* xxx为切入点表达式或以下

* | AspectJ指示器 | 描述                                                         |
  | ------------- | ------------------------------------------------------------ |
  | arg()         | 限制连接点匹配参数为指定类型的执行方法                       |
  | @args()       | 限制连接点匹配参数由指定注解标注的执行方法                   |
  | execution()   | 用于匹配是连接点的执行方法                                   |
  | this()        | 限制连接点匹配AOP 代理的Bean 引用为指定类型的类              |
  | target()      | 限制连接点匹配目标对象为指定类型的类                         |
  | @target()     | 限制连接点匹配特定的执行对象，这些对象对应的类要具备指定类型的注解 |
  | within()      | 限制连接点匹配指定的类型                                     |
  | @within()     | 限制连接点匹配指定注解所标注的类型（当使用Spring AOP时，方法定义在由指定的注解所标注的类里） |
  | @annotation   | 限制匹配带有指定注解连接点                                   |
  
  ```java
  //设置切入点1
  @Pointcut("execution( * com.itheima.service.imp1.*.*(..))")
  private void ptl(){}
  
  @Pointcut("@annotation(run.halo.app.annotation.DisableOnCondition)")
  private void pt2(){}         
  
  //其他通知引用切入点，注意要加()
  @After(" pt1() && pt2()")
  public void test1(){
      ...
  }
  @Around("execution(* com.sharpcj.aopdemo.test1.IBuy.buy(..))")
  public void xxx(ProceedingJoinPoint pj) {
      try {
          System.out.println("Around aaa ...");
          //  调用原方法，其他代码均为对其的增强
          pj.proceed();
          System.out.println("Around bbb ...");
      } catch (Throwable throwable) {
          throwable.printStackTrace();
      }
  }
  ```

> ==**注意 ：使用注解aop的四个通知注解，spring调用时存在问题，并不是按照前置>后置(异常)>最终的顺序执行，而是先执行最终通知然后才执行后置通知，推荐使用环绕通知进行自定义。**==

>   **使用aop功能时，不要自己new对象，而是要用spring的ioc容器提供的对象**

### 整合junit单元测试

应用程序的入口

* main方法
* junit单元测试中，没有main方法也能执行，junit集成了一个main方法，该方法就会判断当前测试类中哪些方法有@Test注解，junit就让有Test注解的方法执行
* junit不会知道我们有没有使用spring，也不会为我们创建spring容器

解决方法

* 导入spring-test的依赖(spring5以上，junit的版本要4.12以上)

  ```xml
  <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-test</artifactId>
              <version>5.1.5.RELEASE</version>
              <scope>test</scope>
  </dependency>
  ```

* 在测试类上方使用`@RunWirh`注解，用spring的main方法代替junit的main方法

* 使用`@ ContextConfiguration`注解，使junit知道，是使用注解还是xml文件创建容器

  ```java
  @ Runwith(SpringJUnit4ClassRunner. class)
  @ ContextConfiguration(classes=SpringConfiguration. class)
  public class UserServiceTest { 
      @Autowried
      private UserService us;
      
      @Test
      public void test1(){
          us.xxx();
      }
  ```

基于web项目的单元测试

```java
@Runwith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes={"SpringConfiguration.class","springmvc.xml")
@WebAppConfiguration
public class WebTest{
    //声明一个模拟请求的对象
    private MockMvc mockMvc;
    //需要一个web容器
    @Autowired
    private WebApplicationContext context;
    
    @Before
    public void sefup({
        mockMvc=MockMvcBuilders.webAppContextSetup(context).build();
    }
                      
    @Test
    public void testLogin()throws Exception{
        //发送post请求到指定路径，携带指定参数
    	MvcResult mvcResult = mockMvc.perform(MockMvcRequestBuilders.post(urlTemplate:"/consumer/Login/auth").param("username","damu").param("password","123")).andReturn();
        System.out.println(mvcResult.getResponse().getcontentAsString());
    }
```



### 事务控制

spring的事务控制的类都处于spring-tx的包内，需要导入依赖。

spring支持Java Transaction API(JPA)以及jdbc的事务，jpa是Java提供的事务管理api，他可以跨越多个数据库和消息进行事务管理，而jdbc的事务管理仅在connection的范围内。

声明式事务是使用aop的方式完成事务，此外还有编程式事务，在代码中处理事务，此种不推荐。

##### 基于xml的事务控制

要开启 Spring 的事务处理功能，在 Spring 的配置文件中创建一个 `DataSourceTransactionManager` 对象：

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <constructor-arg ref="dataSource" />
</bean>
```

> ==注意：为事务管理器指定的 `DataSource` **必须**和用来创建 `SqlSessionFactoryBean` 的是同一个数据源，否则事务管理器就无法工作了。==

实例

```java
// 要想进行事务的接口

package x.y.service;

public interface FooService {

    Foo getFoo(String fooName);

    Foo getFoo(String fooName, String barName);

    void insertFoo(Foo foo);

    void updateFoo(Foo foo);

}
```

接口的实现

```java
package x.y.service;

public class DefaultFooService implements FooService {

    @Override
    public Foo getFoo(String fooName) {
        // ...
    }

    @Override
    public Foo getFoo(String fooName, String barName) {
        // ...
    }

    @Override
    public void insertFoo(Foo foo) {
        // ...
    }

    @Override
    public void updateFoo(Foo foo) {
        // ...
    }
}
```

配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

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

    <!-- 连接池 -->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
        <property name="url" value="jdbc:oracle:thin:@rj-t42:1521:elvis"/>
        <property name="username" value="scott"/>
        <property name="password" value="tiger"/>
    </bean>

    <!-- 事务管理器 -->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>


</beans>
```

也可以用注解@Transactional来进行配置

需要在xml中设置

```xml
<tx:annotation-driven/>
```

##### 基于注解的事务

==@EnableTransactionManagement==

用于开启基于注解的事务管理

配置类

```java
@Configuration
@Aspect
@EnableTransactionManagement //也可以在主启动类上
public class TxConfig{
    
    @Bean
    public DataSource dataSource() throws Exception{
        ComboPooledDataSource dataSource=new ComboPooledDataSour
            dataSource.setUser("root");
        dataSource.setPassword("123456")；
            dataSource.setDriverClass("com.mysql.jdbc.Driver")；
            dataSource.setJdbcUr1("jdbc:mysq1://localhost:3306/test");
        return dataSource;
    }
    
    @Bean
    public JdbcTemplate jdbcTemplate() throws Exception{
        // Spring对@Configuration类会特殊处理；给容器中加组件的方法,多次调用都只是从ioc容器取,而不会新建
        JdbcTemplate jdbcTemplate=new JdbcTemplate(dataSource());
        return jdbcTemplate;
    }
    
    
    @Bean
    public PlatfomTrapsaetionManager transactionManager(){
        // 需要传入datasource，mybatis使用这个事务管理器
        return new DatasourceTransactionManagern(dataSource);
    }
}
```



| 数据访问技术 | 事务管理器的实现             |
| ------------ | ---------------------------- |
| JDBC         | DataSourceTransactionManager |
| Hibernate    | HibernateTransactionManager  |
| JPA          | JPATransactionManager        |
| 分布式事务   | JtaTransactionManager        |

@Transactional

| Property                                                     | Type                                                         | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| [value](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#tx-multiple-tx-mgrs-with-attransactional) | `String`                                                     | Optional qualifier that specifies the transaction manager to be used. |
| [propagation](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#tx-propagation) | `enum`: `Propagation`                                        | 传播行为.                                                    |
| `isolation`                                                  | `enum`: `Isolation`                                          | 隔离级别。传播行为是 `REQUIRED` or `REQUIRES_NEW`时生效      |
| `timeout`                                                    | `int` (in seconds of granularity)                            | 超时时间.传播行为是 `REQUIRED` or `REQUIRES_NEW`时生效       |
| `readOnly`                                                   | `boolean`                                                    | 事务的读写和只读. Only applicable to values of `REQUIRED` or `REQUIRES_NEW`. |
| `rollbackFor`                                                | `Class` 的集合, 必须是 `Throwable`的衍生类                   | 必须回滚的异常.                                              |
| `rollbackForClassName`                                       | 类名的集合. 必须是 `Throwable`的衍生类                       | 必须回滚的异常的名字.                                        |
| `noRollbackFor`                                              | Array of `Class` objects, which must be derived from `Throwable.` | 不用回滚的异常.                                              |
| `noRollbackForClassName`                                     | Array of `String` class names, which must be derived from `Throwable.` | 不用回滚的异常的名字.                                        |
| `label`                                                      | Array of `String` labels to add an expressive description to the transaction. | Labels may be evaluated by transaction managers to associate implementation-specific behavior with the actual transaction. |

用于需要进行事务管理的方法上或是类上(类中所有方法，均使用事务)

### 源码

`BeanFactory接口`

最顶层的ioc容器，简单的获取bean，是否包含指定bean，查询bean等

`HierarchicalBeanFactory`接口是在继承BeanFactory的基础上，实现BeanFactory的父子关系。

`AutowireCapableBeanFactory`接口是在继承BeanFactory的基础上，实现Bean的自动装配功能

`ListableBeanFactory`接口是在继承BeanFactory的基础上，实现Bean的list集合操作功能

`ConfigurableBeanFactory`接口是在继承`HierarchicalBeanFactory`的基础上，实现BeanFactory的全部配置管理功能， 还实现了`SingletonBeanRegistry`是单例bean的注册接口

`ConfigurableListableBeanFactory`接口是继承AutowireCapableBeanFactory，ListableBeanFactory，Configurab

leBeanFactory三个接口的一个综合接口

`DefaultListableBeanFactory`,BeanFactory的基础实现

`FactoryBean接口`

实现该接口，然后实现其中的getObject()方法，此时将返回的Object对象加入到IOC容器中。

`XxxAware接口`

实现接口可以给自己的bean注入spring底层的组件，如实现ApplicationContextAware接口，实现setApplicationContext方法，可以在自己的bean中得到ApplicationContext来进行一些操作。BeanNameAware获取当前Bean的ioc容器的名字。每个XxxAware接口都有对应的XxxAwareProcessor后置处理器来进行操作的。

`BeanFactoryPostProcessor`

`BeanDefinition`

bean定义，定义了一些bean的属性信息，包括bean的名字，全限定类名，作用域，懒加载 ，父类名字，是否自动注入，根据他来创造bean

`BeanDefinitionRegistry`

bean定义的注册器，用于注册BeanDefinition，调用`registerBeanDefinition`方法，其内部缓存维护一个beanDefinitionMap，`ConcurrentHashMap<String, BeanDefinition>`

过程

1、`Resource`资源定位：找到配置文件,包装成`Resource`

2、`BeanDefinition`载入和解析

`DefaultListableBeanFactory`

