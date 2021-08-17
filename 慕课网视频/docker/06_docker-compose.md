# docker-compose

docker-compose.yml

docker-compose up/stop

## docker-compose安装

docker-compose --version

## compose文件的结构和版本

https://docs.docker.com/compose/compose-file/

```
version: "3.8"

services: # 容器
  servicename: # 服务名字，这个名字也是内部 bridge网络可以使用的 DNS name
    image: # 镜像的名字
    command: # 可选，如果设置，则会覆盖默认镜像里的 CMD命令
    environment: # 可选，相当于 docker run里的 --env
    volumes: # 可选，相当于docker run里的 -v
    networks: # 可选，相当于 docker run里的 --network
    ports: # 可选，相当于 docker run里的 -p
  servicename2:

volumes: # 可选，相当于 docker volume create

networks: # 可选，相当于 docker network create
```

## docker-compose命令行使用

docker-compose up -d

docker-compose ps

docker network ls

docker-compose stop

docker-compose rm

这里有一部分没记下来

## docker-compose镜像构建和拉取

如果镜像还没构建，需要使用：

build: ./flask  (默认会去找dockerfile文件)
image: flask-demo:latest

build:
    context: ./flask
    dockerfile: Dockerfile.dev

docker-compose build

docker image ls

docker-compose pull

## docker-compose服务更新

docker-compose up
docker-compose stop

docker-compose build

docker-compose up -d  --build 重新创建

docker-compose up -d 

比较常用的：
docker-compose up -d --remove-orphans 删除不用的image
docker-compose restart
docker-compose up -d --build 重新构建image

## docker-compose 网络

这个好难

## docker-compose 水平扩展

## docker-compose 环境变量

environment

${REDIS_PASSWORD}

创建.env文件，把环境变量写进去，放在.gitignore里面

docker-compose config 配置预览

docker-compose --env-file xxx config

docker-compose --env-file xxx up -d

## 服务依赖和健康检查

depends_on

HEALTHCHECK

