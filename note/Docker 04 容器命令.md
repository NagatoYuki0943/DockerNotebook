https://www.kuangstudy.com/bbs/1478274565913587713

> 参考源

https://www.bilibili.com/video/BV1og4y1q7M4?spm_id_from=333.999.0.0

https://www.bilibili.com/video/BV1kv411q7Qc?spm_id_from=333.999.0.0

> 版本

本文章基于 **Docker 20.10.11**

------

# 查看容器

## **docker ps -aq**

> 语法

```shell
docker ps [参数]
```

> 参数

- `-a`：查看所有容器（包括正在运行的和已经停止的）。
- `-n`：显示最近创建的容器，设置显示个数。
- `-q`：只显示容器的编号。

> 查看正在运行的容器

```shell
[root@sail ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS              PORTS     NAMES
1aaf76d85b9e   centos    "/bin/bash"   About a minute ago   Up About a minute             intelligent_proskuriakova
```

> 查看所有容器

```shell
[root@sail ~]# docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED              STATUS                      PORTS     NAMES
1aaf76d85b9e   centos         "/bin/bash"   About a minute ago   Up About a minute                     intelligent_proskuriakova
7ac04abd5a1f   centos         "/bin/bash"   3 minutes ago        Exited (0) 3 minutes ago              elegant_euler
bbb87116d848   centos         "/bin/bash"   8 minutes ago        Exited (0) 3 minutes ago              focused_boyd
81c83ea42dc0   centos         "/bin/bash"   28 minutes ago       Exited (0) 19 minutes ago             zealous_proskuriakova
52918b3ce8f6   feb5d9fea6a5   "/hello"      11 days ago          Exited (0) 11 days ago                friendly_ramanujan
```

> 显示最近创建的 2 个容器

```shell
[root@sail ~]# docker ps -a -n=2
CONTAINER ID   IMAGE     COMMAND       CREATED       STATUS                   PORTS     NAMES
1aaf76d85b9e   centos    "/bin/bash"   5 hours ago   Up 5 hours                         intelligent_proskuriakova
7ac04abd5a1f   centos    "/bin/bash"   5 hours ago   Exited (0) 5 hours ago             elegant_euler
```

> 只显示容器的 ID

```shell
[root@sail ~]# docker ps -aq
1aaf76d85b9e
7ac04abd5a1f
bbb87116d848
81c83ea42dc0
52918b3ce8f6
```

# 退出容器

## **exit **退出后容器停止

进入容器后，可以使用 `exit` 退出

停止并退出容器（后台方式运行则仅退出）

```shell
[root@sail ~]# docker run -it centos /bin/bash
[root@7ac04abd5a1f /]# exit
exit
[root@sail ~]#

[root@sail ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

由此可见，这样退出后容器也会停止。

## **Ctrl + P + Q** 退出后容器不停止

如果想退出后容器不停止，可以使用 `Ctrl + P + Q` 快捷键退出。

```shell
[root@sail ~]# docker run -it centos /bin/bash
[root@1aaf76d85b9e /]# [root@sail ~]# docker ps # 此时即为使用 Ctrl + P + Q 快捷键的效果
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
1aaf76d85b9e   centos    "/bin/bash"   8 seconds ago   Up 8 seconds             intelligent_proskuriakova
```

# 删除容器

## **docker rm**

> 语法

```shell
docker rm [参数] 容器 [容器...]
```

> 参数

- `-f`：强制删除。

> 删除指定容器（不能删除正在运行的容器）

```shell
[root@sail ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED       STATUS       PORTS     NAMES
1aaf76d85b9e   centos    "/bin/bash"   5 hours ago   Up 5 hours             intelligent_proskuriakova

[root@sail ~]# docker rm 1aaf76d85b9e
Error response from daemon: You cannot remove a running container 1aaf76d85b9ee5002411c1ea390fca05819f19dc400e85127731d37455cb0acc. Stop the container before attempting removal or force remove
```

> 强制删除指定容器

```shell
[root@sail ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED       STATUS       PORTS     NAMES
1aaf76d85b9e   centos    "/bin/bash"   5 hours ago   Up 5 hours             intelligent_proskuriakova

[root@sail ~]# docker rm -f 1aaf76d85b9e
1aaf76d85b9e
```

> 删除所有容器。先使用 `docker ps -aq` 获取所有容器的 ID，再调用 `docker rm -f` 递归删除。

```shell
[root@sail ~]# docker rm -f $(docker ps -aq)
7ac04abd5a1f
bbb87116d848
81c83ea42dc0
52918b3ce8f6

[root@sail ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

> 删除所有容器。使用管道符 `|` 获取 Docker 相关的所有容器 ID 并使用 `docker rm -f` 删除。

```shell
[root@sail ~]# docker ps -a -q|xargs docker rm -f
2e61c4578eac
0ebe32ddfa50
```

# 启动容器

## **docker start id/name**

> 查看所有的容器，容器状态为**关闭**

```shell
[root@sail ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                     PORTS     NAMES
569026bc0955   centos    "/bin/bash"   33 seconds ago   Exited (0) 7 seconds ago             centos03
```

> 运行关闭的容器

```shell
[root@sail ~]# docker start 569026bc0955 
569026bc0955
```

> 再次查看所有的容器，容器状态为**运行**

```shell
[root@sail ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS              PORTS     NAMES
569026bc0955   centos    "/bin/bash"   About a minute ago   Up 1 second                   centos03
```

# 停止容器

## **docker stop id/name**

> 查看所有的容器，容器状态为**运行**

```shell
[root@sail ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS              PORTS     NAMES
569026bc0955   centos    "/bin/bash"   About a minute ago   Up 1 second                   centos03
```

> 关闭运行的容器

```shell
[root@sail ~]# docker stop 569026bc0955
569026bc0955
```

> 再次查看所有的容器，容器状态为**关闭**

```shell
[root@sail ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS                      PORTS     NAMES
569026bc0955   centos    "/bin/bash"   4 minutes ago   Exited (0) 38 seconds ago             centos03
```

# 重启容器

## **docker restart id/name**

> 查看所有的容器，容器状态为**关闭**

```shell
[root@sail ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS                      PORTS     NAMES
569026bc0955   centos    "/bin/bash"   4 minutes ago   Exited (0) 38 seconds ago             centos03
```

> 重启关闭的容器

```shell
[root@sail ~]# docker restart 569026bc0955
569026bc0955
```

> 再次查看所有的容器，容器状态为**运行**

```shell
[root@sail ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS          PORTS     NAMES
569026bc0955   centos    "/bin/bash"   6 minutes ago   Up 20 seconds             centos03
```

# 杀掉容器

## **docker kill id/name**

> 查看所有的容器，容器状态为**运行**

```shell
[root@sail ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS          PORTS     NAMES
569026bc0955   centos    "/bin/bash"   6 minutes ago   Up 20 seconds             centos03
```

> 杀掉运行的容器

```shell
[root@sail ~]# docker kill 569026bc0955
569026bc0955
```

> 再次查看所有的容器，容器状态为**关闭**

```shell
[root@sail ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS                        PORTS     NAMES
569026bc0955   centos    "/bin/bash"   8 minutes ago   Exited (137) 15 seconds ago             centos03
```



https://www.kuangstudy.com/bbs/1478921549741735938

> 参考源

https://www.bilibili.com/video/BV1og4y1q7M4?spm_id_from=333.999.0.0

https://www.bilibili.com/video/BV1kv411q7Qc?spm_id_from=333.999.0.0

# 查看日志

## **docker logs**

> 语法

```shell
 docker logs [参数] 容器
```

> 参数

- `-f`：日志流动输出。
- `-t`：展示时间戳。
- `--tail`：从日志末尾显示的行数。

> 为模拟日志输出效果，我们先编写一段脚本

```shell
while true;do echo sail;sleep 3;done
```

以上脚本实现的效果为：每隔 3 秒输出字符串 sail。

> 以脚本启动容器

```shell
[root@sail ~]# docker run -d centos /bin/sh -c "while true;do echo sail;sleep 3;done"
c3d59f55d600566a5bbd9411ce716fbd338efc1c7c991f5baf1152f021d7e151

[root@sail ~]# docker ps 
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
c3d59f55d600   centos    "/bin/sh -c 'while t…"   7 seconds ago   Up 7 seconds             musing_poincare

[root@sail ~]# docker logs -f -t --tail 10 c3d59f55d600
2021-12-10T03:01:28.607288480Z sail
2021-12-10T03:01:31.609334595Z sail
2021-12-10T03:01:34.611361943Z sail
2021-12-10T03:01:37.613461457Z sail
2021-12-10T03:01:40.615619089Z sail
2021-12-10T03:01:43.617595572Z sail
2021-12-10T03:01:46.619526233Z sail
2021-12-10T03:01:49.621856800Z sail
2021-12-10T03:01:52.623864131Z sail
2021-12-10T03:01:55.626146643Z sail
2021-12-10T03:01:58.628272273Z sail
2021-12-10T03:02:01.630370420Z sail
2021-12-10T03:02:04.632430066Z sail
2021-12-10T03:02:07.634690353Z sail
2021-12-10T03:02:10.636760025Z sail
```

可以看到，按此命令会看到容器最后 10 条日志，且每隔 3 秒滚动输出一条日志。

# 后台启动

## **docker run -d id/name**

> 语法

```shell
docker run -d 镜像
```

> 后台启动镜像

```shell
[root@sail ~]# docker run -d centos
0aee6f74b913f120195ca323892867bba7d72f2671f2f8b17278a3e029ad5bfd

[root@sail ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED        STATUS        PORTS     NAMES
```

> 使用 `docker run -d` 启动，也并不能保证容器一定能在后台运行，如果没有前台使用，容器启动后发现自己没有提供服务，会立刻停止。

前面的 `docker run -d centos /bin/sh -c "while true;do echo sail;sleep 3;done"` 命令，由于启动后运行了脚本打印日志，即提供了服务，所以不会停止。

# 查看容器信息

## **docker inspect image/container**

> 语法

```shell
docker inspect 容器
```

> 示例

```shell
[root@sail ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS         PORTS     NAMES
c3d59f55d600   centos    "/bin/sh -c 'while t…"   24 minutes ago   Up 5 seconds             musing_poincare

[root@sail ~]# docker inspect c3d59f55d600
[
    {
        "Id": "c3d59f55d600566a5bbd9411ce716fbd338efc1c7c991f5baf1152f021d7e151", # 容器的完整ID
        "Created": "2021-12-10T02:59:43.245101004Z", # 创建时间
        "Path": "/bin/sh", # 脚本位置
        "Args": [ # 运行的脚本
            "-c",
            "while true;do echo sail;sleep 3;done"
        ],
        "State": {
            "Status": "running", # 状态：正在运行
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 31277, # 父进程ID
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-12-10T03:23:37.744951214Z",
            "FinishedAt": "2021-12-10T03:04:59.090158114Z"
        },
        "Image": "sha256:5d0da3dc976460b72c77d94c8a1ad043720b0416bfc16c52c45d4847e53fadb6", # 来源镜像
        "ResolvConfPath": "/var/lib/docker/containers/c3d59f55d600566a5bbd9411ce716fbd338efc1c7c991f5baf1152f021d7e151/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/c3d59f55d600566a5bbd9411ce716fbd338efc1c7c991f5baf1152f021d7e151/hostname",
        "HostsPath": "/var/lib/docker/containers/c3d59f55d600566a5bbd9411ce716fbd338efc1c7c991f5baf1152f021d7e151/hosts",
        "LogPath": "/var/lib/docker/containers/c3d59f55d600566a5bbd9411ce716fbd338efc1c7c991f5baf1152f021d7e151/c3d59f55d600566a5bbd9411ce716fbd338efc1c7c991f5baf1152f021d7e151-json.log",
        "Name": "/musing_poincare",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": { # 主机配置
            "Binds": null,
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
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
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
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": { # 其他配置
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/ed38d7c2061c4ebc46138b1e181e8286c0ca9242292945be7e2d4a9cef77eb48-init/diff:/var/lib/docker/overlay2/03139488f80fb2bbb139709a63d45233d63ef0258452b719b74f23e78a6aca7c/diff",
                "MergedDir": "/var/lib/docker/overlay2/ed38d7c2061c4ebc46138b1e181e8286c0ca9242292945be7e2d4a9cef77eb48/merged",
                "UpperDir": "/var/lib/docker/overlay2/ed38d7c2061c4ebc46138b1e181e8286c0ca9242292945be7e2d4a9cef77eb48/diff",
                "WorkDir": "/var/lib/docker/overlay2/ed38d7c2061c4ebc46138b1e181e8286c0ca9242292945be7e2d4a9cef77eb48/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [], # 挂载
        "Config": { # 基本配置
            "Hostname": "c3d59f55d600",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [ # 基本环境变量
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [ # 基本命令
                "/bin/sh",
                "-c",
                "while true;do echo sail;sleep 3;done"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20210915",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": { # 网卡设置
            "Bridge": "",
            "SandboxID": "421b87a4fcfaff65125a707b61b1ba88aa9fc731c452a66f8f1731131721a90a",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/421b87a4fcfa",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "3c3fce24d19469194202087ff3fe08135ebdc969773c75300c54db4190609467",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.4",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:04",
            "Networks": {
                "bridge": { # 桥接类型网卡
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "f3eeb014197a66bb6d740738f4e8148db7f8cb6c0b7432e429ff61a0cf5e0b06",
                    "EndpointID": "3c3fce24d19469194202087ff3fe08135ebdc969773c75300c54db4190609467",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.4",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:04",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

> 不管容器是否运行，都可以使用该命令查看。

# 进入正在运行的容器

容器是一个微型的 Linux 系统，我们通常需要进入容器进行操作。

## **docker exec id/name**

使用 `docker exec` 可以进入容器并开启一个新的终端，可以在里面操作。

> 语法

```shell
docker exec [参数] 容器 路径
```

> 参数

- `-d`：后台运行。
- `-it`：交互模式进入。

```shell
[root@sail ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED        STATUS        PORTS     NAMES
96ed3fe3e7f1   centos    "/bin/bash"   15 hours ago   Up 15 hours             centos01

[root@sail ~]# docker exec -it 96ed3fe3e7f1 /bin/bash
[root@96ed3fe3e7f1 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

[root@96ed3fe3e7f1 /]# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 Dec09 pts/0    00:00:00 /bin/bash
root        15     0  0 03:38 pts/1    00:00:00 /bin/bash
root        30    15  0 03:39 pts/1    00:00:00 ps -ef
```

这种进入方式是单独开了一个新进程的方式。

## **docker attach id/name**

使用 `docker attach` 会进入容器正在执行的终端，不会启动新的进程。

> 语法

```shell
docker attach 容器

[root@sail ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED        STATUS        PORTS     NAMES
96ed3fe3e7f1   centos    "/bin/bash"   17 hours ago   Up 17 hours             centos01

[root@sail ~]# docker attach 96ed3fe3e7f1
[root@96ed3fe3e7f1 /]# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 Dec09 pts/0    00:00:00 /bin/bash
root        33     1  0 05:49 pts/0    00:00:00 ps -ef
```

这种进入方式没有开启新的进程（`/bin/bash` 是 centos 容器的默认终端）。

# 从容器内拷贝文件到主机

> 查看启动的容器

```shell
[root@sail ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED        STATUS        PORTS     NAMES
96ed3fe3e7f1   centos    "/bin/bash"   17 hours ago   Up 17 hours             centos01
```

> 进入容器，创建一个文件

```shell
[root@sail ~]# docker attach 96ed3fe3e7f1
[root@96ed3fe3e7f1 /]# cd /home
[root@96ed3fe3e7f1 home]# touch test.java
[root@96ed3fe3e7f1 home]# ls
test.java
[root@96ed3fe3e7f1 home]# exit
exit
```

> 退出容器后，不管容器是否启动，都可以复制容器中的文件到主机上

```shell
[root@sail ~]# cd /home
[root@sail home]# docker cp 96ed3fe3e7f1:/home/test.java /home
[root@sail home]# ls
admin  f2  f3  sail  test.java
```

> 这种方式是一个手动过程，很不方便，推荐使用**数据卷技术**，可以实现自动同步主机和容器的目录。

详情见：[Docker 12 数据卷](https://www.kuangstudy.com/bbs/1484782140666593282)

# 查看Docker内存占用

## **docker stats**

> 语法

```shell
docker stats [参数] [容器...]
```

> 参数

- `-a`：查看所有容器的内存占用（默认只展示运行的容器）。

```shell
[root@sail home]# docker stats
CONTAINER ID   NAME       CPU %     MEM USAGE / LIMIT   MEM %     NET I/O   BLOCK I/O   PIDS
96ed3fe3e7f1   centos01   0.00%     524KiB / 1.694GiB   0.03%     0B / 0B   0B / 0B     1

[root@sail home]# docker stats -a
CONTAINER ID   NAME              CPU %     MEM USAGE / LIMIT   MEM %     NET I/O   BLOCK I/O   PIDS
0aee6f74b913   brave_rosalind    0.00%     0B / 0B             0.00%     0B / 0B   0B / 0B     0
c3d59f55d600   musing_poincare   0.00%     0B / 0B             0.00%     0B / 0B   0B / 0B     0
569026bc0955   centos03          0.00%     0B / 0B             0.00%     0B / 0B   0B / 0B     0
71a97b830ec5   centos02          0.00%     0B / 0B             0.00%     0B / 0B   0B / 0B     0
96ed3fe3e7f1   centos01          0.00%     524KiB / 1.694GiB   0.03%     0B / 0B   0B / 0B     1
```

