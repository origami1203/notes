/与springboot的集成

引入依赖

```xml
```

配置客户端

```java
@Configuration
public class RestClientConfig extends AbstractElasticsearchConfiguration {

    @Override
    @Bean
    public RestHighLevelClient elasticsearchClient() {

        // 可配置ip，ssl，超时时间、认证、header等
        final ClientConfiguration clientConfiguration = ClientConfiguration.builder()  
            .connectedTo("localhost:9200")
            .build();

        return RestClients.create(clientConfiguration).rest();                         
    }
}
```

对象映射

* `@Document`:用在类上，类似mysql的table
  * `indexName`: 索引名，类似表名.可以使用 SpEL 模版表达式 `"log-#{T(java.time.LocalDate).now().toString()}"`
  * `type`: 已过时
  * `createIndex`: 自动创建索引，默认为true

* `@Id`:标注到主键字段上

* `@Field`:字段上，类似@Column注解
  * `name`: 映射到es中的字段名，不写则默认为字段名
  * `type`:字段类型，可以是 *Text, Keyword, Long, Integer, Short, Byte, Double, Float, Half_Float, Scaled_Float, Date, Date_Nanos, Boolean, Binary, Integer_Range, Float_Range, Long_Range, Double_Range, Date_Range, Ip_Range, Object, Nested, Ip, TokenCount, Percolator, Flattened, Search_As_You_Type*. 详情查看 [Elasticsearch Mapping Types](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)
  * `format`: 内置的时间格式, see the next section [Date format mapping](https://docs.spring.io/spring-data/elasticsearch/docs/4.2.3/reference/html/#elasticsearch.mapping.meta-model.date-formats).
  * `pattern`: 自定义时间格式, see the next section [Date format mapping](https://docs.spring.io/spring-data/elasticsearch/docs/4.2.3/reference/html/#elasticsearch.mapping.meta-model.date-formats).
  * `store`: 是否单独存储原始字段，默认 *false*.
  * `analyzer`, `searchAnalyzer`, `normalizer` for specifying custom analyzers and normalizer.
* `@Transient`:是否序列化到es中，使用该注解，被标注注解不会被保存到es中

> es7中，自定义时间格式, 需要使用 *uuuu* 代替 *yyyy*.

自定义类型转换

```java
@Configuration
public class Config extends AbstractElasticsearchConfiguration {

  @Override
  public RestHighLevelClient elasticsearchClient() {
    return RestClients.create(ClientConfiguration.create("localhost:9200")).rest();
  }

  @Bean
  @Override
  public ElasticsearchCustomConversions elasticsearchCustomConversions() {
    return new ElasticsearchCustomConversions(
      Arrays.asList(new AddressToMap(), new MapToAddress()));       
  }

  @WritingConverter                                                 
  static class AddressToMap implements Converter<Address, Map<String, Object>> {

    @Override
    public Map<String, Object> convert(Address source) {

      LinkedHashMap<String, Object> target = new LinkedHashMap<>();
      target.put("ciudad", source.getCity());
      // ...

      return target;
    }
  }

  @ReadingConverter                                                 
  static class MapToAddress implements Converter<Map<String, Object>, Address> {

    @Override
    public Address convert(Map<String, Object> source) {

      // ...
      return address;
    }
  }
}
```

### ElasticsearchRestTemplate

无需特别配置，只要配置了`RestHighLevelClient`即可使用。

### 查询策略

条件组合查询(`CriteriaQuery`)

```java
Criteria criteria = new Criteria("price").greaterThan(42.0).lessThan(34.0L);
Query query = new CriteriaQuery(criteria);
```

字符串查询(`StringQuery`)，json格式

```java
Query query = new SearchQuery("{ \"match\": { \"firstname\": { \"query\": \"Jack\" } } } ");
SearchHits<Person> searchHits = operations.search(query, Person.class);
```

复杂查询(`NativeSearchQuery`)

```java
Query query = new NativeSearchQueryBuilder()
    .addAggregation(terms("lastnames").field("lastname").size(10)) //
    .withQuery(QueryBuilders.matchQuery("firstname", firstName))
    .build();

SearchHits<Person> searchHits = operations.search(query, Person.class);
```

通过方法名查询(与spring data jpa一致)

```java
interface BookRepository extends Repository<Book, String> {
  List<Book> findByNameAndPrice(String name, Integer price);
}
```

返回值类型

- `List<T>`
- `Stream<T>`
- `SearchHits<T>`
- `List<SearchHit<T>>`
- `Stream<SearchHit<T>>`
- `SearchPage<T>`

mapping：类似数据库中schema,规定数据的类型

### es常用命令

```json
// 集群健康状态
GET /_cat/health?v

// 集群所有节点
GET /_cat/nodes

// 集群所有索引
GET /_cat/indices

// 创建索引，幂等，无法使用post
// 创建名为customer的索引
PUT /customer?pretty

// 显示索引的信息
GET /consumer

// 查询指定id的doc
// 查询索引为customer,id为1的doc
GET /customer/_doc/1?pretty

// 查询bank索引下所有文档，默认返回10条
GET /customer/_search

// 创建doc，不指定id，非幂等，无法使用put
// 使用post方法，不指定id，默认生成一个id，再次发送会创建新的数据，/_doc可以改为/_create
POST /customer/_doc?pretty
{
  "name": "Jane"，
  "age": 18
}

// 创建doc，指定id，幂等
// 再次发送不会创建新的数据，相当于更新，与使用PUT结果一致
POST /customer/_doc/1?pretty
{
  "name": "lily",
  "age": 15
}

// 创建索引下文档，索引不存在会自动创建索引，此时与post一致
// 创建索引为customer,id为1，值为下面json的doc
PUT /customer/_doc/1?pretty
{
  "name": "John",
  "age": 18
}

// 全量更新指定id数据，原doc被新的直接替换，幂等
// id已被创建，则更新索引，不存在则创建
PUT /customer/_doc/1?pretty
{
	"name": "Tom",
	"age": 18
}

// 局部更新文档
// 更新customer索引下id为1的doc，更改名字为指定，age仍为原来的
POST /customer/_update/1/?pretty
{
    "doc": { 
        "name": "Bob"
    }
}

// 更新文档
// 更新customer索引下id为1的doc，更改名字为指定,并添加age字段
POST /customer/_update/1?pretty
{
	"doc": { 
        "name": "Tom",
        "age": 20 
    }
}

// 删除索引
// 删除customer索引
DELETE /customer?pretty

// 删除指定id的doc文档
DELETE /customer/_doc/1?pretty
```

复杂查询

```json
// 查询bank索引下所有文档，默认返回10条，此时后面的json体可以省略
GET /bank/_search
{
  "query": { "match_all": {} }
}

// 查询name 包含nfc的，模糊查询
GET /product/_search
{
    "query": {
        // 模糊查询
        "match": {
            "name": "nfc"
        }
    }
}

// 查询指定条数size
GET /bank/_search
{
  "query": { "match_all": {} },
   // 指定数量
  "size": 1
}

// 分页，from起始位置，from = (页码-1)*size
GET /bank/_search
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}

// 排序sort，以price 升序
GET /bank/_search
{
    "query": { "match_all": {} },
    // 排序
    "sort": [
        {
            "price": "asc"
        },
        {
            "price": "asc"
        },
    ],
    "from": 10,
    "size": 10
}

// 显示指定字段_source，类似 select fiele1,field2 from xxx
GET /bank/_search
{
    "query": { "match_all": {} },
    "sort": [
        {
            "price": "asc"
        }
    ],
    "from": 10,
    "size": 10,
    // 指定字段
    "_source": ["name","age"]
}

// 范围查询range,支持数值类型和date类型的范围查询
// 20>=age>>16, gt大于，gte大于等于，lt小于，lte小于等于，日期类型的可以设置format

GET /consumer/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 16,
        "lte": 20
      }
    }
  }
}

```

条件查询

```json
//bool关键词，相当于条件查询
// must相当于and，age=18且gender=1
GET /consumer/_search
{
    "query": {
        "bool": {
            "must": [
                // 年龄18
                {
                    "match": {
                        "age": 18
                    }
                },
                // 且gender为1
                {
                    "match": {
                        "gender": 1
                    }
                }
            ]
        }
    }
}
// should 相当于 or
// age=18或者gender=1
GET /consumer/_search
{
    "query": {
        "bool": {
            "should": [
                // 年龄18
                {
                    "match": {
                        "age": 18
                    }
                },
                // 且gender为1
                {
                    "match": {
                        "gender": 1
                    }
                }
            ]
        }
    }
}

// must_not 相当于not
// 年龄18，gender不能为0，bool里可以使用filter
GET /consumer/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "age": 18
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "gender": 0
          }
        }
      ]
    }
  }
}// filter 类似与must，但只过滤数据，不计算_score
```

**filter与query区别**

> filter，仅仅只是按照搜索条件过滤出需要的数据而已，不计算任何相关度分数，对相关度没有任何影响。
> query，会计算每个document相对于搜索条件的相关度，并按照相关度进行排序。
>
> filter，不需要计算相关度分数，不需要按照相关度分数进行排序，同时还有内置的机制，自动cache最常使用filter的数据。
> query，相反，要计算相关度分数，按照分数进行排序，而且无法cache结果。x·

删除

`_delete_by_query`查询后删除

```json
POST /customer/_delete_by_query
{
    "query":{
        "term":{
            "age": 18
        }
    }
}
```
### 数据类型

##### 字符串类型

* `string`:已过期的数据类型，es5之前是字符串类型，之后的版本被替换为`text`和`keyword`。

* `text`:类型的数据会被 分词器分析，用于全文检索字段，如文章的内容等。不用于排序，很少用于聚合。

* `keyword`:类型适用于结构化的字段，不会被分词器分析，如邮箱，手机号等，可用于过滤、筛选、过滤、聚合。

##### 数值类型

|     类型     |           取值范围           |
| :----------: | :--------------------------: |
|     long     |       -2^63^ ~ 2^63^-1       |
|   integer    |       -2^21^ ~ 2^21^-1       |
|    short     |         -32768~32767         |
|     byte     |           -128~127           |
|    double    | 64位的双精度IEEE 754浮点类型 |
|  half_float  | 16位的双精度IEEE 754浮点类型 |
|    float     | 32位的双精度IEEE 754浮点类型 |
| scaled_float |      缩放类型的浮点类型      |

* 长度越短，索引的搜索效率越高

##### 日期类型

* `date`:可以是由format指定的任意date类型，或是毫秒数，底层转换为UTC毫秒数

##### 范围类型



### 桶聚合

##### Terms Aggregation

类似`group by`，`text`类型不可使用

```json
// 以gender字段分组,统计，gender=0和gender=1的个数
GET /consumer/_search
{
  "size": 0,
  "aggs": {
    "class_goup": { // 此名称任意
      "terms": {
        "field": "gender"
      }
    }
  }
}

// 结果
"buckets" : [
    {
        "key" : 1,
        "doc_count" : 6, //gender=1的有6个
    },
    {
        "key" : 0,
        "doc_count" : 2, //gende=0的有2个
    }
]
```

还可以进行子聚合，指标聚合

```json
// 以gender分组，然后统计每个组内age最大的
GET /consumer/_search
{
  "size": 0,
  "aggs": {
    "class_goup": {
      "terms": {
        "field": "gender"
      },
      "aggs": {
        "max_age": {
          "max": {
            "field": "age"
          }
        }
      }
    }
  }
}

// 结果
"buckets" : [
    {
        "key" : 1,
        "doc_count" : 6,
        "max_age" : {
            "value" : 18.0	//gender为1的有6个，其中age最大的为18
        }						
    },
    {
        "key" : 0,
        "doc_count" : 2,
        "max_age" : {
            "value" : 14.0	//gender为0的有2个，其中age最大的为14
        }
    }
]
```

##### Filter Aggregation

将符合过滤器条件的数据分到一个bucket中

```json
// 15<age<20的个数
GET /consumer/_search
{
  "size": 0,
  "aggs": {
    "filter_agg": { // 任意名
      "filter": {
        "range": {
          "age": {
            "gte": 15,
            "lte": 20
          }
        }
      }
    }
  }
}
```

##### Filters Aggregation

多过滤器聚合，可以有多个过滤器，分成多个桶

```JSON
// 两个桶
GET /consumer/_search
{
    "size": 0,
    "aggs": {
        "filter_agg": { // 任意名
            "filters": {
                "filters": {
                    "warnings": {  //任意名字
                        "term": {
                            "type": "WARNING"
                        }
                    },
                    "faults": {		//任意名字
                        "term": {
                            "type": "FAULT"
                        }
                    }
                }
            }
        }
    }
}
```



##### Range Aggregation

查询0-20,20-50,50以上每个年龄段的个数

```json
GET /consumer/_search
{
  "size": 0, 
  "aggs": {
    "age_range": {
      "range": {
        "field": "age",	// age字段
        "ranges": [
          {
            "to": 20	// 最小~20
          },{
            "from": 20,
            "to": 50	//20~50
          },{
            "from": 50	// 50以上
          }
        ]
      }
    }
  }
}

// 结果
"aggregations" : {
    "age_range" : {
      "buckets" : [
        {
          "key" : "*-20.0",
          "to" : 20.0,
          "doc_count" : 8
        },
        {
          "key" : "20.0-50.0",
          "from" : 20.0,
          "to" : 50.0,
          "doc_count" : 0
        },
        {
          "key" : "50.0-*",
          "from" : 50.0,
          "doc_count" : 0
        }
      ]
    }
  }
```

##### Date Range Aggregation

专门用与统计日期的范围聚合，上面的范围聚合也可以用于日期，但日期范围聚合可以进行日期的运算

如从现在开始一个月前

```JSON
GET /consumer/_search
{
  "size": 0,
  "aggs": {
    "birthday_range": {
      "date_range": {
        "field": "birthday",	// BIRTHDAY字段日期范围
        "ranges": [
          {
            "from": "now-2y/y",	//从两年前开始到现在
            "to": "now"
          },{
            "from": "now",
            "to": "now+3M/M"	// 从现在到3个月以后
          },{
            "from": "now+3M/M"	// 3个月以后或更久
          }
        ]
      }
    }
  }
}

// 结果
"aggregations" : {
    "birthday_range" : {
        "buckets" : [
            {
                "key" : "2019-01-01T00:00:00.000Z-2021-11-20T08:57:25.445Z",
                "from" : 1.5463008E12,
                "from_as_string" : "2019-01-01T00:00:00.000Z",
                "to" : 1.637398645445E12,
                "to_as_string" : "2021-11-20T08:57:25.445Z",
                "doc_count" : 7
            },
            {
                "key" : "2021-11-20T08:57:25.445Z-2022-02-01T00:00:00.000Z",
                "from" : 1.637398645445E12,
                "from_as_string" : "2021-11-20T08:57:25.445Z",
                "to" : 1.6436736E12,
                "to_as_string" : "2022-02-01T00:00:00.000Z",
                "doc_count" : 1
            },
            {
                "key" : "2022-02-01T00:00:00.000Z-*",
                "from" : 1.6436736E12,
                "from_as_string" : "2022-02-01T00:00:00.000Z",
                "doc_count" : 0
            }
        ]
    }
}
```

##### Date Histogram Aggregation

日期直方图聚合

例如统计每年的生日人数

```json
GET /consumer/_search
{
  "size": 0,
  "aggs": {
    "birthday_month_count": {
      "date_histogram": {
        "field": "birthday",	//birthday字段
        "interval": "year"		//以年为单位，可以以年月日时分秒为间隔
      }
    }
  }
}

// 结果
"aggregations" : {
    "birthday_month_count" : {
        "buckets" : [
            {
                "key_as_string" : "2020-01-01T00:00:00.000Z",	//2021年1个
                "key" : 1577836800000,
                "doc_count" : 1
            },
            {
                "key_as_string" : "2021-01-01T00:00:00.000Z",	//2021年7个
                "key" : 1609459200000,
                "doc_count" : 7
            }
        ]
    }
}
```

##### Missing Aggregation

空值聚合，统计没有指定字段的个数

```json
GET /consumer/_search
{
  "size": 0,
  "aggs": {
    "miss": {
      "missing": {
        "field": "birthday"	// 统计没有birthday字段的数据的个数
      }
    }
  }
}

"aggregations" : {
    "miss" : {
        "doc_count" : 0	// 所有数据都有birthday字段
    }
}
```

##### Children Aggregation

##### Geo Distance Aggregation

例如：方圆几百公里的城市

##### IP Range Aggregation

### 管道聚合

对聚合数据的聚合

##### Avg Bucket Aggregation

例如：求每个班级的平均年龄，再计算所有班级平均年龄的平均年龄

```json
GET /consumer/_search
{
    
}
```

#### 其他

##### 普通查询与nested查询结合

```json
GET /gf_device_alarm_log/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "areaId": {
              "value": 591071973208100864
            }
          }
        },
        {
          "nested": {
            "path": "alarmCode",
            "query": {
              "term": {
                "alarmCode.name": {
                  "value": "普通预警"
                }
              }
            }
          }
        }
      ]
    }
  }
}
```

##### 嵌套聚合

```json
GET /my_index/blogpost/_search
{
  "size" : 0,
  "aggs": {
    "comments": { 
      "nested": {
        "path": "comments"
      },
      "aggs": {
        "by_month": {
          "date_histogram": { 
            "field":    "comments.date",
            "interval": "month",
            "format":   "yyyy-MM"
          },
          "aggs": {
            "avg_stars": {
              "avg": { 
                "field": "comments.stars"
              }
            }
          }
        }
      }
    }
  }
}
```





