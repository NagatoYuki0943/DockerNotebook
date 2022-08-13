

```
docker pull mysql:latest

docker run -d --name mysql1 -p 3360:3307 -e MYSQL_ROOT_PASSWORD=123456 mysql

docker start mysql1
docker stop mysql1

docker exec -it mysql1 /bin/bash
```

