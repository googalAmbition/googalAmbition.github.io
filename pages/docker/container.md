容器是镜像的运行时实例。正如从虚拟机模板上启动 VM 一样，用户也同样可以从单个镜像上启动一个
或多个容器。虚拟机和容器最大的区别是容器更快并且更轻量级——与虚拟机运行在完整的操作系统之
上相比，容器会共享其所在主机的操作系统/内核。下图为使用单个 Docker镜像启动多个容器的示意
图。

![img.png](container.png)

Docker容器类似于一个轻量级的沙箱，Docker利用容器来运行和隔离应用。
容器是镜像的一个运行实例。
可以将其启动、开始、停止、删除，而这些容器都是彼此相互隔离的、互不可见的。
可以把容器看做是一个简易版的Linux系统环境（包括root用户权限、进程空间、用户空间和网络空间等）以及运行在其中的应用程序打包而成的盒子。
容器是基于镜像启动起来的，容 器中可以运行一个或多个进程。
镜像是Docker生命周期 中的构建或打包阶段，而容器则是启动或执行阶段。
镜像自身是只读的。容器从镜像启动的时候，会在镜像的最上层创建一个可写层。

# Docker容器常用命令

## 新建并启动容器

**语法**
```shell
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```
**运行容器**
```shell
docker run -it --rm -p 8080:8080 tomcat:9.0.20-jre8-alpine
```

**常用参数**

docker run命令常用参数比较多，这里仅仅列出开发岗常用参数，请小伙伴们自行查找资料获得更多参数信息
- `-d, --detach=false`: 后台运行容器，并返回容器ID
- `-i, --interactive=false`: 以交互模式运行容器，通常与 -t 同时使用
- `-P, --publish-all=false`: 随机端口映射，容器内部端口随机映射到主机的端口。不推荐各位小伙伴使用该参数
- `-p, --publish=[]`: 指定端口映射，格式为：主机(宿主)端口:容器端口，推荐各位小伙伴们使用
- `-t, --tty=false`: 为容器重新分配一个伪输入终端，通常与 -i 同时使用
- `--name="nginx-lb"`: 为容器指定一个名称
- `-h , --hostname="laosiji"`: 指定容器的hostname
- `-e , --env=[]`: 设置环境变量，容器中可以使用该环境变量
- `--net="bridge"`: 指定容器的网络连接类型，支持 `bridge/host/none/container`: 四种类型
- `--link=[]`: 添加链接到另一个容器；不推荐各位小伙伴使用该参数
- `-v, --volume `: 绑定一个卷
- `--privileged=false`: 指定容器是否为特权容器，特权容器拥有所有的capabilities
- `--restart=no`：指定容器停止后的重启策略
- `no`：容器退出时不重启
- `on-failure`：容器故障退出（返回值非零）时重启
- `always`：容器退出时总是重启,推荐各位小伙伴们使用
- `--rm=false`: 指定容器停止后自动删除容器,不能以`docker run -d`启动的容器

## 容器日志
docker logs : 获取容器的日志

**语法**
```shell
docker logs [OPTIONS] CONTAINER
```

**执行命令**
```shell
docker run -itd --name tomcat9 -p 8080:8080 tomcat:9.0.20-jre8-alpine
docker logs -f tomcat9
```

**常用参数**
- `-f `: 跟踪日志输出
- `--tail `:仅列出最新N条容器日志

## 删除容器
docker rm ：删除一个或多个容器。docker rm命令只能删除处于终止或退出状态的容器，并不能删除还处于运行状态的容器

**语法**
```shell
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```
**执行命令**
```shell
docker run -itd --name tomcat9 -p 8080:8080 tomcat:9.0.20-jre8-alpine
# 需要先停止运行中的容器再删除，否则无法删除容器
docker stop tomcat9
# 按照容器名称删除
docker rm tomcat9
# 按照容器ID删除
docker rm 8dd95a95e687
```
**常用参数**
- `-f` :通过 SIGKILL 信号强制删除一个运行中的容器。
- `-l` :移除容器间的网络连接，而非容器本身。
- `-v` :删除与容器关联的卷。
 
## 列出容器

**语法**
```shell
docker ps [OPTIONS]
```
**执行命令**
```shell
docker run -itd --name tomcat9 -p 8080:8080 tomcat:9.0.20-jre8-alpine
# 查看运行中的容器
docker ps tomcat9
# 查看所有容器
docker ps -a tomcat9
```
**输出详情介绍：**
- CONTAINER ID: 容器 ID。
- IMAGE: 使用的镜像。
- COMMAND: 启动容器时运行的命令。
- CREATED: 容器的创建时间。
- STATUS: 容器状态。  状态有7种：
  - created（已创建）
  - restarting（重启中）
  - running（运行中）
  - removing（迁移中）
  - paused（暂停）
  - exited（停止）
  - dead（死亡）
- PORTS: 容器的端口信息和使用的连接类型（tcp\udp）。
- NAMES: 自动分配的容器名称。

**常用参数**
- `-a` :显示所有的容器，包括未运行的。
- `-q` :只显示容器编号。

**实用技巧**
```shell
# 停止所有运行容器
docker stop $(docker ps -qa)
# 删除所有的容器
docker rm $(docker ps -aq)
docker rm $(docker stop $(docker ps -q))
# 删除所有的镜像
docker rmi $(docker images -q)
```
## 创建容器
docker create ：创建一个新的容器但不启动它。用法同 docker run命令。

**语法**
```shell
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

**执行命令**
```shell
docker create -it --name tomcat9 -p 8080:8080 9.0.20-jre8-alpine
```
## 启动、重启、终止容器
docker start :启动一个或多个已经被停止的容器
docker stop :停止一个运行中的容器
docker restart :重启容器

**语法**
```shell
docker start [OPTIONS] CONTAINER [CONTAINER...]
docker stop [OPTIONS] CONTAINER [CONTAINER...]
docker restart [OPTIONS] CONTAINER [CONTAINER...]
```
**执行命令**
```shell
docker start tomcat9
docker stop tomcat9
docker restart tomcat9
```
## 进入容器
`docker exec` ：在运行的容器中执行命令。早期有attach命令，对于阻塞命令会等待，所以不方便。在
Docker 1.3.0后提供了exec 可以在容器内直接执行任意命令

**语法**
```shell
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```
**执行命令**
```shell
# 有bash命令的linux系统：例如centos
docker run -it --name tomcat9.1 -p 8080:8080 tomcat:9.0.20-jre8-slim
docker exec -it tomcat9.1 /bin/bash
# 没有bash命令的linux系统：例如alpine系统
docker run -it --name tomcat9.2 -p 8081:8080 tomcat:9.0.20-jre8-alpine
docker exec -it tomcat9.2 sh
```
**常用参数**
- `-i` :即使没有附加也保持STDIN 打开
- `-t` :分配一个伪终端

