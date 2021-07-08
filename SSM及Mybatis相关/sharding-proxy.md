server.yaml
```yaml
rules:
 - !AUTHORITY
   users:
    # 代理的用户名和密码，@后面为%或空格，表示允许所有ip访问
     - root@localhost:root
     - sharding@:sharding
   provider:
     type: NATIVE

props:
 max-connections-size-per-query: 1
 executor-size: 16  # Infinite by default.
 proxy-frontend-flush-threshold: 128  # The default value is 128.
   # LOCAL: Proxy will run with LOCAL transaction.
   # XA: Proxy will run with XA transaction.
   # BASE: Proxy will run with B.A.S.E transaction.
 proxy-transaction-type: LOCAL
 xa-transaction-manager-type: Atomikos
 proxy-opentracing-enabled: false
 proxy-hint-enabled: false
 sql-show: false
 check-table-metadata-enabled: false
 lock-wait-timeout-milliseconds: 50000 # The maximum time to wait for a lock
```

```yaml
# 逻辑库名
schemaName: sharding_db

dataSources:
 # 实际的数据库1
 ds_0:
   url: jdbc:mysql://127.0.0.1:3306/demo_ds_0?serverTimezone=UTC&useSSL=false
   username: root
   password:
   connectionTimeoutMilliseconds: 30000
   idleTimeoutMilliseconds: 60000
   maxLifetimeMilliseconds: 1800000
   maxPoolSize: 50
   minPoolSize: 1
   maintenanceIntervalMilliseconds: 30000
 # 实际的数据库2
 ds_1:
   url: jdbc:mysql://127.0.0.1:3306/demo_ds_1?serverTimezone=UTC&useSSL=false
   username: root
   password:
   connectionTimeoutMilliseconds: 30000
   idleTimeoutMilliseconds: 60000
   maxLifetimeMilliseconds: 1800000
   maxPoolSize: 50
   minPoolSize: 1
   maintenanceIntervalMilliseconds: 30000

rules:
- !SHARDING # 分片的配置
 tables:
   # 逻辑表名
   t_order:
     # 对应的实际表有哪几个
     actualDataNodes: ds_${0..1}.t_order_${0..1}
     # 表的分片策略
     tableStrategy:
       standard:
         # 通过哪个字段分表
         shardingColumn: order_id
         # 分表的策略，对应下面shardingAlgorithms中的策略
         shardingAlgorithmName: t_order_inline
     # 主键生成策略
     keyGenerateStrategy:
       column: order_id
       keyGeneratorName: snowflake
   # 逻辑表
   t_order_item:
     actualDataNodes: ds_${0..1}.t_order_item_${0..1}
     tableStrategy:
       standard:
         shardingColumn: order_id
         shardingAlgorithmName: t_order_item_inline
     keyGenerateStrategy:
       column: order_item_id
       keyGeneratorName: snowflake
 bindingTables:
   - t_order,t_order_item
 # 默认的分库策略，如果上面没有为每个表配置分库策略，默认使用的分库策略
 defaultDatabaseStrategy:
   standard:
     shardingColumn: user_id
     shardingAlgorithmName: database_inline
 defaultTableStrategy:··
   none:
 
 # 分片策略
 shardingAlgorithms:
   database_inline:
     # 使用行内表达式方式，也可以自行实现，然后打jar包放到ext-lib下
     type: INLINE
     props:
       # 表示user_id % 2，若结果是0，这保存到ds_0,若结果是1，保存到ds_1
       algorithm-expression: ds_${user_id % 2}
   t_order_inline:
     type: INLINE
     props:
       algorithm-expression: t_order_${order_id % 2}
   t_order_item_inline:
     type: INLINE
     props:
       algorithm-expression: t_order_item_${order_id % 2}
 
 keyGenerators:
   snowflake:
     type: SNOWFLAKE
     props:
       worker-id: 123

```

