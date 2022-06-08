https://www.kuangstudy.com/bbs/1485182032115318785

> 参考源

https://www.bilibili.com/video/BV1og4y1q7M4?spm_id_from=333.999.0.0

https://www.bilibili.com/video/BV1kv411q7Qc?spm_id_from=333.999.0.0

> 版本

本文章基于 **Docker 20.10.11**

------

# Linux 网络

> 查看本地网络信息

```shell
[root@sail ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:16:3e:30:01:20 brd ff:ff:ff:ff:ff:ff
    inet 172.24.19.94/18 brd 172.24.63.255 scope global dynamic eth0
       valid_lft 310201059sec preferred_lft 310201059sec
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:23:ae:ac:24 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
```

有三个网卡信息：

- **lo**：本地。
- **ens**：虚拟机或阿里云服务器地址。
- **docker0**：Docker 网络地址。

# Docker 网络

在 Docker 安装后，主机会为 Docker 分配一个网卡，名为 **docker0** 。

该网卡使用**桥接模式**，使用的是 **veth-pair** 技术。

> 启动两个容器

```shell
[root@sail ~]# docker run -d -p 8081:8080 --name=tomcat01 tomcat
29a06eab16e73f34458b77a520081083fe536d8eb34eb67dbb9c6632fc720687

[root@sail ~]# docker run -d -p 8082:8080 --name=tomcat02 tomcat
442add0d94cef631e0f531dff9d8f55b7e2f1aaeb088f742c3d8e240d4f9cc7d

[root@sail ~]# docker ps
CONTAINER ID   IMAGE     COMMAND             CREATED          STATUS          PORTS                    NAMES
442add0d94ce   tomcat    "catalina.sh run"   4 seconds ago    Up 3 seconds    0.0.0.0:8082->8080/tcp   tomcat02
29a06eab16e7   tomcat    "catalina.sh run"   15 seconds ago   Up 14 seconds   0.0.0.0:8081->8080/tcp   tomcat01
```

> 查看 Linux 网络

```shell
[root@sail tomcat]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:16:3e:30:01:20 brd ff:ff:ff:ff:ff:ff
    inet 172.24.19.94/18 brd 172.24.63.255 scope global dynamic eth0
       valid_lft 310199524sec preferred_lft 310199524sec
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:23:ae:ac:24 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
7: veth4a18f1b@if110: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 52:69:3c:bc:83:4a brd ff:ff:ff:ff:ff:ff link-netnsid 0
9: veth296fd0d@if112: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 76:3c:34:e8:c4:90 brd ff:ff:ff:ff:ff:ff link-netnsid 1
```

Docker 每启动一个容器，就会分配一个 IP。

> 查看容器的内部网络

```shell
[root@sail ~]# docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
6: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

[root@sail ~]# docker exec -it tomcat02 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
8: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

可以看到容器内 IP 与本机 IP 成对出现，这就是 veth-pair 技术。

> 容器访问 docker0 测试

```shell
[root@sail ~]# docker exec -it tomcat01 ping 172.17.0.1
PING 172.17.0.1 (172.17.0.1) 56(84) bytes of data.
64 bytes from 172.17.0.1: icmp_seq=1 ttl=64 time=0.845 ms
64 bytes from 172.17.0.1: icmp_seq=2 ttl=64 time=0.139 ms
64 bytes from 172.17.0.1: icmp_seq=3 ttl=64 time=0.130 ms
64 bytes from 172.17.0.1: icmp_seq=4 ttl=64 time=0.134 ms
64 bytes from 172.17.0.1: icmp_seq=5 ttl=64 time=0.119 ms
64 bytes from 172.17.0.1: icmp_seq=6 ttl=64 time=0.082 ms
```

容器与 docker0 之间是可以访问的。

> 容器与容器之间访问

**docker0** 相当于一个路由器，各个容器都与 docker0 相连，容器之间的通信通过路由器来转发。

![img](Docker 14 网络.assets/kuangstudy4403351a-c00c-465a-96f5-c73377088790.png)

Docker 中的所有网络接口都是虚拟的，相当于内网传递。

> 只要删除容器，对应网络就会删除。

## 容器间网络连接

### docker run —link

每次重启容器或 Linux，IP 就会变化，固定 IP 互联网络就会失效。

如果能使用服务名来连接，而不考虑 IP，就会方便很多。

> 测试使用容器名来 ping

```shell
[root@sail ~]# docker exec -it tomcat01 ping tomcat02
ping: tomcat02: Name or service not known
```

容器之间无法通过容器名来连接。

> 使用 `--link` 启动测试，先启动 tomcat01

```shell
[root@sail ~]# docker run -d -P --name=tomcat01 tomcat
0d39450a3253544ff5a9bf390b450a218b1055c4d1e60fc02b153ab58544d600
```

> 使用 `–-link` 命令启动 tomcat02

```shell
[root@sail ~]# docker run -d -P --name tomcat02 --link tomcat01 tomcat
1901445346baf10ddca6e8a639f0aed72b2cf758046e6da9c28426857c7bb3fd
```

> 在 tomcat02 访问 tomcat01

```shell
[root@node1 ~]# docker exec -it tomcat02 ping tomcat01
PING tomcat01 (172.17.0.2) 56(84) bytes of data.
64 bytes from tomcat01 (172.17.0.2): icmp_seq=1 ttl=64 time=0.120 ms
64 bytes from tomcat01 (172.17.0.2): icmp_seq=2 ttl=64 time=0.184 ms
^C
--- tomcat01 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 3ms
rtt min/avg/max/mdev = 0.120/0.152/0.184/0.032 ms
```

在 tomcat01 访问 tomcat02 则无法访问。

> tomcat02 能够通过容器名访问 tomcat01，原理是 `--link` 通过 tomcat02 在自己容器 hosts 文件中配置了 tomcat01 IP 信息。

```shell
[root@sail ~]# docker exec -it tomcat02 cat /etc/hosts
127.0.0.1    localhost
::1    localhost ip6-localhost ip6-loopback
fe00::0    ip6-localnet
ff00::0    ip6-mcastprefix
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
172.17.0.2    tomcat01 0d39450a3253
172.17.0.3    1901445346ba
```

所以 `--link` 本质就是修改 host 映射。

> 这种方式已经不流行了，建议使用**自定义网络**实现。

## 查看Docker网络信息

### docker network  ls

```shell
[root@sail ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
f3eeb014197a   bridge    bridge    local
28d77e958643   host      host      local
c3ff850e96f0   none      null      local
```

- **bridge**：桥接模式（默认）。自己创建也使用这种模式。
- **host**：和宿主即共享。
- **none**：不配置网络。

## 创建自定义网络

> 启动容器

```shell
[root@sail ~]# docker run -d -P --net bridge --name tomcat01 tomcat
f048aad0addb07e861c138d167cd644c4c99f9d64c99c6b8ab3f7960fde1dce4
```

在我们启动容器的时候默认会有一个网络设置。

> 自定义网络，先使用 `docker network --help` 命令查询一下

```shell
[root@sail ~]# docker network --help
Usage:  docker network COMMAND
Manage networks
Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks
Run 'docker network COMMAND --help' for more information on a command.
```

### docker network create

> 参数

- `-d`：网络模式
- `--subnet`：子网
- `--gateway`：网关

> 创建自定义网络

```shell
[root@sail ~]# docker network create -d bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
801fbbe1b38c81b12ce90aa9139561b5843dca64b4b17718b6e2622369f9be67
```

> 查看创建的网络

```shell
[root@sail ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
f3eeb014197a   bridge    bridge    local
28d77e958643   host      host      local
801fbbe1b38c   mynet     bridge    local
c3ff850e96f0   none      null      local

[root@sail ~]# docker network inspect 801fbbe1b38c
[
    {
        "Name": "mynet",
        "Id": "801fbbe1b38c81b12ce90aa9139561b5843dca64b4b17718b6e2622369f9be67",
        "Created": "2021-12-30T17:39:43.100705632+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

> 以下配置可以看出，自定义网络创建完成。

```shell
"Config": [
    {
        "Subnet": "192.168.0.0/16",
        "Gateway": "192.168.0.1"
    }
]
```

> 启动镜像

```shell
[root@sail ~]# docker run -d -P --net=mynet --name=tomcat01 tomcat 
f3fad0c65fc3eb9a39b1189a25f5a7f664a0b9415df05cca5ee6edb6b7cc1915
[root@sail ~]# docker run -d -P --net=mynet --name=tomcat02 tomcat
68a78759663854c6d83a14fcc0cf45515e61c5e81d10799e96b22ef79c0d478f
```

> 连接测试

```shell
[root@sail ~]# docker exec -it tomcat01 ping tomcat02
64 bytes from tomcat02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.187 ms
64 bytes from tomcat02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.147 ms
^C
--- tomcat02 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 4ms
rtt min/avg/max/mdev = 0.147/0.167/0.187/0.020 ms
```

能够连通说明不同容器处于同一网络下。

> 这种方式可以实现不同集群使用不同的网络，保证集群网络的安全。

如 Redis 集群在 192.160.0.0/16 网段下，MySQL 集群在 192.161.0.0/16 网段下。

## 网络连通

### docker network connect

使用`docker network connect`实现一个容器链接到另一个网段。

> 建立连接

```shell
[root@sail ~]# docker run -d -P --name=tomcat02-net tomcat
5a02cd4172daccc5073907ee6b063560687db5ffdd5041b18fd3ff1055a8984c

[root@sail ~]# docker network connect mynet tomcat02-net
[root@sail ~]#

[root@sail ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
f3eeb014197a   bridge    bridge    local
28d77e958643   host      host      local
801fbbe1b38c   mynet     bridge    local
c3ff850e96f0   none      null      local

[root@sail ~]# docker network inspect 801fbbe1b38c
[
    {
        "Name": "mynet",
        "Id": "801fbbe1b38c81b12ce90aa9139561b5843dca64b4b17718b6e2622369f9be67",
        "Created": "2021-12-30T17:39:43.100705632+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "5a02cd4172daccc5073907ee6b063560687db5ffdd5041b18fd3ff1055a8984c": {
                "Name": "tomcat02-net",
                "EndpointID": "e6503138bcb91a7693576e324df75d1dff594f1c5aa3e08397802c38133eb0e9",
                "MacAddress": "02:42:c0:a8:00:05",
                "IPv4Address": "192.168.0.5/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

这样也可以实现容器链接到自定义网络。

> 查看容器详情

```shell
[root@sail ~]# docker ps
CONTAINER ID   IMAGE     COMMAND             CREATED             STATUS             PORTS                     NAMES
5a02cd4172da   tomcat    "catalina.sh run"   5 minutes ago       Up 5 minutes       0.0.0.0:49159->8080/tcp   tomcat02-net

[root@sail ~]# docker inspect 5a02cd4172da
```

![img](Docker 14 网络.assets/kuangstudy39dee321-5e55-48fa-b3ec-f9d3b3b4d0f0.png)

这里也可以发现容器 tomcat02-net 已经与 mynet 建立了连接。

> 测试连接

```shell
[root@sail ~]# docker exec -it tomcat02 ping tomcat02-net
PING tomcat02-net (192.168.0.2) 56(84) bytes of data.
64 bytes from tomcat02-net.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.121 ms
64 bytes from tomcat02-net.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.064 ms
^C
--- tomcat02-net ping statistics ---
```

网络连通成功。