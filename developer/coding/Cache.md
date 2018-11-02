# 缓存使用规约

> 作者 张磊

使用 @Cacheable、@CachePut 和 @CacheEvict 三个注释标签实现基于方法对象的缓存，减少了对原有的侵入，在不改变代码的前提下可以切换底层缓存实

* [方法级注解](#user-content-1)
  * [@Cacheable 增加缓存](#user-content-1.1)
  * [@CacheEvict 清空缓存](#user-content-1.2)
  * [@CachePut 增加缓存同时调用方法](#user-content-1.3)
  * [缓存过期](#user-content-1.4)
  * [注解参数](#user-content-1.5)
  * [Synchronized](#user-content-1.6)
  * [优化缓存的使用](#user-content-1.7)
* [编码使用](#user-content-2)
* [底层实现方式](#user-content-3)

## <a id="1">1</a> 方法级注解

### <a id="1.1">1.1</a> @Cacheable 增加缓存 

> 使用Cacheable注解实现方法返回对象的自动缓存，当缓存存在时方法体不会被执行，当缓存不存在时方法体执行，并将执行结果缓存

示例说明：

- 定一个UserService服务
- getUserByName方法根据用户名userName从数据库中获取用户对象并返回
- 使用Cacheable定义缓存名称为userCache

```java
import org.springframework.cache.annotation.Cacheable;

@Service
public class UserService {
    @Cacheable(value="userCache")
    public User getUserByName(String userName) {
		// 此处实现从数据库中根据用户名 userName 获取 User对象并返回
    }
}
```

getUserByName方法首次调用的时候在userCache中查找key为userName的数据，如果没有找到则执行方法体内容，并将userName作为key，User对象作为value缓存到userCache中，再次调用getUserByName方法并传递相同的userName时由于userCache已经存在key=userName的缓存，那么方法体将不会被执行

### <a id="1.2">1.2</a> @CacheEvict 清空缓存 

> 使用 @CacheEvict 注解实现缓存的清空

```java
import org.springframework.cache.annotation.CacheEvict;

@Service
public class UserService {
    @CacheEvict(value="userCache", allEntries=true)
    public void cleanUserCache() {
		//此方法体不用写任何实现，调用时即可清空userCache所有缓存
    }
    
    @CacheEvict(value="userCache")
    public void delUser(String userName) {
    	//此处实现删除用户的方法，调用时会自动清除userCache中key=userName的缓存
    }    
}
```

### <a id="1.3">1.3</a> @CachePut 增加缓存同时调用方法

> @Cacheable注解的方法当缓存存在时则不会调用方法内部的逻辑代码，如果我们希望缓存的同时也调用方法体内的实现可以使用 @CachePut 注解

### <a id="1.4">1.4</a> 缓存过期

> 因为@Cache的底层实现不同，所以注解参数中没有提供统一的过期参数，可以参考一下两种方式实现

* 方法同时增加 CacheEvict、 @Scheduled 实现定期调用清除
* 使用@Autowired 获取CacheManger通过编码方式控制缓存

### <a id="1.5">1.5</a> 注解参数

#### value

> 缓存名称

#### key

> 以上两个例子缓存的key都是默认参数userName，如果传递的参数是一个对象的时候可以使用SpEL 表达式指定key

例如：key是user对象的userName属性

```java
import org.springframework.cache.annotation.Cacheable;

@Service
public class UserService {
    @CacheEvict(value="userCache", key="#user.getUserName()")
    public void delUser(User user) {
    	//此处实现删除用户的方法，调用时会自动清除userCache中key=user对象的userName属性的缓存
    } 
}
```

#### condition

> 条件缓存，支持[SpEL](https://docs.spring.io/spring/docs/3.0.x/reference/expressions.html)，当表达式返回值为true时才缓存

更多详情参考[SpringBoot Cache](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-caching.html)

#### cacheResolver

> 用于指定使用那个缓存解析器，非必需。

### <a id="1.6">1.6</a> Synchronized

> 不可以在@CacheXXX的注解的方法上增加synchronized，因为这是无效的，可以单独定义一个synchronized方法，然后在@CacheXXX注解的方法中调用这个同步方法

```java
import org.springframework.cache.annotation.Cacheable;

@Service
public class UserService {
    
    private synchronized delSyncUser(User user){
        //实现删除用户的逻辑
    }
    
    @CacheEvict(value="userCache", key="#user.getUserName()")
    public void delUser(User user) {
		delSyncUser(user)
    } 
}
```

### <a id="1.7">1.7</a> 优化缓存的使用

#### 1.7.1 返回空对象不缓存

使用unless参数定义空对象不缓存

```java
@Cacheable(value="userCache", unless="#result != null")
public User getUserByName(String userName) {

}
```

#### 1.7.2 组合使用注解

```java
class UserService {
	@Cacheable(value = "userCache", unless = "#result != null")
	public User getUserById( long id ) {
   		return userRepository.getById( id );
	}
}
 
class UserRepository {
	@Caching(
      put = {
            @CachePut(value = "userCache", key = "'username:' + #result.username", condition = "#result != null"),
            @CachePut(value = "userCache", key = "#result.id", condition = "#result != null")
      }
	)
	@Transactional(readOnly = true)
	public User getById( long id ) {
	   ...
	}
}
```

#### 1.7.3 事务

创建一个存储访问对象,并声明@CachePut @CacheEvict 和 @Transactional

```java
class UserRepository {
	@CachePut(value = "userCache", key = "#result.username")
	@Transactional(readOnly = true)
	User getByUsername( String username );

	@CacheEvict(value = “userCache”, key = "#p0.username"),
	@Transactional
	void save( User user );
}
```

创建一个Service调用UserRepository

```java
class UserService {
	@Transactional
	User updateAndRefresh( User user ) {
		userRepository.save(user);
		return userRepository.getByUsername( user.getUsername() );
	}
}
```

## <a id="2">2</a> 编码使用

> 注解的方式使用简单，但是缺乏灵活性，例如数据的expire就无法统一设置，我们也可以通过CacheManager对象实现灵活的缓存使用。

```java
@Autowired
private CacheManager cacheManager; //通过注解获取默认cacheManager

public void test(){
    if(this.cacheManager instanceof RedisCacheManager){
        RedisCacheManager redisCacheManager = (RedisCacheManager)this.cacheManager;
        //使用灵灵活的方式操作缓存
        redisCacheManager.xxx 
    }
}
```



## <a id="3">3</a> 底层实现方式

> Cache注解支持多种实现方式，框架默认的是进程内Guava方式，也支持Redis，EHcache等方式，可以通过配置参数定义实现方式

缓存实现方式配置参数，默认采用侦测包的方式，如果检测到redis配置，那么会默认使用redis方式（不建议），也可以通过以下参数定义（建议）

```properties
spring.cache.type=redis
```



> 



