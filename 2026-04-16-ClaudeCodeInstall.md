---
title: WSL 环境下的Claude Code 安装以及接入 GLM 模型
date: 2026-04-16 16:27:47
tags:
  - AI
  - 工具
categories:
  - 工具
abbrlink: claudecode
---

# Claude Code

## 安装

在Windows 中安装Claude Code 需要`wsl2`以下环境安装在 wsl2 中，如果是 Mac、Linux 则直接安装：

- nvm
- node

### wsl

Windows Subsystem Linux，Windows 的 Linux 子系统。安装和使用都很简单，比虚拟机轻便，且可以用 Linux 命令和软件操作 Windows 文件，非常方便。

#### 确保已开启虚拟化！！！

打开任务管理器，检查是否启用虚拟化

![1776341674416](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776341674416.png)

#### 安装wsl2

按下 `Win` 键搜索`PowerShell`以`管理员`运行

输入以下命令

```bash
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

![1776341880957](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776341880957.png)

输入以下命令启动 wsl2，如果是 Windows10默认是 wsl1

```bash
wsl --set-default-version 2
```

#### 重启电脑

#### 下载并安装Ubuntu 到 D 盘

以管理员启动`PowerShell`输入以下命令安装 Ubuntu，后面路径可自行修改。如不写 --location 路径，则会自动安装在 `C 盘`！

```bash
wsl --install -d Ubuntu --location D:\WSL\Ubuntu
```

安装后会要求设置账号密码，请记住自己设置的密码，忘记会很麻烦！！！

> 输入密码时不会显示

成功安装后如下

![1776346041496](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776346041496.png)

绿色的文字每个人不同

```
用户名@电脑名字:~$
```

安装完成后如果关闭了 PowerShell，可以重新打开后输入以下命令查看是否安装成功。

```bash
wsl -l -v
```

>  出现以下界面代表安装成功，没有出现则重新检查上面流程是否安装成功

![1776343082282](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776343082282.png)

> 后续所有操作都是在wsl 中 ！！！

#### 在`wsl`中安装`nvm`，使用nvm安装`Node.js`

nvm 和 npm 是 Node.js 生态系统中的两个重要工具，但它们的功能和用途完全不同。

nvm (Node Version Manager) 是一个用于管理 Node.js 版本的工具。它允许开发者在同一台机器上安装和切换多个版本的 Node.js。

npm (Node Package Manager) 是 Node.js 的包管理工具，主要用于管理项目的依赖和脚本。

> Claude Code 就是通过`npm`安装

##### 使用管理员打开`PowerShell`后进入wsl

如果在 wsl 中，则跳过此步

在 PowerShell 中输入 wsl，输入账号密码进入 Ubuntu，进入后会变成这样

![1776344459890](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776344459890.png)

##### 安装必要工具curl wget和git

输入：

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl wget git -y
```

如果出现以下信息，则输入密码按下回车后继续，等待进度条加载完成后出现上图绿色文字

![1776346705867](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776346705867.png)

确保此处更新完成

##### 安装 nvm

输入此命令安装 nvm

```basn
wget -qO- https://gitee.com/RubyMetric/nvm-cn/raw/main/install.sh | bash
```

安装完成后输入此命令更新环境变量

```bash
source ~/.bashrc
```

输入nvm -v 查看 nvm 版本号，如果显示报错说明 nvm 安装失败了

![1776350382980](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776350382980.png)

##### 安装 Node.js

输入此命令安装 Node.js lts 版本

> LTS (Long-Term Support) 长期支持版本，会持续更新修复漏洞，所有软件都尽量选择 LTS！

```bash
nvm install --lts
```

安装后输入此命令查看Node.js 版本号

```bash
node -v
```

![1776352508232](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776352508232.png)

##### 安装Claude Code

输入以下命令

```bash
npm install -g @anthropic-ai/claude-code
```

安装成功后输入此命令查看版本号，版本可能会不一样，只要能显示就没问题

```bash
claude -v
```

![1776353344731](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776353344731.png)

## 配置

### 绕过检测

输入命令进入claude

```bash
claude
```

进入后大概率会显示如下界面

因为 Claude 首次进入是引导界面，但是因为科学问题，没法访问 Claude 的引导界面，所以出现如下提示

![1776354120138](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776354120138.png)

#### 解决办法有两种，选择一个

在`~/.claude.json`文件中将`hasCompletedOnboarding`字段改为`true`即可跳过引导界面

##### 方法一：脚本直接修改

直接输入：

```bash
sed -i 's/}$/,"hasCompletedOnboarding":true}/' .claude.json
```

这行命令会直接添加好内容



##### 方法二：通过vim编辑

> 在 Linux 中复制的快捷键是`ctrl`+`shift`+`C`，粘贴是`ctrl`+`shift`+`V`

输入以下命令进入编辑模式

```bash
vim ~/.claude.json
```

![1776356643677](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776356643677.png)

按下 `i` 进入输入模式，操控键盘的上下左右键，将光标移动到`userId` 这一行的最后，输入一个`英文`逗号`,`然后再添加

```bash
"hasCompletedOnboarding":true
```

修改后的文件如下

![1776356826179](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776356826179.png)

输入完成后按下`esc`退出输入模式，再输入以下指令，保存并且退出该文件

```bash
:wq
```

### 首次进入

再次输入`claude`进入 Claude，如修改正确，成功进入 Claude 界面，提醒是否信任此文件夹

![1776356951802](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776356951802.png)

小键盘的上下进行选择，回车键确认

这里直接按下回车

![1776357002487](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776357002487.png)

现在可以对话了，但是还是用不了的，因为这只是代表 Claude 可以使用了，但是没有接入模型不能完成 AI 操作。

到这里我们已经非常非常接近成功了！

## 接入大模型



Claude Code 只是一个工具，可以自由选择接入不同的大模型，这里因为 GLM 最近免费，所以先接入 GLM 使用

### 生成API Key

免费领取 token：

> 我正在智谱大模型开放平台 BigModel.cn上打造AI应用，智谱新一代旗舰模型GLM-5已上线， 在推理、代码、智能体综合能力达到开源模型 SOTA 水平，通过我的邀请链接注册即可获得 2000万Tokens 大礼包，期待和你一起在BigModel上畅享卓越模型能力；链接：https://www.bigmodel.cn/invite?icode=F59uQ%2FDhP6VnFseNoMrcX%2Bnfet45IvM%2BqDogImfeLyI%3D

注册后即可领取 2000万 token，够用一段时间了。

点击控制台 -> API Key -> 新建API Key（随意取名）

创建 API Key 后随时在控制台中可以查看，马上就会用到！！！

![1776387098134](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776387098134.png)

##  修改Claude Code配置

修改配置可以跟着我来，也可以看以下官方文档

官方教程：https://docs.bigmodel.cn/cn/coding-plan/tool/claude#claude-code

修改`~/.claude/settings.json` 文件（没有的话创建一个）

输入以下命令编辑此文件（如果无，输入此命令会自动创建新文件）

```bash
vim ~/.claude/settings.json
```

如果输入后，下方出现`[New]`字样，代表新创建的此文件

将以下内容放入文件，如果文件已存在，添加以下内容

> 右侧的`glm-4.5-air`等字样为使用的模型，通过此链接https://bigmodel.cn/finance-center/resource-package/package-mgmt可以查看对应模型的对应用量，随时可以来修改模型，`glm-5.1`最强、消耗最高、费用最高。用完了可以全部换成`glm-4.5-air`够用了。

`ANTHROPIC_AUTH_TOKEN`的值，写上自己的API Key ！！！

```bash
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "写上自己的API Key",
    "ANTHROPIC_BASE_URL": "https://open.bigmodel.cn/api/anthropic",
    "API_TIMEOUT_MS": "3000000",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": 1,
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "glm-4.5-air",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "glm-5-turbo",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "glm-5.1"
  }
}
```

## 使用测试

在 Windows 中任意打开一个文件夹，在 url 栏中输入 PowerShell 按下回车

这样打开的 PowerShell 是带有路径的，如果不会 Linux，后续都可以这样打开

![1776389316198](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776389316198.png)

打开 PowerShell 后输入 wsl 进入 Ubuntu

![1776389946518](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776389946518.png)

输入`claude`进入 claude，选择`Yes, I trust this folder`

如果出现以下信息，代表上一步的模型配置有误

![1776388005422](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776388005422.png)

没问题话就可以输入想要做的事情，按下回车

![1776392065571](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776392065571.png)

如果有对文件进行操作他会提问，按下回车接受更改

![1776392199213](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776392199213.png)

生成完毕

![1776393713260](https://simon-picgo-img.oss-cn-chengdu.aliyuncs.com/1776393713260.png)
