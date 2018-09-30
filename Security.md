# 安全相关配置手册

本文档主要讲解于项目安全相关的配置和开发指南

- [配置文件加密](#1)
- [安全漏洞](#2)
  - [SQL注入漏洞](#2.1)
  - [CSRF跨站请求伪造](#2.2)
  - [点击劫持](#2.2)
  - [不安全的HTTP方法](#2.4)
  - [SSL/TLS受诫礼(BAR-MITZVAH)攻击漏洞](#2.5)
  - [SpringBoot配置不当敏感信息泄漏](#2.6)
  - [Slow HTTP Denial of Service Attack漏洞](#2.7)



## 配置文件加密
> 配置文件中或者启动参数的参数值可以使用加密方式配置

例如以下是未加密的数据库密码配置参数

```properties
spring.datasource.password=123456
```

可以使用密码生成工具生成明文对应的密文后配置成密文格式

```shell
java -jar enctool-0.0.1-SNAPSHOT.jar "123456" 
======================================================================  
微服务配置文件密文生成工具  java -jar enctool-0.0.1-SNAPSHOT.jar 要加密的明文 
----------------------------------------------------------------------         
Copyright© BOCO  
====================================================================== 
明文:123456 
密文:ENC(mBaGBXPu1VFgECoBH5NGWeTdFLy79Ic5)
```

使用生成的密文配置数据库密码的加密配置方式

```properties
spring.datasource.password=ENC(mBaGBXPu1VFgECoBH5NGWeTdFLy79Ic5)
```

## <span id="2"/>安全漏洞

安全漏洞主要是目前已知的一些漏洞的配置或者开发注意事项，大部分漏洞在框架级都有默认配置

### <span id="2.1"/>SQL注入漏洞

> 本框架使用的是Mybatis，所以SQL注入漏洞主要是针对此框架

* 在SQL中尽量使用#，避免使用$
* 如果必须使用$，那么可以通过在Mapper方法上使用@SQLInjection实现检查实现检查和策略

@SQLInjection参数说明

1. value 敏感词定义，可以不填写，默认敏感词如下

```java
String[] value() default {";","'","\"","\\(","\\)","and","or","union","where","limit","select","delete","substr","group","by"};
```

2. ignore 敏感词忽略，默认空，用于要排除的默认敏感词定义

例如要忽略select敏感词，那么使用如下配置

```java
@SQLInjection(ignore = {"select"})
```
3. regex 正则敏感词定义，默认空，使用正则表达式定义敏感词，次参数不可和value, ingore同时使用

例如：要检查参数中是否包含 select 和 >

```java
@SQLInjection(regex = "(select)|(>)")
```


4. policy 检查到疑似注入后的处理策略，默认策略是打印警告日志，可以通过配置终止策略SQLInjectionPolicy.BREAK达到紧致此SQL中断执行的目的

例如：以下策略将在发现疑似注入漏洞发生时中断SQL执行

```java
@SQLInjection(policy = SQLInjectionPolicy.BREAK)
```

### <span id="2.2"/>CSRF跨站请求伪造
### <span id="2.3"/>点击劫持
### <span id="2.4"/>不安全的HTTP方法
### <span id="2.5"/>SSL/TLS受诫礼(BAR-MITZVAH)攻击漏洞
### SpringBoot配置不当敏感信息泄漏 <span id="2.6">2.6</span> 
### Slow HTTP Denial of Service Attack漏洞

<span id="2.7">2.7</span>