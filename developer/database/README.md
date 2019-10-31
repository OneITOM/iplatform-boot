# 数据库开发手册

> 作者 张磊

## 1. 依赖包

```xml
<dependency>
    <groupId>org.iplatform</groupId>
    <artifactId>iplatform-service</artifactId>
</dependency>
```

## 2. 配置数据源

```properties
# 数据库类型(必填)
spring.datasource.platform=mysql
# JDBC驱动(必填)
spring.datasource.dataSourceClassName=com.mysql.jdbc.Driver
# JDBC URL(必填)
spring.datasource.url='jdbc:mysql://oneitom-mysql:3306/authdb?useUnicode=true&characterEncoding=utf-8&autoReconnect=true'
# 数据库用户名(必填)
spring.datasource.username=authdb-user
# 数据库密码(必填)
spring.datasource.password=ENC(mBaGBXPu1VFgECoBH5NGWeTdFLy79Ic5)
# 开启定期检查空闲连接(非必填，默认true)
spring.datasource.test-while-idle=true
# 检查一次空闲连接间隔(非必填，30s)
spring.datasource.time-between-eviction-runs-millis=300000
# 空闲链接清除间隔(非必填，默认30s)
spring.datasource.min-evictable-idle-time-millis=30000
# 检查空闲链接SQL（oracle必填SELECT 1 FROM DUAL，覆盖默认的SELECT 1，mysql非必填）
spring.datasource.validation-query='SELECT 1 FROM DUAL'
# 最大并发连接数(非必填，默认100)
spring.datasource.max-active=100
# 最大空闲连接数(非必填，默认5)
spring.datasource.max-idle=5
# 最小空闲连接数(非必填，默认2)                
spring.datasource.min-idle=2
# sql最大等待时长(非必填，默认30s)
spring.datasource.max-wait=30000
```

## 3. 数据库初始化脚本

> SpringBoot提供了程序启动时自动创建数据库脚本和初始化数据的功能

1. 定义初始化表结构脚本

   在resources目录下创建schema-mysql.sql文件，并在文件中定义建表脚本

   ```sql
   create table empty_test(
   	id int(11) not null auto_increment,
   	is_enable boolean not null default false,
   	create_time timestamp not null,
   	primary key (id)
   ) engine=InnoDB 
   ```

2. 定义初始化数据脚本

   在resources目录下创建data-mysql.sql文件，并在文件中定义初始化数据脚本

   ```sql
   insert into empty_test(is_enable,create_time) values (true,current_timestamp());
   ```

3. *开启启动数据源初始化参数

   ```properties
   # 使用schema-xx.sql、data--xx.sql初始化数据库数据（默认是true）
   spring.datasource.initialize=true
   # 初始化数据过程中出错继续（默认是false）
   spring.datasource.continue-on-error=true
   ```

   **特别注意：spring.datasource.initialize=true的时候每次启动都会执行这两个脚本，所以脚本内不能有删表脚本，除非你明确要删除**

## 4. 定义Mapper

> Mybatis采用接口全注解方式

1. 创建dao包

   ```java
   package [你的项目包路径].service.dao
   ```

2. 创建DO类

   ```java
   package [你的项目包路径].service.domain;
   import java.sql.Date;
   public class TestDO {
   	String id;
   	Date opdatetime;
   	public String getId() {
   		return id;
   	}
   	public void setId(String id) {
   		this.id = id;
   	}
   	public Date getOpdatetime() {
   		return opdatetime;
   	}
   	public void setOpdatetime(Date opdatetime) {
   		this.opdatetime = opdatetime;
   	}
   }
   ```

3. 创建Mapper接口类

   ```java
   package [你的项目包路径].service.dao;
   
   import java.util.List;
   import org.apache.ibatis.annotations.Mapper;
   import org.apache.ibatis.annotations.Select;
   import [你的项目包路径].service.domain.TestDO;
   
   @Mapper
   public interface MyMapper {
   	@Select("select * from empty_test")
   	List<TestDO> getAll();
   }
   ```

## 5. 在Service中调用Mapper

```java
package [你的项目包路径].service;

import java.util.List;
import javax.servlet.http.HttpServletRequest;
import [你的项目包路径].service.dao.TestMapper;
import [你的项目包路径].service.domain.TestDO;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author zhanglei
 */
@Configuration
@Service
@RestController
@RequestMapping("/api/v1/test")
public class TestService {
    private static final Logger logger = LoggerFactory.getLogger(TestService.class);
    
    @Autowired
    TestMapper testMapper;
    
    @RequestMapping(value = "/testdb", method = RequestMethod.GET)
    public ResponseEntity<RestResponse<List<TestDO>>> getAll() {
        RestResponse<List<TestDO>> response = new RestResponse();
        List<TestDO> dbos = testMapper.getAll();
        response.setData(dbos);
        response.setSuccess(Boolean.TRUE);
        return new ResponseEntity<>(response, HttpStatus.OK);
    }
}
```

## 6. 翻页查询

> 框架内部集成了分页插件[PageHelper 4.1.6]("https://pagehelper.github.io/")，示例如下，前端表格基于Bootstrap Table

1. 创建QueryContext，定义Bootstrap Table分页信息

   ```java
   
    package [你的项目包路径].domain;

    public class QueryContext {
        private int start;
        private int limit;

        public int getStart() {
            return start;
        }

        public void setStart(int start) {
            this.start = start;
        }

        public int getLimit() {
            return limit;
        }

        public void setLimit(int limit) {
            this.limit = limit;
        }
    }
    
   ```

2. 创建PageInfoTable，定义Bootstrap Table接收数据类型

   ```java
   
    package [你的项目名称].domain;

    import java.util.List;

    public class PageInfoTable {

        private long total;

        private List rows;

        public long getTotal() {
            return total;
        }

        public void setTotal(long total) {
            this.total = total;
        }

        public List getRows() {
            return rows;
        }

        public void setRows(List rows) {
            this.rows = rows;
        }
    }
   
   ```

   

3. 创建controller，接收Bootstrap Table的请求

   ```java
   @RequestMapping(value = "/allTableData", method = RequestMethod.POST)
   public ResponseEntity<PageInfoTable> getAllTableData(@RequestBody QueryContext queryContext) {
   	PageInfoTable pageInfoTable = null;
   	try {
   		pageInfoTable = testService.getAllTableData(queryContext);
   		return new ResponseEntity<>(pageInfoTable, HttpStatus.OK);
   	} catch (Exception e) {
   		LOG.error("", e);
   		return new ResponseEntity<>(pageInfoTable, HttpStatus.OK);
   	}	
   }
   ```

4. 创建service，实现分页查询

   ```java

    public PageInfoTable getAllTableData(QueryContext queryContext) {
        PageInfoTable pageInfoTable = new PageInfoTable();
        try{
            //启用分页
            PageHelper.startPage(queryContext.getStart(), queryContext.getLimit());
            //紧跟startPage方法后面的第一个Mybatis查询会被分页
            List<Map<String,Object>> dataList = testMapper.getAllTableData();
            //查询结果强转为包含完整分页信息的PageInfo
            PageInfo<Map<String,Object>> pageInfo = new PageInfo<Map<String,Object>>(dataList);
            long total = pageInfo.getTotal();
            //装载pageInfoTable返回前端
            pageInfoTable.setTotal(total);
            pageInfoTable.setRows(dataList);
        }catch(Exception e){
            LOG.error("", e);
        }finally{
            PageHelper.cleanPage();
        }
        return pageInfoTable;
    }
   
   ```

## 7. 多数据源

> 在一个项目中使用多个数据库连接的配置方法

1. 增加一个数据源otherdb

   ```properties
   # 开启支持多数据源
   spring.dynamicdatasource.enable=true
   # 定义数据源名称为bomcbp
   spring.dynamicdatasource.names=otherdb
   # 定义bomcbp数据源的jdbc驱动
   spring.dynamicdatasource.otherdb.dataSourceClassName=oracle.jdbc.driver.OracleDriver
   # 定义bomcbp数据源的url
   spring.dynamicdatasource.otherdb.url="jdbc:oracle:thin:@127.0.0.1:1521:mydb"
   # 定义bomcbp数据源的用户名
   spring.dynamicdatasource.otherdb.username=username
   # 定义bomcbp数据源的密码
   spring.dynamicdatasource.otherdb.password=password
   ```

2. 定义Mapper

   多数据源定义Mapper的方式和默认数据源方式没有差别，只是我们需要在Service的方法中通过方法注解声明此Mapper的调用使用哪个数据源。我们还用上边的testMapper这个例子，在方法上增加注解@TargetDataSource(name="otherdb")定义使用的数据源，这时再调用getAll方法的时候就使用的是otherdb这个数据源了。

   ```java
   @RequestMapping(value = "/testdb", method = RequestMethod.GET)
   @TargetDataSource(name="otherdb")
   public ResponseEntity<RestResponse<List<TestDO>>> getAll() {
       RestResponse<List<TestDO>> response = new RestResponse();
       List<TestDO> dbos = testMapper.getAll();
       response.setData(dbos);
       response.setSuccess(Boolean.TRUE);
       return new ResponseEntity<>(response, HttpStatus.OK);
   }
   ```

## 8. SQL语法兼容

> SQL语法兼容主要用在不同的数据库产品在SQL写法上存在一定的差异，我们可以通过配置定义在程序连接不同的数据库的时候执行匹配这个数据库的SQL语句

目前支持的数据库类型如下

* oracle
* mysql
* h2
* sqlserver
* db2
* postgresql

### 8.1 语法

通过if语法对内置变量_databaseId进行判断，从而构造动态SQL语句

```xml
<if test=\"_databaseId == 'oracle'\"></if>
<if test=\"_databaseId == 'mysql'\"></if>
...
<if test=\"_databaseId == 'db2'\"></if>
```

### 8.2 样例

> 例如：Oracle和MySQL的时间的格式化语法不通，所以通过_databaseId判断数据库类型，从而自动选择执行的SQL语法

```xml
@Select({ "<script>",
    " select"
    + " <if test=\"_databaseId == 'oracle'\">to_char(holiday_day,'YYYY-MM-DD') day</if>"
    + " <if test=\"_databaseId == 'mysql'\">DATE_FORMAT(holiday_day,'%Y-%m-%d') day</if>"
    + "from mytable ",
    "</script>" })
List<Map> getTest();
```
## 9. 事务管理

### 9.1 事务支持

> 主启动类增加@EnableTransactionManagement开启注解式事务的支持 

```java
@SpringBootApplication
@EnableOAuth2Client
@EnableDiscoveryClient
@EnableEurekaClient
@Configuration
@EnableTransactionManagement
@ComponentScan({"org.iplatform.microservices", "org.iplatform.myproject.service"})
public class MyprojectServiceApplication extends IPlatformServiceApplication {

    private static final Logger LOG = LoggerFactory.getLogger(ServiceApplication.class);

    public static void main(String[] args) {
        try {
            run(MyprojectServiceApplication.class, args);
        } catch (Exception e) {
            LOG.error("", e);
        }
    }
}
```

### 9.2 开启事务

> 类或者方法上增加注解@Transactional开启事务，在类上增加意味着此类的所有public方法都进行事务管理

```java
@Transactional(rollbackFor = Exception.class)
public void save(TestDO testDO){
    testMapper.insert(testDO);
}
```
## 10. SQL敏感词过滤

### 10.1 功能开启

> 通过配置的方式开启敏感词过滤功能，目前只实现了针对sql类型进行拦截，其他内容待扩展

```properties
# 开关，默认false
iplatform.sql.danger.enabled: true
# 拦截sql类型
iplatform.sql.danger.sqltype: drop,truncate
```

### 10.2 功能说明

> 敏感词过滤功能开启后，所有的sql执行均会被过滤，如果sql类型在拦截范围内，将终止sql的执行

```
sql拦截类型的支持

alter、createindex、createtable、createview、delete、drop、execute、insert、merge、replace
select、truncate、update、upsert
```
## 11. 数据分片及读写分离

> 框架内部集成了sharding-jdbc 3.1.0，研发人员可通过配置启用该功能，从而实现数据分片及读写分离

### 11.1 添加依赖

```xml
<dependency>
    <groupId>io.shardingsphere</groupId>
    <artifactId>sharding-jdbc-core</artifactId>
    <version>3.1.0</version>
</dependency>
```

### 11.2 功能启用

> 启动脚本调整

```properties
# sharding-jdbc开关，默认false
sharding.jdbc.enable: true
# 读写分离场景时需设置为true，默认false
sharding.jdbc.masterslave: false
```

> 新增分片或读写分离配置sharding-jdbc.yml，与application.yml同级目录，配置内容参考场景样例
>
> 具体属性说明查看官网 https://shardingsphere.apache.org/document/current/cn/overview/

### 11.3 场景说明

> 场景样例地址 [https://github.com/OneITOM/iplatform-boot-example/tree/master/example-shardingjdbc]( https://github.com/OneITOM/iplatform-boot-example/tree/master/example-shardingjdbc)

#### 分表场景

> sharding-tables，对于访问频繁数据量大的表来说，如果要减少访问所需的时间，可以进行分表，具体的分表策略需自定义

#### 分库场景

> sharding-databases，如果要提高数据库的写入能力，可以进行分库，具体的分库策略需自定义

#### 分库分表场景

> sharding-databases-tables，同时支持分库分表

#### 读写分离场景

> master-slave，读写分离支持一主多从，增加、删除、更新操作使用主库，查询操作通过负载均衡策略疏导至不同从库，该场景需要在启动脚本中设置 sharding.jdbc.masterslave: true

#### 分片+读写分离场景

> sharding-master-slave，同时支持数据分片及读写分离
