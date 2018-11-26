# 鉴权规约

> 结合认证服务可以实现服务的单点认证和权限控制

### 1. 获取当前用户

* @RequestMapping

> 通过增加Principal 变量可以在RESTful API中自动注入当前用户登录名

```java
@RequestMapping("/xxx")
public String userinfo(ModelMap map,Principal principa) throws Exception {
    // 可以获取到当前登录名 
    System.out.println(principa.getName())
}
```

* UserDetailsUtil

> 可以通过UserDetailsUtil获取当前登录用户的详细信息，所属部门、角色

```java
@Autowired
private UserDetailsUtil userDetailsUtil;

public void test(){
    UserDetails userDetails = new UserDetails();
    userDetails.setUsername("用户登录名"));
    userDetails = userDetailsUtil.getUserDetails(userDetails);    
}
```

* Session

> 尽量避免使用：只能在UI项目中使用，RESTful调用可能会由于sessionid变化而导致无效

 ```java
UserDetails userDetails = (UserDetails) request.getSession().getAttribute("userDetails");   
 ```

* 用户角色、部门

> 通过获得当前登录用户对象UserDetails，可以获取当前用户的角色和部门

```java
// 获取用户角色列表
userDetails.getAuthorities();

// 获取用户部门列表（一个用户只属于一个部门）
userDetails.getDepartments();
```

### 2. Token

> 框架RESTful API在调用时需要传递访问token

#### 获取当前登录用户Token

> 通过thymeleaf变量获取token

```html
<input type="hidden" name="access_token" th:value="${access_token}"></input>
```

> javascript获取token，注意thymeleaf的script标签的使用方式<script th:inline="javascript">

```javascript
<script th:inline="javascript">
	var token = '"'+/*[[${access_token}]]*/+'"'
</script>
```

#### 提交Token

> 提交后台服务时token不允许拼接在url后以参数形式提交，只能以POST或者Header方式提交

* AJAX

  ```javascript
  $.ajax({
      type: 'POST',
      url: /*[[@{/xxxxx}]]*/,
      headers : {
      	'Authorization':'Bearer '+ /*[[${access_token}]]*/ 
  	},
      success: function(data){  
      	//调用成功
  	},
      error: function(data, textStatus, errorThrown) {
          //调用失败
      }
  });	
  ```

* FORM

  ```html
  <form name="myform" th:action="@{/xxxx}" method="post">
  	<input type="hidden" name="access_token" th:value="${access_token}"></input>
  </form>   
  ```

### 3. 角色鉴权

#### 页面鉴权

> 框架使用的是thymeleaf模版，通过th:if语法控制页面元素显示，以下片段显示了具有ROLE_ADMIN角色的用户才可以看见此<li></li>

```html
<li th:if="${#authorization.expression('hasRole(''ROLE_ADMIN'')')}">
    <!-- 具有ROLE_ADMIN角色的用户才可以看见此片段-->           
</li>    
```

#### 方法鉴权

> 使用@PreAuthorize控制

```java
@PreAuthorize("hasRole('ROLE_ADMIN')")
@RequestMapping(value = "/xxx", method = {RequestMethod.GET})
public String test(ModelMap map) throws Exception {
   // 具有ROLE_ADMIN角色的用户才可以进入此方法
}
```

**为防止越权访问，要求页面鉴权和方法鉴权结合使用**

#### 页面获取认证信息

> 在页面上可以获取认证通过的用户信息

```html
<div th:text="${user.username}">用户登录名</div>  
<div th:text="${user.truename}">用户中文名</div>
<div th:text="${user.email}">邮箱</div>
<div th:text="${user.mobile}">手机号</div>
<div th:text="${user.authorities}">具有的角色列表</div>   
<div th:text="${user.departments}">所属部门</div>
```

### 4. 忽略鉴权

> 每个项目可以通过扩展WebSecurityConfigurerAdapter类实现忽略path的定义

**注意忽略的访问路径映射方法以及后续调用的方法中无法获取认证用户**

例如：下边定义了访问/without_ma和/without_mb的时候不需要认证（不需要传递token）也可以调用

```java
package 项目包xxx.config

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
public class XXXSecurityConfiguration extends WebSecurityConfigurerAdapter {
	@Override
	public void configure(WebSecurity web) throws Exception {
	    //设置认证不拦截规则，自定义跳过认证拦截的路径
	    web.ignoring().antMatchers("/without_ma").antMatchers("/without_mb");
	}
}
```
