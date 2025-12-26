---
title: 在 Android 平板上使用 Termux 和 Visual Studio Code 网页版，实现便携编程
published: 2025-11-01
description: ""
tags: ["编程", "安卓"]
category: ""
draft: false
---
## 前言
此教程假设：
1. 阅读者熟悉 Linux 相关的基本知识，并使用过 Linux 和其命令行一段时间。
2. 阅读者要操作的平板的处理器架构是 Arm64。
3. 阅读者有在官网询问 AI （如 [Deepseek](https://chat.deepseek.com), [ChatGPT](https://chatgpt.com), [Claude](https://claude.ai)）技术问题的能力。

## 下载并安装 Termux
在 [GitHub](https://github.com/termux/termux-app/releases/latest) 或 [F-Droid](https://f-droid.org/packages/com.termux/) 下载 Termux。

## 配置 Termux
### （可选）让 Termux 可以访问内部存储
在 Termux 命令行处键入以下内容并按回车：
```sh
termux-setup-storage
```
此时平板会弹出一个授权选项，同意授权即可。

### 更改 Termux 软件源
在 Termux 命令行处键入以下内容并按回车：
```sh
termux-change-repo
```
Termux 的界面会变成TUI。默认选中“Mirror group”，此时可以用手指点击 \<OK\> 进入下一步。然后 Termux 会让你选择用哪个地区的软件源，点击“Mirrors in Chinese Mainland”后，对应的选项会高亮，然后点 <OK>，Termux 会对中国大陆的软件源测速。完毕后，软件源更换完成。

执行以下命令更新软件包来验证：
```sh
pkg update && pkg upgrade -y
```

## 安装 proot-distro
执行以下命令来安装 proot-distro：
```
pkg install proot -y
```

## 使用 proot-distro 安装 Linux，以 Ubuntu 为例
proot-distro 是 Termux 的一个命令行工具，借助该工具，用户可以轻松地在手机或平板电脑上创建一个轻量级的、功能齐全的 Linux 环境，而无需对设备进行 root。如果你的平板已经 root,可以使用 chroot 安装 Linux，此教程不提供这种安装方式的教程。

注意：此步骤需要使用代理等方式，才能以正常速度下载内容。

执行以下命令安装 proot-distro 下的 Ubuntu：
```sh
pd install ubuntu
```

下载安装完成后，执行以下命令登录 Ubuntu：
```sh
pd login ubuntu
```

如果你想使用其他发行版，可以使用 `pd list` 命令以显示可以安装的 Linux 发行版，再使用 `pd install distro-name` 来安装。

## 配置 Linux
### 创建普通用户
使用以下命令，假设要创建的用户名是 `user`：
```sh
useradd -m user
```
给普通用户创建密码：
```sh
passwd user
```
注：输入密码时，密码不会显示。需要输入两次密码，每次输入完成后，按回车。

### 给普通用户授权 sudo 权限
此处用的是添加到 sudo 用户组的方式，也可以用编辑 sudoer 配置文件的方式。
执行以下命令：
```sh
usermod -aG sudo user
```

切换到普通用户：
```sh
su user
```
此时会发现，命令行左侧由`#`变成了`$`。
验证是否授权成功：
```sh
sudo -i
```
如果提示输入密码，那么就授权成功了。
### 为 Ubuntu 更换镜像源
此处以中国科学技术大学提供的源和教程为例。

备份原镜像源文件：
```sh
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```
替换镜像源为中科大源：
```sh
sudo sed -i -e 's@//ports.ubuntu.com/\? @//ports.ubuntu.com/ubuntu-ports @g' \
            -e 's@//ports.ubuntu.com@//mirrors.ustc.edu.cn@g' \
            /etc/apt/sources.list
```
更新一下软件包来测试，此时可以关掉代理：
```sh
sudo apt update && sudo apt upgrade -y
```
## 在 proot 容器中安装并配置 Visual Studio Code
注意：此步骤可能需要使用代理等方式，才能以正常速度下载内容。进行以下操作的用户是刚才创建的普通用户，而不是 root。
有两种方式，第一种是安装 VSCode 完整版，第二种是只安装命令行版，二选一即可。
### 安装完整版
安装 wget：
```
sudo apt install wget tar -y
```
使用 wget 下载 Visual Studio Code 完整版（复制以下内容后，长按命令行界面，再点 Paste 粘贴）：
```sh
wget -O code.deb "https://code.visualstudio.com/sha/download?build=stable&os=linux-deb-arm64"
```
安装：
```sh
sudo apt install ./code.deb -y
```
### 安装命令行版
安装 wget, tar：
```
sudo apt install wget tar -y
```
使用 wget 下载 Visual Studio Code 命令行版：
```sh
wget -O code.tar.gz "https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-arm64"
```

解压缩：
```sh
tar zxvf ./code.tar.gz
# 以防万一，添加可执行权限
chmod +x ./code/code
```
假设 code 二进制文件的位置在`/home/user/code/code`，现在把`/home/user/code`加入 PATH 环境变量中：
```sh
vi /home/user/.bashrc
```
键入`Shift + G`跳到最后，按`o`键另起一行，复制以下内容后，长按命令行界面，再点 Paste 粘贴：
```sh
export PATH=$PATH:/home/user/code
```
按`Esc`（注意是 Termux 界面下方左上角的 `Esc`），`:wq`和回车保存并退出。

执行以下命令生效：
```sh
source ~/.bashrc
```

## 运行 Visual Studio Code 并开始使用
注意：此步骤可能需要使用代理等方式，才能以正常速度下载内容。

执行以下命令以运行 Visual Studio Code 服务器：
```sh
code serve-web
```
如果没有问题，命令行会输出类似以下的内容：
```
*
* Visual Studio Code Server
*
* By using the software, you agree to
* the Visual Studio Code Server License Terms (https://aka.ms/vscode-server-license) and
* the Microsoft Privacy Statement (https://privacy.microsoft.com/en-US/privacystatement).
*
Web UI available at http://127.0.0.1:8000?tkn=***
```
复制最后的链接，到 Chrome, Edge 等 Chromium 浏览器粘贴，稍等片刻就可使用了。

## 已知的问题
安装插件时，会提示无法验证证书，可以选择忽略并继续安装。

无法使用 Copilot，原因也是无法验证证书。