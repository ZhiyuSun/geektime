# docker存储

## Data Volume

VOLUME 指定文件夹，持久化存储到磁盘上

docker volume ls

docker volume inspect xxxx

docker container run -d -v cron-data:/app my-cron

上面只能在linux下执行

## 