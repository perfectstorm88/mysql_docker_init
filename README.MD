# 需求说明
- 初始化的数据库脚本，需要在数据库首次使用时执行，之后重启应用和数据库，不再执行初始化脚本
- 数据库的数据要进行持久化，mysql容器是否重启销毁不影响数据

# 用法
参加[docker-mysql](https://hub.docker.com/_/mysql)的"Initializing a fresh instance"部分：

>When a container is started for the first time, a new database with the specified name will be created and initialized with the provided configuration variables. Furthermore, it will execute files with extensions .sh, .sql and .sql.gz that are found in /docker-entrypoint-initdb.d. Files will be executed in alphabetical order. You can easily populate your mysql services by mounting a SQL dump into that directory and provide custom images with contributed data. SQL files will be imported by default to the database specified by the MYSQL_DATABASE variable.

就是把初始化脚本，映射到/docker-entrypoint-initdb.d目录下，首次启动时执行
```yml
version: "3.5"
services:
  db:
    image: mysql
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: abc
    volumes:
        - ./mysql/mydb_log.sql:/docker-entrypoint-initdb.d/mydb_log.sql
        - ./mysql/setup.sql:/docker-entrypoint-initdb.d/setup.sql
        - db_data:/var/lib/mysql
  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080 # http://localhost:8080

volumes:
    db_data:
```

首次启动`docker-compose up`时可以看到下面日志，之后再重启就不再执行了
```
2020-01-30 03:24:32+00:00 [Note] [Entrypoint]: /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/mydb_log.sql
2020-01-30 03:24:32+00:00 [Note] [Entrypoint]: /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/setup.sql
```

# 参考:

- [Create database on docker-compose startup](https://stackoverflow.com/questions/43322033/create-database-on-docker-compose-startup):参照样例
- [docker-mysql](https://hub.docker.com/_/mysql):参照

# 附：辅助脚本，重复测试时使用
```bash
# 删除容器
docker ps -a|grep -v CONTAINER|grep -v Up|awk -F " " '{print $1}'|xargs docker rm
# 情况volume
docker volume prune
```