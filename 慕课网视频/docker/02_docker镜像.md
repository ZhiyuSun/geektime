# docker镜像

## 镜像的获取

- pull from registry (online) 从registry拉取
    - public（公有）
    - private（私有）
- build from Dockerfile (online) 从Dockerfile构建
- load from file (offline) 文件导入 （离线）

## 获取镜像和删除

docker image pull nginx

docker image ls

docker image pull nginx:1.20.0

docker image inspect id

docker image rm id

如果镜像有正在使用的容器，是没法删除的，即使停掉也删除不掉

## 镜像的导入导出

docker image save nginx:1.20.0 -o nginx.image

docker image load -i .\nginx.image


## dockerfile

Dockerfile是用于构建docker镜像的文件

Dockerfile里包含了构建镜像所需的“指令”

Dockerfile有其特定的语法规则

## 镜像的构建和分享

docker container commit id sunzhiyu/nginx
把容器转成镜像

## 聊聊scratch镜像

docker container run -it hello


