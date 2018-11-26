# Mongo开发手册

> 作者 yubo

### MongoDB简介和基本概念
MongoDB是一种NoSql非关系型数据库，以键值对(key-value)存储，它的结构不固定，每一条记录可以有不一样的键，每条记录可以根据需要增加一些自己的键值对，这样就不会局限于固定的结构，可以减少一些时间和空间的开销。

与传统关系型数据库对比：

	优势方面：
		+ 快速的读写 
		+ 低廉的成本 
		+ 灵活的数据结构
	
	劣势方面：
		+ 不提供对SQL的支持
		+ 语法不同于传统SQL，学习成本
		+ 集合之间弱关连
<table style="width: 100%; border-collapse: collapse;">
  <tr>
    <th>对比项</th>
    <th>mongoDB</th>
    <th>mysql oracle</th>
  </tr>
  <tr>
    <td style="text-align:center;">表</td>
    <td style="text-align:center;">集合</td>
    <td style="text-align:center;">二维表table</td>
  </tr>
  <tr>
    <td style="text-align:center;">表的一行数据</td>
    <td style="text-align:center;">文档document</td>
    <td style="text-align:center;">一条记录recoder</td>
  </tr>
  <tr>
    <td style="text-align:center;">表字段</td>
    <td style="text-align:center;">键key</td>
    <td style="text-align:center;">字段filed</td>
  </tr>
<tr>
    <td style="text-align:center;">字段值</td>
    <td style="text-align:center;">值value</td>
    <td style="text-align:center;">值value</td>
  </tr>
<tr>
    <td style="text-align:center;">主外键</td>
    <td style="text-align:center;">无</td>
    <td style="text-align:center;">PK FK</td>
  </tr>
<tr>
    <td style="text-align:center;">灵活度扩展性</td>
    <td style="text-align:center;">极高</td>
    <td style="text-align:center;">差</td>
  </tr>
<tr>
    <td style="text-align:center;">索引</td>
    <td style="text-align:center;">有</td>
    <td style="text-align:center;">有</td>
  </tr>
</table>

- 文档(document)是MongoDB中数据的基本单元，非常类似于关系型数据库系统中的行(但是比行要复杂的多)。
- 集合(collection)就是一组文档，如果说MongoDB中的文档类似于关系型数据库中的行，那么集合就如同表。
- MongoDB的单个计算机可以容纳多个独立的数据库，每一个数据库都有自己的集合和权限。
- MongoDB自带简洁但功能强大的JavaScript shell，这个工具对于管理MongoDB实例和操作数据作用非常大。
- 每一个文档都有一个特殊的键”_id”,它在文档所处的集合中是唯一的，相当于关系数据库中的表的主键。

* * *
### 环境依赖
在pom文件引入spring-boot-starter-data-mongodb依赖：

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-data-mongodb</artifactId>
	</dependency>

### 数据源配置
在application.yml文件写入mongodb数据源：
	
1,默认端口无密码：
```properties
spring.data.mongodb.uri=mongodb://[ip]/[databaseName]
```
2,配置端口无密码：
```properties
spring.data.mongodb.uri=mongodb://[ip]:[port]/[databaseName]
```
3,配置端口有密码：
```properties
spring.data.mongodb.uri=mongodb://[username]:[password]@[ip]:[port]/[databaseName]
```
例:spring.data.mongodb.uri=mongodb://bomcbp:bomcbp@192.168.55.30:27017/datashare

### 使用示例
直接在代码中绑定注入即可
```java
@Autowired
private MongoTemplate mongoTemplate;
```
具体API可参见  [MongoTemplate API](https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoTemplate.html)

### Query的翻页和排序
```java
Query query = new Query(criteria);
// 翻页
query.skip((offsite - 1) * pageSize);
query.limit(pageSize);
// 排序(DESC：降序, ASC：升序)
query.with(new Sort(Direction.DESC, key));
```
### MongoDB的查询
Mongodb提供了Criteria对象，帮助我们实现数据的查询功能，何为Criteria对象：

	+ Criteria查询采用面向对象方式进程条件查询
	+ 对查询语句进行了封装
	+ 采用对象的方式来组合各种查询条件


更多的Query查询对象的用法及API可参见 [Query API](https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/query/Query.html)

### Criteria常用查询
前提：mongodb对数据类型敏感，key = 1 和 key = '1'的查询解雇是不一样的

1，等于"="，不等于"<>"
```java
Criteria criteria = new Criteria();
criteria = criteria.and(key).is(value);
criteria = criteria.and(key).ne(value); 
```
```sql
where key = value, where key <> value
```
2，大于">", 大于等于">=", 小于"<", 小于等于"<="
```java
Criteria criteria = new Criteria();
criteria = criteria.and(key).gt(value);
criteria = criteria.and(key).gte(value);
criteria = criteria.and(key).lt(value);
criteria = criteria.and(key).lte(value); 
```
```sql
where key > value, where key >= value, where key < value, where key <= value
```
3, "in", "not in" (参数是数组)
```java
Criteria criteria = new Criteria();
criteria = criteria.and(key).in(values);
criteria = criteria.and(key).nin(values); 
```
```sql
where key in (values), where key not in (values)
```
4，模糊查询"like"
```java
Criteria criteria = new Criteria();
criteria = criteria.and(key).regex(".*?" + queryValue + ".*");
```
```sql
where key like '%value%'
```
以上条件基本可以涵盖绝大部分的查询需求了，只需根据业务场景来灵活组合查询条件即可（andOperator, orOperator）

例：在user集合中，查询所有的[男性],且[年龄]大于等于30的，且[现住址]不在'大东区','苏家屯','于洪区'的，且[老家]在'抚顺'或'阜新'，且姓名中含有'于'字的这些人（就把我给选出来了*^_^*）
```java	
Criteria criteria = new Criteria();
criteria = criteria.and("sex").is("male"); 
criteria = criteria.and("age").gte(30);
criteria = criteria.and("address").nin({'大东区','苏家屯','于洪区'});
criteria = criteria.and("hometown").in({'抚顺','阜新'});
criteria = criteria.and("name").regex(".*?于.*");
Query query = new Query(criteria);
// 查询user集合
return mongoTemplate.find(query, JSONObject.class, "user");  
```
更多的Criteria查询对象的用法及API可参见 [Criteria API](https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/query/Criteria.html)

### 聚合函数
Aggregation简单来说，就是提供数据统计、分析、分类的方法，常用于数据统计报表，配合ECharts绘制统计分析图表等，一个Aggregation操作，接收指定collection的数据集，通过计算后返回result数据

使用mongoTemplate来实现按资源类型统计数据
```java
Criteria criteria = new Criteria();
// 此处只统计PC服务器，小型机和路由器数据
criteria = criteria.and("BMCLASSNAME").in({"ISS","EPS","CMDB_ROUTER"}); 
Aggregation agg = Aggregation.newAggregation(    
    Aggregation.match(criteria),//条件  
    Aggregation.group("BMCLASSNAME").count().as("count"),//分组字段    
    Aggregation.sort(sort),//排序  
    Aggregation.skip(page.getFirstResult()),//过滤  
    Aggregation.limit(pageSize)//页数  
 );    
AggregationResults<JSONObject> outputType=mongoTemplate.aggregate(agg,"cmdb",JSONObject.class);    
List<JSONObject> list=outputType.getMappedResults();
```	
```sql
select BMCLASSNAME,count(1) from cmdb where BMCLASSNAME in ('ISS','EPS','CMDB_ROUTER') group by BMCLASSNAME
```
更多的Aggregation的用法及API可参见 [Aggregation API](https://www.baeldung.com/spring-data-mongodb-projections-aggregations)

### Mongodb的事务 -- 未完待续
### Mongodb的主从 -- 未完待续
