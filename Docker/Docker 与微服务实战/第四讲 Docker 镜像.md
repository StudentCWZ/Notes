# Docker 镜像

## 4.1 Docker 镜像简介

1. **镜像**: 是一种轻量级、可执行的独立软件包，它包含运行某个软件所需的所有内容，我们把应用程序和配置依赖打包形成一个可交付的运行环境(包括代码、运行时需要的库、环境变量和配置文件等)，这个打包好的运行环境就是 image 镜像文件
2. 只有通过这个镜像文件才能生成 Docker 容器实例(类似 Java 中 new 出来一个对象)

### 4.1.1 UnionFS

1. 以我们的 pull 为例，在下载的过程中我们可以看到 Docker 的镜像好像是一层一层的在下载

2. **UnionFS(联合文件系统)**: Union 文件系统(UnionFS)是一种分层、轻量级并且高性能的文件系统，它支持**对文件系统的修改作为一次提交来一层层的叠加**，同时可以将不同目录挂载到同一个虚拟文件系统下(unite serveral directories into a single virtual filesystem)。 ——> 类似花卷
3. Union 文件系统是 Docker 镜像的基础。**镜像可以通过分层来进行继承**，基于基础镜像(没有父镜像)，可以制作各种具体的应用镜像。
4. 特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。

### 4.1.2 Docker 镜像的加载原理

1. Docker 的镜像实际上由一层层的文件系统组成，这种层级的文件系统 UnionFS

2. bootfs(boot file system) 主要包括 bootloader 和 kernel，bootloader 主要是引导加载 kernel，Linux 刚启动时会加载 bootfs 文件系统，**在 Docker 镜像的最底层时引导文件系统 bootfs**。这一层与我们典型的 Linux/Unix 系统是一样的，包含 boot 加载器和内核。当 boot 加载完成之后整个内核就都在内存中了，此时内存的使用权已由 bootfs 转交给内核，此时系统也会卸载 bootfs

3. rootfs(root file system)，在 bootfs 之上。包含的就是典型的 Linux 系统中的 /dev，/proc，/bin，/etc 等标准目录和文件。rootfs 就是各种不同操作系统发行版，比如 Ubuntu、Centos 等等

   <img src="https://studentcwz-pic-bed.oss-cn-guangzhou.aliyuncs.com/img/bootfs%20%E5%92%8C%20rootfs.png" alt="bootfs 和 rootfs" style="zoom:50%;" />

4. 对于一个精简的 OS，rootfs 可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用 Host 的 kernel，自己只需要提供 rootfs 就行了。由此可见对于不同的 Linux 发行版，bootfs 基本是一致的，rootfs 会有差别，因此不同的发行版可以公用 bootfs

### 4.1.3 为什么 Docker 镜像采用这种分层结构

1. 镜像分层最大的一个好处就是共享资源，方便复制迁移，就是为了复用
2. 比如说有多个镜像都从相同的 base 镜像构建而来，那么 Docker Host 只需在磁盘上保存一份 base 镜像
3. 同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。

## 4.2 重点理解

1. **Docker 镜像层都是只读的。容器层是可写的**

2. 当容器启动时，一个新的可写层被加载到镜像顶部。这一层通常被称作容器层，容器层之下的都叫镜像层

3. 所有对容器的改动-无论添加、删除、还是修改文件都只会发生在容器层中。只有容器层是可写的，容器层下面的所有镜像层都是只读的

   <img src="https://studentcwz-pic-bed.oss-cn-guangzhou.aliyuncs.com/img/%E5%AE%B9%E5%99%A8%E5%B1%82%E5%92%8C%E9%95%9C%E5%83%8F%E5%B1%82.png" alt="容器层和镜像层" style="zoom:50%;" />

## 4.3 Docker 镜像 commit 操作

1. docker commit 提交容器副本使之成为一个新的镜像

2. docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]

3. 案例演示：Ubuntu 安装 vim

   - 从 Hub 拉取 Ubuntu 镜像到本地

     ```bash
     $ docker pull ubuntu
     Using default tag: latest
     latest: Pulling from library/ubuntu
     7b1a6ab2e44d: Pull complete
     Digest: sha256:626ffe58f6e7566e00254b638eb7e0f3b11d4da9675088f4781a50ae288f3322
     Status: Downloaded newer image for ubuntu:latest
     docker.io/library/ubuntu:latest
     ```

   - 成功运行

     ```bash
     $ docker run -it ubuntu /bin/bash
     root@707675d49da5:/#
     ```

   - 外网联通情况下，安装 vim(容器内部执行)

     ```bash
     apt-get upgrade
     apt-get -y install vim
     ```

   - 编辑文本

     ```bash
     vim a.txt
     ```

   - 安装完成后，commit 我们自己的新镜像

     ```bash
     $ docker commit -m="add vim cmd" -a="StudentCWZ" 707675d49da5 studentcwz/myubuntu:1.1
     sha256:8fc0d376fbd5503a8e79303c1860b3c5e1be51d01748335988706c83530b8f20
     ```

   - 启动镜像和原来的对比(已经具备 vim 功能)

     ```bash
     $ docker run -it 8fc0d376fbd5 /bin/bash
     root@556f563a36c6:/#
     ```

## 4.4 总结

1. Docker 中镜像分层，**支持通过扩展现有镜像，创建新的镜像**。类似 Java 继承于一个 Base 基础类，自己再按需扩展

2. 新镜像是从 base 镜像一层一层叠加生成。每安装一个软件，就在现有镜像的基础上增加一层

   ![镜像叠加](https://studentcwz-pic-bed.oss-cn-guangzhou.aliyuncs.com/img/%E9%95%9C%E5%83%8F%E5%8F%A0%E5%8A%A0.png)
