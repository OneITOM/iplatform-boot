# 自定义参数规约

> 本框架的参数使用YAML格式定义，除[框架参数](../../Properties.md)外，使用者也可以采用此格式定义自身的业务参数

YAML优点

* 作为JSON超集可以定义复杂类型
* 相比Properties文件尺寸更小
* 相比JSON格式可以写注释

## 自定义参数方式

> 自定参数可以直接写在application.yml 文件中，也可以单独建立一个yml文件（不推荐，需要单独编码读取参数）

举例

> 以下参数样例，演示了多种结构的参数定义方式

YAML格式

```yaml
myapp:
  username: demo
  password: demo123
  url: 'http://xxxxx'
  counter: 1
  enabled: true
  params:
    one: 1
    two: 2
    third: 3
  words:
    - word1
    - word2
    - word3
  pairs:
    - name: name1
      age: 1
    - name: name2
      age: 2
    - name: name3
      age: 3
  array: array1,array2,array3
```

外部定义格式

```properties
myapp.username=demo
myapp.password=demo123
myapp.url='http://xxxxx'
myapp.counter=1
myapp.enabled=true
myapp.params.one=1
myapp.params.two=2
myapp.params.third=3
myapp.words[0]=word1
myapp.words[1]=word2
myapp.words[2]=word3
myapp.pairs[0].name=name1
myapp.pairs[0].age=1
myapp.pairs[1].name=name2
myapp.pairs[1].age=2
myapp.pairs[2].name=name3
myapp.pairs[2].age=3
myapp.array=array1,array2,array3
```

## 格式规约

> 在参数定义的时候请遵循以下规约

* 应用的参数请使用单独的根节点，例如myapp.xxx
* map类型的参数请使用类似params的定义方式
* list类型的参数请使用类似words的定义方式

## 加载规约

> 框架中支持在application.yml 、命令行参数、环境变量等多种方式定义配置参数，并遵循一定的优先加载顺序进行配置加载

优先顺序遵循如下约定，按照以下顺序读取key后将忽略后续的定义key

1. 命令行参数

   ```shell
   java -jar xxx.jar --myapp.username=demo
   ```

2. 集中配置（需要开启集中配置参数）

   ```properties
   # 在启动参数开启集中配置，会到集中配置中加载
   java -jar xxx.jar --spring.cloud.config.enabled=true
   ```

3. 环境变量

   > 因为springboot的参数格式约定为点分隔。但是在linux中使用export方式生命环境变量的时候名称禁止使用点，所以可以使用env [key=value] command的方式设置环境变量，docker内使用bash无此问题

   ```shell
   env myapp.username=demo myapp.password=demo123 java -jar xxx.jar
   ```

4. application.yml

   > 读取项目中application.yml中的配置

## 静态读取配置方法

### 1. @Value

> 使用Spring的类管理器自动装配

```java
@Value("${myapp.username}")
private String username;

@Value("${myapp.counter}")
private Integer counter;

@Value("${myapp.array}")
private String[] array;
```

### 2. ConfigurationProperties

> 使用自定义类加载一组配置

```java
@Component
@ConfigurationProperties(prefix="myapp")
public class MyAppConfig {
    private String username;
    private String password;
    private Integer counter;
    private Boolean enabled;
    private String[] words;
    private String[] array;
    private Pair[] pairs;
    private Map<String,String> params;
    
    //getter
    
    //setter
}
```

## 动态读取配置方法

> 动态读取配置依赖[集中配置服务](../config/README.md)，在程序启动后通过修改集中配置动态修改每个服务的参数，程序中如果要使用动态参数，那么需要使用以下加方法在程序执行时动态获取

```java
IPlatformApplication.getEnvironment().getProperty("myapp.password")
IPlatformApplication.getEnvironment().getProperty("myapp.array", String[].class);
```





