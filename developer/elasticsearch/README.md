# Elasticsearch开发手册

> 作者 于胜强

## 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
    <exclusions>
    	<exclusion>
    		<groupId>org.springframework.data</groupId>
    		<artifactId>spring-data-elasticsearch</artifactId>
    	</exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-elasticsearch</artifactId>
    <version>2.0.11.RELEASE</version>
    <exclusions>
    	<exclusion>
    		<groupId>org.springframework.data</groupId>
    		<artifactId>spring-data-commons</artifactId>
    	</exclusion>
    </exclusions>
</dependency>		
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-commons</artifactId>
    <version>1.12.11.RELEASE</version>
</dependency>
```

## 增加配置参数

```yml
spring:
  data:
    elasticsearch:
      cluster-name: elasticsearch
      cluster-nodes: 192.168.0.1:9300,192.168.0.2:9300,192.168.0.3:9300
```


## 使用elasticsearch

### 自动注入ElasticsearchTemplate

```java

@Autowired(required = false)
private ElasticsearchTemplate elasticsearchTemplate;

```

### 使用示例

> elasticsearchTemplate使用

- 查询
```java
SearchQuery searchQuery = new NativeSearchQueryBuilder()
    .withQuery(matchAllQuery())
    .withFilter(boolFilter().must(termFilter("id", documentId)))
    .build();

Page<SampleEntity> sampleEntities =
    elasticsearchTemplate.queryForPage(searchQuery,SampleEntity.class);
```
- 创建索引
```java
String indexName = "test-log-2018.10.01";
elasticsearchTemplate.createIndex(indexName);

```
- 获取原生Client进行操作
```java
Client client = elasticsearchTemplate.getClient();
// 搜索数据
SearchResponse response = null;
SearchRequestBuilder requestBuilder =  client.prepareSearch(indexName);
response = requestBuilder.setQuery(boolQueryBuilder)
        .setFrom(page).setSize(size).setExplain(true)
        .setFetchSource(true)
        .addSort("timestamp", SortOrder.DESC)
        .execute().actionGet();

```

> Repository 方式使用

```java
/** Entity 声明*/
@Document(indexName = "es-customer", type = "customer", shards = 2, replicas = 1, refreshInterval = "-1")
public class Customer {
    @Id
    private String id;
    private String name;
    // constructor,set,get省略
}

/** Repository */
public interface CustomerRepository extends ElasticsearchRepository<Customer, String> {
    public List<Customer> findByName(String name);
}

```

```java
/** 自动注入 */
@Autowired
private CustomerRepository repository;
```

```java
// 保存
repository.save(new Customer("Alice"));
// 删除
repository.delete(id);
// 数量
repository.count();
// 查询
repository.findAll();
```
