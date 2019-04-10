# 服务调用开发手册

> 作者 王立松

本手册描述如何调用已经注册到注册中心的其他服务接口，具体实现基于spring cloud的Feign，如下示例完成了[myproject](YourFirstProject.md)项目UI服务到SERVICE服务的调用

## 1. 接口定义

> SERVICE服务中定义接口

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



## 2. 客户端定义

> UI服务中定义FeignClient
>
> @FeignClient()中的myproject-service是SERVICE服务的名称 ${spring.application.name}
>
> @RequestMapping()中的myprojectservice是SERVICE服务的上下文路径 ${server.contextPath}

```java
package [你的ui项目包路径].feign;

import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@FeignClient("myproject-service")
public interface MyClient {
	
    @RequestMapping(value = "myprojectservice/api/v1/myservice/getData", method = RequestMethod.GET)
    public ResponseEntity<RestResponse<String>> getData();
	
}
```



## 3. 完成调用

> UI服务中基于上一步定义的MyClient实现对SERVICE服务接口的调用

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
