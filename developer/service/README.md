# Service开发手册

> 作者 王立松

通过本文档你可以了解如何开发一个Service以及如何调用Service

## 包命名

```java
package [你的service项目包路径].service
```

## 类命名

```java
[你的service项目包路径].service.XxxService.java
```

## 定义Service

```java
package [你的service项目包路径].service;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@Service
@RestController
@RequestMapping("/api/v1/test")
public class TestService {
	
	@RequestMapping(value = "/testservice", method = RequestMethod.GET)
    public ResponseEntity<RestResponse<String>> testService() {
        RestResponse<String> response = new RestResponse<String>();
        String returnStr = "I am testService";
        response.setData(returnStr);
        response.setSuccess(Boolean.TRUE);
        return new ResponseEntity<>(response, HttpStatus.OK);
    }
	
}

```

## 调用Service

> 基于Spring Cloud的Feign实现对Service的调用，可以调用注册中心的下任何一个Service

### 1. 定义FeignClient

```java
package [你的ui项目包路径].feign;

import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@FeignClient("[service项目名称]")
public interface TestClient {
	
    @RequestMapping(value = "[service项目contextPath]/api/v1/test/testservice", method = RequestMethod.GET)
    public ResponseEntity<RestResponse<String>> testService();
	
}

```

### 2. 完成Service调用

> 以下示例通过Controller完成对Service的调用

```java
package [你的ui项目包路径].controller;

import [你的ui项目包路径].feign.TestClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class TestController {

	@Autowired
	private TestClient testClient;
	
	@RequestMapping("/testservice")
	@ResponseBody
	public String testService() throws Exception{
		return testClient.testService().getBody().getData();
	}
	
}
```
