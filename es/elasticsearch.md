与springboot的集成

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

* `@Document`:用在类上，类似mysql的表
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

命令

```ini
# 集群健康状态
GET /_cat/health?v
# 集群所有节点
GET /_cat/nodes
# 集群所有索引
GET /_cat/indices
# 创建索引
# 创建名为customer的索引
PUT customer?pretty
# 创建索引下文档，索引不存在会自动创建索引
# 创建索引为customer,id为1，值为下面json的doc
PUT /customer/_doc/1?pretty
{
  "name": "John"
}
# 查询指令索引，指定id的doc
# 查询索引为customer,id为1的doc
GET /customer/_doc/1?pretty
# 删除索引
# 删除customer索引
DELETE /customer?pretty
# 替换doc，原doc被新的替换
# id已被创建，则更新索引，不存在则创建
PUT /customer/_doc/1?pretty
{
  "name": "Tom"
}
# 创建doc，不指定id
# 使用post方法，不指定id，默认生成一个id
POST /customer/_doc?pretty
{
  "name": "Jane"
}
# 更新文档
# 更新customer索引下id为1的doc，更改名字为指定
POST /customer/_update/1/?pretty
{
  "doc": { "name": "Tom" }
}
# 更新文档
# 更新customer索引下id为1的doc，更改名字为指定,并添加age字段
POST /customer/_update/1?pretty
{
  "doc": { "name": "Tom", "age": 20 }
}
```

查询

```ini
# 查询bank索引下所有文档，默认返回10条
GET /bank/_search
{
  "query": { "match_all": {} }
}

# 查询指定条数
GET /bank/_search
{
  "query": { "match_all": {} },
  "size": 1
}

#  
GET /bank/_search
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}
```

