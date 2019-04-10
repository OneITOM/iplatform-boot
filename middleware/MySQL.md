# MySQL部署手册

> 作者 杜臻

## 1. 安装

### 1.1 前提

* 磁盘剩余空间大于5G

* 空闲内存大于2GB

* perl  perl-devel  autoconf  libaio  perl-Data-Dumper 依赖rpm包需要提前安装

  ```properties
  # 安装依赖
  yum -y install perl perl-devel autoconf libaio
  ```

  ------

### 1.2  单节点安装

1. ### 下载安装包

   ```properties
   #下载地址
   https://dev.mysql.com/downloads/mysql/5.6.html#downloads
   选择版本5.6.42
   选择系统Linux-Generic
   下载mysql-5.6.42-linux-glibc2.12-x86_64.tar.gz
   ```

2. 解压缩

   把mysql-5.6.42-linux-glibc2.12-x86_64.tar.gz安装包上传到/usr/local下解压

   ```properties
   tar -xzvf mysql-5.6.42-linux-glibc2.12-x86_64.tar.gz
   mv mysql-5.6.42-linux-glibc2.12-x86_64 mysql
   ```

3. 创建用户组

   ```properties
   groupadd mysql
   useradd mysql -g mysql
   ```

4. 安装

   使用root安装

   ```properties
   cd /usr/local/mysql
   ./scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
   ```

5. 修改目录权限

   ```properties
   cd /usr/local
   chown -R mysql:mysql mysql
   ```

6. 添加开启启动

   ```properties
   cp support-files/mysql.server /etc/init.d/mysql
   # 赋予可执行权限
   chmod +x /etc/init.d/mysql
   # 添加服务
   chkconfig --add mysql 
   # 显示服务列表
   chkconfig --list 
   ```

7. 修改配置

   ```properties
   #修改/etc/my.cnf
   [mysqld]
   skip-name-resolve
   port=3306
   socket=/tmp/mysql.sock
   basedir=/usr/local/mysql
   datadir=/usr/local/mysql/data
   max_connections=500
   character-set-server=utf8
   default-storage-engine=INNODB
   lower_case_table_name=1
   max_allowed_packet=16M
   sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
   user=mysql
   max_allowed_packet=16M
   symbolic-links=0
   ```

8. 启停

   ```properties
   service mysql start
   service mysql stop
   ```

9. 设置密码

   ```properties
   #注意：password可以根据现场需要修改
   /usr/local/mysql/bin/mysqladmin -u root password 'root'
   ```

10. 设置允许远程访问

    ```properties
    #新建用户远程连接mysql数据库
    grant all on *.* to bomc@'%' identified by 'bomc' with grant option; 
    flush privileges;
    #允许任何ip地址(%表示允许任何ip地址)使用bomc帐户和密码(bomc)来访问这个mysql server。
    
    #支持root用户允许远程连接mysql数据库
    grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;
    flush privileges;
    ```

## 2. Docker

> 基于Docker的单节点配置

```yaml
version: '3.2'
services:
  oneitom-mysql:
    image: boco/alpine-mysql:10.1.28
    hostname: oneitom-mysql
    container_name: oneitom-mysql
    restart: always
    networks:
      - oneitom-network
    ports:
      - '3306:3306'
    volumes:
      - ${ONEITOM_VOLUME_PATH}/oneitom-mysql:/app:rw
    environment:
      - TZ=Asia/Shanghai
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=authdb,notifydb,gulfdataflowdb
      - MYSQL_USER=authdb-user,notifydb-user,gulfdataflowdb-user
      - MYSQL_PASSWORD=authdb-password,notifydb-password,gulfdataflowdb-password
    labels:
     - oneitom-mysql-cluster
networks:
  oneitom-network:
    external: true
```


