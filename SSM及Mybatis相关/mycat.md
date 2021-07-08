# MyCat

mycat是一个数据库中间件，可以视为MySQL集群的企业级数据库。用来搭建mysql集群，实现数据的大量存储。

安装mycat到`/usr/local/mycat`下

安装后主要有以下文件夹

*   bin：主要存放可执行文件，linux下的运行 `./mycat console`,首先要 chmod +x * 
    *   mycat 支持的命令{ console | start | stop | restart | status | dump } 

*   conf：目录下存放配置文件，server.xml 是 Mycat 服务器参数调整和用户授权的配置文件，schema.xml 是逻 
    辑库定义和表以及分片定义的配置文件，rule.xml 是分片规则的配置文件，分片规则的具体一些参数信息单独存 
    放为文件，也在这个目录下，配置文件修改，需要重启 Mycat 或者通过 9066 端口 reload. 

*   lib：依赖jar包

*   logs：日志文件

>   注意：Linux 下部署安装 MySQL，默认不忽略表名大小写，需要手动到/etc/my.cnf 下配置 
>   lower_case_table_names=1 使 Linux 环境下 MySQL 忽略表名大小写，否则使用 MyCAT 的时候会提示找不到 
>   表的错误！ 

### 配置文件

安装目录下conf文件夹：

*   schema.xml：定义逻辑库，表，分片节点等内容；
*   rule.xml：定义分片规则;
*   server.xml：定义用户及系统相关变量；

##### ==**schema**==

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">

    <!--
	sechma用于配置逻辑库，与server.xml中的schema对应，server<property name="schemas">mydatabase</property>
	可配置多个逻辑库
	逻辑库的概念和 MYSQL 数据库中 Database 的概念相同，在查询两个不同的逻辑库中表的时候需要切换到该逻辑库下(`use db1`)才可以查询到所需要的表。
	如果不配置 schema 标签，所有的表配置，会属于同一个默认的逻辑库。
	-->
    
    <!-- checkSQLschema这个属性默认就是false，官方文档的意思就是是否去掉表前面的数据库的名称，
    "select * from db1.testtable"，设置为true就会去掉db1。但是如果db1的名称不是schema的名称，那么也不会被去掉，
    因此官方建议不要使用这种语法。同时默认设置为false。-->
    
    <!-- sqlMaxLimit当该值设置为某个数值时。每条执行的 SQL 语句，如果没有加上 limit 语句，MyCat 也会自动的加上所对应的值。
    例如设置值为 100，执行”select * from test_table”,则效果为“selelct * from test_table limit 100”.
    注意:如果运行的 schema 为非拆分库的，那么该属性不会生效。-->
    
    <!--若逻辑表没有进行分表，每个数据节点都是一个逻辑表，可以删除内部的table标签，直接在标签内写入其对应的数据节点即可-->
    <schema name="mycatdb" checkSQLschema="false" sqlMaxLimit="100" >
        
        <!--table标签定义逻辑表-->
    
        <!-- name  定义逻辑表的表名，这个名字就如同我在数据库中执行 create table 命令指定的名字一样，同个 schema 标 签中定义的名字必须唯一。  -->
        <!-- dataNode   该属性定义这个逻辑表所属的 dataNode, 该属性的值需要和 dataNode 标签中 name 属性的值相互对应。 -->
        <!-- rule   该属性用于指定逻辑表要使用的规则名字，规则名字在 rule.xml 中定义，必须与 tableRule 标签中 name 属性属性值一一对应 -->
        <!-- ruleRequired   该属性用于指定表是否绑定分片规则，如果配置为 true，但没有配置具体 rule 的话 ，程序会报错。 -->
        <!-- primaryKey 对应的真实表的主键字段名，自动生成主键时需要设置 -->
        <!-- type   该属性定义了逻辑表的类型，目前逻辑表只有“全局表”和”普通表”两种类型。全局表定义type=”global”，不定义的就是普通表。 -->
        <!-- autoIncrement  主键是否自增长。 -->
        <!-- subTables  分表，分表目前不支持Join。 -->
        <!-- needAddLimit是否自动添加limit，默认是开启状态。关闭请谨慎。 -->
        <!--rule="mod-long"  指定规则在rule中配置-->
        <table name="users" primaryKey="id" dataNode="dn1"/>
        <table name="orders" primaryKey="id" dataNode="dn1"/>
   
        <table name="product" primaryKey="id" dataNode="dn2"/>
        <table name="news" primaryKey="id" dataNode="dn2"/>
        
    </schema>
    
    <!--dataNode标签定义了 MyCat 中的数据节点，也就是我们通常说所的数据分片。一个 dataNode 标签就是 
一个独立的数据分片-->
    <!--  Name  定义数据节点的名字，这个名字需要是唯一的，我们需要在 table 标签上应用这个名字，来建立表与分片对 应的关系。 
-->
    <!-- dataHost   该属性用于定义该分片属于哪个数据库实例，属性值是引用 dataHost 标签上定义的 name 属性。  -->
    <!-- Database   该属性用于定义该分片属性哪个具体数据库实例上的具体库 -->
    <!--下面表示，在实际的数据库localhost1上，有db1和db2两个需要使用mycat的数据库，localhost2上有db3数据库-->
    <dataNode name="dn1" dataHost="localhost1" database="db1" />
    <dataNode name="dn2" dataHost="localhost1" database="db2" />
    <dataNode name="dn3" dataHost="localhost2" database="db3" />
    

    <!--dataHost标签在 mycat 逻辑库中作为最底层的标签存在，直接定义了具 
体的数据库实例、读写分离配置和心跳语句。-->
    <!--  name  唯一标识 dataHost 标签，供上层的标签使用-->
    <!-- maxCon 指定每个读写实例连接池的最大连接。 -->
    <!-- minCon 指定每个读写实例连接池的最小连接，初始化连接池的大小。 -->
    <!-- balance    负载均衡类型，目前的取值有4 种： 
    	“0”, 不开启读写分离机制，所有读操作都发送到当前可用的 writeHost 上。 
    	“1”，全部的 readHost 与 stand by writeHost（非主非从） 参与 select 语句的负载均衡，简单的说，当双主双从模式(M1->S1，M2->S2，并且 M1 与 M2 互为主备)，正常情况下，M2,S1,S2 都参与 select 语句的负载均衡。
    	“2”，所有读操作都随机的在 writeHost、readhost 上分发。
    	  "3”，所有读请求随机的分发到 wiriterHost 对应的 readhost 执行，writerHost 不负担读写压力。
     -->
    <!--writeType 负载均衡类型
	1. writeType=”0”, 所有写操作发送到配置的第一个 writeHost，第一个挂了切到还生存的第二个writeHost，重新启动后已切换后的为准，切换记录在配置文件中:dnindex.properties
	2. writeType=”1”，所有写操作都随机的发送到配置的 writeHost，1.5 以后废弃不推荐。默认0就好了！
    -->
    <!-- dbType 指定后端连接的数据库类型，目前支持二进制的 mysql 协议，还有其他使用 JDBC 连接的数据库。例如：mongodb、oracle、spark 等. 
    -->
    <!-- dbDriver   指定连接后端数据库使用的 Driver，目前可选的值有 native 和 JDBC。使用 native 的话，因为这个值执行的是二进制的 mysql 协议，所以可以使用 mysql 和 mariadb。其他类型的数据库则需要使用 JDBC 驱动来支持。 
    -->
    <!-- switchType 切换方式 
	"-1" 表示不自动切换； 
	"1"  默认值，自动切换；
	"2" 基于 MySQL 主从同步的状态决定是否切换
		心跳语句为 show slave status； 
	"3"  基于 MySQL galary cluster 的切换机制（适合集群）1.4.1版本
		心跳语句为 show status like ‘wsrep%’.     
    -->
    <!--slaveThreshold:指定从节点的最大个数-->
  
    <dataHost name="localhost1" 
              maxCon="1000" 
              minCon="10" 
              balance="0"
              writeType="0" 
              dbType="mysql" 
              dbDriver="native" 
              switchType="1"  
              slaveThreshold="100">
        <heartbeat>select user()</heartbeat>
        
        <!--writeHost和readHost    
		用于标识不同实例，一般 writeHost 我们使用*M1，readHost 我们用*S1。-->
        <!--url 后端实例连接地址。如果是使用 native 的 dbDriver，则一般为 address:port 这种形式。用 JDBC 或其他的 
dbDriver，则需要特殊指定。当使用 JDBC 时则可以这么写：jdbc:mysql://localhost:3306/。 -->
        <!--password    后端存储实例需要的密码-->
        <!--user    后端存储实例需要的用户名字-->
        <!--weight  权重 配置在 readhost 中作为读节点的权重-->
        <!--usingDecrypt    是否对密码加密，默认0。具体加密方法看官方文档。-->
        <writeHost host="hostM1" url="47.94.158.155:3306" user="root"
                   password="TJXtjx_19991007">
        </writeHost>
    </dataHost>
    
    
    <dataHost name="localhost2" 
              maxCon="1000" 
              minCon="10" 
              balance="0"
              writeType="0" 
              dbType="mysql" 
              dbDriver="native" 
              switchType="1"  
              slaveThreshold="100">
        <heartbeat>select user()</heartbeat>
        <writeHost host="hostM2" url="47.94.158.155:3306" user="root"
                   password="TJXtjx_19991007">
        </writeHost>
    </dataHost>

</mycat:schema>
```

![image-20200715144553980](E:%5CJava%5C%E8%B5%84%E6%96%99%5C%E8%B5%84%E6%96%99%5Cmycat.assets%5Cimage-20200715144553980.png)

##### ==**rule**==

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mycat:rule SYSTEM "rule.dtd">
<mycat:rule xmlns:mycat="http://io.mycat/">
    
    <!--rule.xml 里面就定义了我们对表进行拆分所涉及到的规则定义。-->
    
    <!--定义规则-->
    <!--
	name 属性指定唯一的名字，用于标识不同的表规则。 
	内嵌的 rule 标签指定对物理表中的哪一列进行拆分和使用什么路由算法。 
	columns 指定要拆分的列名字。 
	algorithm 使用 function 标签中的 name 属性。连接表规则和具体路由算法。当然，多个表规则可以连接到同一个路由算法上。table 标签内使用。让逻辑表使用这个规则进行分片。 
	-->
    <tableRule name="rule1">
        <rule>
            <columns>id</columns>
            <algorithm>func1</algorithm>
        </rule>
    </tableRule>

    <tableRule name="rule2">
        <rule>
            <columns>user_id</columns>
            <algorithm>func1</algorithm>
        </rule>
    </tableRule>

    <tableRule name="sharding-by-intfile">
        <rule>
            <columns>sharding_id</columns>
            <algorithm>hash-int</algorithm>
        </rule>
    </tableRule>
    <tableRule name="auto-sharding-long">
        <rule>
            <columns>id</columns>
            <algorithm>rang-long</algorithm>
        </rule>
    </tableRule>
    <tableRule name="mod-long">
        <rule>
            <columns>id</columns>
            <algorithm>mod-long</algorithm>
        </rule>
    </tableRule>
    <tableRule name="sharding-by-murmur">
        <rule>
            <columns>id</columns>
            <algorithm>murmur</algorithm>
        </rule>
    </tableRule>
    <tableRule name="crc32slot">
        <rule>
            <columns>id</columns>
            <algorithm>crc32slot</algorithm>
        </rule>
    </tableRule>
    <tableRule name="sharding-by-month">
        <rule>
            <columns>create_time</columns>
            <algorithm>partbymonth</algorithm>
        </rule>
    </tableRule>
    <tableRule name="latest-month-calldate">
        <rule>
            <columns>calldate</columns>
            <algorithm>latestMonth</algorithm>
        </rule>
    </tableRule>
    
    <tableRule name="auto-sharding-rang-mod">
        <rule>
            <columns>id</columns>
            <algorithm>rang-mod</algorithm>
        </rule>
    </tableRule>
    
    <tableRule name="jch">
        <rule>
            <columns>id</columns>
            <algorithm>jump-consistent-hash</algorithm>
        </rule>
    </tableRule>

    <function name="murmur"
        class="io.mycat.route.function.PartitionByMurmurHash">
        <property name="seed">0</property><!-- 默认是0 -->
        <property name="count">2</property><!-- 要分片的数据库节点数量，必须指定，否则没法分片 -->
        <property name="virtualBucketTimes">160</property><!-- 一个实际的数据库节点被映射为这么多虚拟节点，默认是160倍，也就是虚拟节点数是物理节点数的160倍 -->
        <!-- <property name="weightMapFile">weightMapFile</property> 节点的权重，没有指定权重的节点默认是1。以properties文件的格式填写，以从0开始到count-1的整数值也就是节点索引为key，以节点权重值为值。所有权重值必须是正整数，否则以1代替 -->
        <!-- <property name="bucketMapPath">/etc/mycat/bucketMapPath</property> 
            用于测试时观察各物理节点与虚拟节点的分布情况，如果指定了这个属性，会把虚拟节点的murmur hash值与物理节点的映射按行输出到这个文件，没有默认值，如果不指定，就不会输出任何东西 -->
    </function>

    <function name="crc32slot"
              class="io.mycat.route.function.PartitionByCRC32PreSlot">
    </function>
    
    <function name="hash-int"
        class="io.mycat.route.function.PartitionByFileMap">
        <property name="mapFile">partition-hash-int.txt</property>
    </function>
    <function name="rang-long"
        class="io.mycat.route.function.AutoPartitionByLong">
        <property name="mapFile">autopartition-long.txt</property>
    </function>
    <function name="mod-long" class="io.mycat.route.function.PartitionByMod">
        <!-- how many data nodes -->
        <property name="count">3</property>
    </function>

    <function name="func1" class="io.mycat.route.function.PartitionByLong">
        <property name="partitionCount">8</property>
        <property name="partitionLength">128</property>
    </function>
    <function name="latestMonth"
        class="io.mycat.route.function.LatestMonthPartion">
        <property name="splitOneDay">24</property>
    </function>
    <function name="partbymonth"
        class="io.mycat.route.function.PartitionByMonth">
        <property name="dateFormat">yyyy-MM-dd</property>
        <property name="sBeginDate">2015-01-01</property>
    </function>
    
    <function name="rang-mod" class="io.mycat.route.function.PartitionByRangeMod">
            <property name="mapFile">partition-range-mod.txt</property>
    </function>
    
    <function name="jump-consistent-hash" class="io.mycat.route.function.PartitionByJumpConsistentHash">
        <property name="totalBuckets">3</property>
    </function>
</mycat:rule>
```

##### ==**server**==

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mycat:server SYSTEM "server.dtd">
<mycat:server xmlns:mycat="http://io.mycat/">
    <!--server.xml 几乎保存了所有 mycat 需要的系统配置信息。其在代码内直接的映射类为 SystemConfig 类。 -->
    <system>
        <!-- 0为需要密码登陆、1为不需要密码登陆 ,默认为0，设置为1则需要指定默认账户-->
        <property name="nonePasswordLogin">0</property> 
        <property name="useHandshakeV10">1</property>
        <!-- 1为开启实时统计、0为关闭 -->
        <property name="useSqlStat">0</property> 
        <!-- 1为开启全加班一致性检测、0为关闭 -->
        <property name="useGlobleTableCheck">0</property>  
        <!--全局id声称方式  0 本地文件方式  1 数据库方式  2 时间戳方式-->
        <property name="sequnceHandlerType">2</property> 
        <!-- 子查询中存在关联查询的情况下,检查关联字段中是否有分片字段 .默认 false -->
        <property name="subqueryRelationshipCheck">false</property> 
        <!--  <property name="useCompression">1</property>--> <!--1为开启mysql压缩协议-->
        <!--模拟的 mysql 版本号，默认值为 5.6 版本，如非特需，不要修改这个值-->
        <property name="fakeMySQLVersion">5.6.20</property>-->
        <!-- <property name="processorBufferChunk">40960</property> -->

        <!--指定系统可用的线程数，默认值为机器 CPU 核心线程数。 -->
        <property name="processors">8</property>  
        <property name="processorExecutor">32</property> 

        <!--默认为type 0: DirectByteBufferPool | type 1 ByteBufferArena | type 2 NettyBufferPool -->
        <property name="processorBufferPoolType">0</property>
        <!--默认是65535 64K 用于sql解析时最大文本长度 -->
        <!--<property name="maxStringLiteralLength">65535</property>-->
        <!--<property name="backSocketNoDelay">1</property>-->
        <!--<property name="frontSocketNoDelay">1</property>-->
        <!--<property name="processorExecutor">16</property>-->

        <!--使用端口-->
        <property name="serverPort">8066</property> 
        <!--管理端口-->
        <property name="managerPort">9066</property> 
        <property name="idleTimeout">300000</property> 
        <!-- mycat 服务监听的 IP 地址，默认值为 0.0.0.0。 -->
        <property name="bindIp">0.0.0.0</property> 
        <property name="frontWriteQueueSize">4096</property> 
        <property name="processors">32</property> -->

        <!--分布式事务开关，0为不过滤分布式事务，1为过滤分布式事务（如果分布式事务内只涉及全局表，则不过滤），2为不过滤分布式事务,但是记录分布式事务日志-->
        <property name="handleDistributedTransactions">0</property>

        <!--off heap for merge/order/group/limit      1开启   0关闭  -->
        <property name="useOffHeapForMerge">1</property>

        <!--单位为m-->
        <property name="memoryPageSize">64k</property>

        <!--单位为k -->
        <property name="spillsFileBufferSize">1k</property>

        <property name="useStreamOutput">0</property>

        <!--单位为m -->
        <property name="systemReserveMemorySize">384m</property>


        <!--是否采用zookeeper协调切换  -->
        <property name="useZKSwitch">false</property>

        <!-- XA Recovery Log日志路径 -->
        <!--<property name="XARecoveryLogBaseDir">./</property>-->

        <!-- XA Recovery Log日志名称 -->
        <!--<property name="XARecoveryLogBaseName">tmlog</property>-->
        <!--如果为 true的话 严格遵守隔离级别,不会在仅仅只有select语句的时候在事务中切换连接-->
        <property name="strictTxIsolation">false</property>

        <property name="useZKSwitch">true</property>

    </system>

    <!-- ===================全局SQL防火墙设置========================= -->
    <!--白名单可以使用通配符%或着*-->
    <!--例如<host host="127.0.0.*" user="root"/>-->
    <!--例如<host host="127.0.*" user="root"/>-->
    <!--例如<host host="127.*" user="root"/>-->
    <!--例如<host host="1*7.*" user="root"/>-->
    <!--这些配置情况下对于127.0.0.1都能以root账户登录-->

    <firewall>
        <whitehost>
            <!--ip 白名单  用户对应的可以访问的 ip 地址 -->
            <host host="1*7.0.0.*" user="root"/>
        </whitehost>
        <blacklist check="false">
        </blacklist>
    </firewall>


    <!--#############################主要配置这里##################################-->

    <!--标签主要用于定义登录 mycat 的用户和权限。-->
    <!--登录的用户名，连接mycat的用户名-->
    <user name="mycat" defaultAccount="true">
        <!--连接mycat的密码-->
        <property name="password">123456</property>
        <!---该用户可访问的数据库名，这里会和schema.xml中的配置关联，多个用逗号分开，例如需要这个用户需要管理两个数据库db1,db2，则配置db1,dbs-->
        <property name="schemas">mycatdb</property>
        <!--是否只读-->
        <property name="readOnly">true</property> 
        <!--上面定 义了一个用户，用户名为 mycat、密码123456，可访问的 schema 只有 mycatdb 一个。 -->

        <!-- 
  privileges对用户的 schema 及 下级的 table 进行精细化的 DML 权限 
  dml 	insert,update,select,delete 	0000 
 -->
        <!--check 是否开启 DML 权限检查， 默认 false 标识不检查，当然 privileges 节点不配置，等同 check=false-->
        <privileges check="false">
            <schema name="TESTDB" dml="0110" >
                <table name="tb01" dml="0000"></table>
                <table name="tb02" dml="1111"></table>
            </schema>

            <schema name="TESTDB1" dml="0110" >
                <table name="tb01" dml="0000"></table>
                <table name="tb02" dml="1111"></table>
            </schema>
        </privileges>       

    </user>

    <user name="user">
        <property name="password">user</property>
        <property name="schemas">mycatdb</property>
        <!--是否只读-->
        <property name="readOnly">true</property>
    </user>

</mycat:server>
```