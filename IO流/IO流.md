# IO流

### File

`java.io.File`是文件和目录（文件夹）路径名的抽象表示形式，是专门对文件进行操作的类，只对文件本身进行操作，不能对文件内容进行操作。主要用于文件和目录的创建、查找和删除等操作。

##### 构造方法

构造方法可以封装存在的文件或目录，==也可以封装不存在文件或目录==。

*   `public File(String pathname)` ：通过将给定的**路径名字符串**转换为抽象路径名来创建新的 File实例。
*   `public File(String parent, String child)` ：从**父路径名字符串和子路径名字符串**创建新的 File实例。
*   `public File(File parent, String child)` ：从**父抽象路径名和子路径名字符串**创建新的 File实例。

```java
// 根据一个路径得到File对象
File file = new File("E:\\demo\\a.txt");

// 根据一个目录和一个子文件或子目录得到File对象
File file2 = new File("E:\\demo","a.txt");

// 根据一个父File对象和一个子文件或子目录得到File对象
File file3 = new File("e:\\demo");
File file4 = new File(file3,"a.txt");
```

##### 常用方法

###### 创建

==若创建文件或文件夹时未指定盘符,会在当前项目下创建文件或文件夹==

- `public boolean createNewFile();`    创建文件,如果已经存在文件，则不创建

  >   注意：如果要在某个目录下创建这个文件,则这个目录要已经存在，否则会报错

- `public boolean mkdir();`    创建文件夹,如果已经存在，则不创建

  >   注意：如果要在某个目录创建这个文件夹，则这个目录要已经存在，否则虽然不会报错，但是会创建失败

- `public boolean mkdirs();`    创建文件夹,如果父文件夹不存在,会帮你创建出来

  >   mkdirs只能创建目录
  >
  >   ```java
  >   Flie file = new File("c:\\demo\\a.txt");
  >   file.mkdirs();
  >   ```
  >   此时不会创建`a.txt`文件，而是会创建一个==名为a.txt的文件夹==。
  >   

###### 删除

Java中文件和文件夹的删除不走回收站

- `public boolean delete();`
- 功能：删除此抽象路径名表示的文件或目录。如果此路径名表示一个目录，则该目录必须为空才能删除。
	- 返回：当且仅当成功删除文件或目录时，返回true；否则返回false

###### 重命名

- `public boolean renameTo(File dest);`

  >   如果路径名相同，就直接重命名。
  >   如果路径名不同，就将原路径文件剪切到新的路径并重命名

  当且仅当重命名成功时，返回true；否则返回false

###### 判断

- `public boolean isDirectory();`	判断是否是目录
- `public boolean isFile();`    判断是否是文件
- `public boolean exists();`    判断是否存在
- `public boolean canRead();`    判断是否可读
- `public boolean canWrite();`    判断是否可写
- `public boolean isHidden();`    判断是否隐藏

###### 获取

如果抽象路径名不表示一个目录，或者发生I/0错误，则返回null，目录内为空则返回[ ]

- 基本获取功能

	- `public String getAbsolutePath();`    获取绝对路径并以字符串形式返回
	- `public String getpath();`    获取相对路径并以字符串形式返回
	- `public String getName();`    获取文件或文件夹名称
	- `public long length();`    获取文件大小(字节大小)
	- `public long lastModified();`    获取最后一次的修改时间(毫秒值)

- 高级获取功能

	- `public String[] list();`    返回一个字符串数组，这些字符串是指定目录中的文件和目录。 
	- `public String[] list(FilenameFilter filter);`   返回一个字符串数组，这些字符串指定此抽象路径名表示的目录中满足指定过滤器的文件和目录。若filter为null,则返回所有对象
	- `public File[] listFiles();`    返回一个File数组，这些File表示此目录中的文件。(可以调用这些File进行操作)
	- `public File[] listFiles(FilenameFilter filter);`    返回抽象路径名数组，这些路径名表示此抽象路径名表示的目录中满足指定过滤器的文件和目录。

### 字节流

将硬盘上的文件读取成流或是将流转换为硬盘上的文件

##### FileInputStream

其是InputStream(字节输入流)的子实现类

###### 构造方法

- `public FileInputStream(File file)`
- 创建inputStream流来读取指定的文件类
	- 如果指定文件不存在，或者它是一个目录，而不是一个常规文件，抑或因为其他某些原因而无法打开进行读取，则抛出FileNotFoundException。
	
- `public FileInputStream(String name)`
- 创建inputStream流来读取指定位置的文件
	- 如果指定文件不存在，或者它是一个目录，而不是一个常规文件，抑或因为其他某些原因而无法打开进行读取，则抛出FileNotFoundException。

###### 成员方法

- `public int read()`
- 从 此输入流中读取一个数据字节。
	- 返回值:  下一个数据字节，如果己到达文件末尾，则返回-1。
	
- `public int read(byte[] b)`
- 从 此输入流中将最多b.length个字节的数据读入一个byte数组中。(根据指定的数组大小)
	- 读入缓冲区的字节总数，如果因为己经到达文件末尾而没有更多的数据，则返回-1。(注意:返回值是实际读入的字节总数,而不是设定的字节数组的长度)
	
- `public void read(byte[]b, int off,int len)`
- 与上面类似,不过在读取时,如果最后一次读取的字节数不是刚好与字节数组相同,那么它会把上一次字节数组没被读取的字节覆盖的部分也保留下来,就会造成与原数据不一致,因此,要根据返回值对读取的字节数进行截取

```java
public class Input {
    public static void main(String args[]) {
        FileInputStream inputStream = null;
        try {
            inputStream = new FileInputStream("a.txt");
            int len = 0;
            byte[] bys = new byte[1024];
            while ((len = inputStream.read(bys)) != -1) {
                System.out.println(new String(bys,0,len));
            }
        
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                inputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

    }
}
```

##### 2.2 FileOutputStream

其是OutputStream(字节输出流)的基于文件的子实现

###### 2.2.1 构造方法

- `FileOutputstream(File file);`
- 创建一个向指定File对象表示的文件中写入数据的文件输出流。
	- 如果该文件存在，但它是一个目录，而不是一个常规文件；或者该文件不存在，但无法创建它；抑或因为其他某些原因而无法打开，则抛出FilelNotFoundException。
	
- `FileOutputStream(String name);`
- 创建一个向具有指定名称的文件中写入数据的输出文件流。
	- 如果该文件存在，但它是一个目录，而不是一个常规文件；或者该文件不存在，但无法创建它；抑或因为其他某些原因而无法打开它，则抛出FilelNotFoundException。

###### 2.2.2 成员方法

- `public abstract void write(int b)`
- 把一个字节b斜此文件输出流中
	
- `public void write(byte[] b)`
- 将b.length个字节从指定byte数组写入此文件输出流中。
	- 一般我们直接输入字符串,但是成员方法并没有直接将字符串转为字节流的方法,一般使用String的getBytes()方法将字符串转为byte数组
	
- `public void write(byte[] b, int off,int len)`
- 将byte数组从off索引开始,长度为len的字节写入文件输出流中
	
- `public void close()`
- 关闭此输出流并释放与此流有关的所有系统资源。close的常规协定是：该方法将关闭输出流。关闭的流不能执行输出操作，也不能重新打开。
	- 为什么一定要close（）呢？
	A：让流对象变成垃圾，这样就可以被垃圾回收器回收了
B：通知系统去释放跟该文件相关的资源

其他问题

- 如何实现写入数据的换行

	- 将换行字符串\n转换为字节数组写入到需要换行处

	  >   Windows的换行符是 \r\n
	  >   Linux的换行符是 \n
	  >   Mac的换行符是 \r
	
- 如何实现数据的追加

	- 使用public FileOutputStream(File file, boolean append)或者public FileOutputStream(String name, boolean append)构造方法
	- 如果第二个参数为true，则将字节写入文件末尾处，而不是写入文件开始处。

### 转换流

字节流读取字符类，可能会因为编码问题出现乱码，转换流即是给字节流指定编码，让其按照指定的编码集去读取字符。

##### InputStreamReader

字符流读入数据

###### 构造方法

- `InputStreamReader(InputStream is);`     用默认的编码读取数据
- `InputStreamReader(InputStream is,String charsetName);`   用指定的编码读取数据

###### 成员方法

- `public int read();`    一次读取一个字符
- `public int read(char[] chs);`    一次读取一个字符数组
- `public int read(char[] cbuf, int offset,int length);`    一次读取字符数组的一部分

##### OutputStreamWriter

字符流输出数据

###### 构造方法

- `public OutputStreamWriter(OutputStream out);`
    - 创建使用默认字符编码的OutputStreamWriter。
    - 根据自己的系统和语言设定

- `public OutputStreamwriter(OutputStream out,String charsetName)`
    - 创建使用指定字符集的OutputStreamWriter.
    - charsetName---受支持 charset的名称(如"GBK"  "UTF-8"等)

###### 成员方法

- `public void write(int c);`   写入一个字符
- `public void write(char[] cbuf);`    写入一个字符数组
- `public void write(char[] cbuf,int off,int len);`    写入一个字符数组从off开始,len长度的部分
- `public void write(String str);` 写一个字符串
- `public void write(String str,int off,int len); `写一个字符串的一部分

### 字符流

字符流 = 字节流 + 编码集，==字符流是转换流的便捷类==

`new FileReader(file)` 等同于`new InputStreamReader(new FileInputStream(file, true))`

`new FileWeader(file)` 等同于`new OutputStreamWriter(new FileOutputStream(file, true))`

##### FileReader

用来读取字符文件的便捷类。使用默认编码和默认大小缓冲区

###### 构造方法

- `public FileReader(String fileName);`    在给定从中读取数据的文件名的情况下创建一个新FileReader.
- `public FileReader(File file);`    在给定从中读取数据的File的情况下创建一个新FileReader.

###### 成员方法

与其他类似

##### FileWriter

###### 构造方法

- `public FileWriter(File file);`    根据给定的File对象构造一个FileWriter对象.
- `public FileWriter(File file,boolean append);`     根据给定的File对象构造一个可追加字节FileWriter对象.
- `public FileWriter(String filelame);`    根据给定的文件名构造一个FileWriter对象.
- `public FileWriter(File file, boolean append)` 追加方式

###### 成员方法

与其他类似

### 缓冲流

缓冲流,也叫高效流，是对4个`FileXxx` 流的“增强流”。

**缓冲流的基本原理**：

>   1.  使用了底层流对象从具体设备上获取数据，并将数据存储到缓冲区的数组内。
>   2.  通过缓冲区的read()方法从缓冲区获取具体的字符数据，这样就提高了效率。
>   3.  如果用read方法读取字符数据，并存储到另一个容器中，直到读取到了换行符时，将另一个容器临时存储的数据转成字符串返回，就形成了readLine()功能。

在创建流对象时，会创建一个内置的默认大小的缓冲区数组，通过缓冲区读写，减少系统IO次数，从而提高读写的效率。

缓冲书写格式为`BufferedXxx`，按照数据类型分类：

-   **字节缓冲流**：`BufferedInputStream`，`BufferedOutputStream`
-   **字符缓冲流**：`BufferedReader`，`BufferedWriter`

##### 字节缓冲区流

用于对字节流进行高效缓冲

- `public BufferedInputStream(InputStream in)`
- 创建一个BufferedInputStream并保存其参数,即输入流in,以便将来使用.
	
- `public BufferedOutputStream(OutputStream out)`
- 创建一个BufferedOutputStream并保存其参数,即输入流out,以便将来使用.

##### 字符缓冲区流

- `public BufferedReader(Reader in);`    创建一个使用默认大小输入缓冲区的缓冲字符输入流.
- `public String readLine();` 读取一行文本(不包括换行符)
	
- `public BufferedWriter(Writer out);`    创建一个使用默认大小输出缓冲区的缓冲字符输出流.
- `public void newLine();`    根据系统写入一个行分隔符.

### 内存操作流

上面的FileXxx是对文件进行io操作，而java中也可以直接对内存进行io操作

将内存中的数组等转换为流或将流写入到内存的数组中等

##### 字节数组

- ByteArraylnputStream

  ByteArrayInput Strean 包含一个内部缓冲区，该缓冲区包含从流中读取的字节。内部计数器跟踪 read 万法要提供的下一个字节。
  关闭ByteArrayImputStrean无效。此类中的方法在关闭此流后仍可被调用，而不会产生任何IOException。

	- `public ByteArrayInputStream(byte[] buf);`    构造
	- `public int read()`
	- `public int read(byte[]b, int off,int len)`

- ByteArrayOutputStream

  此类实现了一个输出流,其中的数据被写入一个byte数组.缓冲区会随着数据的不断写入而自动增长.可使用toByteArray()和 tostring()获取数据.
  关闭ByteArrayoutputStream 无效。此类中的方法在关闭此流后仍可调用，而不会产生任何IOException。

	- `public ByteArrayOutputStream();`    创建一个新的byte数组输出流。缓冲区的容量32字节，如有必要可增加其大小。也可自定义大小
	- `public void write(int b)`
	- `public void write(byte[]b, int off,int len)`
	- `public byte[] toByteArray();`    转换为byte数组,

##### 字符数组

- CharArrayReader

	- `public CharArrayReader(char[] buf);`    构造
	- `public int read()` 读取单个字符.
	- `public int read(char[] b,int off,int len)` 将字符读入数组的某一部分.

- CharArrayWrite

	- `public CharArrayWriter()`;    创建一个新的CharArrayWriter.
	- `public void write(int c)` 将一个字符写入缓冲区.
	- `public void write(char[]c,int off,int len)`将字符写入缓冲区.
	- `public void write(String str,int off,int len)` 将字符串的某一部分写入缓冲区.
	- `public char[] toCharArray()` 复制输入数据所得到的char数组.

##### 字符串

- StringReader

	- `public StringReader(String s);`    创建一个新字符串reader.
	- `public int read()`读取单个字符.
	- `public int read(char[]cbuf,int off,int 1en);`    将字符读入数组的某一部分.

- StringWriter

	- `public StringWriter()`   使用默认初始字符串缓冲区大小创建一个新字符串 writer.
	- `public void write(int c)`写入单个字符.
	- `public void write(char[]cbuf,int off,int 1en)`写入字符数组的某一部分.
	- `public void write(String str)`写入一个字符串.
	- `public void write(String str,int off,int 1en)`写入字符串的某一部分.
	- `public StringBuffer getBuffer()`返回该字符串缓冲区本身.
	- `public String toString()`以字符串的形式返回该缓冲区的当前值.

### 数据流

用于输入输出基本数据类型的流，byte  short  int  long  float  double  boolean  char

##### DataInputStream(数据输入流)

- `public DataInputStream(InputStream in);`    构造方法

- `public final int readInt();`    读入一个int类型数据到基础输入流

    ​	`readChar(int i)`	写入int类型
    ​	`readBoolean(boolean v)`
    ​	`readByte(int v)`
    ​	`readBytes(String s)`
    ​	`readLong(long v)`
    ​	`readChars(String s)`
    ​	`read(byte[] b,int off,int len)`

- close( )方法

##### DataOutputStream(数据输出流)

- `public Data0utputStream(outputStream out);`    构造方法
- `public final void writeChar(int v);`    写入一个char类型到基础输出流

  ​	`writeInt(int i)`	写入int类型
  ​	`writeBoolean(boolean v)`
  ​	`writeByte(int v)`
  ​	`writeBytes(String s)`
  ​	`writeLong(long v)`
  ​	`writeChars(String s)`
  ​	`write(byte[] b,int off,int len)`

- close( )方法

### 打印流

我们在控制台打印输出，是调用`print`方法和`println`方法完成的,他们就来自`java.io.PrintStream`

>   只有写数据的，没有读取数据。只能操作目的地，不能操作数据源。
>
>   可以操作任意类型的数据。
>
>   如果启动了自动刷新，能够自动刷新。
>
>   该流是可以直接操作文本文件的。

##### 字节打印流

*   PrintStream

##### 字符打印流

- printWriter

### RondomAcessFile

随机访问文件流

RandomAccessFile类不属于流，是Object类的子类。但它融合了InputStream和OutputStream的功能。支持对随机访问文件的读取和写入。

##### 构造

>   mode的选择
>
>   “r”以只读方式打开。调用结果对象的任何write 方法都将导致抛出IOException。
>
>   "rw"打开以便读取和写入。如果该文件尚不存在，则尝试创建该文件。(一般用这个)
>
>   “rws"打开以便读取和写入，对于“rw"，还要求对文件的内容或元数据的每个更新都同步写入到底层存储设备。
>
>   “rwd”打开以便读取和写入，对于“rw”，还要求对文件内容的每个更新都同步写入到底层存储设备。

- `public RandomAccessFile(File file, String mode)`   创建从指定文件读写的随机文件访问流)
- `public RandomAccessFile(String name, String mode)`    创建从指定路径文件读写的随机访问文件流

##### 读写

- write可写入各种类型
- read()    一次读取一个字节或字节数组

##### 指针

- seek()  设置偏移量,从这里读写
- getFilepointer()    获取当前偏移量

### 合并流

SequencelnputStream合并流，将两个流合并

SequenceInputStream 表示其他输入流的逻辑串联。它从输入流的有序集合开始，并从第一个输入流开始读取，直到到达文件末尾，接着从第二个输入流读取，依次类推，直到到达包含的最后一个输入流的文件末尾为止。

##### 构造

- SequenceInputStream(InputStream sl,Inputstream s2)
- SequenceInputStream(Enumeration e)

### 序列化流

Java 提供了一种对象**序列化**的机制。用一个字节序列可以表示一个对象，该字节序列包含该`对象的数据`、`对象的类型`和`对象中存储的属性`等信息。字节序列写出到文件之后，相当于文件中**持久保存**了一个对象的信息。

将对象保存为一个文件，成为序列化，将其从文件重新读取为一个对象，成为反序列化。

```mermaid
graph LR
A[字节] -->|ObjectInputStream反序列化| B[对象]
B -->|ObjectOutputStream序列化| A
```

该类要实现可序列化接口`java.io.Serializable`，增加一个任意的固定的序列化id值`private static final long serialVersionUID = 24952475619L`。类中所有属性必须可序列化，无需序列化的属性使用``transient`修饰。

```java
public class Employee implements java.io.Serializable {
   
    private static final long serialVersionUID = 1L;
    
    public String name;
    public String address;
    public int eid; 

    public void addressCheck() {
        System.out.println("Address  check : " + name + " -- " + address);
    }
}
```

##### ObjectOutputStream

序列化

`public ObjectOutputStream(OutputStream out)`

`public final void writeObject (Object obj)`

##### ObjectInputStream

反序列化

`public ObjectInputStream(IntputStream in)`

`public final void readObject (Object obj)`

### Properties

Hashtable的子类,Map中的方法都可以用，该集合没有泛型。

它是一个可以持久化的属性集。可以从流中加载或存储。键值对可以存储到集合中，也可以存储到持久化的设备上。

##### 特有方法

因为Properties继承于Hashtable，所以可对Properties对象应用put和putAll方法。但不建议使用这两个方法，因为它们允许调用者插入其键或值不是String的项。相反，应该使用setProperty方法。

- `public Obiect setProperty(String key, String value)`

  调用Hashtable的方法put。使用getProperty 方法提供并行性。强制要求为属性的键和值使用字符串。返回值是Hashtable调用put的结果。

- `public String getProperty(String key)`    通过键找值
- `public Set<String> stringPropertyNames()`    返回键的Set集合

##### 与IO流的结合使用

- `public void load(Reader reader):`把中的数据读取到集合中

```java
Properties prop = new Properties();

//注意:这个文件的数据必须是键值对形式
Reader r = new FileReader("prop.txt");
prop.load(r);
r.close());
```

- `public void store(Writer writer,String comments)` 把集合中的数据存储到文件，comments表示对文件进行描述的字符串

  ```java
  Properties prop = new Properties();
  prop.setproperty("张三","27");
  prop.setProperty("李四","30");
  prop.setProperty("王五","18");
  Writer w=new FileWriter("name.txt");
  prop.store(w,"helloworld");
  w.close();
  ```
  
  





