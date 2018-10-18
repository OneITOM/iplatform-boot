# 获取配置参数规约

> 作者 张磊

本文介绍了在框架中如何获取配置参数值

## @Value

> 凡是通过spring注解 @Service，@Constoller，@RestConstoller，@Component等实例化的类可以通过@Value获取参数
>
> **注意：@Value的的值是在类实例化时初始化的**

```java
@Service
public class MyService {
    
    @Value("${server.host:null}")
    String serverHost;
    
    public void testMethod() {
        System.out.println(serverHost)
    }    
}
```



## @Autowired

> 凡是通过spring注解 @Service，@Constoller，@RestConstoller，@Component等实例化的类可以通过@autowired方式获取Environment对象
>
> **注意:这种方式和@Value的区别是，可以获取注册中心实时变化的参数值**

```java
@Service
public class MyService {
    
    @Autowired
    Environment environment;
    
    public void testMethod(Environment environment) {
        String serverHost = environment.getProperty("server.host");
    }    
}
```



## 接口方式

> 对于普通类可以通过实现EnvironmentAware的方式在代码中获取配置参数

```java
public class MyClass implements EnvironmentAware {
    @Override
    public void setEnvironment(Environment environment) {
        String serverHost = environment.getProperty("server.host");
    }    
}
```



 ## 全局静态上下文中获取

> 在主启动类的run方法中返回的ConfigurableApplicationContext对象中获取

```java
public class MyServiceApplication extends IPlatformServiceApplication {
    
    private static final Logger LOG = LoggerFactory.getLogger(EmptyServiceApplication.class);
    private static ConfigurableApplicationContext context;
    public static void main(String[] args) throws Exception {
        try{
            context = run(EmptyServiceApplication.class, args);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}


public class MyClass {

    public void testMethod(Environment environment) {
        Environment environment = MyServiceApplication.context.getEnvironment();
        String serverHost = environment.getProperty("server.host");
    }    
}
```



## 配置中心动态参数

参考[集中配置开发手册](../developer/config/README.md)

 