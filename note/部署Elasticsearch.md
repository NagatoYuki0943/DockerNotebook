https://www.kuangstudy.com/bbs/1481803742688288770

> 参考源

https://www.bilibili.com/video/BV1og4y1q7M4?spm_id_from=333.999.0.0

https://www.bilibili.com/video/BV1kv411q7Qc?spm_id_from=333.999.0.0

> 版本

本文章基于 **Docker 20.10.11**

------

部署 **Elasticsearch** 可以参考 Docker Hub 官方文档：https://hub.docker.com/_/elasticsearch

# 启动

Elasticsearch 十分耗内存，建议启动前先尽量腾出内存空间。

Elasticsearch 需要暴露的端口很多，启动时需要比较复杂的配置。

Elasticsearch 的数据一般需要放置到安全目录，这里又涉及到**数据卷技术**了。

详情见：[Docker 12 数据卷](https://www.kuangstudy.com/bbs/1484782140666593282)

> 启动 Elasticsearch 

```shell
docker run 
-d 
--name elasticsearch01 
-p 9200:9200 						# 配置多个端口
-p 9300:9300 						# 配置多个端口
-e "discovery.type=single-node" 	# -e 配置环境 
elasticsearch:7.6.2
```

```shell
[root@sail sail]# docker run -d --name elasticsearch01 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.6.2
Unable to find image 'elasticsearch:7.6.2' locally
7.6.2: Pulling from library/elasticsearch
ab5ef0e58194: Pull complete 
c4d1ca5c8a25: Pull complete 
941a3cc8e7b8: Pull complete 
43ec483d9618: Pull complete 
c486fd200684: Pull complete 
1b960df074b2: Pull complete 
1719d48d6823: Pull complete 
Digest: sha256:1b09dbd93085a1e7bca34830e77d2981521a7210e11f11eda997add1c12711fa
Status: Downloaded newer image for elasticsearch:7.6.2
6eccd8a777d91764963c2464b30db78b189f5c867633255e996bae088ace6029

[root@sail sail]# docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                                            NAMES
6eccd8a777d9   elasticsearch:7.6.2   "/usr/local/bin/dock…"   8 seconds ago   Up 7 seconds   0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   elasticsearch01
```

> 查看 Docker 内存占用

```shell
[root@sail sail]# docker stats
CONTAINER ID   NAME              CPU %     MEM USAGE / LIMIT     MEM %     NET I/O   BLOCK I/O       PIDS
6eccd8a777d9   elasticsearch01   0.00%     1.238GiB / 1.694GiB   73.06%    0B / 0B   292MB / 389kB   43
```

由此可以看出，Elasticsearch 是多么的耗内存。

> 启动 Elasticsearch 时建议限制其最大内存占用。

```shell
[root@sail sail]# docker run -d --name elasticsearch02 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:7.6.2
e6d7fe80cf2961817bd1184b98fd320dce4d9071b80ffb78aa413f484fa498ae

[root@sail sail]# docker stats
CONTAINER ID   NAME              CPU %     MEM USAGE / LIMIT     MEM %     NET I/O   BLOCK I/O       PIDS
e6d7fe80cf29   elasticsearch02   0.78%     379.8MiB / 1.694GiB   21.89%    0B / 0B   106MB / 733kB   44
```

这样 Elasticsearch 的内存占用就会小很多了。

# 测试验证

> Linux 中可以使用 `curl` 命令模拟网页访问。

```shell
[root@sail sail]# curl localhost:9200
{
  "name" : "e6d7fe80cf29",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "MIfPING6SFGVR88knFCf6w",
  "version" : {
    "number" : "7.6.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
    "build_date" : "2020-03-26T06:34:37.794943Z",
    "build_snapshot" : false,
    "lucene_version" : "8.4.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

![img](部署Elasticsearch.assets/kuangstudy89a8801b-2e63-4d25-bc87-abaf822663e9.png)

> 网页此内容代表 Elasticsearch 部署完成。