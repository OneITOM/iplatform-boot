# Controller开发手册

> 作者 王立松

本文档演示如何给[myproject](YourFirstProject.md)项目增加一个Controller

## 包命名

包以controller命名

```java
package [你的ui项目包路径].controller
```

## 类命名

类以Controller结尾，例如MyController

```java
[你的ui项目包路径].controller.MyController.java
```

## 定义Controller

> 增加注解@Controller和@RequestMapping
>
> toMypage方法完成页面跳转，getData方法返回测试数据

```java
package [你的ui项目包路径].controller;

import java.security.Principal;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import [你的ui项目包路径].feign.MyClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequestMapping("/mycontroller")
public class MyController {

    @Autowired
    private MyClient myClient;

    @RequestMapping("/toMypage")
    public String toMypage(ModelMap map, HttpServletRequest request, HttpServletResponse response, Principal principal)
		throws Exception {
	String data = myClient.getData().getBody().getData();
	map.put("data", data);
	return "pages/mypage";
    }

    @RequestMapping("/getData")
    @ResponseBody
    public String getData() throws Exception {
	return myClient.getData().getBody().getData();
    }

}

```

## 访问Controller

### 1. FORM表单请求

```html
index.html

<!DOCTYPE html>
<html lang="zh-CN" xmlns:th="http://www.thymeleaf.org"
	xmlns:layout="http://www.ultraq.net.nz/web/thymeleaf/layout"
	layout:decorator="base/mainlayout">
<body>
	<div layout:fragment="content"
		class="col-sm-9 col-sm-offset-3 col-md-10 col-md-offset-2 main">	
		<form name="testform" th:action="@{/mycontroller/toMypage}" method="GET">
			<input type="hidden" name="access_token" th:value="${access_token}"></input>
		</form>
		<a href="javascript:document.testform.submit();">testJump</a>
	</div>
</body>
</html>
```

### 2. ajax请求

```html
pages/test/test.html

<!DOCTYPE html>
<html lang="zh-CN" xmlns:th="http://www.thymeleaf.org"
	xmlns:layout="http://www.ultraq.net.nz/web/thymeleaf/layout"
	layout:decorator="base/mainlayout">
<body>
	<div layout:fragment="content"
		class="col-sm-9 col-sm-offset-3 col-md-10 col-md-offset-2 main">
		<div th:text="${testData}"></div>
		<script th:src="@{/js/plugins/jquery/jquery.min.js}"></script>
		<script type="text/javascript" th:inline="javascript">/*<![CDATA[*/	
			
		    var CONTEXT_PATH = /*[[@{/}]]*/;
	    	var ACCESS_TOKEN = /*[[${access_token}]]*/;
	    	
	    	$(function() {
	    		$.ajax({
			    type:'GET',
			    headers: {'Authorization': 'Bearer ' + ACCESS_TOKEN},
			    url: CONTEXT_PATH + 'mycontroller/getData',
			    data:{},
			    success: function(result){
			     	console.log(result);
			    }
			});
		});

		/*]]>*/
		</script>		
	</div>
</body>
</html>
```
