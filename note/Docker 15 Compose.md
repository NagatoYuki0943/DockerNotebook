https://www.kuangstudy.com/bbs/1485184548689350658

> 参考源

https://www.bilibili.com/video/BV1og4y1q7M4?spm_id_from=333.999.0.0

https://www.bilibili.com/video/BV1kv411q7Qc?spm_id_from=333.999.0.0

> 版本

本文章基于 **Docker 20.10.11**

------

# 简介

使用 **Docker** 的时候，定义 **Dockerfile** 文件，然后使用 `docker build`、`docker run` 等命令操作容器。

然而微服务架构的应用系统一般包含若干个微服务，每个微服务一般都会部署多个实例，如果每个微服务都要手动启停，这样效率很低，也不方便管理。

> 使用 **Docker Compose** 可以轻松、高效的管理容器，它是一个用于定义和运行多容器 Docker 的应用程序工具。

## yaml 官方示例

https://docs.docker.com/compose/compose-file/compose-file-v3/#compose-file-structure-and-examples

```shell
version: "3.9"
services:
  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        max_replicas_per_node: 1
        constraints:
          - "node.role==manager"
  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - "5000:80"
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure
  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - "5001:80"
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints:
          - "node.role==manager"
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints:
          - "node.role==manager"
networks:
  frontend:
  backend:
volumes:
  db-data:
```

**depends_on**：依赖关系，如 web 依赖 redis 和 db，通过 depends_on 表明关系。

```shell
version: "3.9"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

# 安装

Docker Compose 是 Docker 的一个开源项目，目前托管到了 GitHub，需要前往 GitHub 下载。

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/2.2.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```

> 由于存放在 GitHub，国内网络限制导致不太稳定，不推荐使用。

推荐使用 [道客](https://www.daocloud.io/) 提供的 [Docker 极速下载](http://get.daocloud.io/#install-compose) 进行安装。

```shell
curl -L https://get.daocloud.io/docker/compose/releases/download/v2.2.3/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```

> 安装

```shell
[root@sail ~]# curl -L https://get.daocloud.io/docker/compose/releases/download/v2.2.3/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   423  100   423    0     0    394      0  0:00:01  0:00:01 --:--:--   394
100 23.5M  100 23.5M    0     0  8670k      0  0:00:02  0:00:02 --:--:-- 20.3M

[root@sail ~]# cd /usr/local/bin/
[root@sail bin]# ls
docker-compose
```

这样即表示安装成功。

> 授权

```shell
[root@sail ~]# chmod +x /usr/local/bin/docker-compose
[root@sail ~]#
```

## 查看版本

### docker-compose version

```shell
[root@sail bin]# docker-compose version
Docker Compose version v2.2.3
```

显示了版本即代表 Docker Compose 安装完成。

# 卸载

```shell
rm /usr/local/bin/docker-compose
```

由于 **Linux 一切皆文件**，删除此文件夹即可完成 Docker Compose 的卸载。

# 使用

## 构建

> 创建项目目录

```shell
[root@sail sail]# mkdir docker-compose
[root@sail sail]# cd docker-compose
[root@sail docker-compose]#
```

> 创建 app.py

```shell
[root@sail docker-compose]# vim app.py
```

```python
import time
import redis
from flask import Flask
app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)
def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)
@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

redis 是应用容器中 redis 容器的主机名，在同一网络下可以通过服务名访问，端口默认 6379。

> 创建 requirements.txt

```shell
[root@sail docker-compose]# vim requirements.txt
[root@sail docker-compose]# cat requirements.txt 
flask
redis
```

> 创建 Dockerfile

```shell
[root@sail docker-compose]# vim Dockerfile

FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

> 创建 docker-compose.yml

```shell
[root@sail docker-compose]# vim docker-compose.yml

version: "3.3"
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

这个文件定义了两个服务：web 和 redis。

## 启动

### docker-compose up

> 运行应用

在项目目录中，运行 `docker-compose up` 来启动应用程序。

> 第一次启动需要安装很多环境，比较缓慢。

```shell
[root@sail docker-compose]# docker-compose up
[+] Running 7/7
 ⠿ redis Pulled                                                                                                                                                                                              14.1s
   ⠿ 59bf1c3509f3 Pull complete                                                                                                                                                                               3.0s
   ⠿ 719adce26c52 Pull complete                                                                                                                                                                               3.1s
   ⠿ b8f35e378c31 Pull complete                                                                                                                                                                               3.3s
   ⠿ d034517f789c Pull complete                                                                                                                                                                               9.0s
   ⠿ 3772d4d76753 Pull complete                                                                                                                                                                               9.0s
   ⠿ 211a7f52febb Pull complete                                                                                                                                                                               9.1s
Sending build context to Docker daemon     725B
Step 1/10 : FROM python:3.7-alpine
3.7-alpine: Pulling from library/python
59bf1c3509f3: Already exists 
8786870f2876: Pull complete 
45d4696938d0: Pull complete 
ef84af58b2c5: Pull complete 
c3c9b71b9a69: Pull complete 
Digest: sha256:d64e0124674d64e78cc9d7378a1130499ced66a7a00db0521d0120a2e88ac9e4
Status: Downloaded newer image for python:3.7-alpine
 ---> a1034fd13493
Step 2/10 : WORKDIR /code
 ---> Running in e23e4b173abf
Removing intermediate container e23e4b173abf
 ---> 41eb64157cfc
Step 3/10 : ENV FLASK_APP=app.py
 ---> Running in cdefb769398d
Removing intermediate container cdefb769398d
 ---> ab741ac5cb17
Step 4/10 : ENV FLASK_RUN_HOST=0.0.0.0
 ---> Running in 4976c1da428c
Removing intermediate container 4976c1da428c
 ---> 5a5c24d67db6
Step 5/10 : RUN apk add --no-cache gcc musl-dev linux-headers
 ---> Running in 53043bd38e33
fetch https://dl-cdn.alpinelinux.org/alpine/v3.15/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.15/community/x86_64/APKINDEX.tar.gz
(1/13) Installing libgcc (10.3.1_git20211027-r0)
(2/13) Installing libstdc++ (10.3.1_git20211027-r0)
(3/13) Installing binutils (2.37-r3)
(4/13) Installing libgomp (10.3.1_git20211027-r0)
(5/13) Installing libatomic (10.3.1_git20211027-r0)
(6/13) Installing libgphobos (10.3.1_git20211027-r0)
(7/13) Installing gmp (6.2.1-r0)
(8/13) Installing isl22 (0.22-r0)
(9/13) Installing mpfr4 (4.1.0-r0)
(10/13) Installing mpc1 (1.2.1-r0)
(11/13) Installing gcc (10.3.1_git20211027-r0)
(12/13) Installing linux-headers (5.10.41-r0)
(13/13) Installing musl-dev (1.2.2-r7)
Executing busybox-1.34.1-r3.trigger
OK: 139 MiB in 48 packages
Removing intermediate container 53043bd38e33
 ---> 73e9550df596
Step 6/10 : COPY requirements.txt requirements.txt
 ---> c5a73d6f1fe1
Step 7/10 : RUN pip install -r requirements.txt
 ---> Running in 826790d0bfbb
Collecting flask
  Downloading Flask-2.0.2-py3-none-any.whl (95 kB)
Collecting redis
  Downloading redis-4.1.0-py3-none-any.whl (171 kB)
Collecting itsdangerous>=2.0
  Downloading itsdangerous-2.0.1-py3-none-any.whl (18 kB)
Collecting click>=7.1.2
  Downloading click-8.0.3-py3-none-any.whl (97 kB)
Collecting Werkzeug>=2.0
  Downloading Werkzeug-2.0.2-py3-none-any.whl (288 kB)
Collecting Jinja2>=3.0
  Downloading Jinja2-3.0.3-py3-none-any.whl (133 kB)
Collecting packaging>=21.3
  Downloading packaging-21.3-py3-none-any.whl (40 kB)
Collecting deprecated>=1.2.3
  Downloading Deprecated-1.2.13-py2.py3-none-any.whl (9.6 kB)
Collecting importlib-metadata>=1.0
  Downloading importlib_metadata-4.10.0-py3-none-any.whl (17 kB)
Collecting wrapt<2,>=1.10
  Downloading wrapt-1.13.3-cp37-cp37m-musllinux_1_1_x86_64.whl (78 kB)
Collecting zipp>=0.5
  Downloading zipp-3.7.0-py3-none-any.whl (5.3 kB)
Collecting typing-extensions>=3.6.4
  Downloading typing_extensions-4.0.1-py3-none-any.whl (22 kB)
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.0.1-cp37-cp37m-musllinux_1_1_x86_64.whl (30 kB)
Collecting pyparsing!=3.0.5,>=2.0.2
  Downloading pyparsing-3.0.6-py3-none-any.whl (97 kB)
Installing collected packages: zipp, typing-extensions, wrapt, pyparsing, MarkupSafe, importlib-metadata, Werkzeug, packaging, Jinja2, itsdangerous, deprecated, click, redis, flask
Successfully installed Jinja2-3.0.3 MarkupSafe-2.0.1 Werkzeug-2.0.2 click-8.0.3 deprecated-1.2.13 flask-2.0.2 importlib-metadata-4.10.0 itsdangerous-2.0.1 packaging-21.3 pyparsing-3.0.6 redis-4.1.0 typing-extensions-4.0.1 wrapt-1.13.3 zipp-3.7.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
WARNING: You are using pip version 21.2.4; however, version 21.3.1 is available.
You should consider upgrading via the '/usr/local/bin/python -m pip install --upgrade pip' command.
Removing intermediate container 826790d0bfbb
 ---> c1483947fad2
Step 8/10 : EXPOSE 5000
 ---> Running in cd59e0408b47
Removing intermediate container cd59e0408b47
 ---> 05c632dea80d
Step 9/10 : COPY . .
 ---> 8c89b910e366
Step 10/10 : CMD ["flask", "run"]
 ---> Running in 1d9d071fee96
Removing intermediate container 1d9d071fee96
 ---> de4639486b50
Successfully built de4639486b50
Successfully tagged docker-compose_web:latest
Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them
[+] Running 3/3
 ⠿ Network docker-compose_default    Created                                                                                                                                                                  0.3s
 ⠿ Container docker-compose-redis-1  Created                                                                                                                                                                  0.0s
 ⠿ Container docker-compose-web-1    Created                                                                                                                                                                  0.0s
Attaching to docker-compose-redis-1, docker-compose-web-1
docker-compose-redis-1  | 1:C 07 Jan 2022 08:36:26.687 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
docker-compose-redis-1  | 1:C 07 Jan 2022 08:36:26.687 # Redis version=6.2.6, bits=64, commit=00000000, modified=0, pid=1, just started
docker-compose-redis-1  | 1:C 07 Jan 2022 08:36:26.687 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
docker-compose-redis-1  | 1:M 07 Jan 2022 08:36:26.687 * monotonic clock: POSIX clock_gettime
docker-compose-redis-1  | 1:M 07 Jan 2022 08:36:26.688 * Running mode=standalone, port=6379.
docker-compose-redis-1  | 1:M 07 Jan 2022 08:36:26.688 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
docker-compose-redis-1  | 1:M 07 Jan 2022 08:36:26.688 # Server initialized
docker-compose-redis-1  | 1:M 07 Jan 2022 08:36:26.688 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
docker-compose-redis-1  | 1:M 07 Jan 2022 08:36:26.688 * Ready to accept connections
docker-compose-web-1    |  * Serving Flask app 'app.py' (lazy loading)
docker-compose-web-1    |  * Environment: production
docker-compose-web-1    |    WARNING: This is a development server. Do not use it in a production deployment.
docker-compose-web-1    |    Use a production WSGI server instead.
docker-compose-web-1    |  * Debug mode: off
docker-compose-web-1    |  * Running on all addresses.
docker-compose-web-1    |    WARNING: This is a development server. Do not use it in a production deployment.
docker-compose-web-1    |  * Running on http://172.18.0.2:5000/ (Press CTRL+C to quit)
```

> 查看镜像

```shell
[root@sail docker-compose]# docker images
REPOSITORY           TAG          IMAGE ID       CREATED          SIZE
docker-compose_web   latest       6df8b78250a1   44 seconds ago   185MB
redis                alpine       3900abf41552   5 weeks ago      32.4MB
python               3.7-alpine   a1034fd13493   5 weeks ago      41.8MB
```

启动 Docker Compose 时，会自动拉取需要的镜像。

> 查看容器

```shell
[root@sail docker-compose]# docker ps
CONTAINER ID   IMAGE                COMMAND                  CREATED         STATUS         PORTS                    NAMES
78a6f8b03a49   docker-compose_web   "flask run"              8 minutes ago   Up 8 minutes   0.0.0.0:5000->5000/tcp   docker-compose-web-1
b4da6da4364f   redis:alpine         "docker-entrypoint.s…"   8 minutes ago   Up 8 minutes   6379/tcp                 docker-compose-redis-1
```

可以看到容器命名都带有数字，是因为需要集群管理，数字代表副本序号。

> 查看网络

```shell
[root@sail docker-compose]# docker network ls
NETWORK ID     NAME                     DRIVER    SCOPE
b89f719e94e0   bridge                   bridge    local
619a5845a105   docker-compose_default   bridge    local
28d77e958643   host                     host      local
801fbbe1b38c   mynet                    bridge    local
c3ff850e96f0   none                     null      local
```

项目中的内容都在同个网络下。

> 访问测试

```shell
[root@sail docker-compose]# curl localhost:5000
Hello World! I have been seen 1 times.
[root@sail docker-compose]# curl localhost:5000
Hello World! I have been seen 2 times.
[root@sail docker-compose]# curl localhost:5000
Hello World! I have been seen 3 times.
```

Docker Compose 启动完成。

## 停止

### docker-compose stop

> 示例

```shell
[root@sail docker-compose]# docker-compose stop
[+] Running 2/2
 ⠿ Container docker-compose-redis-1  Stopped                                               0.2s
 ⠿ Container docker-compose-web-1    Stopped
```

## 停止并删除容器和网络

### docker-compose down

> 示例

```shell
[root@sail docker-compose]# docker-compose down
[+] Running 3/3
 ⠿ Container docker-compose-web-1    Removed                                                                                                                              10.2s
 ⠿ Container docker-compose-redis-1  Removed                                                                                                                               0.2s
 ⠿ Network docker-compose_default    Removed
 
[root@sail docker-compose]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

[root@sail docker-compose]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
b89f719e94e0   bridge    bridge    local
28d77e958643   host      host      local
801fbbe1b38c   mynet     bridge    local
c3ff850e96f0   none      null      local
```

可以看出，**容器**和**网络**都被删除了。

# 更多配置

https://docs.docker.com/compose/compose-file/compose-file-v3/#compose-file-structure-and-examples