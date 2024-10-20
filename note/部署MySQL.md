

```
docker pull mysql:latest

#                              主机端口:容器端口
docker run -d --name mysql1 -p 3307:3306 -e MYSQL_ROOT_PASSWORD=root mysql

docker start mysql1
docker stop mysql1

docker exec -it mysql1 /bin/bash
```

