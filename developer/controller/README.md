# Controller开发手册

> 作者 王立松

通过本文档你可以了解如何开发一个Controller以及如何访问Controller

## 包命名

```java
package [你的ui项目包路径].controller
```

## 类命名

```java
[你的ui项目包路径].controller.XxxController.java
```

## 定义Controller

> showtest 完成页面跳转，同时将测试数据返回到前端
>
> getTestData 供ajax调用，返回测试数据

```java
package [你的ui项目包路径].controller;

import java.security.Principal;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import [你的ui项目包路径].feign.TestClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequestMapping("/test")
public class TestController {

	@Autowired
	private TestClient testClient;

	@RequestMapping("/showtest")
	public String showtest(ModelMap map, HttpServletRequest request, HttpServletResponse response, Principal principal)
			throws Exception {
		String data = testClient.testService().getBody().getData();
		map.put("testData", data);
		return "pages/test/test";
	}

	@RequestMapping("/getTestData")
	@ResponseBody
	public String getTestData() throws Exception {
		return testClient.testService().getBody().getData();
	}

}

```

## 访问Controller

### 1. 页面跳转

```html
index.html

<!DOCTYPE html>
<html lang="zh-CN" xmlns:th="http://www.thymeleaf.org"
	xmlns:layout="http://www.ultraq.net.nz/web/thymeleaf/layout"
	layout:decorator="base/mainlayout">
<body>
	<div layout:fragment="content"
		class="col-sm-9 col-sm-offset-3 col-md-10 col-md-offset-2 main">	
		<form name="testform" th:action="@{/test/showtest}" method="GET">
			<input type="hidden" name="access_token" th:value="${access_token}"></input>
		</form>
		<a href="javascript:document.testform.submit();">testJump</a>
	</div>
</body>
</html>
```

### 2. ajax调用

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
			     url: CONTEXT_PATH + 'test/getTestData',
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
