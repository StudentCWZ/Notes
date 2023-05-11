# Docker 常用命令

## 3.1 帮助启动类命令

1. 启动 Docker

   ```bash
   systemctl start docker
   ```

2. 停止 Docker

   ```bash
   systemctl stop docker
   ```

3. 重启 Docker

   ```bash
   systemctl restart docker
   ```

4. 查看 Docker 状态

   ```bash
   systemctl status docker
   ```

5. 开机启动

   ```bash
   systemctl enable docker
   ```

6. 查看 Docker 概要信息：

   ```bash
   docker info
   ```

7. 查看 Docker 总体帮助文档

   ```bash
   docker --help
   ```

8. 查看 Docker 命令帮助文档

   ```bash
   docker 具体命令 --help
   ```

## 3.2 镜像命令

### 3.2.1 docker images

1. 列出本地主机上的镜像

   ```bash
   docker images
   ```

2. 各个选项说明

   - REPOSITORY: 表示镜像的仓库源
   - TAG: 镜像的标签
   - IMAGE ID: 镜像 ID
   - CREATED: 镜像创建时间
   - SIZE: 镜像大小

3. 同一仓库源可以有多个 TAG 版本，代表这个仓库源的不同个版本，我们使用 REPOSITORY:TAG 来定义不同的镜像

4. 如果你不指定一个镜像的版本标签，例如你只使用 ubuntu，Docker 将默认使用 ubuntu:latest 镜像

5. OPTIONS 说明

   - -a: 列出本地所有的镜像(含历史映像层)
   - -q: 只显示镜像 ID

### 3.2.2 docker search

1. 网站(<https://hub.docker.com> 或阿里云)

2. 命令

   - docker search [OPTIONS] 镜像名字

   - 不带 OPTIONS 查询

     ```bash
     docker search hello-world
     ```

   - 展示结果说明

     |    参数     |       说明       |
     | :---------: | :--------------: |
     |    NAME     |     镜像名称     |
     | DESCRIPTION |     镜像说明     |
     |    STARS    |     点赞数量     |
     |  OFFICIAL   |   是否是官方的   |
     |  AUTOMATED  | 是否是自动构建的 |

   - OPTIONS 说明

     - `--limit`: 只列出 N 个镜像，默认为 25 个

       ```bash
       docker search --limit 5 hello-world
       ```

### 3.2.3 docker pull

1. docker pull 命令作用：下载镜像

2. docker pull 镜像名字:TAG

3. docker pull 镜像名字

   - 没有 TAG 就是最新版

   - 等价于：docker pull 镜像名字:latest

     ```bash
     docker pull hello-world
     ```

4. 带 TAG 的镜像下载

   ```bash
   docker pull redis:6.0.8
   ```

### 3.2.4 docker system df

1. 查看镜像/容器/数据卷所占的空间

   ```bash
   docker system df
   ```

2. 展示结果说明

   |    参数     |   说明   |
   | :---------: | :------: |
   |    TYPE     |   类型   |
   |    TOTAL    |   数量   |
   |   ACTIVE    |  活动中  |
   |    SIZE     |   大小   |
   | RECLAIMABLE | 可伸缩性 |

### 3.2.5 docker rmi

1. 删除某个镜像

   ```bash
   docker rmi hello-world
   ```

2. 强制删除

   ```bash
   docker rmi -f hello-world
   ```

3. 删除多个

   ```bash
   docker rmi -f 镜像名1:TAG 镜像名2:TAG
   ```

4. ==删除全部==

   ```bash
   docker rmi -f $(docker image -qa)
   ```

### 3.2.6 docker 虚悬镜像

1. 仓库名、标签名都是 None 的镜像，俗称虚悬镜像(dangling image)

## 3.3 容器命令

### 3.3.1 前提

1. 有镜像才能创建容器，这是根本前提(下载一个 CentOS 或者 Ubuntu 镜像演示)
   - docker pull centos
   - docker pull ubuntu
   - 本次演示用 ubuntu 演示

### 3.3.2 新建 + 启动容器

1. 相关命令

   ```bash
   docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
   ```

2. OPTIONS 说明

   - 有些是一个减号，有些是两个减号

     - `--name="容器新名字"`: 为容器指定一个名称
     - `-d`: 后台运行容器并返回容器 ID，也即启动守护式容器(后台运行)
     - `-i`: ==以交互模式运行容器，通常与 -t 同时使用==
     - `-t`: ==为容器重新分配一个伪输入终端，通常与 -i 同时使用，即启动交互式容器(前台有伪终端，等待交互)==
     - `-P`: 随机端口映射，大写 P
     - `-p`: 指定端口映射，小写 p

   - 说明

     |             参数              |                说明                |
     | :---------------------------: | :--------------------------------: |
     |   -p hostPort:containerPort   |       端口映射 -p 8080:8080        |
     | -p ip:hostPort:containerPort  | 配置监听地址 -p 10.0.0.100:8080:80 |
     |     -p ip::containerPort      |   随机分配端口 -p 10.0.0.100::80   |
     | -p hostPort:containerPort:udp |     指定协议 -p 8080:8080 tcp      |
     |      -p 81:80 -p 443:443      |              指定多个              |

3. **启动交互式容器(前台命令行)**

   ```bash
   docker run -it --name=myubuntu ubuntu:latest /bin/bash
   ```

   - 使用镜像 ubuntu:latest 以**交互模式**启动一个容器，在容器内执行 /bin/bash 命令
   - 参数说明
     - `-i`: 交互式操作
     - `-t`: 终端
     - `ubuntu`: ubuntu 镜像
     - `/bin/bash`: 放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash，要退出终端，直接输入 exit

### 3.3.3 列出当前所有正在运行的容器

1. 相关命令

   ```bash
   docker ps [OPTIONS]
   ```

2. OPTIONS 说明
   - `-a`: 列出当前所有**正在运行**的容器+**历史上运行过**的
   - `-l`: 显示最近创建的容器
   - `-n`: 显示最近 n 个创建的容器
   - `-q`: **静默模式，只显示容器编号**

### 3.3.4 退出容器

1. 两种退出方式
   - exit: run 进去容器，exit 退出，容器停止
   - ctrl+p+q: run 进去容器，ctrl+p+q 退出，容器不停止

### 3.3.5 启动已停止运行的容器

1. 相关命令

   ```bash
   docker start 容器ID或者容器名
   ```

### 3.3.6 重启容器

1. 相关命令

   ```bash
   docker restart 容器ID或者容器名
   ```

### 3.3.7 停止容器

1. 相关命令

   ```bash
   docker stop 容器ID或者容器名
   ```

### 3.3.8 强制停止容器

1. 相关命令

   ```bash
   docker kill 容器ID或者容器名
   ```

### 3.3.9 删除已停止的容器

1. 相关命令

   ```bash
   docker rm 容器ID或者容器名
   ```

2. 强制删除

   ```bash
   docker rm -f 容器ID或者容器名
   ```

3. **一次性删除多个容器实例**
   - `docker rm -f $(docker ps -qa)`
   - `Docker ps -qa | xargs docker rm`

### 3.3.10 重要

#### 3.3.10.1 启动守护式容器(后台服务器)、

1. 在大部分场景下，我们希望 docker 的服务是在后台运行的，我们可以通过 -d 指定容器的后台运行模式

2. 相关命令

   ```bash
   docker run -d 容器名
   ```

3. **注意**：

   - 很重要的要说明的一点：**Docker 容器后台运行，就必须要有一个前台进程**
   - 容器运行的命令如果不是那些**一直挂起的命令**(比如运行 top，tail)，就是会自动退出的
   - 这个是 Docker 的机制问题，比如你的 Web 容器，我们以 nginx 为例，正常情况下，我们配置启动服务只需要启动响应的 service 即可。例如，service nginx start。但是，这样做，nginx 为后台进程模式运行，就会导致 Docker 前台没有运行的应用，这样的容器后台启动后，会立即自杀因为他觉得他没事可做了。
   - 所以最佳的解决方案是，**将你要运行的程序以前台进程的形式运行，常见就是命令行模式，表示我还有交互操作，别中断**

4. redis 前后台启动

   - 前台交互式启动

     ```bash
     docker run -it redis:6.0.8
     ```

   - 后台守护式启动

     ```bash
     docker run -d redis:6.0.8
     ```

#### 3.3.10.2 查看容器日志

1. 相关命令

   ```bash
   docker logs 容器ID
   ```

#### 3.3.10.3 查看容器内运行的进程

1. 相关命令

   ```bash
   docker top 容器ID
   ```

#### 3.3.10.4 查看容器内部细节

1. 相关命令

   ```bash
   docker inspect 容器ID
   ```

#### 3.3.10.5 进入正在运行的容器并以命令行交互

1. 相关命令

   ```bash
   docker exec -it 容器ID bashShell
   ```

2. 重新进入

   ```bash
   docker attach 容器ID
   ```

3. **两者区别**
   - **attach 直接进入容器启动命令的终端。不会启动新的进程，用 exit 退出，会导致容器停止**
   - **exec 是在容器中打开新的终端，并且可以启动新的进程，用 exit 退出，不会导致容器的停止**
4. 推荐大家使用 docker exec 命令，因为退出容器终端，不会导致容器的停止
5. 用之前的 redis 容器实例进入试试
   - docker exec -it 容器ID /bin/bash
   - docker exec -it 容器ID redis-cli
   - 一般使用 -d 后台启动的程序，再用 exec 进入对应容器实例

#### 3.3.10.6 从容器内拷贝文件到主机上

1. 容器 -> 主机

2. 相关命令

   ```bash
   docker cp 容器ID:容器内路径 目的主机的路径
   ```

#### 3.3.10.7 导入和导出容器

1. export 导出容器的内容流作为一个 tar 归档文件[对应 import 命令]
2. import 从 tar 包中的内容创建一个新的文件系统再导入为镜像[对应 export]
3. 案例
   - `docker export 容器ID > 文件名.tar`
   - `cat 文件名.tar ｜ docker import -镜像用户/镜像名:镜像版本号`

## 3.4 总结

1. Docker 常用命令图解

   <img src="https://studentcwz-pic-bed.oss-cn-guangzhou.aliyuncs.com/img/Docker%20commands.png" alt="Docker commands" style="zoom: 67%;" />

2. Docker 常用命令表格

   |  命令   |                           英文说明                           |                           中文说明                           |
   | :-----: | :----------------------------------------------------------: | :----------------------------------------------------------: |
   | attach  |                Attach to a running container                 |            当前 shell 下 attach 连接指定运行镜像             |
   |  build  |                Build an image from Dockerfile                |                   通过 Dockerfile 定制镜像                   |
   | commit  |         Create a new image from a container changes          |                    提交当前容器为新的镜像                    |
   |   cp    | Copy files/folders from the containers filesystem to the host path |              从容器中拷贝指定文件或目录到宿主机              |
   | create  |                    create a new container                    |            创建一个新的容器，同 run，但不启动容器            |
   |  diff   |         Inspect changes on a container's filesystem          |                     查看 docker 容器变化                     |
   | events  |             Get real time events from the server             |                从 docker 服务获取容器实时事件                |
   |  exec   |             Run a command in existing container              |                   在已存在的容器上运行命令                   |
   | export  |     Stream the contents of a container as a tar archive      |      导出容器的内容流作为一个 tar 归档文件[对应 import]      |
   | history |                 Show the history of an image                 |                     展示一个镜像形成历史                     |
   | images  |                         List images                          |                       列出系统当前镜像                       |
   | import  | Create a new filesystem image from the contents of a tarball |    从 tar 包中的内容创建一个新的文件系统映像[对应 export]    |
   |  info   |               Display system-wide information                |                       显示系统相关信息                       |
   | inspect |         Return low-level information on a container          |                       查看容器详细信息                       |
   |  kill   |                   Kill a running container                    |                    kill 指定 docker 容器                     |
   |  load   |               Load an image from a tar archive               |            从一个 tar 包中加载一个镜像[对应 save]            |
   |  login  |       Register or Login to the docker registry server        |                注册或登录一个 docker 源服务器                |
   | logout  |              Logout from Docker registry server              |                 从当前 Docker Registry 退出                  |
   |  logs   |                Fetch the logs of a container                 |                     输出当前容器日志信息                     |
   |  port   | Lookup the public-facing port which is NAT-ed to PRIVATE_PORT |               查看映射端口对应的容器内部源端口               |
   |  pause  |            Pause all processes within a container            |                           暂停容器                           |
   |   ps    |                       List containers                        |                         列出容器列表                         |
   |  pull   |  Pull an image or a repository from docker registry server   |          从 docker 镜像源服务器拉取指定镜像或库镜像          |
   |  push   | Push an image or a repository to the docker registry server  |           推出指定镜像或者库镜像至 docker 源服务器           |
   | restart |                 Restart a running container                  |                        重启运行的容器                        |
   |   rm    |                Remove one or more containers                 |                     移除一个或者多个容器                     |
   |   rmi   |                  Remove one or more images                   | 移除一个或多个镜像[无容器使用该镜像才可以删除，否则需要删除相关容器才可继续或 -f 强制删除] |
   |   run   |               Run a command in a new container               |                创建一个新的容器并运行一个命令                |
   |  save   |                Save an image to a tar archive                |             保存一个镜像为一个 tar 包[对应 load]             |
   | search  |            Search for an image on the Docker Hub             |                   在 docker Hub 中搜索镜像                   |
   |  start  |                   Start stopped containers                   |                           启动容器                           |
   |  stop   |                   Stop running containers                    |                           停止容器                           |
   |   tag   |                Tag an image into a repository                |                       给源中镜像打标签                       |
   |   top   |         Lookup the running processes of a container          |                   查看容器中运行的进程信息                   |
   | unpause |                  Unpause a paused container                  |                         取消暂停容器                         |
   | version |             Show the docker version information              |                      查看 docker 版本号                      |
   |  wait   |   Block until a containers stops，then print its exit code    |                  截取容器停止时的退出状态值                  |
