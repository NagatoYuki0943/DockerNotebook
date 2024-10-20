# 拉取

```sh
docker pull postgres
```

# 环境变量

## `POSTGRES_PASSWORD`: 唯一必须的环境变量

此环境变量是使用 PostgreSQL 映像所必需的。它不能为空或 undefined。此环境变量设置 PostgreSQL 的超级用户密码。默认超级用户由 `POSTGRES_USER` 环境变量定义。

## `POSTGRES_USER`

此可选环境变量与 `POSTGRES_PASSWORD` 结合使用，以设置用户及其密码。此变量将创建具有超级用户权限的指定用户和同名数据库。如果未指定，则将使用 `postgres` 的默认用户。

## `POSTGRES_DB`

此可选环境变量可用于为首次启动映像时创建的默认数据库定义不同的名称。如果未指定，则将使用 `POSTGRES_USER` 的值。

## `POSTGRES_INITDB_ARGS`

这个可选的环境变量可用于将参数发送到 `postgres initdb`。该值是一个以空格分隔的参数字符串，正如 `postgres initdb` 所期望的那样。这对于添加数据页校验和等功能非常有用： `-e POSTGRES_INITDB_ARGS="--data-checksums"` .

## `POSTGRES_INITDB_WALDIR`

此可选环境变量可用于定义 Postgres 事务日志的另一个位置。默认情况下，事务日志存储在主 Postgres 数据文件夹 （`PGDATA）` 的子目录中。有时，可能需要将事务日志存储在不同的目录中，该目录可能由具有不同性能或可靠性特征的存储提供支持。

## `POSTGRES_HOST_AUTH_METHOD`

此可选变量可用于控制`所有`数据库、`所有用户`和`所有`地址的`主机`连接的 `auth-method`。如果未指定，则使用 [`scram-sha-256` 密码身份验证](https://www.postgresql.org/docs/14/auth-password.html)（在 14+ 中;`md5 的 md5` 版本）。在未初始化的数据库上，这将通过以下近似行填充 `pg_hba.conf`：

## `PGDATA`

这个可选变量可用于定义数据库文件的另一个位置 - 如子目录。默认值为 `/var/lib/postgresql/data`。如果您使用的数据卷是文件系统挂载点（如 GCE 持久磁盘），或无法提供给 `postgres` 用户的远程文件夹（如某些 NFS 挂载），或包含文件夹/文件（例如 `lost+found`），则 Postgres `initdb` 需要在挂载点内创建一个子目录来包含数据。

# run

```sh
# -d 背景执行
docker run -d \
    --name postgres1 \
    -p 5432:5432 \
    -e POSTGRES_PASSWORD=postgres \
    -e POSTGRES_DB=mb \
    postgres

# windows d盘的目录,在 wsl 中可以看到这个目录,但是windows中看不到, 解释: https://www.perplexity.ai/search/docker-e-can-shu-she-zhi-duo-g-PHp4jSfpQiWgiVcfQW6Qjw
# /mnt/D/sql/PostgreSQL/17/data1
docker run -d \
    --name postgres1 \
    -p 5432:5432 \
    -e POSTGRES_PASSWORD=postgres \
    -e POSTGRES_DB=mb \
    -v /mnt/D/sql/PostgreSQL/17/data1:/var/lib/postgresql/data \
    postgres
```

