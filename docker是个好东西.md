# 删除所有镜像

docker rm $(docker ps -a)

# addr查看ip端口

docker exec -it ID/NAME

# 查看镜像版本

cat /etc/issue
cat /etc/os-release

# 端口映射

-p 9999:8060

# 停止

docker stop ID

# 删除

docker rm ID

# 开始

docker start ID

# 本机端口：docker端口 /bin/bash

docker run -it -p

# 进入容器

docker attach ID

# 重启ssh

/etc/init.d/ssh restart

systemctl stop docker.socket
systemctl stop docker
systemctl start docker
systemctl enable docker
systemctl restart docker

# dockerfile

## FROM
基础镜像
* 必须jdk，nginx等等

## RUN
命令执行
* 后面一般跟shell

## LABEL
添加元数据，键值对的形式


## CMD（可以被覆盖）
指定容器创建时的默认命令

## ENTRYPOINT（不可覆盖）
指定容器创建时的主要命令

## EXPOSE（暴露端口）
容器运行时监听特定的端口

## ENV
环境变量

## ADD
将文件、目录或远程URL复制到镜像中。

## COPY
将文件或目录复制到镜像中。

## WORKDIR
设置后续指令的工作目录。

## USER	
指定后续指令的用户上下文。

* 比如root

## ARG
定义在构建过程中传递给构建器的变量，可使用 "docker build" 命令设置。

## ONBUILD
当该镜像被用作另一个构建过程的基础时，添加触发器。

## STOPSIGNAL
设置发送给容器以退出的系统调用信号。

## HEALTHCHECK
定义周期性检查容器健康状态的命令。

## SHELL
覆盖Docker中默认的shell，用于RUN、CMD和ENTRYPOINT指令。

## docker build
构建镜像
例如 docker build -t nginx:v3 .
这个点是上下文路径
不写默认是当前文件夹
docker会把上下文目录下所有的文件打包给docker引擎使用

## 例子

```dockerfile
# 使用官方 OpenJDK 11 镜像作为基础镜像
FROM openjdk:11-jre-slim

# 设置工作目录
WORKDIR /app

# 将构建好的 Spring Boot 可执行 JAR 文件复制到镜像中
COPY target/my-spring-boot-app.jar /app/my-spring-boot-app.jar

# 暴露 Spring Boot 应用程序的默认端口
EXPOSE 8080

# 容器启动时执行的命令
CMD ["java", "-jar", "my-spring-boot-app.jar"]

```
docker build -t my-spring-boot-app .
docker run -p 8080:8080 my-spring-boot-app










