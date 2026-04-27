---
title: 在1Panel中使用Jenkins--创建Springboot任务
tags:
  - Jenkins
  - 1Panel
  - 运维
Categories:
  - 1Panel
abbrlink: 485afe2b
date: 2025-09-17 16:30:35
---

通过 jenkins，我们可以实现让 jenkins 使用 git 拉取 springboot 项目，编译后使其在服务器中运行。



为此我们需要配置 `Maven`，`jdk` 环境，以及编写使其运行的`脚本`。在这里我使用的是 `docker` 容器运行的方式。

## 环境准备

我们需要 jenkins 容器中拥有 `Maven`，`jdk`。如果没有的请在本机中下载 Maven 和 jdk，然后编辑 jenkins容器，添加新的挂载，将宿主机的 Maven 和 jdk 挂载的`/opt`路径下，如：

![image-20250917164048526](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250917164048526.png)

### 1.创建任务

输入项目名

选择 Maven 项目

点击确定

![image-20250917165224922](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250917165224922.png)

### 2.配置任务

输入项目描述

#### 源码管理

输入项目 git 地址，最好是 `ssh` 地址

![image-20250917165350399](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250917165350399.png)

如果是私有仓库需要添加凭证-Credentials

指定对应的分支

![image-20250917165633396](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250917165633396.png)

#### build

添加Maven options

``` bash
clean install -Dmaven.test.skip=true
```

![image-20250917171257028](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250917171257028.png)

#### Post Steps

##### 添加`执行 shell`

![image-20250917172235469](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250917172235469.png)

用于做发送前预处理

如以下命令，将 `Dockerfile` 拷贝到`Maven`打包生成的`target` 目录中，更改文件名再打成`tar` 包，方便后续文件传输及处理

``` bash
cp Dockerfile-test target/
cd target
mv Dockerfile-test Dockerfile
tar -czvvf app.tar.gz app.jar Dockerfile
```

注意，因为后续要用 `Dockerfile`创建镜像，所以项目根目录中需要包含`Dockerfile`

样例如下，根据使用更改:

```bash
# jre基础环境
FROM openjdk:11

# 维护者信息
MAINTAINER chenximeng

# 设置环境变量-运行时也可传参进来耍哈
ENV JAVA_OPTS ""

RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone

# 添加jar包到容器中 -- tips: xx.jar 和 Dockerfile 在同一级
ADD *.jar /home/app.jar

#拷贝宿主机上的/opt/apiclient_key.pem到容器的/opt/目录下
COPY apiclient_key.pem /opt/apiclient_key.pem

# 对外暴漏的端口号
# [注：EXPOSE指令只是声明容器运行时提供的服务端口，在运行时只会开启程序自身的端口！！]
EXPOSE 9887

# 以exec格式的CMD指令 -- 可实现优雅停止容器服务
# "sh", "-c" : 可通过exec模式执行shell  =》 获得环境变量
CMD ["sh", "-c", "echo \"****** 运行命令：java -jar ${JAVA_OPTS} /home/app.jar --spring.profiles.active=prod\" && java -jar ${JAVA_OPTS} /home/app.jar --spring.profiles.active=prod"]

```



##### 添加`send files or execute commands over SSH`

![image-20250917171519436](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250917171519436.png)

选择需要发送到的服务器

![image-20250917171551094](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250917171551094.png)

将指定文件发送到服务器，再执行运行脚本

![image-20250917172610248](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250917172610248.png)

参考脚本如下：

``` bash
#!/bin/bash

# 设置变量
HOME_DIR="/apps/delivery-helper-test"
APP_DIR="$HOME_DIR/api"
LOG_DIR="$HOME_DIR/logs"

CONTAINER_NAME="delivery-api-test"
DOCKER_IMAGE="delivery-api-test"

# 记录日志文件
LOG_FILE="$LOG_DIR/deployment_api_log.txt"
echo "Deployment Script Execution - $(date)" > "$LOG_FILE"


# 停止并删除 Docker 容器
echo "[INFO] Checking Docker container: $CONTAINER_NAME" | tee -a "$LOG_FILE"

# 检查容器是否存在（无论是否运行）
if docker ps -a -q --filter "name=$CONTAINER_NAME" | grep -q .; then
    # 如果容器在运行，则先停止
    if docker ps -q --filter "name=$CONTAINER_NAME" | grep -q .; then
        echo "[INFO] Stopping running container: $CONTAINER_NAME" | tee -a "$LOG_FILE"
        if ! docker stop "$CONTAINER_NAME"; then
            echo "[ERROR] Failed to stop running container: $CONTAINER_NAME" | tee -a "$LOG_FILE"
            exit 1
        fi
    fi

    # 删除容器
    echo "[INFO] Removing container: $CONTAINER_NAME" | tee -a "$LOG_FILE"
    if docker rm "$CONTAINER_NAME"; then
        echo "[SUCCESS] Container $CONTAINER_NAME removed successfully." | tee -a "$LOG_FILE"
    else
        echo "[ERROR] Failed to remove container: $CONTAINER_NAME" | tee -a "$LOG_FILE"
        exit 1
    fi
else
    # 如果容器不存在
    echo "[INFO] Container $CONTAINER_NAME does not exist. Skipping." | tee -a "$LOG_FILE"
fi

#删除原内容
rm -rf "$APP_DIR"/*
echo "[INFO] Remove old file success" | tee -a

#移动 app.tar.gz 到 APP_DIR
echo "[INFO] Moving $HOME_DIR/app.tar.gz to $APP_DIR" | tee -a "$LOG_FILE"
if mv "$HOME_DIR"/app.tar.gz "$APP_DIR"; then
    echo "[SUCCESS] Moved app.tar.gz to $APP_DIR." | tee -a "$LOG_FILE"
else
    echo "[ERROR] Failed to move app.tar.gz to $APP_DIR." | tee -a "$LOG_FILE"
    exit 1
fi

# 切换到 APP_DIR 并解压 app.tar.gz
echo "[INFO] Changing directory to $APP_DIR and extracting app.tar.gz" | tee -a "$LOG_FILE"
cd "$APP_DIR" || { echo "[ERROR] Failed to change directory to $APP_DIR" | tee -a "$LOG_FILE"; exit 1; }

if tar -xzf app.tar.gz; then
    echo "[SUCCESS] Extracted app.tar.gz in $APP_DIR." | tee -a "$LOG_FILE"
    # 解压后删除 app.tar.gz 文件
    rm -f app.tar.gz
    echo "[INFO] Removed app.tar.gz after extraction." | tee -a "$LOG_FILE"
else
    echo "[ERROR] Failed to extract app.tar.gz in $APP_DIR." | tee -a "$LOG_FILE"
    exit 1
fi

# 从 HOME_DIR 复制 apiclient_key.pem 到 APP_DIR
echo "[INFO] Copying apiclient_key.pem from $HOME_DIR to $APP_DIR" | tee -a "$LOG_FILE"
if cp "$HOME_DIR/apiclient_key.pem" "$APP_DIR"; then
    echo "[SUCCESS] Moved apiclient_key.pem from $HOME_DIR to $APP_DIR." | tee -a "$LOG_FILE"
else
    echo "[ERROR] Failed to move apiclient_key.pem from $HOME_DIR to $APP_DIR." | tee -a "$LOG_FILE"
    exit 1
fi

# 构建新的 Docker 镜像
echo "[INFO] Building Docker image: $DOCKER_IMAGE" | tee -a "$LOG_FILE"

if docker build -t "$DOCKER_IMAGE" .; then
    echo "[SUCCESS] Built Docker image: $DOCKER_IMAGE" | tee -a "$LOG_FILE"
else
    echo "[ERROR] Failed to build Docker image: $DOCKER_IMAGE" | tee -a "$LOG_FILE"
    exit 1
fi

# 运行新的 Docker 容器
echo "[INFO] Running Docker container: $CONTAINER_NAME" | tee -a "$LOG_FILE"
if docker run -d -p 9860:9860 -p 2100:2100 -v "$LOG_DIR:/logs" --restart always --name "$CONTAINER_NAME" "$DOCKER_IMAGE"; then
    echo "[SUCCESS] Docker container $CONTAINER_NAME is running." | tee -a "$LOG_FILE"
else
    echo "[ERROR] Failed to start Docker container: $CONTAINER_NAME" | tee -a "$LOG_FILE"
    exit 1
fi

# 脚本执行完毕
echo "[INFO] Deployment completed successfully." | tee -a "$LOG_FILE"

```

