当你开始大规模使用docker时，你会发现需要了解很多关于网络的知识。docker作为目前最火的轻量级 容器技术，有很多令人称道的功能，如docker的镜像管理。然而，docker同样有着很多不完善的地方，
网络方面就是Docker比较薄弱的部分。因此，作为一名运维工程师有必要深入了解docker的网络知识， 以满足更高的网络需求。作为一名微服开发工程师，简单了解docker网络环节即可。本章节首先介绍了
Docker自身的3种local网络工作方式，然后介绍一些docker自定义网络模式。

docker安装后会自动创建3种网络:

- bridge
- host
- none

```shell
docker network ls
```

## docker网络理论部分

docker使用Linux桥接网卡，在宿主机虚拟一个docker容器网桥(docker0)
，docker启动一个容器时会根据docker网桥的网段分配给容器一个IP地址，称为Container-IP，同时docker网桥是每个容器的默认网关。因为在同一宿主机内的容器都接入同一个网桥，这样容器之间就能够通过容器的Container-IP直接通信。

docker网桥是宿主机虚拟出来的，并不是真实存在的网络设备，外部网络是无法寻址到的，这也意味着 外部网络无法通过直接Container-IP访问到容器。如果容器希望外部访问能够访问到，可以通过映射容
器端口到宿主主机（端口映射），即docker run创建容器时候通过 -p 或 -P 参数来启用，访问容器的时 候就通过**[宿主机IP]**:**[容器端口]**访问容器。
```shell
# 使用命令查看docker网络部分
docker info
```

## 网络模式

|  Docker网络模式     |  配置    |  说明    |
| :--: | :------------: | :--- |
|  host模式    |  –net=host    |  容器和宿主机共享Network namespace。容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口    |
|  container模式    |  –net=container:NAME_or_ID    | 容器和另外一个容器共享Network namespace。 kubernetes中的pod就是多个容器共享一个 Network namespace。 创建的容器不会创建自己的网卡，配置自己的 IP， 而是和一个指定的容器共享IP、端口范围。 |
|  none模式    |  –net=none    | 容器有独立的Network namespace，并没有对 其进行任何网络设置， 如分配veth pair 和网桥连接，配置IP等。 该模式关闭了容器的网络功能。 |
|  bridge模式    |     –net=bridge | （默认为该模式）。此模式会为每一个容器分 配、设置IP等， 并将容器连接到一个docker0虚拟网桥，通过 docker0网桥 以及Iptables nat表配置与宿主机通信 |
|   Macvlan network   | 无 | 容器具备Mac地址，使其显示为网络上的物理 设备 |
|  Overlay    | 无 | (覆盖网络）： 利用VXLAN实现的bridge模式 |

