# Docker 容器数据卷

## 7.1 注意

1. 一旦使用容器卷记得加入 `--privileged=true`
2. 原因：
   - Docker 挂载主机目录访问**如果出现 cannot open directory.:Permission denied**
   - 解决办法：在挂载目录后多加一个 `--privilege=true` 参数即可
   - 如果是 CentOS 7 安全模块会比之前系统版本加强，不安全的先会禁止，所以目录挂载的情况被默认为不安全的行为
   - 在 SELinux 里面挂载目录被禁止掉了，如果要开启，我们一般使用 `--privilege=true` 命令，扩大容器的权限解决挂载目录没有权限的问题，也即使用该参数，container 内的 root 拥有真正的 root 权限，否则。container 内的 root 只是外部的一个普通用户权限

## 7.2 容器数据卷概念

1. 卷就是目录或文件，存在于一个或多个容器中，由 docker 挂载到容器，但不属于联合文件系统，因此能够绕过 Union File System 提供一些用于持续存储或共享数据的特性

2. 卷的设计目的就是**数据持久化**，完全独立于容器的生存周期，因此 Docker 不会在容器删除时删除其挂载的数据卷

3. 有点类似我们 Redis 里面的 rdb 和 aof 文件

4. 将 docker 容器内的数据保存进宿主机的磁盘中

5. 运行一个带有容器卷存储功能的容器实例

   ```bash
   docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录 镜像名
   ```

## 7.3 容器数据卷的作用

1. 当将应用与运行环境打包成镜像，run 后形成容器实例运行，但是我们对数据的要求希望是**持久化**的
2. Docker 容器产生的数据。如果不备份，那么当容器实例删除后，容器内的数据自然也就没有了。
3. 为了能保存数据在 Docker 中我们使用卷
4. 容器数据卷的特点
   - 数据卷可在容器之间共享或重用数据
   - 卷中的更改可以直接实时生效
   - 数据卷中的更改不会包含在镜像的更新中
   - 数据卷的生命周期一直持续到没有容器使用它为止

## 7.4 容器数据卷案例

### 7.4.1 宿主 VS 容器之间映射添加容器卷

1. 直接命令添加

   - 命令

     ```bash
     docker run --privileged=true -v /宿主机绝对路径目录:/容器内目录 镜像名
     ```

   - 具体操作

     - 运行镜像，挂载相关容器数据卷

       ```bash
       docker run -it --privileged=true -v /Users/cuiweizhi/Downloads/host_data:/tmp/docker_data --name=u1 ubuntu
       ```

     - 查看容器内 /tmp/docker_data

       ```bash
       root@ac61efc0b89d:/# cd /tmp/
       root@ac61efc0b89d:/tmp# ll
       total 0
       drwxrwxrwt 1 root root 22 May 16 03:22 ./
       drwxr-xr-x 1 root root  6 May 16 03:22 ../
       drwxr-xr-x 2 root root 64 May 16 03:22 docker_data/
       root@ac61efc0b89d:/tmp# cd docker_data/
       root@ac61efc0b89d:/tmp/docker_data# ll
       total 0
       drwxr-xr-x 2 root root 64 May 16 03:22 ./
       drwxrwxrwt 1 root root 22 May 16 03:22 ../
       ```

     - 新增 /tmp/docker_data 目录下 dockerin.txt 文件

       ```bash
       root@ac61efc0b89d:/tmp/docker_data# touch dockerin.txt
       root@ac61efc0b89d:/tmp/docker_data# ll
       total 0
       drwxr-xr-x 3 root root 96 May 16 03:23 ./
       drwxrwxrwt 1 root root 22 May 16 03:22 ../
       -rw-r--r-- 1 root root  0 May 16 03:23 dockerin.txt
       ```

     - 在宿主机查看相应目录是否有一样文件

       ```bash
       cd Downloads/host_dat
       pwd
       /Users/cuiweizhi/Downloads/host_data
       ll
       total 0
       -rw-r--r--  1 cuiweizhi  staff     0B  5 16 11:23 dockerin.txt
       ```

     - 宿主机新建 hostin.txt 文件

       ```bash
       touch hostin.txt
       ```

     - 查看 docker 容器中 /tmp/docker_data 目录下是否有 hostin.txt

       ```bash
       root@ac61efc0b89d:/tmp/docker_data# ll
       total 0
       drwxr-xr-x 4 root root 128 May 16 03:24 ./
       drwxrwxrwt 1 root root  22 May 16 03:22 ../
       -rw-r--r-- 1 root root   0 May 16 03:23 dockerin.txt
       -rw-r--r-- 1 root root   0 May 16 03:24 hostin.txt
       ```

     - 容器内新增文件 a.txt

       ```bash
       root@ac61efc0b89d:/tmp/docker_data# echo 'hello docker' > a.txt
       root@ac61efc0b89d:/tmp/docker_data# ll
       total 4
       drwxr-xr-x 5 root root 160 May 16 03:25 ./
       drwxrwxrwt 1 root root  22 May 16 03:22 ../
       -rw-r--r-- 1 root root  13 May 16 03:25 a.txt
       -rw-r--r-- 1 root root   0 May 16 03:23 dockerin.txt
       -rw-r--r-- 1 root root   0 May 16 03:24 hostin.txt
       ```

     - 宿主机修改 a.txt

       ```bash
       vim a.txt
       ```

     - 容器内查看修改是否同步

       ```bash
       root@ac61efc0b89d:/tmp/docker_data# cat a.txt
       hello docker
       host update
       ```

2. 查看数据卷是否挂载成功

   - 命令

     ```bash
     docker inspect ac61efc0b89d
     ```

   - 结果

     ```json
     [
         {
             "Id": "ac61efc0b89d247de82a4cd831d4cf07e2aa36291eeb41acc5b80f1eccc4867d",
             "Created": "2023-05-16T03:22:34.598438596Z",
             "Path": "/bin/bash",
             "Args": [],
             "State": {
                 "Status": "running",
                 "Running": true,
                 "Paused": false,
                 "Restarting": false,
                 "OOMKilled": false,
                 "Dead": false,
                 "Pid": 256,
                 "ExitCode": 0,
                 "Error": "",
                 "StartedAt": "2023-05-16T03:22:35.106745646Z",
                 "FinishedAt": "0001-01-01T00:00:00Z"
             },
             "Image": "sha256:3b418d7b466ac6275a6bfcb0c86fbe4422ff6ea0af444a294f82d3bf5173ce74",
             "ResolvConfPath": "/var/lib/docker/containers/ac61efc0b89d247de82a4cd831d4cf07e2aa36291eeb41acc5b80f1eccc4867d/resolv.conf",
             "HostnamePath": "/var/lib/docker/containers/ac61efc0b89d247de82a4cd831d4cf07e2aa36291eeb41acc5b80f1eccc4867d/hostname",
             "HostsPath": "/var/lib/docker/containers/ac61efc0b89d247de82a4cd831d4cf07e2aa36291eeb41acc5b80f1eccc4867d/hosts",
             "LogPath": "/var/lib/docker/containers/ac61efc0b89d247de82a4cd831d4cf07e2aa36291eeb41acc5b80f1eccc4867d/ac61efc0b89d247de82a4cd831d4cf07e2aa36291eeb41acc5b80f1eccc4867d-json.log",
             "Name": "/u1",
             "RestartCount": 0,
             "Driver": "overlay2",
             "Platform": "linux",
             "MountLabel": "",
             "ProcessLabel": "",
             "AppArmorProfile": "",
             "ExecIDs": null,
             "HostConfig": {
                 "Binds": [
                     "/Users/cuiweizhi/Downloads/host_data:/tmp/docker_data"
                 ],
                 "ContainerIDFile": "",
                 "LogConfig": {
                     "Type": "json-file",
                     "Config": {}
                 },
                 "NetworkMode": "default",
                 "PortBindings": {},
                 "RestartPolicy": {
                     "Name": "no",
                     "MaximumRetryCount": 0
                 },
                 "AutoRemove": false,
                 "VolumeDriver": "",
                 "VolumesFrom": null,
                 "ConsoleSize": [
                     50,
                     180
                 ],
                 "CapAdd": null,
                 "CapDrop": null,
                 "CgroupnsMode": "private",
                 "Dns": [],
                 "DnsOptions": [],
                 "DnsSearch": [],
                 "ExtraHosts": null,
                 "GroupAdd": null,
                 "IpcMode": "private",
                 "Cgroup": "",
                 "Links": null,
                 "OomScoreAdj": 0,
                 "PidMode": "",
                 "Privileged": true,
                 "PublishAllPorts": false,
                 "ReadonlyRootfs": false,
                 "SecurityOpt": [
                     "label=disable"
                 ],
                 "UTSMode": "",
                 "UsernsMode": "",
                 "ShmSize": 67108864,
                 "Runtime": "runc",
                 "Isolation": "",
                 "CpuShares": 0,
                 "Memory": 0,
                 "NanoCpus": 0,
                 "CgroupParent": "",
                 "BlkioWeight": 0,
                 "BlkioWeightDevice": [],
                 "BlkioDeviceReadBps": [],
                 "BlkioDeviceWriteBps": [],
                 "BlkioDeviceReadIOps": [],
                 "BlkioDeviceWriteIOps": [],
                 "CpuPeriod": 0,
                 "CpuQuota": 0,
                 "CpuRealtimePeriod": 0,
                 "CpuRealtimeRuntime": 0,
                 "CpusetCpus": "",
                 "CpusetMems": "",
                 "Devices": [],
                 "DeviceCgroupRules": null,
                 "DeviceRequests": null,
                 "MemoryReservation": 0,
                 "MemorySwap": 0,
                 "MemorySwappiness": null,
                 "OomKillDisable": null,
                 "PidsLimit": null,
                 "Ulimits": null,
                 "CpuCount": 0,
                 "CpuPercent": 0,
                 "IOMaximumIOps": 0,
                 "IOMaximumBandwidth": 0,
                 "MaskedPaths": null,
                 "ReadonlyPaths": null
             },
             "GraphDriver": {
                 "Data": {
                     "LowerDir": "/var/lib/docker/overlay2/b2b5b96a63fddef4d31309253e2406f2e55df5de2d7ebedeafa0ba53c2c60156-init/diff:/var/lib/docker/overlay2/98fddfea80cd63dd14cf9821b28b94ed48bfa32ce47324ae7e310d185d7aafdb/diff",
                     "MergedDir": "/var/lib/docker/overlay2/b2b5b96a63fddef4d31309253e2406f2e55df5de2d7ebedeafa0ba53c2c60156/merged",
                     "UpperDir": "/var/lib/docker/overlay2/b2b5b96a63fddef4d31309253e2406f2e55df5de2d7ebedeafa0ba53c2c60156/diff",
                     "WorkDir": "/var/lib/docker/overlay2/b2b5b96a63fddef4d31309253e2406f2e55df5de2d7ebedeafa0ba53c2c60156/work"
                 },
                 "Name": "overlay2"
             },
             "Mounts": [
                 {
                     "Type": "bind",
                     "Source": "/Users/cuiweizhi/Downloads/host_data",
                     "Destination": "/tmp/docker_data",
                     "Mode": "",
                     "RW": true,
                     "Propagation": "rprivate"
                 }
             ],
             "Config": {
                 "Hostname": "ac61efc0b89d",
                 "Domainname": "",
                 "User": "",
                 "AttachStdin": true,
                 "AttachStdout": true,
                 "AttachStderr": true,
                 "Tty": true,
                 "OpenStdin": true,
                 "StdinOnce": true,
                 "Env": [
                     "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
                 ],
                 "Cmd": [
                     "/bin/bash"
                 ],
                 "Image": "ubuntu",
                 "Volumes": null,
                 "WorkingDir": "",
                 "Entrypoint": null,
                 "OnBuild": null,
                 "Labels": {
                     "org.opencontainers.image.ref.name": "ubuntu",
                     "org.opencontainers.image.version": "22.04"
                 }
             },
             "NetworkSettings": {
                 "Bridge": "",
                 "SandboxID": "5276a91d08cfc38f51b5891d614a5363e8eb2f197eede8ba17b76806d3bd8003",
                 "HairpinMode": false,
                 "LinkLocalIPv6Address": "",
                 "LinkLocalIPv6PrefixLen": 0,
                 "Ports": {},
                 "SandboxKey": "/var/run/docker/netns/5276a91d08cf",
                 "SecondaryIPAddresses": null,
                 "SecondaryIPv6Addresses": null,
                 "EndpointID": "fdf0f4673489e3697713a9bf3cd8904773da42150aa049df4c21ecb97b38ffa8",
                 "Gateway": "172.17.0.1",
                 "GlobalIPv6Address": "",
                 "GlobalIPv6PrefixLen": 0,
                 "IPAddress": "172.17.0.2",
                 "IPPrefixLen": 16,
                 "IPv6Gateway": "",
                 "MacAddress": "02:42:ac:11:00:02",
                 "Networks": {
                     "bridge": {
                         "IPAMConfig": null,
                         "Links": null,
                         "Aliases": null,
                         "NetworkID": "68a9b2f0c968afef45f158b7c6e09bc19a2940ab3a7de1ed465080f93d5fe9b6",
                         "EndpointID": "fdf0f4673489e3697713a9bf3cd8904773da42150aa049df4c21ecb97b38ffa8",
                         "Gateway": "172.17.0.1",
                         "IPAddress": "172.17.0.2",
                         "IPPrefixLen": 16,
                         "IPv6Gateway": "",
                         "GlobalIPv6Address": "",
                         "GlobalIPv6PrefixLen": 0,
                         "MacAddress": "02:42:ac:11:00:02",
                         "DriverOpts": null
                     }
                 }
             }
         }
     ]
     ```

3. 如果容器停止情况分析

   - 停止容器

     ```bash
     docker stop ac61efc0b89d
     ```

   - 宿主机新增文件

     ```bash
     touch c.txt
     ```

   - 查看文件目录

     ```bash
     ll
     total 8
     -rw-r--r--  1 cuiweizhi  staff    25B  5 16 11:26 a.txt
     -rw-r--r--  1 cuiweizhi  staff     0B  5 16 11:44 c.txt
     -rw-r--r--  1 cuiweizhi  staff     0B  5 16 11:23 dockerin.txt
     -rw-r--r--  1 cuiweizhi  staff     0B  5 16 11:24 hostin.txt
     ```

   - 再次启动

     ```bash
     docker start ac61efc0b89d
     ```

   - 进入容器

     ```bash
     docker exec -it ac61efc0b89d /bin/bash
     ```

   - 查看容器目录

     ```bash
     cd /tmp/host_data
     ll
     total 8
     -rw-r--r--  1 cuiweizhi  staff    25B  5 16 11:26 a.txt
     -rw-r--r--  1 cuiweizhi  staff     0B  5 16 11:44 c.txt
     -rw-r--r--  1 cuiweizhi  staff     0B  5 16 11:23 dockerin.txt
     -rw-r--r--  1 cuiweizhi  staff     0B  5 16 11:24 hostin.txt
     ```

4. 容器和宿主机之间数据共享
   - **Docker 修改，主机同步获得**
   - **主机修改，docker 同步获得**
   - **docker 容器 stop，主机修改，docker 容器重启数据依然同步**

### 7.4.2 读写规则映射添加说明

#### 7.4.2.1 读写(默认)

1. 命令(默认同上案例，默认就是 rw)

   ```bash
   docker run --privileged=true -v /宿主机绝对路径目录:/容器内目录:rw 镜像名
   ```

#### 7.4.2.2 只读

1. 容器实例内部被限制，只能读取不能写

2. 命令

   ```bash
   docker run --privileged=true -v /宿主机绝对路径目录:/容器内目录:ro 镜像名
   ```

### 7.4.3 卷的继承和共享

1. 容器一完成和宿主机的映射

   - 启动容器

     ```bash
     docker run -it --privileged=true -v /Users/cuiweizhi/Downloads/host_data:/tmp/docker_data --name=u1 ubuntu
     ```

   - 进入目录

     ```bash
     cd /tmp/docker_data/
     ```

   - 查看目录文件(当前为空)

     ```bash
     root@022e9e7845f4:/tmp/docker_data# ll
     total 0
     drwxr-xr-x 2 root root 64 May 16 06:04 ./
     drwxrwxrwt 1 root root 22 May 16 06:04 ../
     ```

   - 新建文件 u1data.txt

     ```bash
     touch u1data.txt
     ```

   - 宿主机新建文件 host.txt

     ```bash
     touch host.txt
     ```

   - 容器查看 /tmp/docker_data 目录

     ```bash
     root@022e9e7845f4:/tmp/docker_data# ls
     host.txt  u1data.txt
     ```

2. 容器二继承容器一的卷规则

   - 容器继承命令

     ```bash
     docker run -it --privileged=true --volumes-from 父类 --name u2 ubuntu
     ```

   - 进入容器，查看 /tmp/docker_data 目录

     ```bash
     root@be2edf6cd401:/# cd /tmp/docker_data/
     root@be2edf6cd401:/tmp/docker_data# ll
     total 0
     drwxr-xr-x 4 root root 128 May 16 06:06 ./
     drwxrwxrwt 1 root root  22 May 16 06:13 ../
     -rw-r--r-- 1 root root   0 May 16 06:06 host.txt
     -rw-r--r-- 1 root root   0 May 16 06:05 u1data.txt
     ```

   - 容器二新增文件 u2data.txt

     ```bash
     root@be2edf6cd401:/tmp/docker_data# touch u2data.txt
     ```

   - 查看宿主机 /Users/cuiweizhi/Downloads/host_data 目录

     ```bash
     ll
     total 0
     -rw-r--r--  1 cuiweizhi  staff     0B  5 16 14:06 host.txt
     -rw-r--r--  1 cuiweizhi  staff     0B  5 16 14:05 u1data.txt
     -rw-r--r--  1 cuiweizhi  staff     0B  5 16 14:15 u2data.txt
     ```

   - 查看容器一 /tmp/docker_data 目录

     ```bash
     root@022e9e7845f4:/tmp/docker_data# ll
     total 0
     drwxr-xr-x 5 root root 160 May 16 06:15 ./
     drwxrwxrwt 1 root root  22 May 16 06:04 ../
     -rw-r--r-- 1 root root   0 May 16 06:06 host.txt
     -rw-r--r-- 1 root root   0 May 16 06:05 u1data.txt
     -rw-r--r-- 1 root root   0 May 16 06:15 u2data.txt
     ```

   - 退出容器一

     ```bash
     root@022e9e7845f4:/tmp/docker_data# exit
     exit
     ```

   - 宿主机新增文件 host2.txt

     ```bash
     touch host2.txt
     ```

   - 查看容器二 /tmp/docker_data 目录

     ```bash
     root@be2edf6cd401:/tmp/docker_data# ll
     total 0
     drwxr-xr-x 6 root root 192 May 16 06:19 ./
     drwxrwxrwt 1 root root  22 May 16 06:13 ../
     -rw-r--r-- 1 root root   0 May 16 06:06 host.txt
     -rw-r--r-- 1 root root   0 May 16 06:19 host2.txt
     -rw-r--r-- 1 root root   0 May 16 06:05 u1data.txt
     -rw-r--r-- 1 root root   0 May 16 06:15 u2data.txt
     ```

   - 容器二新增 u2datav2.txt

     ```bash
     touch u2datav2.txt
     ```

   - 查看宿主机 /Users/cuiweizhi/Downloads/host_data 目录

     ```bash
     ll
     total 0
     -rw-r--r--  1 cuiweizhi  staff     0B  5 16 14:06 host.txt
     -rw-r--r--  1 cuiweizhi  staff     0B  5 16 14:19 host2.txt
     -rw-r--r--  1 cuiweizhi  staff     0B  5 16 14:05 u1data.txt
     -rw-r--r--  1 cuiweizhi  staff     0B  5 16 14:15 u2data.txt
     -rw-r--r--  1 cuiweizhi  staff     0B  5 16 14:21 u2datav2.txt
     ```

   - 容器一再次启动

     ```bash
     docker start 022e9e7845f4
     ```

   - 进入容器一交互式终端

     ```bash
     docker exec -it u1 /bin/bash
     ```

   - 查看容器一 /tmp/docker_data 目录

     ```bash
     root@022e9e7845f4:/tmp/docker_data# ll
     total 0
     drwxr-xr-x 7 root root 224 May 16 06:21 ./
     drwxrwxrwt 1 root root  22 May 16 06:04 ../
     -rw-r--r-- 1 root root   0 May 16 06:06 host.txt
     -rw-r--r-- 1 root root   0 May 16 06:19 host2.txt
     -rw-r--r-- 1 root root   0 May 16 06:05 u1data.txt
     -rw-r--r-- 1 root root   0 May 16 06:15 u2data.txt
     -rw-r--r-- 1 root root   0 May 16 06:21 u2datav2.txt
     ```
