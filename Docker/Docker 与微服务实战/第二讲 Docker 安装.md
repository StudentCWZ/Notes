# Docker 安装

## 2.1 前提说明

### 2.1.1 CentOS Docker 安装

1. Docker 并非是一个通用的容器工具，它依赖于已存在并运行的 Linux 内核环境。
2. Docker 实际上是在已经运行的 Linux 下制造了一个隔离的文件环境，因此它执行的效率几乎等同于所部署的 Linux 主机。
3. 因此，Docker 必须部署在 Linux 内核系统上。如果其他系统想部署 Docker 就必须安装一个虚拟 Linux 环境。
4. 在 Windows 上部署 Docker 的方法都是先安装一个虚拟机，并在安装 Linux 系统的虚拟机中运行 Docker


### 2.1.2 前提条件

1. 目前，CentOS 仅发行版本中的内核支持 Docker。
2. Docker 运行在 CentOS 7(64 bit) 上，要求系统为 64 位、Linux 系统内核版本为 3.8 以上，这里选用 CentOS 7.x

### 2.1.3 查看自己的内核

1. 查看自己内核：**uname** 命令用于打印当前系统相关信息(内核版本号、硬件架构、主机名和操作系统类型等)

   ```bash
   [root@atguifu sysconfig] # uname -r
   3.10.0-1127.19.1.el7.x86_64
   [root@atguifu sysconfig] #
   ```

## 2.2 Docker 基本组成

### 2.2.1 镜像

1. Docker 镜像(image)就是一个**只读**的模板。镜像可以用来创建 Docker 容器，**一个镜像可以创建很多容器**。

2. 它也相当于是一个 root 文件系统。比如官方镜像 CentOS:7 就包含了完整的一套 CentOS:7 最小系统的 root 文件系统。

3. 相当于容器的源代码，**Docker 镜像文件类似于 Java 的类模板，而 Docker 容器实例类似于 Java 中 new 出来的实例对象**

   <center><b>容器与镜像的关系类似于面向对象编程中的对象与类</b></center>

   | Docker | 面向对象 |
   | :----: | :------: |
   |  容器  |   对象   |
   |  镜像  |    类    |

### 2.2.2 容器

1. **从面向对象角度**
   - Docker 利用容器(Container)独立运行的一个或一组应用，应用程序或服务运行在容器里面，容器就类似于一个虚拟化的运行环境，**容器就是用镜像创建的运行实例**。
   - 就像是 Java 中类和实例对象一样，镜像是静态的定义，容器是镜像运行时的实体。
   - 容器为镜像提供了一个标准化的和隔离的运行环境，它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。
2. **从镜像容器角度**
   - 可以把容器看做是一个简易版的 Linux 环境(包括 root 用户权限、进程空间、用户空间和网络空间等)和运行在其中的应用程序。

### 2.2.3 仓库

1. 仓库(Repository)是集中存放镜像文件的场所。
2. 仓库(Repository)和仓库注册服务器(Registry)是有区别的。
3. 仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签(tag)。
4. 仓库分为公开仓库(Public)和私有仓库(Private)两种形式。
5. 最大的公开仓库是 [Docker Hub](<https://hub.dcoker.com/>)，存放了数量庞大的镜像供用户下载。
6. 国内的公开仓库包括阿里云、网易云等。

### 2.2.4 小结

1. Docker 本身是一个容器运行载体或称之为管理引擎。我们把应用程序和配置依赖打包好形成一个可交付的运行环境，这个打包好的运行环境就是 image 镜像文件。只有通过这个镜像文件才能生成 Docker 容器实例(类似 Java 中 new 出来一个对象)。
2. image 文件可以看作容器的模板。Docker 根据 image 文件生成容器的实例。同一个 image 文件，可以生成多个同时运行的容器实例。
3. **镜像文件**
   - image 文件生成的容器实例，本身也是一个文件，成为镜像文件
4. **容器实例**
   - 一个容器运行一种服务，当我们需要的时候，就可以通过 docker 客户端创建一个对应的运行实例，也就是我们的容器。
5. **仓库**
   - 至于仓库，就是一堆镜像的地方，我们可以把镜像发布到仓库中，需要的时候从仓库中拉下来就可以。

## 2.3 Docker 平台架构

1. **Docker 平台架构(入门版)**

   <img src="https://studentcwz-pic-bed.oss-cn-guangzhou.aliyuncs.com/img/Docker%20%E5%B9%B3%E5%8F%B0%E6%9E%B6%E6%9E%84%20--%20%E5%85%A5%E9%97%A8%E7%89%88.png" alt="Docker 平台架构 -- 入门版" style="zoom:80%;" />

2. **Docker 工作原理**

   - Docker 是一个 Client-Server 结构的系统，Docker 守护进程运行在主机上，然后通过 Socket 连接从客户端访问，守护进程从客户端接受命令并管理运行在主机上的容器。

   - **容器，是一个运行时环境，就是我们前面说到的集装箱。**

     <img src="https://studentcwz-pic-bed.oss-cn-guangzhou.aliyuncs.com/img/Docker%20%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86.png" alt="Docker 工作原理" style="zoom: 50%;" />

3. **Docker 平台架构(架构版)**

<img src="https://studentcwz-pic-bed.oss-cn-guangzhou.aliyuncs.com/img/Docker%20%E5%B9%B3%E5%8F%B0%E6%9E%B6%E6%9E%84-%E6%9E%B6%E6%9E%84%E7%89%88.png" alt="Docker 平台架构-架构版" style="zoom: 50%;" />

4. **Docker 整体架构及底层通信原理简述**
   - Docker 是一个 C/S 模式的架构。后端是一个松耦合架构，众多模块各司其职
   - Docker 运行的基本流程为：
     - 用户是使用 Docker Client 与 Docker Daemon 建立通信，并发送请求给后者
     - Docker Daemon 作为 Docker 架构中的主体部分，首先提供 Docker Server 的功能使其可以接受 Docker Client 的请求。
     - Docker Engine 执行 Docker 内部的一系列工作，每一项工作都是以一个 Job 的形式存在
     - Job 的运行过程中，当需要容器镜像时，则从 Docker Registry 中下载镜像，并通过镜像管理驱动 Graph driver 将下载镜像以 Graph 的形式存储
     - 当需要为 Docker 创建网络环境时，通过网络管理驱动 Network driver 创建并配置 Docker 容器网络环境。
     - 当需要限制 Docker 容器运行资源或执行用户指令等操作时，则通过 Exec driver 来完成
     - Libcontainer 是一项独立的容器管理包，Network driver 以及 Exec driver 都是通过 Libcontainer 来实现具体对容器进行的操作

## 2.4 安装步骤

### 2.4.1 CentOS 7 安装 Docker

1. [安装地址](<https://docs.docker.com/engine/install/centos/>)

2. **Docker 安装步骤**

   - 确定你是 CentOS 7 及以上版本

     ```bash
     cat /etc/redhat-release
     ```

   - 卸载旧版本

     ```bash
     sudo yum remove docker \
                       docker-client \
                       docker-client-latest \
                       docker-common \
                       docker-latest \
                       docker-latest-logrotate \
                       docker-logrotate \
                       docker-engine
     ```

   - yum 安装 gcc 相关

     - CentOS 7 能上外网

     - 安装 gcc

       ```bash
       yum -y install gcc
       ```

     - 安装 gcc-c++

       ```bash
       yum -y install gcc-c++
       ```

   - 安装需要的软件包

     ```bash
     sudo yum install -y yum-utils
     ```

   - 设置 stable 镜像仓库

     ```bash
     sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
     ```

     - **注意：上面可能会报错**

       - [Erron 14] curl#35 - TCP connection reset by peer
       - [Erron 12] curl#35 - Timeout

     - **推荐做法**

       ```bash
       sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
       ```

   - 更新 yum 软件包索引

     ```bash
     yum makecache fast
     ```

   - 安装 Docker CE

     ```bash
     sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
     ```

   - 启动 Docker

     ```bash
     sudo systemctl start docker
     ```

   - 查看服务是否启动

     ```bash
     ps -ef | grep docker
     ```

   - 查看 Docker 版本

     ```bash
     docker version
     ```

   - 测试

     ```bash
     sudo docker run hello-world
     ```

   - 卸载(暂时用不到)

     - 第一步

       ```bash
       sudo yum remove docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
       ```

     - 第二步

       ```bash
       sudo rm -rf /var/lib/docker
       sudo rm -rf /var/lib/containerd
       ```

## 2.5 阿里云镜像加速

1. 访问[阿里云](<https://www.aliyun.com/>)
2. 注册一个属于自己的阿里云账户(可复用淘宝账号)
3. 获得加速器地址连接
4. 配置本机 Docker 运行镜像加速器
5. 重新启动 Docker 后台服务： systemctl restart docker
6. Linux 系统下配置完加速器需要检查是否生效

## 2.6 永远的 Hello, world

1. 启动 Docker 后台容器

   ```bash
   docker run hello-world
   ```

2. Docker run 干了什么？

   <img src="https://studentcwz-pic-bed.oss-cn-guangzhou.aliyuncs.com/img/docker%20run%20%E5%B9%B2%E4%BA%86%E4%BB%80%E4%B9%88.png" alt="docker run 干了什么" style="zoom:50%;" />

## 2.6 底层原理

### 2.6.1 为什么 Docker 比 VM 虚拟机快

1. **Docker 有着比虚拟机更少的抽象层**。由于 Docker 不需要 Hypervisor 实现硬件资源虚拟化，运行在 Docker 容器上的程序直接使用的都是实际物理机的硬件资源。因为在 CPU、内存利用率上 Docker 将会在效率上有明显优势。
2. **Docker 利用的是宿主机的内核，而不需要 Guest OS**。 因此，当新建一个容器时，Docker 不需要和虚拟机一样加载一个操作系统内核。从而避免引寻、加载操作系统内核几个比较费时费资源的过程。
3. 当新建一个虚拟机时，虚拟机软件需要加载 Guest OS，这个新建过程是**分钟级别**的，而 Docker 由于直接利用宿主机的操作系统，则省略了这个过程，因此新建一个 Docker 容器只需要**几秒钟**。
4. 两者对比

|            |       Docker 容器        |            虚拟机            |
| :--------: | :----------------------: | :--------------------------: |
|  操作系统  |     与宿主机共享 OS      |  宿主机 OS 上运行虚拟机 OS   |
|  存储大小  |  镜像小，便于存储和运输  |           镜像庞大           |
|  运行性能  |    几乎无额外性能损失    | 操作系统额外的 CPU、内存消耗 |
|   移植性   | 轻便、灵活，适用于 Linux |  笨重，和虚拟化技术耦合度高  |
| 硬件亲和性 |      面向软件开发者      |        面向硬件运维者        |
|  部署速度  |        快速，秒级        |        较慢，10s 以上        |
