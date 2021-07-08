```shell
# 当前数据库
db
# 若有则选择，若没有则创建数据库
use [dbname]
# 暂时所有数据库
show dbs

# 相当于SELECT * FROM collectionname
db.collectionname.find( {} )
#
db.inventory.find( { status: "D" } )
```

```java
// 用于实体类上，将entity转换为mongo，collection类似mysql中的表名
@Document(collection= "xxx")

// 该字段需要建立索引
@Indexed
// 复合索引
@CompoundIndex(name = "index_name", def = "{'name': 1, 'value': -1}")
public class MongoDBDemo {
    String name;
    String value;
}
// 映射到mongo中的别名
@Field("fName") 
private String firstName;
// 不需要映射的字段
@Transient
```



