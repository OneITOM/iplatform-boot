# 获取配置参数规约

> 作者 张磊

框架中所有配置参数都可以通过 Environment 获取，配置参数包含启动参数，System.env等参数

## @Autowired

> 凡是通过spring注解 @Service，@Constoller，@RestConstoller，@Component等实例化的类可以通过@autowired方式获取Environment对象

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

 ## ConfigurableApplicationContext

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



 