# Type 接口 

Type接口完整的定义：

```java
public interface java.lang.reflect.Type {
    /**
     * Returns a string describing this type, including information about any type parameters.
     * @implSpec The default implementation calls {@code toString}.
     * @return a string describing this type
     * @since 1.8
     */
    default String getTypeName() {
        return toString();
    }
}
```

所有已知子接口：`GenericArrayType, ParameterizedType, TypeVariable<D>, WildcardType`

所有已知实现类：Class

*Type is the common superinterface for all types in the Java programming language. These include raw types, parameterized types, array types, type variables and primitive types.*

Type 是 Java 编程语言中【所有类型】的公共高级接口。它们包括原始类型、参数化类型(泛型)、数组类型、类型变量和基本类型。

注意区分Type与Class的区别，这里的Class是Type的一种，而像数组、枚举等"类型"是Class的一种。

### Type接口的来历

我们知道，Type是JDK5开始引入的，其引入主要是为了泛型，没有泛型的之前，只有所谓的原始类型。此时，所有的原始类型都通过字节码文件类Class类进行抽象。Class类的一个具体对象就代表一个指定的原始类型。 

泛型出现之后，也就扩充了数据类型。从只有原始类型扩充了参数化类型、类型变量类型、泛型数组类型，也就是Type的子接口。 

那为什么没有统一到Class下，而是增加一个Type呢？（Class也是种类的意思，Type是类型的意思） 

是为了程序的扩展性，最终引入了Type接口作为Class，ParameterizedType，GenericArrayType，TypeVariable和WildcardType这几种类型的总的父接口。这样实现了Type类型参数接受以上五种子类的实参或者返回值类型就是Type类型的参数。

### Java中的所有类型 

-   raw type：原始类型，对应Class 。这里的Class不仅仅指平常所指的类，还包括数组、接口、注解、枚举等结构。
-   primitive types：基本类型，仍然对应Class
-   parameterized types：参数化类型，对应ParameterizedType，带有类型参数的类型，==**即常说的泛型**==，如：List<T>、Map<Integer, String>、List<? extends Number>。
-   type variables：类型变量，对应TypeVariable<D>，如参数化类型中的E、K等类型变量，表示泛指任何类。
-   array types：(泛型)数组类型，对应GenericArrayType，比如List<T>[]，T[]这种。注意，这不是我们说的一般数组，而是表示一种==**【元素类型是参数化类型或者类型变量的】数组类型**==。

注意：WildcardType代表通配符表达式，或泛型表达式，比如【?】【? super T】【? extends T】。虽然WildcardType是Type的一个子接口，但并不是Java类型中的一种。

### 测试代码

```java
public class Test {
    public static void main(String[] args) throws NoSuchMethodException, SecurityException {
        Method method = Test.class.getMethod("testType", List.class, List.class, List.class, List.class, List.class, Map.class);
        
        Type[] types = method.getGenericParameterTypes();//按照声明顺序返回方法的 形参类型的 Type 对象数组 
        for (Type type : types) {
            ParameterizedType pType = (ParameterizedType) type;//最外层都是ParameterizedType
            Type[] types2 = pType.getActualTypeArguments();//返回表示此类型【实际类型参数】的 Type 对象的数组，即最外层泛型接口内的类型
            for (int i = 0; i < types2.length; i++) {
                Type type2 = types2[i];
                System.out.println(i + "  类型【" + type2 + "】\t类型接口【" + type2.getClass().getInterfaces()[0].getSimpleName() + "】");
            }
        }
    }

    public <T> void testType(List<String> a1, 
                             List<ArrayList<String>> a2, 
                             List<T> a3, 
                             List<? extends Number> a4, 
                             List<ArrayList<String>[]> a5, 
                             Map<String, Integer> a6) {
    }
}
```

运行结果

```
0  类型【class java.lang.String】	类型接口【Serializable】
0  类型【java.util.ArrayList<java.lang.String>】	类型接口【ParameterizedType】
0  类型【T】	类型接口【TypeVariable】
0  类型【? extends java.lang.Number】	类型接口【WildcardType】
0  类型【java.util.ArrayList<java.lang.String>[]】	类型接口【GenericArrayType】
0  类型【class java.lang.String】	类型接口【Serializable】
1  类型【class java.lang.Integer】	类型接口【Serializable】
```

### **Type 接口的四个子接口**

##### ParameterizedType 泛型/参数化类型【重要】

```java
public interface java.lang.reflect.ParameterizedType extends Type
```

ParameterizedType 表示参数化类型，带有类型参数的类型，即常说的泛型，如：List<T>、Map<Integer, String>、List<? extends Number>。

方法

-   `Type[]  getActualTypeArguments()` 返回表示此类型实际类型参数的 Type 对象的数组。【重要】

-   -   简单来说就是：==**获得参数化类型中<>里的类型参数的类型**==。
    -   因为可能有多个类型参数，例如Map<K, V>，所以返回的是一个Type[]数组。
    -   注意：无论<>中有几层<>嵌套，这个方法仅仅脱去最外层的<>，之后剩下的内容就作为这个方法的返回值，所以其返回值类型不一定。

-   `Type  getOwnerType()`  返回 Type 对象，表示此类型是其成员之一的类型。

-   -   如果此类型为顶层类型，则返回 null（大多数情况都是这样）。

-   `Type  getRawType()` 返回 Type 对象，表示声明此类型的类或接口。

-   -   简单来说就是：返回最外层<>前面那个类型，例如Map<K ,V>，返回的是Map类型。

```java
public class Test {
    public static void main(String[] args) throws Exception {
        Method method = Test.class.getMethod("test", Map.Entry.class);
        Type[] types = method.getGenericParameterTypes();
        for (Type type : types) {
            ParameterizedType pType = (ParameterizedType) type;
            System.out.println(pType + " ★ " 
                               + pType.getOwnerType() + " ★ " 
                               + pType.getRawType() + " ★ " 
                               + Arrays.toString(pType.getActualTypeArguments()));
            /** 
             * java.util.Map.java.util.Map$Entry<T, U> ★ 
             * interface java.util.Map ★ 
             * interface java.util.Map$Entry ★ 
             * [T, U]
             */
        }
    }

    public static <T, U> void test(Map.Entry<T, U> mapEntry) {
    }
}
```

##### TypeVariable\<D> 类型变量/泛指任何类【掌握】

```java
public interface java.lang.reflect.TypeVariable<D extends GenericDeclaration> extends Type
```

类型变量，如参数化类型中的E、K等类型变量，表示泛指任何类。

方法

-   `Type[]  getBounds()` 返回表示此类型变量上边界的 Type 对象的数组。

-   -   返回：表示此类型变量的上边界的 Type 的数组
    -   注意，如果未显式声明上边界，则上边界为 Object。  

-   `D  getGenericDeclaration()` 返回 GenericDeclaration 对象，该对象表示声明此类型变量的一般声明。

-   -   返回：为此类型变量声明的一般声明。

-   `String  getName()` 返回此类型变量的名称，它出现在源代码中。

```java
public static <T extends Person, U> void main(String[] args) throws Exception {
    Method method = Test.class.getMethod("main", String[].class);
    // 返回声明顺序的 TypeVariable 对象的数组
    TypeVariable<?>[] tvs = method.getTypeParameters();
    System.out.println("声明的类型变量有：" + Arrays.toString(tvs));//[T, U]

    for (int i = 0; i < tvs.length; i++) {
        GenericDeclaration gd = tvs[i].getGenericDeclaration();
        
        System.out.println("【GenericDeclaration】" + gd);//public static void com.bqt.Test.main(java.lang.String[]) throws java.lang.Exception
        
        System.out.println(gd.getTypeParameters()[i] == tvs[i]);//true。    GenericDeclaration和TypeVariable两者相互持有对方的引用

        System.out.println(tvs[i] 
                           + " " 
                           + tvs[i].getName() 
                           + "  " 
                           + Arrays.toString(tvs[i].getBounds()));//T  T  [class com.bqt.Person] 和 U  U  [class java.lang.Object]
    }
}
```

##### GenericArrayType (泛型)数组类型 

```java
public interface java.lang.reflect.GenericArrayType extends Type
```

GenericArrayType 表示一种(泛型)数组类型，其组件类型为参数化类型或类型变量。

比如List<T>[]，T[]这种。注意，这不是我们说的一般数组，而是表示一种【元素类型是参数化类型或者类型变量的】数组类型。

方法

-   `Type  getGenericComponentType()` 返回表示此数组的组件类型的 Type 对象。此方法创建数组的组件类型。

-   -   获取泛型数组中元素的类型，要注意的是：无论从左向右有几个[]并列，这个方法仅仅脱去最右边的[]之后剩下的内容就作为这个方法的返回值。

##### WildcardType 通配符(泛型)表达式 

```java
public interface java.lang.reflect.WildcardType extends Type
```

WildcardType代表通配符表达式，或泛型表达式，比如【?】【? super T】【? extends T】。虽然WildcardType是Type的一个子接口，但并不是Java类型中的一种。

方法

-   `Type[]  getLowerBounds()` 返回表示此类型变量下边界的 Type 对象的数组。注意，如果不存在显式声明的下边界，则下边界为类型 null。在此情况下，将返回长度为零的数组。
-   `Type[]  getUpperBounds()` 返回表示此类型变量上边界的 Type 对象的数组。注意，如果不存在显式声明的上边界，则上边界为 Object。