# dockerfile

## 基础镜像的选择

- 官方镜像优于非官方的镜像，如果没有官方镜像，则尽量选择Dockerfile开源的
- 固定版本tag而不是每次都使用latest
- 尽量选择体积小的镜像

## 通过RUN执行指令

## 文件复制和目录操作

COPY 和 ADD 都可以把local的一个文件复制到镜像里，如果目标目录不存在，则会自动创建

ADD 比 COPY高级一点的地方就是，如果复制的是一个gzip等压缩文件时，ADD会帮助我们自动去解压缩文件。

因此在 COPY 和 ADD 指令中选择的时候，可以遵循这样的原则，所有的文件复制均使用 COPY 指令，仅在需要自动解压缩的场合使用 ADD。

WORKDIR 类似于CD

## 构建参数和环境变量 (ARG vs ENV)

ARG 和 ENV 是经常容易被混淆的两个Dockerfile的语法，都可以用来设置一个“变量”。 但实际上两者有很多的不同。

docker image build -f .\Dockerfile-env -t ipinfo-env .

docker container run -it ipinfo-env

ARG只存在于镜像构建的时候
ENV会作为环境变量永久保存

## 容器启动命令 CMD

CMD可以用来设置容器启动时默认会执行的命令。

- 容器启动时默认执行的命令
- 如果docker container run启动容器时指定了其它命令，则CMD命令会被忽略
- 如果定义了多个CMD，只有最后一个会被执行。

## 容器启动命令 ENTRYPOINT

ENTRYPOINT 也可以设置容器启动时要执行的命令，但是和CMD是有区别的。
- CMD 设置的命令，可以在docker container run 时传入其它命令，覆盖掉 CMD 的命令，但是 ENTRYPOINT 所设置的命令是一定会被执行的。
- ENTRYPOINT 和 CMD 可以联合使用，ENTRYPOINT 设置执行的命令，CMD传递参数

## Shell 格式和 Exec 格式


