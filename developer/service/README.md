# Service开发手册

> 作者 王立松

本文档演示如何给[myproject](YourFirstProject.md)项目增加一个Service

## 包命名

> 包以service命名

```java
package [你的service项目包路径].service
```

## 类命名

> 类以Service结尾，例如MyService

```java
[你的service项目包路径].service.MyService.java
```

## 定义Service

> 增加注解@Service、@RestController和@RequestMapping

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
@RequestMapping("/api/v1/myservice")
public class MyService {
	
    @RequestMapping(value = "/getData", method = RequestMethod.GET)
    public ResponseEntity<RestResponse<String>> getData() {
        RestResponse<String> response = new RestResponse<String>();
        String returnStr = "I am service";
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
public interface MyClient {
	
    @RequestMapping(value = "[service项目contextPath]/api/v1/myservice/getData", method = RequestMethod.GET)
    public ResponseEntity<RestResponse<String>> getData();
	
}

```

### 2. 完成Service调用

> 以下示例为Controller对Service的调用

```java
package [你的ui项目包路径].controller;

import [你的ui项目包路径].feign.MyClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequestMapping("/mycontroller")
public class MyController {

    @Autowired
    private MyClient myClient;
	
    @RequestMapping("/getData")
    @ResponseBody
    public String getData() throws Exception{
	return myClient.getData().getBody().getData();
    }
	
}
```
