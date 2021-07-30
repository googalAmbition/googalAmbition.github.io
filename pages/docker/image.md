## images命令
**通过使用如下两个命令，列出本机已有的镜像：**
```shell
docker images
docker image ls
```

## save命令
**压缩一个镜像**
```shell
mkdir -p /data
cd /data

docker save tomcat:9.0.20 -o tomcat9.tar

docker save tomcat:9.0.20 > tomcat9.tar
```
常用参数
-o :输出到的文件
**批量**
```shell
docker save \
ubuntu:20.04 \
alpine:3.12.1 \
debian:10.6-slim \
centos:7.8.2003 \
-o linux.tar

```

## load命令

```shell
mkdir -p /data
cd /data
docker load -i linux.tar
docker load < tomcat9.0.20.tar
```
常用参数
- `--input , -i `: 指定导入的文件。
- `--quiet , -q` : 精简输出信息。

## search命令
```shell
docker search tomcat
```
常用参数
- `-f, --filter filter `: 过滤输出的内容；
- `--limit int` ：指定搜索内容展示个数;
- `--no-index `: 不截断输出内容；
- `--no-trunc `：不截断输出内容；

## inspect命令
- 通过 docker inspect 命令，我们可以获取镜像的详细信息，其中，包括创建者，各层的数字摘要
等。
- docker inspect 返回的是 JSON格式的信息，如果您想获取其中指定的一项内容，可以通过 -f 来指
定，如获取镜像大小

```shell
docker inspect tomcat:9.0.20-jre8-alpine
docker inspect -f {{".Size"}} tomcat:9.0.20-jre8-alpine
```

## history命令
通过 docker history命令，可以列出各个层的创建信息，例如：查看 tomcat:9.0.20-jre8-alpine的各层
信息
```shell
docker history tomcat:9.0.20-jre8-alpine
```

##tag命令
标记本地镜像，将其归入某一仓库。
```shell
docker tag tomcat:9.0.20-jre8-alpine lagou/tomcat:9
```

## rmi命令
通过如下两个都可以删除镜像：
```shell
docker rmi tomcat:9.0.20-jre8-alpine
docker image rm tomcat:9.0.20-jre8-alpine
```
常用参数
- `-f, -force` : 强制删除镜像，即便有容器引用该镜像；
- `-no-prune `: 不要删除未带标签的父镜像；

除了通过标签名称来删除镜像，我们还可以通过制定镜像 ID, 来删除镜像。一旦制定了通过 ID 来删除镜
  像，它会先尝试删除所有指向该镜像的标签，然后在删除镜像本身。

```shell
docker rmi ee7cbd482336
```

## 清理镜像
我们在使用 Docker 一段时间后，系统一般都会残存一些临时的、没有被使用的镜像文件，可以通过以
下命令进行清理。执行完命令后，还是告诉我们释放了多少存储空间！
```shell
docker image prune
```
常用参数
- `-a, --all `: 删除所有没有用的镜像，而不仅仅是临时文件；
- `-f, --force` ：强制删除镜像文件，无需弹出提示确认；
