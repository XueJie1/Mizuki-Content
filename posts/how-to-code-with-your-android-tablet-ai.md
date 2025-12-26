---
title: 在 Android 平板上使用 Termux 和 Visual Studio Code 网页版，实现便携编程（Gemini 润色版）
published: 2025-11-01
description: ""
tags: ["编程", "安卓"]
category: ""
draft: false
---

### **〇、前言**

您是否曾想过，将手中的安卓平板转变为一个功能强大的便携式编程工作站？本教程将引导您通过 Termux 和 `proot-distro`，在**无需 Root** 的情况下，安装一个完整的 Linux 发行版（以 Ubuntu 为例），并运行 VS Code Web 服务。最终，您只需一个浏览器，即可随时随地享受桌面级的编码体验。

**在开始之前，请确保您：**

1.  **具备 Linux 基础：** 熟悉基本的命令行操作，理解文件系统和用户权限等概念。
2.  **确认设备架构：** 本教程的操作及软件下载均针对 **Arm64 (aarch64)** 架构的平板电脑。绝大多数现代安卓设备都符合此条件。
3.  **善用 AI 助手：** 遇到问题时，能够清晰地向 [ChatGPT](https://chat.openai.com)、[Claude](https://claude.ai) 或 [Deepseek](https://chat.deepseek.com) 等 AI 工具描述问题，这将是您强大的后援。

---

### **一、基础环境：安装与配置 Termux**

Termux 是我们通往 Linux 世界的大门。

#### **1.1 下载并安装 Termux**

> **重要提示：** 请**不要**从 Google Play 商店下载 Termux，该版本已停止更新且存在诸多问题。

请从以下官方渠道获取最新版本：
*   **[GitHub Releases](https://github.com/termux/termux-app/releases/latest)** (推荐)
*   **[F-Droid](https://f-droid.org/packages/com.termux/)**

#### **1.2 （可选）授予存储权限**

为了让 Termux 内的程序能够读写平板的内部存储（例如 `/storage/emulated/0` 目录），请在 Termux 命令行中执行：

```sh
termux-setup-storage
```
系统将弹出授权请求，请点击“允许”。

#### **1.3 更换为国内镜像源**

更换镜像源是确保软件包下载速度的关键一步。

```sh
termux-change-repo
```
执行后，一个文本用户界面（TUI）将会出现：
1.  保持默认选中的 “Mirror group” 不动，直接用手指点击屏幕上的 `<OK>`。
2.  使用方向键或手指点选 “Mirrors in Chinese Mainland”。
3.  再次点击 `<OK>`，Termux 会自动测试中国大陆各镜像源的速度并选择最优的一个。

完成后，运行以下命令更新并升级所有基础软件包，以验证更换是否成功：

```sh
pkg update && pkg upgrade -y
```

---

### **二、核心步骤：通过 proot-distro 安装 Ubuntu**

`proot-distro` 是一个强大的脚本工具，它利用 `proot` 技术，让我们可以在 Termux 中轻松地安装和管理一个完整的 Linux 发行版，全程无需 Root 权限。

> **网络提醒：** 下载 Linux 根文件系统较大，此步骤**强烈建议**在全局代理环境下进行，否则可能会极其缓慢或失败。

#### **2.1 安装 proot-distro**

```sh
pkg install proot-distro -y
```

#### **2.2 安装 Ubuntu**

我们将以 Ubuntu 为例进行安装。`pd` 是 `proot-distro` 的缩写。

```sh
pd install ubuntu
```
该命令会自动下载 Ubuntu 的根文件系统并进行配置。

> **小贴士：** 您可以通过 `pd list` 命令查看所有支持的 Linux 发行版，然后使用 `pd install <发行版名称>` 来安装您偏爱的一款。

#### **2.3 登录 Ubuntu 环境**

安装完毕后，执行以下命令即可“登录”到这个全新的 Ubuntu 系统中：

```sh
pd login ubuntu
```
登录成功后，您会看到命令提示符变为 `root@localhost:~#`，这表示您已身处 Ubuntu 的 `root` 用户环境中。

---

### **三、精细调校：配置 Ubuntu 子系统**

一个纯净的系统还需要一些基础配置才能顺手使用。

#### **3.1 创建一个普通用户**

直接使用 `root` 用户操作并非最佳实践。让我们创建一个日常使用的普通用户（假设用户名为 `dev`）：

```sh
# -m 标志会自动创建用户的家目录 /home/dev
useradd -m dev
```

为新用户设置密码：
```sh
passwd dev
```
按照提示输入两次密码。注意，输入过程中密码是不可见的。

#### 3.2 授予 sudo 权限

为了让 `dev` 用户能执行需要管理员权限的操作，我们需要将其加入 `sudo` 用户组。

```sh
usermod -aG sudo dev
```

现在，切换到新创建的用户：
```sh
su - dev
```
您会发现命令提示符从 `#` 变成了 `$`，表示已成功切换。

可以通过 `sudo whoami` 命令验证 `sudo` 是否生效。如果系统提示输入 `dev` 用户的密码，并且随后输出了 `root`，则表示配置成功。

#### **3.3 为 Ubuntu 更换国内镜像源**

和配置 Termux 一样，为子系统更换镜像源也至关重要。由于我们是 Arm64 架构，需要使用 `ubuntu-ports` 仓库。

1.  **备份原始源文件：**
    ```sh
    sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
    ```
2.  **编辑源文件：**
    ```sh
    sudo nano /etc/apt/sources.list
    ```
3.  **清空文件内容，并粘贴以下中科大镜像源配置：**
    ```ini
    # 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
    deb https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse
    # deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse
    deb https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse
    # deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse
    deb https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse
    # deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse
    deb https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy-security main restricted universe multiverse
    # deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy-security main restricted universe multiverse
    ```
    *(注：以上配置适用于 Ubuntu 22.04 (Jammy)，请根据您的版本自行调整代号)*

4.  **保存并退出** (在 `nano` 中，按 `Ctrl+X` -> `Y` -> `Enter`)。

5.  **刷新软件包列表进行验证。此时可以关闭代理，体验国内源的极速。**
    ```sh
    sudo apt update && sudo apt upgrade -y
    ```

---

### **四、核心应用：安装与运行 VS Code**

> **网络提醒：** 下载 VS Code 同样建议在代理环境下进行。
> 以下操作均在 `dev` 普通用户下进行。

我们提供两种安装方式，**任选其一**即可。

#### **方案一：安装 VS Code 命令行版 (推荐，更轻量)**

1.  **安装依赖：**
    ```sh
    sudo apt install wget tar -y
    ```
2.  **下载 VS Code CLI (ARM64 for Linux)：**
    ```sh
    wget -O vscode-cli.tar.gz "https://code.visualstudio.com/sha/download?build=stable&os=cli-linux-arm64"
    ```
3.  **解压到家目录：**
    ```sh
    tar -xzf vscode-cli.tar.gz -C ~/
    # 重命名为一个更简洁的目录名
    mv ~/vscode-cli-linux-arm64 ~/vscode-cli
    ```
4.  **将 `code` 命令加入环境变量，方便随时调用：**
    ```sh
    # 编辑 .bashrc 文件
    nano ~/.bashrc
    ```
    在文件末尾另起一行，添加：
    ```sh
    export PATH="$PATH:$HOME/vscode-cli"
    ```
    保存并退出后，执行以下命令使配置立即生效：
    ```sh
    source ~/.bashrc
    ```

#### **方案二：安装 VS Code 完整版 (.deb)**

1.  **安装依赖：**
    ```sh
    sudo apt install wget -y
    ```
2.  **下载 VS Code .deb 包 (ARM64)：**
    ```sh
    wget -O code.deb "https://code.visualstudio.com/sha/download?build=stable&os=linux-deb-arm64"
    ```
3.  **使用 apt 安装 (它会自动处理依赖关系)：**
    ```sh
    sudo apt install ./code.deb -y
    ```

---

### **五、见证奇迹：启动并访问 VS Code Web**

无论您选择哪种安装方式，启动 Web 服务的命令都是相同的。

```sh
code serve-web
```

> **网络提醒：** 首次运行会下载 VS Code Server，请确保网络通畅。

成功后，终端会显示类似如下信息：
```
*
* Visual Studio Code Server
*
* By using the software, you agree to the...
*
Web UI available at http://127.0.0.1:8000/?tkn=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

**复制 `http://127.0.0.1:8000` 开始的完整链接**，打开平板上的 Chrome、Edge 等现代浏览器，粘贴并访问。稍等片刻的加载后，一个功能完整的 VS Code 界面就会呈现在您眼前！

您现在可以打开终端、安装插件、管理文件、编写代码，享受在平板上无缝衔接的开发乐趣了。

---

### **六、已知问题与展望**

*   **插件证书验证：** 在当前 `proot` 环境下，部分需要严格证书验证的插件（如官方的 GitHub Copilot）可能无法正常工作。安装普通插件时若遇到证书提示，可以选择忽略并继续安装。
*   **性能：** `proot` 环境会有一定的性能损耗，但在现代平板的性能下，用于 Web 开发、脚本编写等任务已绰绰有余。

恭喜您，您已成功将 Android 平板武装成了一台强大的便携开发利器！