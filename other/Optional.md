### 简介

从 Java 8 引入了一个 *Optional*  类。Optional 类主要解决的问题是臭名昭著的空指针异常（NullPointerException）。

本质上，这是一个包含有可选值的包装类，这意味着 Optional 类既可以含有对象也可以为空。

假设下面一个例子：

```java
String isocode = user.getAddress().getCountry().getIsocode().toUpperCase();
```

里面每一步都可能有NPE，因此我们需要做判断。

```java
if (user != null) {
    Address address = user.getAddress();
    if (address != null) {
        Country country = address.getCountry();
        if (country != null) {
            String isocode = country.getIsocode();
            if (isocode != null) {
                isocode = isocode.toUpperCase();
            }
        }
    }
}
```

### 创建Optional

```java
// 创建一个空的Optional
Optional.empty();

// 创建一个非空T类型的Optional
Optional.of(T value);

// 创建一个可为null的T类型的Optional
Optional.ofNullable(T value)
```

### **get**

获取Optional内的值value，value为null，可能会抛出异常。

### **ifPresent() **

检查值value是否存在

### **orElse**方法

如果 Optional 中有值则将其返回，否则返回 orElse 方法传入的参数。

```java
T orElse(T other);

// 例子
// 查询id为1的user，若不存在，获取指定的User
User user = Optional.ofNullable(user).orElse(createNewUser());
```

### **orElseGet**

orElseGet 与 orElse 方法的区别在于，orElseGet 方法传入的参数为一个 Supplier 接口的实现 —— 当 Optional 中有值的时候，返回值；当 Optional 中没有值的时候，返回从该 Supplier 获得的值。

```java
User user = Optional.ofNullable(user).orElseGet(() -> createNewUser());
```

>   orElse()方法获取非空值时，仍会创建user对象，使用orElseGet则不会。

### **orElseThrow**

orElseThrow 与 orElse 方法的区别在于，orElseThrow 方法当 Optional 中有值的时候，返回值；没有值的时候会抛出异常，抛出的异常由传入的 exceptionSupplier 提供。

### **map**，**flatMap**，**filter**

与stream的用法一致。

Java 9 新方法

### **or()**

*or()* 方法与 *orElse()* 和 *orElseGet()* 类似，它们都在对象为空的时候提供了替代情况。*or()* 的返回值是由 *Supplier* 参数产生的另一个 *Optional* 对象。

```java
User result = Optional.ofNullable(user)
      .or( () -> Optional.of(new User("default","1234")))
```

### **ifPresentOrElse() **

*ifPresentOrElse()* 方法需要两个参数：一个 *Consumer* 和一个 *Runnable*。如果对象包含值，会执行 *Consumer* 的动作，否则运行 *Runnable*。

### **stream()**

将实例转换为 Stream对象