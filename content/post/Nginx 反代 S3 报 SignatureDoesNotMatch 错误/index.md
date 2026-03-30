---
title: Nginx 反代 S3 报 SignatureDoesNotMatch 错误
description: 反代 MiniO 等 S3 服务时的签名相关问题
date: 2026-03-28 10:32:22+0800
# image: cover.jpg
categories:
    - IT-Techs
tags:
    - Nginx
    - S3
    - MiniO
    - Proxy
---

## 现象

访问通过 Nginx 反代的 MinIO 时, 出现以下报错:

`SignatureDoesNotMatch: The request signature we calculated does not match the signature you provided.`

直接通过内网 IP 访问没问题, 过 Nginx 不行; 切换到旧的 V2 签名协议能通, V4 不行

## 原因

在 Nginx 配置中使用了: `proxy_set_header Host $host;`

## 解析

S3 的 V4 签名协议 对请求头极其敏感. 它在计算哈希时, 会将 Host 字段作为一个关键参数

`$host`: Nginx 的这个变量不包含端口号（如果是非标准端口）

`$http_host`: 这个变量会完整保留客户端请求中的 域名:端口

如果 S3 服务运行在非 80/443 端口（例如 9000）, 客户端生成的签名里包含端口, 在 Nginx 转发时被丢弃, 服务器端校验时就会认为请求被篡改, 直接拒绝访问

## 解决方案

修改 Nginx 配置文件中的 Host

```Nginx
# 使用 $http_host 保留原始端口信息
proxy_set_header Host $http_host;

# 建议同时关闭缓冲，提升大文件备份稳定性
proxy_request_buffering off;
proxy_buffering off;
```

### 补充

如果你是在反代 MiniO Console 后发现无法在 WebUI 操作桶内文件 请尝试检查这个问题