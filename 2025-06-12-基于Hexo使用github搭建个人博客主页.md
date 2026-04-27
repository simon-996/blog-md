---
title: 基于Hexo使用Github Pages搭建个人博客主页
tags:
  - 建站
  - github
  - Hexo
  - 博客
categories: 个人博客
abbrlink: b9b5bbd
date: 2025-06-12 13:07:41
---

## 前言

### 为什么需要一个个人博客主页？

1. 记录学习的知识点，可以随时看到所学知识进行回顾。
2. 记录学习过程中遇到的 bug 或者小问题。当以后自己或者别人遇到同样的问题，可以随时打开博客进行查看。
3. 记录某些软件的安装或配置。当遇到换设备或者重装软件时，可以快速进行回顾。

### Github Pages是什么？

[Github Page](https://pages.github.com/) 是 Github 推出的一个免费的静态网站托管服务。

简单来说就是我们可以`免费`，`0成本`的搭建一个可以公网访问的个人博客。

### Hexo是什么？

[Hexo](https://hexo.io/zh-cn/) 是一个基于 `nodejs`环境，快速、简洁且高效的静态博客框架。

可以通过 `nodejs` 快速的在本地搭建环境，并且一键推送到 Github 进行静态页面部署。







> 需要首先拥有一个github账号

## 创建github仓库，并且配置ssh

### 本机生成` SSH key`(安装git后进行)

打开终端，输入如下命令

```bash
ssh-keygen -t rsa -C "your_email@example.com"
```

一直按回车，直到出现如下提示

```bash
Your identification has been saved in /c/Users/you/.ssh/id_rsa.
# Your public key has been saved in /c/Users/you/.ssh/id_rsa.pub.
# The key fingerprint is:
# 01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db your_email@example.com
```

生成的文件分为` id_rsa` （私钥） 和 ` id_rsa.pub` （公钥）

Windows中文件位于:` C:\Users\用户\.ssh`

Mac和Linux中生成的文件位于` ~/.ssh`

### 在github中添加SSH 公钥

这一步是为了让本地免密提交github代码

首先打开github.com

登录账号

在首页点击右上角` 头像`

点击` Settings`

在左侧侧边栏点击` Access` 中的 ` SSH and GPG Keys`

![image-20250612132418343](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250612132418343.png)

点击右侧的` New SSH key` 按钮

![image-20250612132444183](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250612132444183.png)

填写对应内容，`title` 为SSH key的名字，Key为本机` id_rsa.pub` （公钥）的内容

![image-20250612132953213](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250612132953213.png)

填写完成后到`终端` 或`cmd` 输入以下命令进行测试

```bash
ssh -T git@github.com
```

出现以下内容代表测试成功

![image-20250612132921418](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250612132921418.png)

### 创建github仓库

需要创建一个与用户名同名的` github仓库` 来使用github ` page`

回到github.com 首页

点击` Create New Repository`

创建一个命名为`用户名.github.io`的仓库，勾选创建` READNE.md`文件用于测试

![image-20250612150710495](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/image-20250612150710495.png)

### 本地创建Hexo

#### 环境搭建

##### node

hexo
