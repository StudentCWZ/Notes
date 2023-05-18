# Docker 常规安装简介

## 8.1 总体步骤

1. 搜索镜像
2. 拉取镜像
3. 查看镜像
4. 启动镜像
   - 服务端口映射
5. 停止容器
6. 移除容器

## 8.2 安装 tomcat

1. Docker Hub 上面查找 tomcat 镜像

   ```bash
   docker search tomcat
   ```

2. 从 Docker Hub 上拉取 tomcat 镜像到本地

   ```bash
   docker pull tomcat
   ```

3. Docker images 查看是否有拉取到的 tomcat

   ```bash
   docker images -a
   ```

4. 使用 tomcat 镜像创建容器实例(也叫运行镜像)

   - 相关命令

     ```bash
     docker run -d -p 8080:8080 --name t1 tomcat
     ```

     - `-p` 小写，主机端口:docker 容器端口
     - `-P` 大写，随机分配端口
     - `-i`: 交互
     - `-t`: 终端
     - `-d`: 后台

5. 访问 tomcat 首页

   - 问题：最后访问 404

   - 解决

     - 可能没有映射端口或者没有关闭防火墙

     - 把 webapps.dist 目录换成 webapps

       - 进入容器

         ```bash
         docker exec -it e6528d1a1837 /bin/bash
         ```

       - 进入 /usr/local/tomcat/webapps 目录

         ```bash
         cd /usr/local/tomcat/webapps/
         ```

       - 发现为空文件夹

         ```bash
         ls -al
         total 0
         drwxr-xr-x 1 root root  0 May  4 16:12 ./
         drwxr-xr-x 1 root root 16 May  4 16:12 ../
         ```

       - 删除 webapps 文件夹

         ```bash
         rm -r webapps
         ```

       - 将 webapps.dist 文件夹重命名为 webapps

         ```bash
         mv webapps.dist/ webapps
         ```

       - 修改后访问成功

6. 免修改版说明

   - 下载旧版本镜像

     ```bash
     docker pull tomcat:8.5.40
     ```

   - 运行容器

     ```bash
     docker run -d -p 8080:8080 --name mytomcat8 tomcat:8.5.40
     ```

   - 访问成功

## 8.3 安装 mysql

1. Docker Hub 上面查找 mysql 镜像

   ```bash
   docker search mysql
   ```

2. 从 Docker Hub 上(阿里云加速器)拉取镜像到本地标签为 5.7

   ```bash
   docker pull mysql:5.7
   ```

3. 使用 mysql5.7 镜像创建容器(也叫运行镜像)

   - 简单版

     - 使用 mysql 镜像

       ```bash
       docker run -p 13306:3306 -e MYSQL_ROOT_PASSWORD=localhost123 -d mysql:5.7
       ```

     - 查看容器状态

       ```bash
       docker ps
       ```

     - 进入容器实例

       ```bash
       docker exec -it e4e594617b41 /bin/bash
       ```

     - 容器内部进入数据库

       ```bash
       mysql -u root -p
       ```

     - 建库建表插入数据

       ```mysql
       mysql> create database db01;
       Query OK, 1 row affected (0.00 sec)
       mysql> use db01;
       Database changed
       mysql> create table t1(id int, name varchar(20));
       Query OK, 0 rows affected (0.02 sec)
       mysql> insert into t1 values(1, 'z3');
       Query OK, 1 row affected (0.01 sec)
       mysql> select * from t1;
       +------+------+
       | id   | name |
       +------+------+
       |    1 | z3   |
       +------+------+
       1 row in set (0.00 sec)
       ```

     - 外部也来连接运行在 docker 上的 mysql 容器实例服务

       <img src="https://studentcwz-pic-bed.oss-cn-guangzhou.aliyuncs.com/img/%E5%A4%96%E9%83%A8%E8%BF%9E%E6%8E%A5%20docker%20mysql.png" alt="外部连接 docker mysql"  />

     - 问题

       - 新插入记录无法插入中文

         ```mysql
         INSERT INTO t1 VALUES(2, '王五');
         1366 - Incorrect string value: '\xE7\x8E\x8B\xE4\xBA\x94' for column 'name' at row 1
         ```

       - Docker 上默认字符集编码隐患

       - Docker 里面 mysql 容器实例查看，内容如下(注意：这不是在 Navicat 上操作):

         ```mysql
         mysql> SHOW VARIABLES LIKE 'character%';
         +--------------------------+----------------------------+
         | Variable_name            | Value                      |
         +--------------------------+----------------------------+
         | character_set_client     | latin1                     |
         | character_set_connection | latin1                     |
         | character_set_database   | latin1                     |
         | character_set_filesystem | binary                     |
         | character_set_results    | latin1                     |
         | character_set_server     | latin1                     |
         | character_set_system     | utf8                       |
         | character_sets_dir       | /usr/share/mysql/charsets/ |
         +--------------------------+----------------------------+
         8 rows in set (0.00 sec)
         ```

       - 删除容器后，里面的 mysql 数据如何办？(引出容器数据卷)

   - 实战版

     - 新建 mysql 容器实例

       ```bash
       docker run -d -p 13306:3306 --privileged=true -v /Users/cuiweizhi/Downloads/mysql/log:/var/log/mysql -v /Users/cuiweizhi/Downloads/mysql/data:/var/lib/mysql -v /Users/cuiweizhi/Downloads/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=localhost123 --name mymysql mysql:5.7
       ```

     - 查看容器状态

       ```bash
       docker ps
       ```

     - 新建 my.cnf

       - 通过容器卷同步给 mysql 容器实例

         - 进入 /Users/cuiweizhi/Downloads/mysql/conf 目录

           ```bash
           cd /Users/cuiweizhi/Downloads/mysql/conf
           ```

         - 新建 my.cnf

           ```bash
           vim my.cnf
           ```

         - 添加如下内容

           ```text
           [client]
           default_character_set=utf8
           [mysqld]
           collation_server=utf8_general_ci
           character_set_server=utf8
           ```

     - 重新启动 mysql 容器实例再重新进入并查看字符编码

       ```bash
       docker restart mymysql
       ```

       - 进入容器

         ```bash
         docker exec -it mymysql /bin/bash
         ```

       - 容器内部进入数据库

         ```bash
         mysql -u root -p
         ```

       - 查看字符编码

         ```bash
         mysql> SHOW VARIABLES LIKE 'character%';
         +--------------------------+----------------------------+
         | Variable_name            | Value                      |
         +--------------------------+----------------------------+
         | character_set_client     | utf8                       |
         | character_set_connection | utf8                       |
         | character_set_database   | utf8                       |
         | character_set_filesystem | binary                     |
         | character_set_results    | utf8                       |
         | character_set_server     | utf8                       |
         | character_set_system     | utf8                       |
         | character_sets_dir       | /usr/share/mysql/charsets/ |
         +--------------------------+----------------------------+
         8 rows in set (0.00 sec)
         ```

     - 再新建库新建表再插入中文测试

       ```bash
       mysql> create database db01;
       Query OK, 1 row affected (0.00 sec)
       mysql> use db01;
       Database changed
       mysql> create table t1(id int, name varchar(20));
       Query OK, 0 rows affected (0.02 sec)
       mysql> insert into t1 values(1, 'z3');
       Query OK, 1 row affected (0.01 sec)
       mysql> insert into t1 values(2, '王五');
       Query OK, 1 row affected (0.01 sec)
       mysql> select * from t1;
       +------+--------+
       | id   | name   |
       +------+--------+
       |    1 | z3     |
       |    2 | 王五   |
       +------+--------+
       2 rows in set (0.00 sec)
       ```

     - 结论

       - 之前 DB 无效
       - 修改字符集操作+重启 mysql 容器实例
       - 之后 DB 有效，需要新建
       - **Docker 安装完 MySQL 并 run 出容器后，建议清先修改字符集编码再新建 mysql 库-表-插数据**

     - 假如将当前容器实例删除，再重新来一次，之前建 db01 实例还有吗？

       - 删除容器

         ```bash
         docker rm -f mymysql
         ```

       - 再次新启动一个容器

         ```bash
         docker run -d -p 13306:3306 --privileged=true -v /Users/cuiweizhi/Downloads/mysql/log:/var/log/mysql -v /Users/cuiweizhi/Downloads/mysql/data:/var/lib/mysql -v /Users/cuiweizhi/Downloads/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=localhost123 --name mymysql mysql:5.7
         ```

       - 进入容器

         ```bash
         docker exec -it mymysql /bin/bash
         ```

       - 容器内部进入数据库

         ```bash
         mysql -u root -p
         ```

       - 查看数据

         ```mysql
         mysql> use db01;
         Reading table information for completion of table and column names
         You can turn off this feature to get a quicker startup with -A

         Database changed
         mysql> select * from t1;
         +------+--------+
         | id   | name   |
         +------+--------+
         |    1 | z3     |
         |    2 | 王五   |
         +------+--------+
         2 rows in set (0.01 sec)
         ```

## 8.4 安装 redis

1. 从 Docker Hub 上(阿里云加速器)拉取 redis 镜像到本地标签为 6.0.8

   ```bash
   docker pull redis:6.0.8
   ```

2. 入门命令

   - 运行容器

     ```bash
     docker run -d -p 16379:6379 redis:6.0.8
     ```

   - 进入容器

     ```bash
     docker exec -it 9747aa694397 /bin/bash
     ```

   - 进入容器内部的使用 redis

     ```bash
     redis-cli
     127.0.0.1:6379> set k1 v1
     OK
     127.0.0.1:6379> get k1
     "v1"
     ```

3. 命令提醒：容器卷机的加入 `--privileged=true`

   - Docker 挂载主机目录 Docker 访问出现 cannot open directory: Permission denied
   - 解决方法：在挂载目录后多加一个 `--privileged=true` 参数即可

4. 在宿主机下新建目录 /Users/cuiweizhi/Downloads/app/redis

   ```bash
   mkdir -p /app/redis
   ```

5. 将一个 redis.conf 文件模板拷贝进 /Users/cuiweizhi/Downloads/app/redis 目录下

   ```bash
   cp ../../../../../usr/local/etc/redis.conf .
   ```

6. /Users/cuiweizhi/Downloads/app/redis 目录下修改 redis.conf 文件

   - 修改文件

     ```bash
     vim redis.conf
     ```

     - 运行 redis 外部连接(注释掉 127.0.0.1)

       ```text
       # bind 127.0.0.1
       ```

     - daemonize no
       - 将 daemonize yes 注释起来或者 daemon no 设置，因为该配置和 docker run 中 -d 参数冲突，会导致容器一直启动失败

7. 容器数据卷挂载

   ```bash
   docker run -p 16379:6379 --name myredis --privileged=true -v /Users/cuiweizhi/Downloads/app/redis/redis.conf:/etc/redis/redis.conf -v /Users/cuiweizhi/Downloads/app/redis/data:/data -d redis:6.0.8 redis-server /etc/redis/redis.conf
   ```

8. 查看容器状态

   ```bash
   docker ps
   ```

9. 进入容器实例

   ```bash
   docker exec -it myredis /bin/bash
   ```

10. 使用 redis-cli 连接上来

    ```bash
    redis-cli
    127.0.0.1:6379> set k1 v1
    OK
    127.0.0.1:6379> get k1
    "v1"
    127.0.0.1:6379> ping
    PONG
    127.0.0.1:6379> select 15
    OK
    127.0.0.1:6379[15]> select 10
    OK
    ```

11. 请证明 docker 启动使用了我们指定的配置文件

    - 修改 redis.conf(容器外部)

      ```bash
      vim redis.conf
      ```

    - 修改数据数量为 10

    - 重启容器

      ```bash
      docker restart myredis
      ```

    - 进入容器实例

      ```bash
      docker exec -it myredis /bin/bash
      ```

    - 使用 redis-cli 连接上来(第二次)

      ```bash
      redis-cli
      127.0.0.1:6379> select 15
      (error) ERR DB index is out of range
      ```

    - 证明 docker 启动使用了我们指定的配置文件
