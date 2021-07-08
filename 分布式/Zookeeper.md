### 安装

下载zoopeeker，解压到`/usr/local`下

在zookeeper目录下创建data目录

进⼊到 zookeeper 的 `conf` ⽬录，复制 `zoo_sample.cfg` 得到 `zoo.cfg`

将其中的 `dataDir` 修改为上⾯刚创建的 `data` ⽬录

### 命令

```shell
bin/zkServer.sh start # 启动
bin/zkServer.sh status	#查询状态
bin/zkServer.sh stop	# 停止
```

### 集群

单ip多节点

```ini
# zoo.cfg中更改
dataDir=/usr/local/zookeeper/data		# 前面创建的数据目录
dataLogDir=/usr/local/zookeeper/logs
clientPort=2181	# 端口号
# 添加zookeeper集群，每台机器一个不同的Serverid
server.1=192.168.1.1:2888:3888	# 前面为ip，2888为通信端口号，3888为选举端口号
server.2=192.168.1.1:2888:3888
server.3=192.168.1.1:2888:3888
server.4=192.168.1.1:2888:3888
```

>   注意：
>
>   同一IP上搭建多个节点的集群时，必须要注意端口问题，端口必须不一致才行；
>
>   创建多个节点集群时，在dataDir目录下必须创建myid文件，myid文件用于zookeeper验证server序号等，myid文件只有一行，并且为当前server的序号，例如server.1的myid就是1，server2的myid就是2等。

多ip多节点

单ip的客户端端口号不能相同，多ip配置文件可以相同