---
title: 计网指北：使用 Nginx L4 反向代理 Minecraft 加速玩家访问
description: Nginx 反代 MC 服务器
date: 2025-09-01 20:48:16+0000
image: cover.jpg
categories:
    - IT-Techs
tags:
    - Nginx
    - Proxy
    - Minecraft
---
## 前言

本文主要受众为使用家庭宽带/机房托管等单线路网络的服主
在为全国各省市使用不同运营商的玩家提供服务时
遇到某些地区玩家反馈网络连接高延迟/丢包等问题的


这是本文章的内容简要：

- Nginx Stream 模块反向代理的核心配置方法
- 一些补充玩法

✨>_0 准备工作

在开始之前，请确保你已具备以下条件：

- **一台正在运行的 Minecraft 服务器**: 并且你知道它的 **IP 地址** 和 **游戏端口**（默认为 25565）。
- **一台拥有公网 IP 且线路优良的服务器 (VPS/ECS)**: 这台服务器将用来安装和运行 Nginx。
- (可选) **一个你自己的域名**: 并已将其 DNS 解析管理权交由 Cloudflare 或其他 DNS 服务商。
- (可选) **基础的 Linux 命令行操作能力**: 我们将通过 SSH 连接到服务器进行配置。


### 1.1 安装 Nginx

本步骤旨在确保你的服务器上安装了支持 TCP 代理的 Nginx。

首先，通过 SSH 登录到你的 Nginx 服务器。

```bash
# 更新你的包管理器缓存
$ sudo apt update  # 适用于 Debian/Ubuntu
$ sudo yum update  # 适用于 CentOS/RHEL

# 安装 Nginx。大多数现代发行版的官方源都包含了 stream 模块。
$ sudo apt install nginx -y
$ sudo yum install nginx -y

# 验证 Nginx 是否安装成功并查看版本信息
$ nginx -v
# 你应该会看到类似 `nginx version: nginx/1.18.0` 的输出
```

**！** Nginx 的 TCP/UDP 代理功能依赖于 **--with-stream** 模块。幸运的是，几乎所有主流 Linux 发行版官方仓库中的 Nginx 包都**默认编译**了这个模块。如果你的 Nginx 版本非常古老或来源特殊，你可能需要手动编译。可以通过 nginx -V (大写V) 命令查看编译参数，确认是否包含 ngx_stream_module。

### 1.2 Nginx 核心配置

与我们熟知的用于网站的 http 块不同，代理OSI模型四层的 TCP/UDP 流量（比如 Minecraft 的游戏数据）需要使用 stream 块。

编辑 Nginx 的主配置文件 nginx.conf：

```bash
# 使用你喜欢的文本编辑器打开主配置文件
$ sudo nano /etc/nginx/nginx.conf
```

在文件的**最外层**（与 http 块平级，**不要**放在 http 块内部！）添加以下内容：

```nginx
# ================= Stream (TCP/UDP Proxy) Settings =================

stream {
    # 引用一个外部的配置文件目录，让配置更整洁
    # 这样我们就可以在 /etc/nginx/streams-available/ 目录中为每个服务创建独立的配置文件
    include /etc/nginx/streams-enabled/*;
}
```

**！** 将 stream 配置独立出来是一种非常好的工程实践。这让你的主配置文件 nginx.conf 保持干净，并且便于管理多个不同的 TCP/UDP 代理服务。

创建用于存放 stream 配置的目录：

```bash
# 创建可用配置目录和启用配置目录
$ sudo mkdir /etc/nginx/streams-available
$ sudo mkdir /etc/nginx/streams-enabled
```

### 2. 为你的 Minecraft 服务器创建反向代理配置

现在，我们将为你的 MC 服务器创建一个专属的配置文件。

```bash
# 在 'streams-available' 目录下创建一个新的配置文件
$ sudo nano /etc/nginx/streams-available/minecraft.conf
```

在打开的空文件中，粘贴以下内容，并**根据注释修改为你自己的信息**：

```nginx
# 定义一个上游服务器组，名字叫 'mc_backend'
upstream mc_backend {
    # 你的 Minecraft 服务器的真实内网地址和端口
    # 如果 Nginx 和 MC 服务器在同一台机器上，就用 127.0.0.1
    server 127.0.0.1:25565;
}

# 定义一个服务器监听块
server {
    # 让 Nginx 监听公网的 25565 端口，准备接收玩家的连接
    listen 25565;

    # 将所有接收到的流量，原封不动地转发给我们刚刚定义的 'mc_backend' 上游服务器组
    proxy_pass mc_backend;

    # (可选) 开启 proxy_protocol，用于向后端传递玩家的真实 IP 地址 ➊
    # proxy_protocol on;
}
```

保存并退出文件 (Ctrl+X, Y, Enter)。

现在，我们需要**启用**这个配置：

```bash
# 创建一个从 'available' 到 'enabled' 的软链接
# 这就像在 Windows 桌面上创建一个快捷方式
$ sudo ln -s /etc/nginx/streams-available/minecraft.conf /etc/nginx/streams-enabled/
```

➊ proxy_protocol 是什么？  
**！** 当你使用反向代理时，你的 Minecraft 服务器看到的**所有玩家的 IP 地址**都会变成 Nginx 服务器的地址（例如 127.0.0.1）。这会导致封禁 (ban) 和白名单 (whitelist) 功能失效。  
**！** 开启 proxy_protocol 后，Nginx 会在转发数据时额外附加一个头部信息，包含了玩家的**真实 IP**。你需要在你的 MC 服务器端（例如 Spigot/Paper 的配置文件 spigot.yml 中设置 bungeecord: true）也开启对这个协议的支持，才能正确获取玩家 IP。

**重载 Nginx 并测试**

    - 回到你的 Nginx 服务器终端。

    - 首先，测试配置文件语法是否有误：

        ```bash
        $ sudo nginx -t
        # 如果一切正常，你会看到：
        # nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
        # nginx: configuration file /etc/nginx/nginx.conf test is successful
        ```

    - 如果语法正确，平滑地重载 Nginx 使配置生效：

        
        ```bash
        $ sudo systemctl reload nginx
        ```

如果连接失败，请优先检查**服务器的防火墙（安全组）**是否已经**放行了 TCP 25565 端口**的入站流量。这是最常见的错误原因。

---
## 额外使用例


#### 懒人方案

正文演示的方法是非常标准的 Nginx 配置最佳实践
但如果你只需要反代一两个服务器 从效率的角度大可不必使用层级目录配置
直接在主配置文件 `nginx.conf` 里加入以下内容就可以了

```nginx
stream {

	server {
		listen {Port} [udp];
	    proxy_pass {IP:Port};
	}
	
}
```

##### 最简实现

如果你甚至懒得装 Nginx 还可以用如下方法
###### Linux: iptables

- **这是什么**：Linux 内核自带的防火墙和网络地址转换 (NAT) 工具。
- **如何实现**：通过设置 PREROUTING 和 POSTROUTING 链的 NAT 规则，可以实现端口转发，达到反向代理的效果。

```bash
	# 假设后端 MC 服务器的 IP 是 1.1.1.1

    # 1. 开启内核 IP 转发
    $ sudo sysctl -w net.ipv4.ip_forward=1

    # 2. 设置 DNAT 规则 (将所有访问本机 25565 端口的流量，目标地址改为 MC 服务器)
    $ sudo iptables -t nat -A PREROUTING -p tcp --dport 25565 -j DNAT --to-destination 1.1.1.1:25565

    # 3. 设置 SNAT/MASQUERADE 规则 (将来自后端 MC 服务器的返回流量，源地址伪装成本机)
    $ sudo iptables -t nat -A POSTROUTING -p tcp -d 1.1.1.1 --dport 25565 -j MASQUERADE
    ```

###### Windows: netsh

- **这是什么：Windows 系统内置的端口代理/转发命令行工具。
- **如何实现：在 Windows 服务器上，一行命令即可设置。

```powershell
# 以管理员身份运行 PowerShell 或 CMD
# 将所有访问本机 IPv4 25565 端口的 TCP 流量，转发到目标服务器

netsh interface portproxy add v4tov4 listenport=25565 listenaddress=0.0.0.0 connectport=25565 connectaddress=1.1.1.1
```

##### 加速 Hypixel

是的，你还可以用一条**直连线路更优的专线**来反向代理 **Hypixel 等服务器**， 规避普通家庭宽带国际出口的拥堵，实现类似游戏加速器的效果。

```
# 定义 Hypixel 的上游

upstream hypixel_backend {

    # 直接使用域名，Nginx 会自动解析
    # resolver 指令是必要的，用于解析动态域名
    
    resolver 8.8.8.8;
    server mc.hypixel.net:25565;
}

server {

    # 监听你的 VPS 的 25565 端口
    
    listen 25565;
    proxy_pass hypixel_backend;
}

```
**！** **注意**：这种方法**仅供个人或小范围朋友使用**。公开提供此类服务可能违反 Hypixel 的用户协议，这只是一个利用技术提升个人游戏体验的有趣实践。