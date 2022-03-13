Docker Compose 是 Docker 官方编排项目之一，进行多容器应用的部署和管理，Compose 定位是定义和运行多个 Docker 容器的应用，它就会自动帮我们构建镜像、配置网络等功能

使用 docker-compose 我们主要的任务是编写 docker-compose.yml 文件

```
version: '3' # 定义版本，不指定默认为版本 1，新版本功能更多

services: # 定义不同的应用服务
   db: # 名称，它也是 network 中 DNS 名称
     image: mysql:5.7 # 镜像，如果像自定义镜像可以不指定这个参数，而用 build
     volumes: # 定义数据卷，类似 -v
       - db_data:/var/lib/mysql
       - .:/aaa # 挂载当前目录到容器中的 /aaa 无需使用绝对路径
     restart: always # 类似 --restart
     # 'no' 默认，不自动重启，以为 no 是 yaml 关键字所以加引号
     # always 总是自动重启
     # on-failure 当失败时自动重启，也就是 exit code 不为 0 时
     # unless-stopped 除非手动停止，否者一直重启
     environment: # 定义环境变量，类似 -e
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
   wordpress: # 第二个容器
     labels:
       com.example.description: "This label will appear on all containers for the web service"
     # 为容器添加 Docker 元数据（metadata）信息。例如可以为容器添加辅助说明信息。
     depends_on: # 帮助 compose 理解容器之间的关系
     # db 将会在 wordpress 之前被启动
     # 关闭时 wordpress 将会在 db 之前关闭
     # 我们指定只启动 wordpress，db 也会跟着启动
       - db
     image: wordpress:latest
     ports: # 端口，类似 -p
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress

volumes: # 可选，需要创建的数据卷，类似 docker volume create
  db_data:

networks: # 可选，需要创建的网络，类似 docker network create
```

写好 Dockerfile 后，通过配置 docker-compose.yml 的 build 字段去指定 Dockerfile

```
version: '3'
services:
  ruijieportal-svc:
    # image: aicregistry.azurecr.cn/platform/ruijieportal/ue:v1.0.0.1
    build:
      context: ..
      dockerfile: ./devops/Dockerfile
    ports:
      - "80:80"
```

然后执行下面的语句启动应用

```
docker-compose up -d
```

其他命令

```
# 停止服务
docker-compose down

# 列出所有运行中的容器
docker-compose ps

# 查看日志
docker-compose logs

# 重新构建服务
docker-compose build

# 重启服务
docker-compose restart
```
