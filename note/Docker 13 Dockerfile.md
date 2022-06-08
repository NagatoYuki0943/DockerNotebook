https://www.kuangstudy.com/bbs/1485174981762945026

> 参考源

https://www.bilibili.com/video/BV1og4y1q7M4?spm_id_from=333.999.0.0

https://www.bilibili.com/video/BV1kv411q7Qc?spm_id_from=333.999.0.0

> 版本

本文章基于 **Docker 20.10.11**

------

# 简介

![img](Docker 13 Dockerfile.assets/kuangstudybd744e77-1e24-41e0-9737-d169690f30cf.png)

Dockerfile 是用来构建 Docker 镜像的文件，可以理解为**命令参数脚本**。

Dockerfile 是面向开发的，想要打包项目，就要编写 Dockerfile 文件。

> 由于 Docker 的流行，Docker 镜像逐渐替代 **jar** 或者 **war** 成为企业的交付标准。

# 官方 Dockerfile

> 首先看一下官方的 Dockerfile。

这里以 centos 的镜像为例。

> 在 [Docker Hub](https://hub.docker.com/) 搜索 centos 镜像，选择官方镜像。

![img](Docker 13 Dockerfile.assets/kuangstudy07af1e35-15eb-4529-8d12-ef5ea0c1ea8e.png)

> 点击 **The CentOS Project** 。

![img](Docker 13 Dockerfile.assets/kuangstudyaa509032-be45-4ece-ad06-ab89df41112f.png)

> 选择版本分支。这里以 **CentOS-7.6.1810** 为例。

![img](Docker 13 Dockerfile.assets/kuangstudyb7697c68-22ef-4905-9170-62378aef5f0b.png)

> 查看 Dockerfile。

![img](Docker 13 Dockerfile.assets/kuangstudy55cc0f0a-1230-46df-a5dc-771be68096e0.png)

![img](Docker 13 Dockerfile.assets/kuangstudy62bf47da-a420-4fd3-90db-035c70d5280c.png)

> 可以看出官方的 Dockerfile 是很简洁的，这就是 Docker 极致精简的原因。

不过，官方镜像都是基础包，很多功能是没有的，比如 centos 的官方镜像是没有 `vim` 命令的。

像这种基础命令比较常用，如果想要使用可以在容器内安装，但这样很不方便，因为每个新启动的容器都要安装。

我们通常会选择自己搭建镜像，这就要用到 Dockerfile 了。

# 命令

以上面的 centos 官方镜像的 Dockerfile 为例。

```shell
FROM scratch
ADD centos-7-docker.tar.xz /

LABEL org.label-schema.schema-version="1.0" \
    org.label-schema.name="CentOS Base Image" \
    org.label-schema.vendor="CentOS" \
    org.label-schema.license="GPLv2" \
    org.label-schema.build-date="20181204"
   
CMD ["/bin/bash"]
```

Docker Hub 中 99% 的镜像都是从`FROM scratch` 开始的。

> 规则

- 每个指令都必须是大写字母。
- 按照从上到下顺序执行。
- **#** 表示注释。
- 每一条指令都会创建一个新的镜像层。

> 解释

- `FROM`：基础镜像，比如 centos。
- `MAINTAINER`：镜像是谁写的。建议以此格式：`姓名<邮箱>`。
- `RUN`：镜像构建时需要运行的命令。
- `ADD`：添加，比如添加一个 tomcat 压缩包。
- `WORKDIR`：镜像的工作目录。
- `VOLUME`：挂载的目录。
- `EXPOSE`：指定暴露端口，跟 -p 一个道理。
- `RUN`：最终要运行的。
- `CMD`：指定这个容器启动的时候要运行的命令，只有最后一个会生效，而且可被替代。
- `ENTRYPOINT`：指定这个容器启动的时候要运行的命令，可以追加命令。
- `ONBUILD`：当构建一个被继承Dockerfile 这个时候运行ONBUILD指定，触发指令。
- `COPY`：将文件拷贝到镜像中。
- `ENV`：构建的时候设置环境变量。

## 构建镜像

### docker build

Dockerfile 编写好后，需要使用 `docker build` 命令运行。

> 语法

```shell
docker build [参数] 路径 | 网络地址 | -
```

> 参数

- `-f`：指定要使用的Dockerfile路径。
- `-t`：镜像的名字及标签，通常 **name:tag** 或者 **name** 格式；可以在一次构建中为一个镜像设置多个标签。
- `-m`：设置内存最大值。

> Docker 守护进程执行 Dockerfile 中的指令前，首先会对 Dockerfile 进行语法检查，有语法错误时会返回报错信息。

```shell
Error response from daemon: Unknown instruction: RUNCMD
```

## 查看构建记录

### docker history

> 语法

```shell
docker history 镜像
```

## CMD 与 ENTRYPOINT 区别

### CMD 命令演示

> 编写 Dockerfile

```shell
[root@sail dockerfile]# vim Dockerfile-cmd-test
[root@sail dockerfile]# cat Dockerfile-cmd-test 
FROM centos
CMD ["ls","-a"]
```

> 构建镜像

```shell
[root@sail dockerfile]# docker build -f Dockerfile-cmd-test -t cmdtest .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM centos
 ---> 5d0da3dc9764
Step 2/2 : CMD ["ls","-a"]
 ---> Running in 0a743e929fff
Removing intermediate container 0a743e929fff
 ---> 1683c0790d49
Successfully built 1683c0790d49
Successfully tagged cmdtest:latest

[root@sail dockerfile]# docker images
REPOSITORY        TAG       IMAGE ID       CREATED          SIZE
cmdtest           latest    1683c0790d49   13 minutes ago   231MB
```

> 运行镜像

```shell
[root@sail dockerfile]# docker run cmdtest
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
```

此时 Dockerfile 中编写的命令生效了。

> 追加 `-l` 命令

```shell
[root@sail dockerfile]# docker run cmdtest -l
docker: Error response from daemon: OCI runtime create failed: container_linux.go:380: starting container process caused: exec: "-l": executable file not found in $PATH: unknown.
ERRO[0000] error waiting for container: context canceled
```

没有达到预期的 `ls -al` 命令。

> `CMD` 是**替换**的方式， `-l` 不是命令，所以报错。

### ENTRYPOINT 命令演示

> 编写 Dockerfile

```shell
[root@sail dockerfile]# vim Dockerfile-entrypoint-test
[root@sail dockerfile]# cat Dockerfile-entrypoint-test 
FROM centos
ENTRYPOINT ["ls","-a"]
```

> 构建镜像

```shell
[root@sail dockerfile]# docker build -f Dockerfile-entrypoint-test -t entrypoint-test .
Sending build context to Docker daemon  3.072kB
Step 1/2 : FROM centos
 ---> 5d0da3dc9764
Step 2/2 : ENTRYPOINT ["ls","-a"]
 ---> Running in a02d55ae0a00
Removing intermediate container a02d55ae0a00
 ---> 795973a0ed43
Successfully built 795973a0ed43
Successfully tagged entrypoint-test:latest

[root@sail dockerfile]# docker images
REPOSITORY        TAG       IMAGE ID       CREATED          SIZE
entrypoint-test   latest    795973a0ed43   22 seconds ago   231MB
```

> 运行镜像

```shell
[root@sail dockerfile]# docker run entrypoint-test
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
```

此时 Dockerfile 中编写的命令也生效了。

> 追加 `-l` 命令

```shell
[root@sail dockerfile]# docker run entrypoint-test -l
total 56
drwxr-xr-x   1 root root 4096 Dec 23 06:46 .
drwxr-xr-x   1 root root 4096 Dec 23 06:46 ..
-rwxr-xr-x   1 root root    0 Dec 23 06:46 .dockerenv
lrwxrwxrwx   1 root root    7 Nov  3  2020 bin -> usr/bin
drwxr-xr-x   5 root root  340 Dec 23 06:46 dev
drwxr-xr-x   1 root root 4096 Dec 23 06:46 etc
drwxr-xr-x   2 root root 4096 Nov  3  2020 home
lrwxrwxrwx   1 root root    7 Nov  3  2020 lib -> usr/lib
lrwxrwxrwx   1 root root    9 Nov  3  2020 lib64 -> usr/lib64
drwx------   2 root root 4096 Sep 15 14:17 lost+found
drwxr-xr-x   2 root root 4096 Nov  3  2020 media
drwxr-xr-x   2 root root 4096 Nov  3  2020 mnt
drwxr-xr-x   2 root root 4096 Nov  3  2020 opt
dr-xr-xr-x 106 root root    0 Dec 23 06:46 proc
dr-xr-x---   2 root root 4096 Sep 15 14:17 root
drwxr-xr-x  11 root root 4096 Sep 15 14:17 run
lrwxrwxrwx   1 root root    8 Nov  3  2020 sbin -> usr/sbin
drwxr-xr-x   2 root root 4096 Nov  3  2020 srv
dr-xr-xr-x  13 root root    0 Oct 31 15:00 sys
drwxrwxrwt   7 root root 4096 Sep 15 14:17 tmp
drwxr-xr-x  12 root root 4096 Sep 15 14:17 usr
drwxr-xr-x  20 root root 4096 Sep 15 14:17 var
```

运行了预期的 `ls -al` 命令。

> `ENTRYPOINT` 是**追加**的方式。

Docker 中许多命令都十分相似，我们需要了解他们的区别，最好的方式就是这样对比测试。

# 实战

## 创建包含vim命令的centos镜像

> 编写 Dockerfile

```shell
[root@sail dockerfile]# vim Dockerfile-centos-test

[root@sail dockerfile]# cat Dockerfile-centos-test

FROM centos

MAINTAINER sail<yifansailing@163.com>

ENV MYPATH /usr/local

WORKDIR $MYPATH

RUN yum -y install vim

RUN yum -y install net-tools

EXPOSE 81

CMD echo $MYPATH

CMD echo "---end---"

CMD ["/bin/bash"]
```

> 构建镜像

```shell
[root@sail dockerfile]# docker build -f Dockerfile-centos-test -t centos-test .
Sending build context to Docker daemon  5.632kB
Step 1/10 : FROM centos
 ---> 5d0da3dc9764
Step 2/10 : MAINTAINER sail<yifansailing@163.com>
 ---> Running in 8b7340768878
Removing intermediate container 8b7340768878
 ---> 9616888f3b10
Step 3/10 : ENV MYPATH /usr/local
 ---> Running in 2c73446a56ff
Removing intermediate container 2c73446a56ff
 ---> be89377d4c2c
Step 4/10 : WORKDIR $MYPATH
 ---> Running in db113c4f7cb2
Removing intermediate container db113c4f7cb2
 ---> fb41ece5d944
Step 5/10 : RUN yum -y install vim
 ---> Running in eccee60c0389
CentOS Linux 8 - AppStream                       12 MB/s | 8.4 MB     00:00    
CentOS Linux 8 - BaseOS                         1.7 MB/s | 3.6 MB     00:02    
CentOS Linux 8 - Extras                          17 kB/s |  10 kB     00:00    
Last metadata expiration check: 0:00:01 ago on Sat Dec 25 05:09:40 2021.
Dependencies resolved.
================================================================================
 Package             Arch        Version                   Repository      Size
================================================================================
Installing:
 vim-enhanced        x86_64      2:8.0.1763-16.el8         appstream      1.4 M
Installing dependencies:
 gpm-libs            x86_64      1.20.7-17.el8             appstream       39 k
 vim-common          x86_64      2:8.0.1763-16.el8         appstream      6.3 M
 vim-filesystem      noarch      2:8.0.1763-16.el8         appstream       49 k
 which               x86_64      2.21-16.el8               baseos          49 k
Transaction Summary
================================================================================
Install  5 Packages
Total download size: 7.8 M
Installed size: 30 M
Downloading Packages:
(1/5): gpm-libs-1.20.7-17.el8.x86_64.rpm        582 kB/s |  39 kB     00:00    
(2/5): vim-filesystem-8.0.1763-16.el8.noarch.rp 1.2 MB/s |  49 kB     00:00    
(3/5): vim-common-8.0.1763-16.el8.x86_64.rpm     40 MB/s | 6.3 MB     00:00    
(4/5): vim-enhanced-8.0.1763-16.el8.x86_64.rpm  7.1 MB/s | 1.4 MB     00:00    
(5/5): which-2.21-16.el8.x86_64.rpm             252 kB/s |  49 kB     00:00    
--------------------------------------------------------------------------------
Total                                           6.4 MB/s | 7.8 MB     00:01     
warning: /var/cache/dnf/appstream-02e86d1c976ab532/packages/gpm-libs-1.20.7-17.el8.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 8483c65d: NOKEY
CentOS Linux 8 - AppStream                      1.6 MB/s | 1.6 kB     00:00    
Importing GPG key 0x8483C65D:
 Userid     : "CentOS (CentOS Official Signing Key) <security@centos.org>"
 Fingerprint: 99DB 70FA E1D7 CE22 7FB6 4882 05B5 55B3 8483 C65D
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : which-2.21-16.el8.x86_64                               1/5 
  Installing       : vim-filesystem-2:8.0.1763-16.el8.noarch                2/5 
  Installing       : vim-common-2:8.0.1763-16.el8.x86_64                    3/5 
  Installing       : gpm-libs-1.20.7-17.el8.x86_64                          4/5 
  Running scriptlet: gpm-libs-1.20.7-17.el8.x86_64                          4/5 
  Installing       : vim-enhanced-2:8.0.1763-16.el8.x86_64                  5/5 
  Running scriptlet: vim-enhanced-2:8.0.1763-16.el8.x86_64                  5/5 
  Running scriptlet: vim-common-2:8.0.1763-16.el8.x86_64                    5/5 
  Verifying        : gpm-libs-1.20.7-17.el8.x86_64                          1/5 
  Verifying        : vim-common-2:8.0.1763-16.el8.x86_64                    2/5 
  Verifying        : vim-enhanced-2:8.0.1763-16.el8.x86_64                  3/5 
  Verifying        : vim-filesystem-2:8.0.1763-16.el8.noarch                4/5 
  Verifying        : which-2.21-16.el8.x86_64                               5/5 
Installed:
  gpm-libs-1.20.7-17.el8.x86_64         vim-common-2:8.0.1763-16.el8.x86_64    
  vim-enhanced-2:8.0.1763-16.el8.x86_64 vim-filesystem-2:8.0.1763-16.el8.noarch
  which-2.21-16.el8.x86_64             
Complete!
Removing intermediate container eccee60c0389
 ---> 9f54f48660ac
Step 6/10 : RUN yum -y install net-tools
 ---> Running in 6caa7361b001
Last metadata expiration check: 0:00:08 ago on Sat Dec 25 05:09:40 2021.
Dependencies resolved.
================================================================================
 Package         Architecture Version                        Repository    Size
================================================================================
Installing:
 net-tools       x86_64       2.0-0.52.20160912git.el8       baseos       322 k
Transaction Summary
================================================================================
Install  1 Package
Total download size: 322 k
Installed size: 942 k
Downloading Packages:
net-tools-2.0-0.52.20160912git.el8.x86_64.rpm   1.0 MB/s | 322 kB     00:00    
--------------------------------------------------------------------------------
Total                                           449 kB/s | 322 kB     00:00     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : net-tools-2.0-0.52.20160912git.el8.x86_64              1/1 
  Running scriptlet: net-tools-2.0-0.52.20160912git.el8.x86_64              1/1 
  Verifying        : net-tools-2.0-0.52.20160912git.el8.x86_64              1/1 
Installed:
  net-tools-2.0-0.52.20160912git.el8.x86_64                                     
Complete!
Removing intermediate container 6caa7361b001
 ---> a9431f90fd3f
Step 7/10 : EXPOSE 81
 ---> Running in ad67fa23940a
Removing intermediate container ad67fa23940a
 ---> b5bd21416741
Step 8/10 : CMD echo $MYPATH
 ---> Running in fb1d08538689
Removing intermediate container fb1d08538689
 ---> 5c5def0bbb85
Step 9/10 : CMD echo "---end---"
 ---> Running in a9d955b6b389
Removing intermediate container a9d955b6b389
 ---> ad95558eb658
Step 10/10 : CMD ["/bin/bash"]
 ---> Running in 190651202e7b
Removing intermediate container 190651202e7b
 ---> d58be7785771
Successfully built d58be7785771
Successfully tagged centos-test:latest
```

> 查看构建的镜像

```shell
[root@sail dockerfile]# docker images
REPOSITORY        TAG       IMAGE ID       CREATED              SIZE
centos-test       latest    d58be7785771   About a minute ago   323MB
```

> 查看本地镜像的构建记录

```shell
[root@sail dockerfile]# docker history centos-test
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
d58be7785771   5 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
ad95558eb658   5 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B        
5c5def0bbb85   5 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B        
b5bd21416741   5 minutes ago   /bin/sh -c #(nop)  EXPOSE 81                    0B        
a9431f90fd3f   5 minutes ago   /bin/sh -c yum -y install net-tools             27.3MB    
9f54f48660ac   5 minutes ago   /bin/sh -c yum -y install vim                   64.8MB    
fb41ece5d944   5 minutes ago   /bin/sh -c #(nop) WORKDIR /usr/local            0B        
be89377d4c2c   5 minutes ago   /bin/sh -c #(nop)  ENV MYPATH=/usr/local        0B        
9616888f3b10   5 minutes ago   /bin/sh -c #(nop)  MAINTAINER sail<yifansail…   0B        
5d0da3dc9764   3 months ago    /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      3 months ago    /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B        
<missing>      3 months ago    /bin/sh -c #(nop) ADD file:805cb5e15fb6e0bb0…   231MB
```

> 运行测试

```shell
[root@sail ~]# docker run -it centos-test
[root@530551bc2162 local]# pwd
/usr/local
[root@530551bc2162 local]# vim test.java
[root@530551bc2162 local]#
```

默认的工作目录正是 Dockerfile 中设置的 `/usr/local` ，且可以使用 `vim` 命令了。

## 自定义tomcat环境镜像

> 编写 Dockerfile

```shell
FROM centos

MAINTAINER sail<yifansailing@163.com>

COPY readme.txt /usr/local/readme.txt

ENV MYPATH /usr/local/

WORKDIR $MYPATH

ADD jdk-8u301-linux-x64.tar.gz $MYPATH

ADD apache-tomcat-9.0.55.tar.gz $MYPATH

RUN yum -y install vim

ENV JAVA_HOME $MYPATH/jdk1.8.0_301-amd64

ENV CLASSPATH $JAVA_HOME/lib/

ENV CATALINA_HOME $MYPATH/apache-tomcat-9.0.55

ENV CATALINA_BASH $MYPATH/apache-tomcat-9.0.55

ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/bin:$CATALINA_HOME/lib

EXPOSE 8080

CMD $CATALINA_HOME/bin/startup.sh && tail -F $CATALINA_HOME/logs/catalina.out
```

其中的 **readme.txt** 一般作为镜像说明文件，可以在里面编写镜像的信息。

> 构建镜像

```shell
[root@sail tomcat]# docker build -t tomcat-test .
Sending build context to Docker daemon  157.1MB
Step 1/15 : FROM centos
 ---> 5d0da3dc9764
Step 2/15 : MAINTAINER sail<yifansailing@163.com>
 ---> Using cache
 ---> 9616888f3b10
Step 3/15 : COPY readme.txt /usr/local/readme.txt
 ---> da792df641f8
Step 4/15 : ENV MYPATH /usr/local/
 ---> Running in e4a5b13decd7
Removing intermediate container e4a5b13decd7
 ---> 7b1e6970b4b3
Step 5/15 : WORKDIR $MYPATH
 ---> Running in 835dabd080dd
Removing intermediate container 835dabd080dd
 ---> 7be17b1556ee
Step 6/15 : ADD jdk-8u301-linux-x64.tar.gz $MYPATH
 ---> 480721043fda
Step 7/15 : ADD apache-tomcat-9.0.55.tar.gz $MYPATH
 ---> c7bfa13bfcd1
Step 8/15 : RUN yum -y install vim
 ---> Running in 85532523d784
CentOS Linux 8 - AppStream                      9.0 MB/s | 8.4 MB     00:00    
CentOS Linux 8 - BaseOS                         5.6 MB/s | 3.6 MB     00:00    
CentOS Linux 8 - Extras                          20 kB/s |  10 kB     00:00    
Dependencies resolved.
================================================================================
 Package             Arch        Version                   Repository      Size
================================================================================
Installing:
 vim-enhanced        x86_64      2:8.0.1763-16.el8         appstream      1.4 M
Installing dependencies:
 gpm-libs            x86_64      1.20.7-17.el8             appstream       39 k
 vim-common          x86_64      2:8.0.1763-16.el8         appstream      6.3 M
 vim-filesystem      noarch      2:8.0.1763-16.el8         appstream       49 k
 which               x86_64      2.21-16.el8               baseos          49 k
Transaction Summary
================================================================================
Install  5 Packages
Total download size: 7.8 M
Installed size: 30 M
Downloading Packages:
(1/5): gpm-libs-1.20.7-17.el8.x86_64.rpm        973 kB/s |  39 kB     00:00    
(2/5): vim-filesystem-8.0.1763-16.el8.noarch.rp 726 kB/s |  49 kB     00:00    
(3/5): vim-enhanced-8.0.1763-16.el8.x86_64.rpm   10 MB/s | 1.4 MB     00:00    
(4/5): which-2.21-16.el8.x86_64.rpm             901 kB/s |  49 kB     00:00    
(5/5): vim-common-8.0.1763-16.el8.x86_64.rpm     27 MB/s | 6.3 MB     00:00    
--------------------------------------------------------------------------------
Total                                           6.6 MB/s | 7.8 MB     00:01     
warning: /var/cache/dnf/appstream-02e86d1c976ab532/packages/gpm-libs-1.20.7-17.el8.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 8483c65d: NOKEY
CentOS Linux 8 - AppStream                      1.6 MB/s | 1.6 kB     00:00    
Importing GPG key 0x8483C65D:
 Userid     : "CentOS (CentOS Official Signing Key) <security@centos.org>"
 Fingerprint: 99DB 70FA E1D7 CE22 7FB6 4882 05B5 55B3 8483 C65D
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : which-2.21-16.el8.x86_64                               1/5 
  Installing       : vim-filesystem-2:8.0.1763-16.el8.noarch                2/5 
  Installing       : vim-common-2:8.0.1763-16.el8.x86_64                    3/5 
  Installing       : gpm-libs-1.20.7-17.el8.x86_64                          4/5 
  Running scriptlet: gpm-libs-1.20.7-17.el8.x86_64                          4/5 
  Installing       : vim-enhanced-2:8.0.1763-16.el8.x86_64                  5/5 
  Running scriptlet: vim-enhanced-2:8.0.1763-16.el8.x86_64                  5/5 
  Running scriptlet: vim-common-2:8.0.1763-16.el8.x86_64                    5/5 
  Verifying        : gpm-libs-1.20.7-17.el8.x86_64                          1/5 
  Verifying        : vim-common-2:8.0.1763-16.el8.x86_64                    2/5 
  Verifying        : vim-enhanced-2:8.0.1763-16.el8.x86_64                  3/5 
  Verifying        : vim-filesystem-2:8.0.1763-16.el8.noarch                4/5 
  Verifying        : which-2.21-16.el8.x86_64                               5/5 
Installed:
  gpm-libs-1.20.7-17.el8.x86_64         vim-common-2:8.0.1763-16.el8.x86_64    
  vim-enhanced-2:8.0.1763-16.el8.x86_64 vim-filesystem-2:8.0.1763-16.el8.noarch
  which-2.21-16.el8.x86_64             
Complete!
Removing intermediate container 85532523d784
 ---> e091ece0364d
Step 9/15 : ENV JAVA_HOME $MYPATH/jdk1.8.0_301-amd64
 ---> Running in 473066cf57f4
Removing intermediate container 473066cf57f4
 ---> 0a8963a2c1ab
Step 10/15 : ENV CLASSPATH $JAVA_HOME/lib/
 ---> Running in 78a2cb9b06cd
Removing intermediate container 78a2cb9b06cd
 ---> 3dd34a2857b4
Step 11/15 : ENV CATALINA_HOME $MYPATH/apache-tomcat-9.0.55
 ---> Running in 4ca540479e3d
Removing intermediate container 4ca540479e3d
 ---> fa38f4581510
Step 12/15 : ENV CATALINA_BASH $MYPATH/apache-tomcat-9.0.55
 ---> Running in 31dc5b38478c
Removing intermediate container 31dc5b38478c
 ---> 8ae919106bf6
Step 13/15 : ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/bin:$CATALINA_HOME/lib
 ---> Running in d3fe1f81fab7
Removing intermediate container d3fe1f81fab7
 ---> dd8b07b2adfd
Step 14/15 : EXPOSE 8080
 ---> Running in 1f1601f2dcc2
Removing intermediate container 1f1601f2dcc2
 ---> 9078648b7a2e
Step 15/15 : CMD $CATALINA_HOME/bin/startup.sh && tail -F $CATALINA_HOME/logs/catalina.out
 ---> Running in 6a3b2aefaf44
Removing intermediate container 6a3b2aefaf44
 ---> 23a538c107a0
Successfully built 23a538c107a0
Successfully tagged tomcat-test:latest
```

> 查看镜像

```shell
[root@sail tomcat]# docker images
REPOSITORY        TAG       IMAGE ID       CREATED          SIZE
tomcat-test       latest    23a538c107a0   25 minutes ago   673MB
```

> 启动镜像

```shell
[root@sail tomcat]# docker run -d -p 8080:8080 --name sail-tomcat -v /home/sail/tomcat/webapps:/usr/local/apache-tomcat-9.0.55/webapps -v /home/sail/tomcat/logs:/usr/local/apache-tomcat-9.0.55/logs tomcat-test 
9d391e13efdc495206429dbdb0392180a7bd3a4750cbc1419c31c80cd69c6b7b
[root@sail tomcat]#
```

启动时将 tomcat 的 **webapps** 和 **logs** 目录都挂载到了本机。

> 查看挂载目录

```shell
[root@sail tomcat]# ls /home/sail/tomcat
logs  webapps
```

这里找到了挂载到本机的两个目录，说明挂载成功了。

> 进入容器

```shell
[root@sail tomcat]# docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                    NAMES
9d391e13efdc   tomcat-test   "/bin/sh -c '$CATALI…"   24 minutes ago   Up 24 minutes   0.0.0.0:8080->8080/tcp   sail-tomcat

[root@sail tomcat]# docker exec -it 9d391e13efdc /bin/bash
[root@9d391e13efdc local]# ls
apache-tomcat-9.0.55  bin  etc    games  include    jdk1.8.0_301  lib  lib64  libexec  readme.txt  sbin  share  src

[root@9d391e13efdc local]# cd apache-tomcat-9.0.55/
[root@9d391e13efdc apache-tomcat-9.0.55]# ls
BUILDING.txt  CONTRIBUTING.md  LICENSE    NOTICE    README.md  RELEASE-NOTES  RUNNING.txt  bin  conf  lib  logs  temp  webapps  work
```

jdk 和 readme.txt 都是具备了的，且 tomcat 目录下的文件也是完整的。

> 查看挂载文件

这里以 logs 为例，我们先进入 tomcat 容器中的 logs 文件夹查看日志内容。

```shell
[root@9d391e13efdc apache-tomcat-9.0.55]# cd logs
[root@9d391e13efdc logs]# ls
catalina.out
[root@9d391e13efdc logs]# cat catalina.out 
/usr/local//apache-tomcat-9.0.55/bin/catalina.sh: line 504: /usr/local//jdk1.8.0_301-amd64/bin/java: No such file or directory
```

然后再退出查看主机上挂载的 logs 文件夹。

```shell
[root@9d391e13efdc logs]# exit
exit
[root@sail tomcat]# cd /home/sail/tomcat/logs
[root@sail logs]# ls
catalina.out
[root@sail logs]# cat catalina.out 
/usr/local//apache-tomcat-9.0.55/bin/catalina.sh: line 504: /usr/local//jdk1.8.0_301-amd64/bin/java: No such file or directory
```

两个地方 logs 下的文件内容一致，说明挂载成功。

# 发布镜像到 Docker Hub

> 注册账号

如果没有 Docker Hub 账号，先注册账号：https://hub.docker.com/

> 登录 Docker Hub 账号

```shell
[root@sail logs]# docker login -u asailing
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
Login Succeeded
```

## 发布镜像

### docker push

> 直接发布镜像

```shell
[root@sail logs]# docker push centos-test
Using default tag: latest
The push refers to repository [docker.io/library/centos-test]
de70c523870b: Preparing 
909db45c4bc4: Preparing 
74ddd0ec08fa: Preparing 
denied: requested access to the resource is denied
```

访问资源被拒绝了。拒绝的原因是我们没有带标签，默认的 latest 标签是不能被识别的。

## 指定镜像标签

### docker tag

我们可以使用 `docker tag` 命令给镜像加一个标签。

> 必须以 `账号名/镜像名:标签` 的格式命令才能提交。

```shell
[root@sail logs]# docker tag d58be7785771 asailing/centos:1.0
[root@sail logs]# docker images
REPOSITORY        TAG       IMAGE ID       CREATED        SIZE
asailing/centos   1.0       d58be7785771   29 hours ago   323MB
centos-test       latest    d58be7785771   29 hours ago   323MB
```

此时会多出一个相同 ID 但是标签和名字不同的镜像。

> 再次发布镜像

```shell
[root@sail logs]# docker push asailing/centos:1.0
The push refers to repository [docker.io/asailing/centos]
de70c523870b: Pushed 
909db45c4bc4: Pushed 
74ddd0ec08fa: Pushed 
1.0: digest: sha256:ecefaae6c5a2cab84693175ea3b18d0d0a7aa0160e33a0bf3eb4ab626b10f0f1 size: 953
```

这样就能发布成功了。且可以发现，**镜像的发布也是分层发布的**。

# 配置国内镜像站

由于对国外网络的限制，发布镜像到 DockerHub 是比较缓慢的。

这里可以使用配置 **Docker 国内镜像站**的方式实现加速。

运行以下命令即可：

```shell
[root@sail ~]# curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
docker version >= 1.12
{
  "registry-mirrors": ["http://f1361db2.m.daocloud.io"]
}
Success.
You need to restart docker to take effect: sudo systemctl restart docker

[root@sail ~]# systemctl restart docker
[root@sail ~]#
```

该脚本可以将 `--registry-mirror` 加入到 Docker 配置文件 `/etc/docker/daemon.json` 中。

适用于 **Ubuntu14.04**、**Debian**、**CentOS6** 、**CentOS7**、**Fedora**、**Arch Linux**、**openSUSE Leap 42.1**，其他版本可能有细微不同。

> 去 Docker Hub 上以 **账号名/镜像名** 搜索我们刚发布的镜像，发现是可以搜索到的。

![img](Docker 13 Dockerfile.assets/kuangstudy5d0bfcf4-7830-42d9-a019-6b3fdbf9d82c.png)

> 查看详情也可以镜像的具体信息。

![img](Docker 13 Dockerfile.assets/kuangstudyc57c1644-bd6a-4e74-8d90-764c3a279b6c.png)

DIGEST 的值正是刚才发布后返回值 `ecefaae6c5a2cab84693175ea3b18d0d0a7aa0160e33a0bf3eb4ab626b10f0f1` 的缩写。

且镜像的大小是小于我们本地镜像的，说明**发布的过程中也会压缩镜像**。

> 拉取我们发布的镜像

```shell
[root@sail logs]# docker pull asailing/centos:1.0
1.0: Pulling from asailing/centos
Digest: sha256:ecefaae6c5a2cab84693175ea3b18d0d0a7aa0160e33a0bf3eb4ab626b10f0f1
Status: Image is up to date for asailing/centos:1.0
docker.io/asailing/centos:1.0
```

无法拉取。原因很简单，因为我们本地存在了同名镜像。

> 我们先删除这个镜像再拉取

```shell
[root@sail logs]# docker rmi -f d58be7785771
Untagged: asailing/centos:1.0
Untagged: asailing/centos@sha256:ecefaae6c5a2cab84693175ea3b18d0d0a7aa0160e33a0bf3eb4ab626b10f0f1
Untagged: centos-test:latest
Untagged: sail/centos:1.0
Deleted: sha256:d58be7785771bd95d8016fa5807a486d6c50e195879eddd88cb602172fc51ffe
Deleted: sha256:ad95558eb65801f5871215837558156c5e33ba351b3b52e0a50aac045abb46c1
Deleted: sha256:5c5def0bbb85d8779d02f115c3d072fe9adb1fd07556ee8c5a130823ecf6811d
Deleted: sha256:b5bd21416741daec348f417dbea1b73001e257f1e63a5d2abddabc8554fca611
Deleted: sha256:a9431f90fd3f23387c456ad5b925dbb9531beece3eab825848db99db29c6a1fa
Deleted: sha256:9f54f48660acb350921aefab74769e51fc7917a1e1e730d3df2edd1513517c42
Deleted: sha256:fb41ece5d944c667762945fdf7275a1d267acd92fe9dc56709fc3adaca6f087f
Deleted: sha256:be89377d4c2ccea11308d8196ba53f03985882db015e01ed8b54fc114f4ba058
Deleted: sha256:9616888f3b103230ed5f378af4afc11b7ce7ed3d96653e5bd918c49152bbdf8c

[root@sail logs]# docker pull asailing/centos:1.0
1.0: Pulling from asailing/centos
a1d0c7532777: Already exists 
0594d57f8468: Already exists 
9c13f720f33e: Already exists 
Digest: sha256:ecefaae6c5a2cab84693175ea3b18d0d0a7aa0160e33a0bf3eb4ab626b10f0f1
Status: Downloaded newer image for asailing/centos:1.0
docker.io/asailing/centos:1.0

[root@sail logs]# docker images
REPOSITORY        TAG       IMAGE ID       CREATED        SIZE
asailing/centos   1.0       d58be7785771   29 hours ago   323MB
```

拉取成功，且大小又恢复到了之前本地的镜像大小，说明**拉取的过程中也会解压镜像**。

> 启动拉取的镜像

```shell
[root@sail logs]# docker run -it asailing/centos:1.0 /bin/bash
[root@168c9e550886 local]# vim test.java
[root@168c9e550886 local]#
```

`vim` 命令也是可以使用的，镜像发布成功。

# 发布镜像到阿里云镜像仓库

> 登录阿里云，点击**我的阿里云**

![img](Docker 13 Dockerfile.assets/kuangstudy42018025-d25d-4838-815f-cd683749715b.png)

![img](Docker 13 Dockerfile.assets/kuangstudy76398d1b-d43b-4d2c-8960-ab9c66e02182.png)

> 创建实例

这里以创建个人版实例为例。

我这里已经创建好了，如果没有创建点击创建即可。

![img](Docker 13 Dockerfile.assets/kuangstudyf40bb3ef-d6e3-4fcf-b2cf-56a68f3a3936.png)

> 进入镜像仓库

创建好个人实例后，点击进入。

![img](Docker 13 Dockerfile.assets/kuangstudy599e3dcd-15a1-4996-bb6d-4c25e60a9344.png)

> 创建命名空间

![img](Docker 13 Dockerfile.assets/kuangstudycb118d7b-e673-46f3-8fc8-963b8a26a5d8.png)

一个账号只能创建 3 个命名空间，需要谨慎创建。

创建好后就是这样。

![img](Docker 13 Dockerfile.assets/kuangstudy7f2b0bf2-190f-4272-b2a3-857d1c65abc2.png)

> 创建镜像仓库

![img](Docker 13 Dockerfile.assets/kuangstudy33cd03cb-bfd7-4ca8-8d1e-59b87b1370dd.png)

> 点击下一步，创建本地仓库

![img](Docker 13 Dockerfile.assets/kuangstudya3427a43-12f3-4d21-a579-6336f884034b.png)

![img](Docker 13 Dockerfile.assets/kuangstudy8a8bbfee-29b9-46a0-9644-f8788e2804d0.png)

至此，我们就创建好了阿里云的镜像仓库，具体的操作步骤上图也写得非常清楚。

> 退出登录的账号

如果之前登录了 Docker Hub 账号或者其他阿里云账号，先退出账号。

```shell
[root@sail logs]# docker logout
Removing login credentials for https://index.docker.io/v1/
```

> 登录阿里云账号

```shell
[root@sail logs]# docker login --username=内有千丘外无锋 registry.cn-hangzhou.aliyuncs.com
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
Login Succeeded
```

> 设置镜像标签

```shell
[root@sail logs]# docker tag d58be7785771 registry.cn-hangzhou.aliyuncs.com/asailing/sail:1.0
[root@sail logs]# docker images
REPOSITORY                                        TAG       IMAGE ID       CREATED        SIZE
asailing/centos                                   1.0       d58be7785771   32 hours ago   323MB
registry.cn-hangzhou.aliyuncs.com/asailing/sail   1.0       d58be7785771   32 hours ago   323MB
```

> 提交镜像

```shell
[root@sail logs]# docker push registry.cn-hangzhou.aliyuncs.com/asailing/sail:1.0
The push refers to repository [registry.cn-hangzhou.aliyuncs.com/asailing/sail]
de70c523870b: Pushed 
909db45c4bc4: Pushed 
74ddd0ec08fa: Pushed 
1.0: digest: sha256:ecefaae6c5a2cab84693175ea3b18d0d0a7aa0160e33a0bf3eb4ab626b10f0f1 size: 953
```

> 查看提交的镜像

![img](Docker 13 Dockerfile.assets/kuangstudyfb28acbd-766d-4818-b6df-2fb889ef1072.png)

提交的镜像可以在这里查看。

> 拉取镜像

先删除本地镜像， 再拉取测试。

```shell
[root@sail logs]# docker rmi -f d58be7785771
Untagged: asailing/centos:1.0
Untagged: asailing/centos@sha256:ecefaae6c5a2cab84693175ea3b18d0d0a7aa0160e33a0bf3eb4ab626b10f0f1
Untagged: registry.cn-hangzhou.aliyuncs.com/asailing/sail/centos:1.0
Untagged: registry.cn-hangzhou.aliyuncs.com/asailing/sail:1.0
Untagged: registry.cn-hangzhou.aliyuncs.com/asailing/sail@sha256:ecefaae6c5a2cab84693175ea3b18d0d0a7aa0160e33a0bf3eb4ab626b10f0f1
Deleted: sha256:d58be7785771bd95d8016fa5807a486d6c50e195879eddd88cb602172fc51ffe

[root@sail logs]# docker pull registry.cn-hangzhou.aliyuncs.com/asailing/sail:1.0
1.0: Pulling from asailing/sail
a1d0c7532777: Already exists 
0594d57f8468: Already exists 
9c13f720f33e: Already exists 
Digest: sha256:ecefaae6c5a2cab84693175ea3b18d0d0a7aa0160e33a0bf3eb4ab626b10f0f1
Status: Downloaded newer image for registry.cn-hangzhou.aliyuncs.com/asailing/sail:1.0
registry.cn-hangzhou.aliyuncs.com/asailing/sail:1.0

[root@sail logs]# docker images
REPOSITORY                                        TAG       IMAGE ID       CREATED        SIZE
registry.cn-hangzhou.aliyuncs.com/asailing/sail   1.0       d58be7785771   32 hours ago   323MB
```

> 启动镜像

```shell
[root@sail logs]# docker run -it registry.cn-hangzhou.aliyuncs.com/asailing/sail:1.0
[root@c612099a94a2 local]# vim test.java
[root@c612099a94a2 local]#
```

`vim` 命令可以使用，说明提交镜像成功。