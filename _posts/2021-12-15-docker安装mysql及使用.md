---
title: 使用Docker搭建MySQL服务
tags: Docker
---

# 使用Docker搭建MySQL服务

### 一、安装docker

* Windows 可以直接到官网下载 `docker desktop`

* Arch linux  `yay -S docker` 

可以在shell中输入以下命令检查是否成功安装： `sudo docker version`

### 二、建立镜像

1. 拉取官方镜像（我们这里选择5.7，如果不写后面的版本号则会自动拉取最新版）

   ```shell
   docker pull mysql:5.7   # 拉取 mysql 5.7
   docker pull mysql       # 拉取最新版mysql镜像
   ```

   [MySQL文档地址](https://hub.docker.com/_/mysql/)

2. 检查是否拉取成功

   ```shell
   $ sudo docker images
   ```

3. 一般来说数据库容器不需要建立目录映射

   ```shell
   $ sudo docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
   ```

   - –name：容器名，此处命名为`mysql`
   - -e：配置信息，此处配置mysql的root用户的登陆密码
   - -p：端口映射，此处映射 主机3306端口 到 容器的3306端口
   - -d：后台运行容器，保证在退出终端后容器继续运行

4. 如果要建立目录映射

   ```shell
   $ docker run --privileged=true -p 3306:3306 --name mysql5 \                
   -v ~/docker/mysql5/conf:/etc/mysql/conf.d \
   -v ~/docker/mysql5/logs:/var/log/mysql \
   -v ~/docker/mysql5/data:/var/lib/mysql \
   -e MYSQL_ROOT_PASSWORD=123456 \
   -d mysql:5.7
   ```

   - -v：主机和容器的目录映射关系，":"前为主机目录，之后为容器目录


1. 检查容器是否正确运行

   ```shell
   $ docker container ls
   ```

   - 可以看到容器ID，容器的源镜像，启动命令，创建时间，状态，端口映射信息，容器名字

### 三、连接mysql

1. 进入docker本地连接mysql客户端

   ```shell
   $ sudo docker exec -it mysql bash
   mysql -uroot -p123456
   ```

2. 使用远程连接软件时要注意一个问题

   我们在创建容器的时候已经将容器的3306端口和主机的3306端口映射到一起，所以我们应该访问：

   ```
   host: 127.0.0.1
   port: 3306
   user: root
   password: 123456
   ```

3. 如果你的容器运行正常，但是无法访问到MySQL，一般有以下几个可能的原因：

   - 防火墙阻拦

     ```shell
     # 开放端口：
     $ systemctl status firewalld
     $ firewall-cmd  --zone=public --add-port=3306/tcp -permanent
     $ firewall-cmd  --reload
     # 关闭防火墙：
     $ sudo systemctl stop firewalld
     ```

   - 需要进入docker本地客户端设置远程访问账号

     ```shell
     $ sudo docker exec -it mysql bash
     $ mysql -uroot -p123456
     mysql> grant all privileges on *.* to root@'%' identified by "你要的密码";
     ```

     原理：

     ```shell
     # mysql使用mysql数据库中的user表来管理权限，修改user表就可以修改权限（只有root账号可以修改）
     
     mysql> use mysql;
     Database changed
     
     mysql> select host,user,password from user;
     +--------------+------+-------------------------------------------+
     | host                    | user      | password                                                                 |
     +--------------+------+-------------------------------------------+
     | localhost              | root     | *A731AEBFB621E354CD41BAF207D884A609E81F5E      |
     | 192.168.1.1            | root     | *A731AEBFB621E354CD41BAF207D884A609E81F5E      |
     +--------------+------+-------------------------------------------+
     2 rows in set (0.00 sec)
     
     mysql> grant all privileges  on *.* to root@'%' identified by "password";
     Query OK, 0 rows affected (0.00 sec)
     
     mysql> flush privileges;
     Query OK, 0 rows affected (0.00 sec)
     
     mysql> select host,user,password from user;
     +--------------+------+-------------------------------------------+
     | host                    | user      | password                                                                 |
     +--------------+------+-------------------------------------------+
     | localhost              | root      | *A731AEBFB621E354CD41BAF207D884A609E81F5E     |
     | 192.168.1.1            | root      | *A731AEBFB621E354CD41BAF207D884A609E81F5E     |
     | %                       | root      | *A731AEBFB621E354CD41BAF207D884A609E81F5E     |
     +--------------+------+-------------------------------------------+
     3 rows in set (0.00 sec)
     ```



4. 如果你的mysql容器无法插入中文数据

```shell
cd [你挂载的容器conf.d目录]
#按照上方docker run的话我这里是cd ~/docker/mysql5/conf
vim my.cnf
#插入如下
[client]                                    #针对客户端的设置
default-character-set = utf8    #指定字符集(mariadb默认是拉丁文)

[mysqld]                               #设置mysql-server相关信息
collation_server = utf8_general_ci   
character_set_server = utf8

#重启mysql
docker restart mysql5
#之后再新建数据库！！！！！更改之后旧的数据库还是不能插入中文
#示例
create database gofaquan;

use gofaquan;

create table test_utf8_again(name varchar(20));

INSERT INTO gofaquan.test_utf8_again (name)
VALUES (' 你爸爸');

#插入成功
```

