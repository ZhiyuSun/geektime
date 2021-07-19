# docker基本

docker version

docker info

docker image ls

docker --help

## 镜像和容器

image是一个只读的文件

docker container ls
docker container --help
docker container run nginx
docker stop xxx
docker contanier ps
docker container rm 1a

最好加上完整的container或image

## 命令行小技巧

docker container ps -aq

docker container stop $(docker container ps -qa)

docker container stop id1 id2

docker container rm id1 id2

不能删除正在运行中的容器，要停掉再删除，或者加强制删除参数-f

## attached和detached模式

docker container run -p 80:80 nginx

windows的attached模式，但是不能ctrl+c

detached就是在后台执行

docker run -d -p 80:80 nginx

docker attach id

docker container ps -a

不推荐attach模式，建议-d

## 容器的交互模式

docker container logs id

docker contianer logs -f id

docker container run -it ubuntu sh

docker container run -d -p 80:80 nginx

非常频繁的使用，交互的进入shell里面
docker exec -it id sh
退出时，容器不会停止

docker container run -it busybox sh

## windows的docker engine

## 容器和虚拟机

容器就是进程

## 创建容器时发生了什么

