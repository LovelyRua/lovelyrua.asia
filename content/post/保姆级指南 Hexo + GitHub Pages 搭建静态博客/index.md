---
title: 保姆级指南 Hexo + GitHub Pages 搭建静态博客
description: 保姆级指南 Hexo + GitHub Pages 搭建静态博客
date: 2025-04-09 22:22:22+0800
# image: cover.jpg
categories:
    - IT-Techs
tags:
    - Blog
    - Pages
    - Hexo
---
> 》》 将心比心，换位思考；务实求索，无限进步 《《

### `##`前言`##`

**本文将基于实操使用 Hexo 静态博客框架, 并借助 GitHub Pages 和 Cloudflare CDN 服务, 搭建一个无需租用服务器, 无需支付任何费用即可访问的高性能个人博客站点. 本文旨在帮助读者了解以下内容:**

 * [静态网站](https://baike.baidu.com/item/%E9%9D%99%E6%80%81%E7%BD%91%E7%AB%99/2776875)的基本原理
 * [GitHub Pages](https://pages.github.com/) 的使用
 * [CDN](https://zh.wikipedia.org/zh-cn/%E5%85%A7%E5%AE%B9%E5%82%B3%E9%81%9E%E7%B6%B2%E8%B7%AF) 加速的优势
 * [Node.js](https://zh.wikipedia.org/wiki/Node.js) 基础操作

---

### ✨>_0 准备工作

**在开始之前, 请确保您的计算机已安装以下环境：**

* **Node.js 环境** ([官方下载地址](https://nodejs.org/)) : Hexo 基于 Node.js 运行, 您需要安装 Node.js.
* **Git** ([官方下载地址](https://git-scm.com/downloads)) : 用于将博客部署到 GitHub.
* **GitHub 账号** ([注册地址](https://github.com/join)) : 用于存储博客部署后的静态文件.

可选 :
* 自己的 **域名** 和 **Cloudflare 账号**, 用于后续配置 Cloudflare CDN 及个性化域名.

#### 创建 Github Pages 仓库
登录你的 Github 账号并访问:
https://github.com/new

在 **Repository name** 一框填入: **{你的用户名}.github.io** . 如 **`lovelyrua.github.io`**
点击 右下角绿色按钮 **`Create Repository`**

接下来便可跟随本文步骤指引搭建你的博客

---

## 安装 Hexo

> 本步骤参考 ([Hexo官方文档 - 建站](https://hexo.io/zh-cn/docs/setup))

选择一个合适目录储存项目, 这里创建一个 `hexo-blog` 文件夹为例:

```bash
  # 创建并进入项目目录; `&&` 表示前一个命令成功执行后执行后一个命令
  $ mkdir hexo-blog && cd hexo-blog

  # 在当前目录下使用 nodejs 提供的包管理工具 `npm` 来安装 hexo 命令行工具 (hexo-cli).
  # 如果想要在任何目录下都能直接使用 `hexo` 命令, 可以使用 `npm install hexo-cli -g` 进行全局安装 ➊
  $ npm install hexo-cli

  # 初始化 hexo 项目 'blog', hexo 会在当前目录下创建文件夹并从 github 拉取模板.
  # `npx` 是 npm 自带的包执行工具, 它会查找本地 'node_modules/.bin' 中的命令
  # 如果上一步骤进行了全局安装, 这里就可以直接执行 `hexo init blog`
  $ npx hexo init blog && cd blog

  # 安装项目依赖. 此命令会根据目录下的 'package.json' 文件中 "dependencies" 字块安装所需依赖
  $ npm install

  # 启动 hexo 测试服务端
  $ npx hexo server
  # `server` 也可直接缩写成 `s`

  # 以上操作若过程无误 控制台会输出:
  # `INFO  Hexo is running at http://localhost:4000/ .`
```

此时在浏览器访问 http://localhost:4000/ 即可看到 hexo 的初始主题页面.

_**如果启动或后续步骤出现类似** `Error: Cannot find module 'xxx'` 的报错, 说明依赖安装不完整
用 `npm install {报错提示缺少的依赖}` 命令**补充安装依赖即可**;
若还是无法解决 可以尝试删除 node_modules 文件夹和 package-lock.json 文件后重新运行 `npm install`_

➊ 全局安装: 
>! 当使用 `-g` 参数全局安装一个包时, npm 会将可执行文件链接到系统的 PATH 环境变量所包含的目录中 (如 Node.js 安装目录), 这样系统就能在任何位置识别 hexo 命令.
>! 如果全局安装后命令仍不可用, 可能需要重启终端或检查系统 PATH 环境变量配置.

---

## 创建第一篇文章

**作为一个静态博客框架, Hexo 不像 WordPress, QQ空间 或 新浪微博 那样提供在线编辑器让你随时随地编辑发布
它的工作流程更接近于开发者编写代码: `在本地创建文件` - `编辑内容` - `然后通过命令生成最终的网页文件并部署`**

> 本步骤参考 ([Hexo官方文档 - 写作](https://hexo.io/zh-cn/docs/writing))

#### 使用命令创建新文章:

确保终端位于 Hexo 项目的根目录 (即 `blog` 文件夹内). 然后执行以下命令:

```bash
  $ npx hexo new post "我的第一篇文章"
  # `new` 是创建命令, 'post' 是文章的布局 (layout), "我的第一篇文章" 是文章标题
  # 如果标题包含空格, 最好用引号括起来

  # 控制台会输出类似信息:
  # INFO  Created: ~/your/path/to/hexo-blog/blog/source/_posts/我的第一篇文章.md
```

>! 这个命令会在 source/_posts/ 目录下创建一个名为 "我的第一篇文章.md" 的 Markdown 文件.
>! `post` 是 Hexo 默认的文章布局, 你也可以创建其他布局 (如 `draft` 草稿, `page` 独立页面等).

#### 编辑文章内容:

使用你喜欢的文本编辑器 (如 `VS Code`, `Sublime Text`, `Obsidian` 等) 打开刚刚创建的 .md 文件
你会看到文件顶部有一段由 `---` 包裹的内容, 这叫做 _Front-matter_, 用于定义这篇文章的元数据:
```md
---
layout: post  # 文章布局, 自动根据命令生成
title: 我的第一篇文章  # 文章标题, 自动根据命令生成
date: 2025-04-09 20:15:00  # 文章创建时间, 自动生成
tags: [hello, markdown]  # 文章标签, 可以自行添加
---

# 这里是正文的开始
欢迎来到我的新博客!

这是我的第一篇文章，使用 **Markdown** 语法编写。

*   列表项1
*   列表项2
```
你可以在 --- 下方开始使用 [Markdown](https://zh.wikipedia.org/wiki/Markdown) 语法编写你的文章正文
可以修改 title, 添加 tags (多个标签用逗号分隔或使用 YAML 列表格式). date 通常保持自动生成的时间即可.

#### 本地预览:

编辑并保存好文章后回到终端 如果之前的 hexo server 还在运行, 可以先按 Ctrl+C 停止. 然后重新启动服务器进行预览：

```bash
# 清理之前构建的文件 (可选, 但修改配置或主题后需要执行)
npx hexo clean

# 生成静态文件 (可选, server 命令通常会自动处理)
npx hexo generate

# 启动本地服务器
npx hexo server
```

刷新浏览器中的 http://localhost:4000/, 你应该能在首页看到你新创建的文章 "我的第一篇文章" 的摘要或标题, 点击即可进入阅读.

***本节涉及的命令及其缩写:***
>! npm i == npm install
>! hexo n == hexo new
>! hexo g == hexo generate
>! hexo s == hexo server
>! hexo d == hexo deploy
>! hexo clean (无缩写)

---

## 将 Hexo 部署到 GitHub
现在你已经在本地创建并预览了你的第一篇文章. 下一步就是将博客发布到互联网上, 让其他人也能访问.

**静态博客的优势在于, 生成的网站文件无需复杂的服务器端处理;
只要将这些文件托管在任何可公开访问的 Web 服务器上, 站点就能运行.**

GitHub Pages 提供了一个免费的静态网站托管服务, 非常适合部署 Hexo 生成的静态博客. 
Hexo 提供了一键部署功能, 可以方便地将生成的静态文件推送到指定的 Git 仓库.
在此将演示 Hexo 提供的一键部署功能

> 本步骤参考 ([Hexo官方文档 - 一键部署](https://hexo.io/docs/one-command-deployment))

#### 安装 Git 部署插件:

要使用一键部署, 首先需要根据自己要部署到的平台安装对应的 deployer 插件
安装针对 GitHub (或其他 Git 仓库) 的部署插件 `hexo-deployer-git`:

```
  $ npm install hexo-deployer-git
```
#### 配置 _config.yml:
在部署前需要编辑 Hexo 项目的设置
在 `_config.yml`内填入必要的信息:

```
# 找到文件末尾的 `deploy` 配置项 
  deploy:
     type: git
     repo: https://github.com/用户名/用户名.github.io.git  # 修改仓库地址
     # ！！极其重要：替换成你自己的仓库地址！！
     # 例如: https://github.com/LovelyRua/lovelyrua.github.io.git
     branch: main  # GitHub Pages 仓库默认分支名
     # message: "Site updated: {{ now('YYYY-MM-DD HH:mm:ss') }}"  # 可选, 自定义 commit 信息
```

>! type: 一键部署方式的类型, 这里必须是 git
>!
>! repo: 务必替换成你自己的 GitHub Pages 仓库地址. 这个仓库通常命名为 `{你的GitHub用户名}.github.io`. 可以使用 HTTPS 或 SSH 格式的地址.
>! 
>! branch: 指定要将静态文件推送到哪个分支. 对于 `用户名.github.io` 这种仓库, 通常是 `main` 或 `master` 分支作为 GitHub Pages 的源. 具体依赖于你的仓库设置.

保存文件后 执行部署命令

```
  $ npx hexo deploy
  # 这个命令会先执行 `hexo generate` (生成静态文件到 public 目录)
  # 然后将 public 目录的内容推送到你配置的 repo 和 branch。
```

**首次部署可能遇到的情况:**

**Git 身份未配置**: 如果你之前没有在本机配置过 Git 的用户名和邮箱, 可能会看到类似以下的提示:

```ps
  PS X:\your\path\hexo-blog\blog> npx hexo deploy
	INFO  Validating config
	INFO  Deploying: git
	INFO  Setting up Git deployment...
	Initialized empty Git repository in X:/your/path/hexo-blog/blog/.deploy_git/.git/
	Author identity unknown
	
	*** Please tell me who you are.
	
	Run
	
	git config --global user.email "you@example.com"
	git config --global user.name "Your Name"
	
	to set your account's default identity.
	Omit --global to set the identity only in this repository.
	
	...
```

这是因为 Git 必须提供提交时必要的身份信息
**如上情况只需按照提示 通过这两条命令设置 Git 的默认身份信息:**

```
  $ git config --global user.email "{你的邮箱}"
  $ git config --global user.name "{你的名字}"
```

设置完成后再重新执行 `npx hexo deploy`
验证部署结果:

```
  INFO  Deploy done: git
```

**当终端输出如上日志即说明部署成功了**
访问你的 Github Pages 仓库, 即可看到编译出的网页静态文件.
***稍等片刻 GitHub Pages 需要一点时间来更新***

### 🎉 恭喜你!
至此步骤在浏览器打开 `https://你的用户名.github.io/`
就可以访问你刚刚部署到 Github Pages 的博客了

***如需编写新文章 只需重复从 **步骤1.2** 开始的操作***

---

## 通过 Cloudflare CDN 加速国内访问

>! **先说结论：Cloudflare 免费版不等于中国大陆 CDN。**
>! 免费版会使用 Cloudflare 全球网络，能提供代理、缓存和 HTTPS 等功能，但访客不一定会连接到中国大陆节点。
>! 不同地区、运营商和时段的线路表现也可能不同，配置后应以实际测试为准。
>! Cloudflare 官方的中国网络是 Enterprise 方案的额外付费服务，并且要求域名完成 ICP 备案。

如果你已经有自己的域名，可以把域名接入 Cloudflare，再由 Cloudflare 反向代理到 GitHub Pages。
访问路径会变为：

`访客 -> Cloudflare -> 你的用户名.github.io`

以下使用 `example.com` 作为示例域名，实际配置时请替换成你自己的域名。

### 将域名接入 Cloudflare

注册并登录 [Cloudflare](https://dash.cloudflare.com/)，点击 **Add a domain**，输入你的根域名，例如 `example.com`。

选择 Free 套餐后，Cloudflare 会分配两条 Nameserver 地址。前往购买域名的平台，将域名原有的 Nameserver 替换成 Cloudflare 提供的地址。

Nameserver 修改后不会立即生效。等待 Cloudflare 控制台中的域名状态变为 **Active**，再继续配置。

### 配置自定义域名

先在 Hexo 项目的 `source` 目录创建一个名为 `CNAME` 的文件，文件中只填写要使用的完整域名：

```text
www.example.com
```

`source/CNAME` 会在 Hexo 构建时被复制到 `public` 目录，这样每次执行 `hexo deploy` 时都不会丢失 GitHub Pages 的域名配置。

然后重新部署一次：

```bash
  $ npx hexo clean
  $ npx hexo deploy
```

打开 GitHub Pages 仓库，进入 **Settings -> Pages**，在 **Custom domain** 中填写同一个域名，例如 `www.example.com`，然后点击 **Save**。

>! 建议先在 GitHub 中保存自定义域名，再配置公开的 DNS 解析，以免域名被其他 GitHub Pages 仓库抢先绑定。

### 添加 DNS 记录

回到 Cloudflare 控制台，进入 **DNS -> Records**，为 `www` 添加一条记录：

| Type | Name | Target | Proxy status |
| --- | --- | --- | --- |
| CNAME | `www` | `你的用户名.github.io` | DNS only |

例如 GitHub 用户名为 `lovelyrua`，Target 就填写 `lovelyrua.github.io`。
不要在 Target 后面附加仓库名、`https://` 或路径。

如果还希望通过根域名 `example.com` 访问博客，可以再添加以下四条 A 记录：

| Type | Name | IPv4 address | Proxy status |
| --- | --- | --- | --- |
| A | `@` | `185.199.108.153` | DNS only |
| A | `@` | `185.199.109.153` | DNS only |
| A | `@` | `185.199.110.153` | DNS only |
| A | `@` | `185.199.111.153` | DNS only |

这里先保持灰色云朵，即 **DNS only**。等待 GitHub Pages 检测 DNS 配置并签发 HTTPS 证书后，再开启 Cloudflare 代理，可以减少证书验证失败或配置排查困难的情况。

DNS 记录可能需要一段时间才能完全生效。回到 GitHub 的 **Settings -> Pages**，确认域名检查通过，并勾选 **Enforce HTTPS**。

### 开启 Cloudflare 代理和 HTTPS

GitHub Pages 已经可以通过自定义域名正常使用 HTTPS 后，返回 Cloudflare 的 DNS 页面，将博客域名对应记录的 **Proxy status** 改为 **Proxied**，也就是橙色云朵。

然后进入 **SSL/TLS -> Overview**：

* 推荐选择 **Full (strict)**，Cloudflare 到 GitHub Pages 的连接也会验证 HTTPS 证书。
* 不要选择 **Flexible**，否则可能出现重定向循环，且 Cloudflare 到源站之间不会得到完整的 HTTPS 保护。

再进入 **SSL/TLS -> Edge Certificates**，开启 **Always Use HTTPS**，将 HTTP 请求统一跳转到 HTTPS。

### 缓存设置

启用橙色云朵后，Cloudflare 默认会缓存图片、CSS、JavaScript 和字体等静态资源，通常不需要额外配置。
HTML 页面默认不会被缓存，这对刚开始使用的博客反而更省心：文章部署后不容易因为旧缓存而看不到更新。

如果后续确实要缓存 HTML，可以在 **Caching -> Cache Rules** 中单独创建规则。但这样做以后，每次发布文章都要考虑缓存刷新，因此不建议一开始就使用 “Cache Everything”。

### CDN 优选（可选）

Cloudflare 默认使用 Anycast：同一个 IP 会在多个数据中心广播，再由网络路由决定访客连接到哪里。
所谓“CDN 优选”，就是在当前网络中测试 Cloudflare 的多个 Anycast IP，再让域名解析到延迟、丢包和下载速度表现较好的 IP。

>! CDN 优选不是 Cloudflare 官方提供的功能，也不能把免费版变成中国大陆 CDN。
>! 它只是在某些运营商线路上绕过不理想的自动选路，结果具有地区性和时效性。
>! 一个在电信宽带上表现良好的 IP，在移动网络或其他省份可能更慢。

#### 测试优选 IP

可以使用开源工具 [CloudflareSpeedTest](https://github.com/XIU2/CloudflareSpeedTest) 测试当前网络到 Cloudflare IP 的延迟、丢包和下载速度。
从项目的 Releases 页面下载与你的系统对应的版本，解压后运行：

```ps
  PS X:\your\path\cfst> .\cfst.exe
```

工具会先筛选延迟和丢包，再对候选 IP 进行下载测速，最终结果保存在 `result.csv`。
不要只看最低延迟，应优先选择 **无丢包、下载速度稳定** 的结果。

测速时还要注意：

* 关闭代理软件，否则测到的可能是代理服务器到 Cloudflare 的线路。
* 尽量在博客主要读者使用的网络上测试，而不是只在服务器上测试。
* 分别在白天和晚高峰测试几次，不要根据一次结果就修改解析。
* 测速会产生大量连接，请勿长时间、高并发地反复扫描。

#### 先在本机验证

不要立即修改正式域名。可以先用 `curl --resolve` 临时指定连接 IP：

```bash
  $ curl -I --resolve www.example.com:443:优选IP https://www.example.com/
```

例如：

```bash
  $ curl -I --resolve www.example.com:443:104.16.0.1 https://www.example.com/
```

这里的 IP 仅用于演示，不代表它适合你的线路。命令中的域名仍然是 `www.example.com`，
因此 HTTPS 的 SNI 和 HTTP Host 不会被改成 IP。响应正常后，再用浏览器或多次下载测试确认效果。

#### 将域名解析到优选 IP

确认优选 IP 可用后，可以在 Cloudflare 的 **DNS -> Records** 中将博客域名改为 A 记录：

| Type | Name | IPv4 address | Proxy status |
| --- | --- | --- | --- |
| A | `www` | `测试得到的优选 IP` | DNS only |

这里必须使用 **DNS only（灰色云朵）**。如果重新打开 **Proxied（橙色云朵）**，
Cloudflare 会再次返回自动分配的 Anycast IP，手动指定的优选 IP 也就失去作用。

修改后可以查询解析结果：

```bash
  $ nslookup www.example.com
```

返回地址应当是刚刚填写的优选 IP。随后检查网站 HTTPS、页面、图片和静态资源是否都能正常加载。

这种配置虽然显示为灰色云朵，但请求仍会到达 Cloudflare 的网络，再由请求中的域名转发到 GitHub Pages。
不过它不属于 Cloudflare 官方承诺的标准接入方式，可能遇到证书、IP 调整、路由变化或 Cloudflare 策略变更。
如果出现 SSL 错误、`1000`/`1001` 错误或网站无法访问，应立即恢复前文的标准配置：

| Type | Name | Target | Proxy status |
| --- | --- | --- | --- |
| CNAME | `www` | `你的用户名.github.io` | Proxied |

#### 使用第三方优选域名

网上也有维护“优选域名”的公共服务，使用时通常把 `www` 的 CNAME 指向对方提供的域名，并保持 **DNS only**。
这类服务会替你更换解析 IP，配置更省事，但域名解析结果由第三方控制；服务停止、被污染或改错记录时，你的网站也会受影响。

因此不建议在没有审计服务来源的情况下，把主域名直接交给公共优选域名。
如果一定要使用，最好先用单独的测试子域名，例如 `cf-test.example.com`，观察一段时间后再决定是否切换。

对个人博客来说，优选配置还需要定期维护。建议保留原始 CNAME 配置，并每隔一段时间从不同运营商重新测试；
当优选线路的实际提升不明显时，使用 Cloudflare 标准橙云代理通常更加稳定。

### 验证是否生效

打开博客并确认以下项目：

* 浏览器地址栏使用的是你的自定义域名，并且 HTTPS 证书正常。
* GitHub Pages 的 **Custom domain** 和 Hexo 的 `source/CNAME` 内容一致。
* Cloudflare DNS 页面中的博客记录已经变为橙色云朵。
* 浏览器开发者工具的 Network 面板中，请求响应头出现 `server: cloudflare`；静态资源还可能出现 `cf-cache-status: HIT`。

第一次访问某个资源时，`cf-cache-status` 可能为 `MISS`，表示 Cloudflare 尚未缓存；再次访问后才可能变成 `HIT`。

如果开启代理后出现 `526` 错误，先把 DNS 记录改回 **DNS only**，确认 GitHub Pages 已成功签发自定义域名证书，再重新开启代理。
如果网站出现反复跳转，检查 Cloudflare SSL/TLS 模式是否误设成了 **Flexible**。

至此 Cloudflare 已接入完成。它能隐藏 GitHub Pages 的直接访问地址、提供静态资源缓存和统一的 HTTPS 配置，但对于中国大陆访问速度是否有提升，仍应使用不同运营商网络实际测试，不能只看 Cloudflare 控制台中的状态判断。
