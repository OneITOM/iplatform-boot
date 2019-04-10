# 文档服务集成手册

> 作者 于胜强

>
> 框架提供DFSSClient实现文件的集中管理。

## 1. 前提

* 已经启动了注册中心
* 已经启动认证服务
* 已经启动文档服务

## 2. 参数配置

> 指定文件库与对应密钥

```yml
spring.dfss.bucketname: bomc
spring.dfss.secretkey: bomc
```

## 3. 使用DFSSClient

### 自动注入DFSSClient

```java
@Autowired(required = false)
DFSSClient dfssClient;
```


### 文件管理接口

> 上传文件

```java
# 
public String add(String fileName, File file) throws Exception
# 
public String add(String fileName, InputStream inputStream) throws Exception


```

> 下载文件

```java
# 
public File get(String fileId, String filePath) throws Exception

```

> 删除文件

```java
public void delete(String fileId) throws Exception
```

> 更新文件

```java
#
public String update(String fileId, String fileName, File file) throws Exception
#
public String update(String fileId, String fileName, InputStream inputStream) throws Exception

```

> 获取文件信息

```java
public DfssFileDO info(String fileId) throws Exception
```
