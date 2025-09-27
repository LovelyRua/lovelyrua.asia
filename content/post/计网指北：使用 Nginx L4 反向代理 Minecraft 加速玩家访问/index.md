---
title: 计网指北：使用 Nginx L4 反向代理 Minecraft 加速玩家访问
description: Nginx 反代 MC 服务器
date: 2025-09-01 20:48:16+0800
# image: cover.jpg
categories:
    - IT-Techs
tags:
    - Nginx
    - Proxy
    - Minecraft
---
### 前言

本文主要受众为使用家庭宽带/机房托管等单线路网络的服主

在为全国各省市使用不同运营商的玩家提供服务时

遇到某些地区玩家反馈网络连接高延迟/丢包等问题的



本文的内容简要：

- Nginx Stream 模块反向代理的核心配置方法
- 一些补充玩法

文末我会推荐一种 相比传统 VPS/ECS 弹性极高 极大程度节约轻业务成本的解决方案


✨>_0 准备工作

在开始之前，请确保你已具备以下条件：

- **一台正在运行的 Minecraft 服务器**: 并且你知道它的 **IP 地址** 和 **游戏端口**（默认为 25565）。
- **一台拥有公网 IP 且线路优良的服务器 (VPS/ECS)**: 这台服务器将用来安装和运行 Nginx。
- (可选) **一个你自己的域名**: 并已将其 DNS 解析管理权交由 Cloudflare 或其他 DNS 服务商。
- (可选) **基础的 Linux 命令行操作能力**: 我们将通过 SSH 连接到服务器进行配置。


## 安装 Nginx

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

**！** Nginx 的 TCP/UDP 代理功能依赖于 **`ngx_stream_module`** 模块。幸运的是，几乎所有主流 Linux 发行版官方仓库中的 Nginx 包都**默认编译**了这个模块。如果你的 Nginx 版本非常古老或来源特殊，你可能需要手动编译。可以通过 nginx -V (大写V) 命令查看编译参数，确认是否包含 `--with-stream`。

## Nginx 核心配置

与我们熟知的用于网站的 `http` 块不同，代理OSI模型四层的 TCP/UDP 流量（比如 Minecraft 的游戏数据）需要使用 `stream` 块。

编辑 Nginx 的主配置文件 nginx.conf：

```bash
# 使用你喜欢的文本编辑器打开主配置文件
$ sudo nano /etc/nginx/nginx.conf
```

在文件的**最外层**（与 `http` 块平级，**不要**放在 `http` 块内部！）添加以下内容：

```nginx
# ================= Stream (TCP/UDP Proxy) Settings =================

stream {
    # 引用一个外部的配置文件目录，让配置更整洁
    # 这样我们就可以在 /etc/nginx/streams-available/ 目录中为每个服务创建独立的配置文件
    include /etc/nginx/streams-enabled/*;
}
```

**！** 将 `stream` 配置独立出来是一种非常好的工程实践。这让你的主配置文件 `nginx.conf` 保持干净，并且便于管理多个不同的 TCP/UDP 代理服务。

创建用于存放 stream 配置的目录：

```bash
# 创建可用配置目录和启用配置目录
$ sudo mkdir -p /etc/nginx/streams-available
$ sudo mkdir -p /etc/nginx/streams-enabled
```

## 为你的 Minecraft 服务器创建反向代理配置

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
    # 如果是IP地址，可以直接填写。如果是域名，强烈建议参考下文“加速Hypixel”中的 resolver 配置
    server your_minecraft_server_ip:25565;
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

### 最简实现

如果你甚至懒得装 Nginx 还可以用如下方法
#### Linux: iptables

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

#### Windows: netsh

- **这是什么：Windows 系统内置的端口代理/转发命令行工具。
- **如何实现：在 Windows 服务器上，一行命令即可设置。

```powershell
# 以管理员身份运行 PowerShell 或 CMD
# 将所有访问本机 IPv4 25565 端口的 TCP 流量，转发到目标服务器

netsh interface portproxy add v4tov4 listenport=25565 listenaddress=0.0.0.0 connectport=25565 connectaddress=1.1.1.1
```

### 加速 Hypixel (动态域名解析示例)

是的，你还可以用一条**直连线路更优的专线**来反向代理 **Hypixel 等服务器**， 规避普通家庭宽带国际出口的拥堵，实现类似游戏加速器的效果。

```
# 定义 Hypixel 的上游

upstream hypixel_backend {

    # 当后端是域名时，必须使用 resolver 指令！
    # Nginx 默认只在启动时解析一次域名，如果域名 IP 变了，Nginx 不会知道。
    # resolver 指令告诉 Nginx 使用指定的 DNS 服务器，并周期性地重新解析域名。
    # 这里的 8.8.8.8 是 Google DNS，valid=300s 表示每 300 秒（5分钟）刷新一次。

    resolver 8.8.8.8 valid=300s;
    server mc.hypixel.net:25565;
}

server {

    # 监听你的 VPS 的 25565 端口
    
    listen 25565;
    proxy_pass hypixel_backend;
}

```
**！** **注意**：这种方法**仅供个人或小范围朋友使用**。公开提供此类服务可能违反 Hypixel 的用户协议，这只是一个利用技术提升个人游戏体验的有趣实践。

### 推荐：雨云云应用

对于这种需求 使用一台完整的VPS/ECS主机实际产生了不小的资源及成本浪费

在此强烈推荐使用的方案：**雨云云应用**

这是一种以单个容器为最小单位的弹性极高的云服务

使用云应用 我们可以节省相比ECS十倍甚至九倍的成本

雨云提供的社区应用商店功能还很大程度上简化了传统安装部署的人力和时间消费

![[Pasted image 20250926143228.png]](https://pic.lovelyrua.asia:81/api/assets/5335f4ce-f8f5-4e25-a4cb-6f654999e91d/original?key=IYz7LMuntz9bMrdmgVMKTyHJMWIK4DUzkkOSjAErkiA3UL_hxjcg2zhSCabSWWEuF1Y)

接下来我会逐步演示如何通过雨云云应用部署 L4 反向代理服务

![[Pasted image 20250926143439.png]](https://pic.lovelyrua.asia:81/api/assets/a15d515d-bee3-4658-9b4e-62159d9fda35/original?key=IYz7LMuntz9bMrdmgVMKTyHJMWIK4DUzkkOSjAErkiA3UL_hxjcg2zhSCabSWWEuF1Y)

首先在应用商店界面找到需要安装的应用 这里可以使用 Nginx Proxy Manager

![[Pasted image 20250926143559.png]](https://pic.lovelyrua.asia:81/api/assets/04868b2c-052c-4abe-9398-7b5d93adfdce/original?key=IYz7LMuntz9bMrdmgVMKTyHJMWIK4DUzkkOSjAErkiA3UL_hxjcg2zhSCabSWWEuF1Y)
![[Pasted image 20250926143614.png]](https://pic.lovelyrua.asia:81/api/assets/03a4eb2f-5983-4ef7-aa0e-35df67d6b770/original?key=IYz7LMuntz9bMrdmgVMKTyHJMWIK4DUzkkOSjAErkiA3UL_hxjcg2zhSCabSWWEuF1Y)

这里的端口填写不用担心 稍后都可以进行修改

点击安装应用之后就可以看到 Nginx Proxy Manager 已经出现在应用管理界面里

几秒内应用就会自动完成部署并启动

![[Pasted image 20250926143643.png]](https://pic.lovelyrua.asia:81/api/assets/b2bdbc52-002c-4662-930a-bb64898247ed/original?key=IYz7LMuntz9bMrdmgVMKTyHJMWIK4DUzkkOSjAErkiA3UL_hxjcg2zhSCabSWWEuF1Y)

安装完成后 我们需要调整容器应用对外暴露的端口

在 `我的项目 - 应用管理 - 应用 - 服务` 页面可以对应用进行端口的配置

![[Pasted image 20250926143807.png]](https://pic.lovelyrua.asia:81/api/assets/bf6d5996-c707-4f50-a65a-35f68b2385db/original?key=IYz7LMuntz9bMrdmgVMKTyHJMWIK4DUzkkOSjAErkiA3UL_hxjcg2zhSCabSWWEuF1Y)

Nginx Proxy Manager 默认会暴露 WebUI 控制台的端口和 HTTP & HTTPS 的端口

由于这里我们是进行 MC 服务器的反代 可以直接将用不到的 80 和 443 端口 修改成自己需要的端口（这一步骤可按照个人喜好自行配置）

端口映射输入框中前一个框代表容器本身监听的内部端口 后一个框表示实际暴露到外部访问 IP 的端口

如这里设置 25565:8500 就意味着稍后我们需要将反向代理监听在 :25565 并使用端口8500 访问雨云提供的 IP 就可以连接到我们的服务器

![[Pasted image 20250926143933.png]](https://pic.lovelyrua.asia:81/api/assets/d340be74-0862-49fb-acd5-d8b94500ba17/original?key=IYz7LMuntz9bMrdmgVMKTyHJMWIK4DUzkkOSjAErkiA3UL_hxjcg2zhSCabSWWEuF1Y)

(容器内部我们让 Nginx 监听 25565 端口，然后雨云平台将这个内部端口映射到公网 IP 的 8500 端口上)

接下来访问 Nginx Proxy Manager 的 WebUI 控制台来进行反向代理的实际配置

默认的公网地址是从集群共享 IP 中随机分配的

如有独立公网 IP 的需求 可以在`我的项目 - 设置 - IP 地址管理`添加独立 IP 地址(会产生相应费用)

![[Pasted image 20250926150719.png]](https://pic.lovelyrua.asia:81/api/assets/49d4b406-dc22-4b43-afaf-0c4d107bd059/original?key=IYz7LMuntz9bMrdmgVMKTyHJMWIK4DUzkkOSjAErkiA3UL_hxjcg2zhSCabSWWEuF1Y)

后台的地址是在服务界面看到的公网 IP 地址加控制台服务的外部端口

比如在这里就是 `http://110.42.111.57:41998`

![[Pasted image 20250926151255.png]](https://pic.lovelyrua.asia:81/api/assets/343aee91-ea9d-472b-ba47-012c4e518b77/original?key=IYz7LMuntz9bMrdmgVMKTyHJMWIK4DUzkkOSjAErkiA3UL_hxjcg2zhSCabSWWEuF1Y)

默认账户密码是

admin@example.com

changeme

第一次登录之后先跟随指引修改管理员账户密码

要创建四层代理 我们需要到 Streams 配置页面 点击右上角的 Add Stream

![[Pasted image 20250926151740.png]](https://pic.lovelyrua.asia:81/api/assets/eb01e70a-c339-4c76-ac6b-319212920bba/original?key=IYz7LMuntz9bMrdmgVMKTyHJMWIK4DUzkkOSjAErkiA3UL_hxjcg2zhSCabSWWEuF1Y)
![[Pasted image 20250926151810.png]](https://pic.lovelyrua.asia:81/api/assets/89105190-758c-4c65-9e41-c386fdf77fb3/original?key=IYz7LMuntz9bMrdmgVMKTyHJMWIK4DUzkkOSjAErkiA3UL_hxjcg2zhSCabSWWEuF1Y)
![[Pasted image 20250926151846.png]](https://pic.lovelyrua.asia:81/api/assets/4e617f1b-199f-4207-926c-f37184f2a467/original?key=IYz7LMuntz9bMrdmgVMKTyHJMWIK4DUzkkOSjAErkiA3UL_hxjcg2zhSCabSWWEuF1Y)

这里的 Incoming Port 需要填写对应我们刚刚在雨云防火墙设置的 **内部端口**

刚才的演示中我们填写了 25565 故此处需要填 25565

Forward Host&Port 填写的是被代理的服务的主机名或 IP 地址

比如被代理的是 mc.lovelyrua.asia:25566 这里就填写 mc.lovelyrua.asia 和 25566

如果如上步骤无误 保存之后访问`110.42.111.57:8500`就可以访问到经过代理后的服务了

如果要代理多个服务器 通过一样的步骤开放端口并添加代理配置即可

看完本文内容应该也已经对雨云云应用的使用有了一定了解

最后分享一下自己使用一段时间的费用

我个人轻度使用下来 每星期成本仅五元 且共享网络的弹性带宽空间上限也更大

按量计费的流量每百GB也只要5元左右

![[Pasted image 20250926155848.png]](https://pic.lovelyrua.asia:81/api/assets/4d51d745-86b8-4c35-9861-167cfffaddd8/original?key=IYz7LMuntz9bMrdmgVMKTyHJMWIK4DUzkkOSjAErkiA3UL_hxjcg2zhSCabSWWEuF1Y)
![[Pasted image 20250926160825.png]](https://pic.lovelyrua.asia:81/api/assets/0aa28780-1c57-4ba3-ae65-737e0904cc85/original?key=IYz7LMuntz9bMrdmgVMKTyHJMWIK4DUzkkOSjAErkiA3UL_hxjcg2zhSCabSWWEuF1Y)

##### 补充：雨云机房反代前后延迟

![[a1d52736-c142-40bd-8053-5176d526dce3.png]](https://pic.lovelyrua.asia:81/api/assets/7d78ac9c-556f-48f6-982b-44e0822239d4/original?key=IYz7LMuntz9bMrdmgVMKTyHJMWIK4DUzkkOSjAErkiA3UL_hxjcg2zhSCabSWWEuF1Y)
![[2aeca52f-24ae-4283-9d56-5dc2818a3d89.png]](https://pic.lovelyrua.asia:81/api/assets/418f26e7-fb4d-4519-b0d0-a6e1b5dde3fc/original?key=IYz7LMuntz9bMrdmgVMKTyHJMWIK4DUzkkOSjAErkiA3UL_hxjcg2zhSCabSWWEuF1Y)

虽然直观数据差别不大 但是对于本身网络环境不佳的用户来说三线优化线路会带来质的飞跃