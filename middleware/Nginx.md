# Nginx部署手册

> 作者 xx

## 1. 安装

### 1.1 前提

* 磁盘剩余空间大于1GB
* 空闲内存大于1GB

### 2.2 安装

1. 下载安装包

```bash
wget http://nginx.org/download/nginx-1.14.0.tar.gz
wget http://distfiles.macports.org/openssl/openssl-1.0.1f.tar.gz
wget http://zlib.net/zlib-1.2.11.tar.gz
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.40.tar.gz
```

2. 安装依赖

   如果已安装，请忽略

```bash
yum install gcc-c++ -y
```
3. 安装Nginx及相关组件

   openssl安装

```properties
tar -xzvf openssl-1.0.1f.tar.gz
cd openssl-1.0.1f
./config && make && make install
```

​       pcre安装

```properties
tar -xzvf pcre-8.40.tar.gz
cd pcre-8.40
./config && make && make install
```

​      zlib安装

```properties
tar -xzvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./config && make && make install
```

​      Nginx安装

```properties
tar -xzvf nginx-1.14.0.tar.gz
cd nginx-1.14.0
./config && make && make install
```

4. 启动

```properties
cd /usr/local/nginx/sbin
./nginx
```

5. 验证安装

   浏览器访问http://ip是否可以登录

6. 停止

   ```properties
   cd /usr/local/nginx/sbin
   ./nginx -s stop
   ```

## 2. Docker

> 基于Docker的单节点配置

```yaml

```


