# 拉取

```sh
docker pull redis
```

# run

```sh
# -d 背景执行
docker run -d --name redis1 -p 6379:6379 redis

# 持久存储
docker run -d --name redis1 -p 6379:6379 redis redis-server --save 60 1 --loglevel warning
```
