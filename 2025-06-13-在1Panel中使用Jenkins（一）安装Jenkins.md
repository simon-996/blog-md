---
title: 在1Panel中使用Jenkins（一）安装Jenkins
tags:
  - Jenkins
  - 1Panel
  - 运维
Categories:
  - 1Panel
abbrlink: ad1ea3d
date: 2025-06-13 14:25:02
---

#### 1Panel是什么

> 1Panel 是一个现代化、开源的 Linux 服务器运维管理面板。

1Panel基于`Docker`，所有安装的软件都是Docker`容器`，对服务器入侵性较小

### Jenkins是什么

**Jenkins** 是一个开源的自动化服务器软件，用于实现持续集成和持续交付（CI/CD）流程。它提供了强大的工具和插件来帮助开发团队自动构建、测试和部署软件。

简单的来说就是可以自动化部署软件。

例如，平时我们的Java项目，部署需要以下步骤

1. 在本地将项目打成jar包
2. 将jar包发送到服务器
3. 删除旧的jar包
4. 运行新的jar包

这通常需要耗费大量的时间。而在使用Jenkins创建好job后，我们部署java项目只需要以下步骤：

1. 将代码推送到git仓库
2. 在Jenkins中点击构建

如果配置了Webhook的话只需要一步

1. 将代码推送到git仓库

所以使用Jenkins后，我们可以更简单、快捷的部署项目，将更多的精力放在项目开发上。

![image-20250917165005384](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250917165005384.png)

#### 前置条件

- 服务器中安装 [1Panel]([在线安装 - 1Panel 文档](https://1panel.cn/docs/v2/installation/online_installation/))
- 本地防火墙开放以下使用到的所有端口
- 如果是云服务器，需要在安全组中开放以下使用到的所有端口

### 环境准备（按需安装）

#### 安装Maven

部署java项目尤其是SpringBoot项目，需要在Jenkins使用Maven进行打包

这里我们使用在本机安装Maven，然后挂载到Jenkins进行使用

##### 下载Maven

进入[Maven官网]([Download Apache Maven – Maven](https://maven.apache.org/download.cgi))下载一个`.tar.gz`包到`/opt`目录后解压

参考以下命令:

```bash
cd /opt

sudo wget https://dlcdn.apache.org/maven/maven-3/3.9.10/binaries/apache-maven-3.9.10-bin.tar.gz

sudo tar -zxvf apache-maven-3.9.10-bin.tar.gz
```

##### 配置镜像

进入`conf`目录，修改settings.xml

```bash
cd /opt/apache-maven-3.9.10/conf

sudo vim settings.xml
```

在`mirrors`中添加以下内容

```bash
<mirror>
	<id>alimaven</id>
	<mirrorOf>central</mirrorOf>
	<name>阿里云公共仓库</name>
	<url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

#### 安装nodejs

##### 下载nodejs

为了方便版本管理，我们选择`nvm`安装

参考以下命令:

```bash
cd /opt

# Download and install nvm:
sudo curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# in lieu of restarting the shell
\. "$HOME/.nvm/nvm.sh"

# Download and install Node.js:
nvm install 22

# Verify the Node.js version:
node -v # Should print "v22.16.0".
nvm current # Should print "v22.16.0".

# Download and install pnpm:
corepack enable pnpm

# Verify pnpm version:
pnpm -v

```

确保以下命令运行正常 

```bash
npm -v
pnpm -v
node -v
```

##### 常见错误

如果安装时遇到如下报错：

![image-20250613022854935](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250613022854935.png)

代表gcc版本不够，需要运行

```bash
sudo apt update
sudo apt install -y build-essential
```



##### 环境变量

如果想要配置环境变量，可以在`~/.bashrc`中添加以下内容：

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

如果想要配置nvm下载nodejs的镜像，可以在`~/.bashrc`中添加以下内容：

```bash
export NVM_NODEJS_ORG_MIRROR=https://npmmirror.com/mirrors/node
```



### 安装jenkins

##### 下载

在1Panel`应用商店`中搜索Jenkins并点击安装

![image-20250613005023137](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250613005023137.png)

填写自己想要的端口号，勾选端口外部访问。

![image-20250613024747551](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250613024747551.png)

点击`编辑compose文件` 将自己的`Maven`和`nodejs`挂载到Jenkins中

比如我的路径是

- /opt/apache-maven-3.9.10
- /home/xgadmin/.nvm/versions/node/v22.16.0

![image-20250613025817787](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250613025817787.png)

输入以下命令可以查看nvm默认node路径

``` bash
 nvm which current
```

点击确认后等待下载完成

##### 配置

通过ip:端口进入Jenkins，首次进入需要配置

![image-20250613143047528](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250613143047528.png)

在1Panel 应用商店中点击日志，查看管理员密码

![image-20250613143135831](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250613143135831.png)

![image-20250613143159980](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250613143159980.png)

将密码输入后点击`继续`

点击`安装推荐的插件`

![image-20250613143315224](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250613143315224.png)

等待安装完成

创建一个管理员账号，`用户名`就是登录用的账号

![image-20250613145242536](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250613145242536.png)

##### 安装常用插件

点击`Manage Jenkins`

![image-20250613145448509](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250613145448509.png)

点击`Plugins`

![image-20250613145537568](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250613145537568.png)

点击`Availbale plugins`

搜索并下载

1. Maven Integration
2. NodeJS
3. Publish Over SSH
