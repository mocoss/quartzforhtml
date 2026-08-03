---
title: Quartz静态站点搭建与发布完整教程
description: 从Quartz搭建静态HTML，到Nginx/Caddy发布，再到阿里云ESA解决80/443端口封禁的完整操作指南
tags:
  - Quartz
  - Nginx
  - Caddy
  - 阿里云ESA
  - 静态站点
  - Obsidian
date: 2026-07-29
---

# Quartz静态站点搭建与发布完整教程

## 目录

- [[#一、Quartz 静态站点搭建]]
  - [[#1.1 环境准备]]
  - [[#1.2 下载并初始化 Quartz]]
  - [[#1.3 本地预览与构建]]
  - [[#1.4 Obsidian 配置优化]]
  - [[#1.5 内容结构规划]]
- [[#二、Nginx 部署静态站点]]
  - [[#2.1 Nginx 安装]]
  - [[#2.2 静态站点配置]]
  - [[#2.3 性能优化配置]]
  - [[#2.4 HTTPS 配置]]
- [[#三、Caddy 部署静态站点]]
  - [[#3.1 Caddy 安装]]
  - [[#3.2 Caddyfile 基础配置]]
  - [[#3.3 高级配置与优化]]
  - [[#3.4 运维管理命令]]
- [[#四、阿里云 ESA 解决端口封禁]]
  - [[#4.1 原理说明]]
  - [[#4.2 开通 ESA 服务]]
  - [[#4.3 添加站点配置]]
  - [[#4.4 DNS 解析配置]]
  - [[#4.5 回源规则配置]]
  - [[#4.6 SSL 证书配置]]
  - [[#4.7 验证与测试]]
- [[#五、完整部署流程汇总]]
- [[#六、常见问题与排错]]

---

## 一、Quartz 静态站点搭建

### 1.1 环境准备

Quartz 是基于 Node.js 的静态站点生成器，专为 Obsidian 笔记发布设计，原生支持双链语法。

**必需环境：**
- Git：版本控制与代码同步
- Node.js：运行 Quartz 构建
- npm：包管理工具（Node.js 自带）
- Obsidian：笔记编辑

**安装 Git：**
```bash
# Ubuntu/Debian
sudo apt update && sudo apt install git -y

# 配置 Git
git config --global user.name "你的用户名"
git config --global user.email "你的邮箱"

# 验证安装
git --version
```

**安装 Node.js（推荐 LTS 版本）：**
```bash
# Ubuntu/Debian 使用 NodeSource 仓库
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs

# 验证安装
node -v
npm -v
```

### 1.2 下载并初始化 Quartz

**克隆 Quartz 仓库：**
```bash
# 创建项目目录并克隆
git clone https://github.com/jackyzha0/quartz.git my-quartz-site
cd my-quartz-site
```

> 💡 **网络问题解决**：如果克隆失败，可尝试设置 Git 使用 HTTP/1.1
> ```bash
> git config --global http.version HTTP/1.1
> git clone --depth=1 https://github.com/jackyzha0/quartz.git my-quartz-site
> ```

**安装依赖：**
```bash
npm i
```

**初始化 Quartz：**
```bash
npx quartz create
```

初始化选项说明：
1. **Choose how to initialize the content** → 选择 `Empty Quartz`（空项目）
2. **Choose how Quartz should resolve links in your content** → 选择 `Treat links as shortest path`（适配 Obsidian 双链）
3. 其他选项保持默认，按回车即可

### 1.3 本地预览与构建

**本地开发预览：**
```bash
npx quartz build --serve
```

启动成功后会显示：
```
Started a Quartz server listening at http://localhost:8080
```

浏览器访问 `http://localhost:8080` 即可预览站点。

> ⚠️ 停止服务：在终端按 `Ctrl + C`

**生产环境构建：**
```bash
npx quartz build
```

构建完成后，静态文件输出到 `public/` 目录，这就是我们需要部署的内容。

### 1.4 Obsidian 配置优化

**打开项目文件夹：**
在 Obsidian 中选择「打开本地文件夹作为仓库」，选择整个 `my-quartz-site` 目录（不要只选 content）。

**隐藏不必要的文件：**
进入 `设置 → 文件与链接 → 排除文件`，添加以下内容：
```
node_modules/
quartz/
public/
.github/
.git/
package.json
package-lock.json
quartz.config.ts
quartz.layout.ts
tsconfig.json
```

**设置图片保存位置：**
进入 `设置 → 文件与链接`，找到「附件默认存放位置」：
- 选择：`当前文件所在文件夹下的指定子文件夹`
- 子文件夹名称：`assets`

**设置新建笔记默认位置：**
进入 `设置 → 文件与链接`，找到「新建笔记的存放位置」：
- 选择：`指定的文件夹`
- 文件夹：`content`

### 1.5 内容结构规划

**推荐目录结构：**
```
content/
├── index.md              # 首页
├── 教程/
│   ├── index.md          # 分类首页
│   ├── Quartz搭建指南.md
│   └── assets/           # 图片资源
├── 运维/
│   ├── index.md
│   ├── Nginx配置详解.md
│   └── assets/
└── 工具/
    ├── index.md
    └── ...
```

**首页示例（content/index.md）：**
```markdown
---
title: 首页
description: 我的技术笔记与知识花园
---

# 欢迎来到我的知识花园

这里是我的公开笔记博客，记录技术学习与实践总结。

## 内容分类

- [[教程/index|教程笔记]]
- [[运维/index|运维实践]]
- [[工具/index|工具推荐]]

## 最近更新

- [[教程/Quartz搭建指南]]
- [[运维/Nginx配置详解]]
```

**分类首页示例（content/教程/index.md）：**
```markdown
---
title: 教程笔记
description: 各类技术教程与操作指南
tags:
  - 教程
---

# 教程笔记

这里整理各类技术教程与详细操作指南。

## 站点搭建

- [[Quartz搭建指南]]
- [[Obsidian使用技巧]]

## 服务器运维

- [[Nginx配置详解]]
- [[Caddy快速入门]]
```

---

## 二、Nginx 部署静态站点

### 2.1 Nginx 安装

**Ubuntu/Debian 安装：**
```bash
sudo apt update && sudo apt install nginx -y

# 启动并设置开机自启
sudo systemctl enable --now nginx

# 验证安装
curl -I http://localhost
```

**CentOS/RHEL 安装：**
```bash
sudo yum install nginx -y
sudo systemctl enable --now nginx
```

**Nginx 核心目录：**
| 路径 | 说明 |
|------|------|
| `/etc/nginx/nginx.conf` | 主配置文件 |
| `/etc/nginx/conf.d/*.conf` | 站点配置目录 |
| `/etc/nginx/sites-available/` | 站点配置存放（Ubuntu） |
| `/etc/nginx/sites-enabled/` | 已激活站点（软链接） |
| `/var/log/nginx/access.log` | 访问日志 |
| `/var/log/nginx/error.log` | 错误日志 |

### 2.2 静态站点配置

**创建站点根目录：**
```bash
# 创建网站目录
sudo mkdir -p /var/www/quartz-site

# 上传 Quartz 构建产物（public 目录内容）
# 假设本地构建好的 public 目录在 /tmp/public
sudo cp -r /tmp/public/* /var/www/quartz-site/

# 设置权限
sudo chown -R www-data:www-data /var/www/quartz-site
```

**创建 Nginx 配置文件：**
```bash
sudo nano /etc/nginx/conf.d/quartz-site.conf
```

**基础配置（HTTP）：**
```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;

    root /var/www/quartz-site;
    index index.html;

    # 日志
    access_log /var/log/nginx/quartz-access.log;
    error_log /var/log/nginx/quartz-error.log warn;

    # 静态文件处理
    location / {
        try_files $uri $uri/ =404;
    }

    # 静态资源缓存
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff2|svg|woff|ttf)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # 禁止访问隐藏文件
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }
}
```

**验证并生效配置：**
```bash
# 检查配置语法
sudo nginx -t

# 重载配置
sudo systemctl reload nginx
```

### 2.3 性能优化配置

**全局优化（/etc/nginx/nginx.conf）：**
```nginx
worker_processes auto;
worker_rlimit_nofile 65535;

events {
    worker_connections 4096;
    multi_accept on;
    use epoll;
}

http {
    server_tokens off;

    # 连接优化
    keepalive_timeout 65;
    keepalive_requests 1000;

    # Gzip 压缩
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_min_length 1000;
    gzip_types
        text/plain text/css text/xml text/javascript
        application/json application/javascript application/xml
        application/x-font-ttf font/opentype image/svg+xml;

    # 文件缓存
    open_file_cache max=10000 inactive=30s;
    open_file_cache_valid 60s;
    open_file_cache_min_uses 2;

    include /etc/nginx/conf.d/*.conf;
}
```

### 2.4 HTTPS 配置

> 💡 如果使用阿里云 ESA，HTTPS 可由 ESA 边缘节点处理，源站可只用 HTTP。如需源站直接 HTTPS，可参考以下配置。

**使用 Certbot 申请证书：**
```bash
# 安装 Certbot
sudo apt install certbot python3-certbot-nginx -y

# 申请并自动配置证书
sudo certbot --nginx -d example.com -d www.example.com
```

**手动 HTTPS 配置：**
```nginx
# HTTP 重定向到 HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}

# HTTPS 站点
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name example.com www.example.com;

    root /var/www/quartz-site;
    index index.html;

    # SSL 证书路径
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # SSL 安全配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;

    # 其他配置同 HTTP 版本...
}
```

---

## 三、Caddy 部署静态站点

### 3.1 Caddy 安装

Caddy 是一款现代化 Web 服务器，最大特点是**自动 HTTPS**，配置极简。

**Ubuntu/Debian 官方 APT 安装（推荐）：**
```bash
# 安装依赖
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl

# 添加 GPG 密钥
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg

# 添加 APT 源
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list

# 安装 Caddy
sudo apt update && sudo apt install caddy -y

# 验证安装
caddy version
systemctl status caddy
```

**一键脚本安装：**
```bash
curl -fsSL https://get.caddyserver.com | bash -s personal
```

**Docker 部署：**
```bash
docker pull caddy:latest

docker run -d -p 80:80 -p 443:443 \
  -v /path/to/Caddyfile:/etc/caddy/Caddyfile \
  -v /path/to/site:/var/www \
  -v caddy_data:/data \
  -v caddy_config:/config \
  --name mycaddy caddy
```

### 3.2 Caddyfile 基础配置

Caddy 的配置文件称为 Caddyfile，语法简洁直观。

**配置文件位置：** `/etc/caddy/Caddyfile`

**最简静态站点配置（3 行搞定）：**
```caddyfile
example.com {
    root * /var/www/quartz-site
    file_server
}
```

> ✨ **自动 HTTPS**：只要域名已解析到服务器，Caddy 会自动：
> 1. 向 Let's Encrypt 申请证书
> 2. 配置 HTTPS 加密
> 3. 设置 HTTP → HTTPS 跳转
> 4. 证书到期前自动续期

**完整静态站点配置：**
```caddyfile
example.com {
    # 网站根目录
    root * /var/www/quartz-site

    # 启用静态文件服务
    file_server

    # 启用 Gzip 压缩
    encode gzip zstd

    # 安全响应头
    header {
        X-Content-Type-Options "nosniff"
        X-Frame-Options "DENY"
        Strict-Transport-Security "max-age=31536000; includeSubDomains"
        Referrer-Policy "strict-origin-when-cross-origin"
        -Server
    }

    # 静态资源缓存
    @static {
        path *.jpg *.jpeg *.png *.gif *.ico *.css *.js *.woff2 *.svg *.woff *.ttf
    }
    header @static Cache-Control "public, max-age=2592000, immutable"

    # 访问日志
    log {
        output file /var/log/caddy/quartz-access.log {
            roll_size 100mb
            roll_keep 5
        }
        format console
    }
}
```

**多站点配置：**
```caddyfile
# 主站
example.com {
    root * /var/www/main-site
    file_server
}

# 博客子域名
blog.example.com {
    root * /var/www/quartz-site
    file_server
    encode gzip
}

# API 反向代理
api.example.com {
    reverse_proxy localhost:3000
}
```

### 3.3 高级配置与优化

**自定义端口（端口被封时使用）：**
```caddyfile
# 监听自定义端口，配合 ESA 使用
:8080 {
    root * /var/www/quartz-site
    file_server
    encode gzip
}
```

**同时监听 HTTP 和 HTTPS 自定义端口：**
```caddyfile
http://example.com:8080, https://example.com:8443 {
    root * /var/www/quartz-site
    file_server
    encode gzip

    # 使用自定义证书（如果 ESA 回源用 HTTPS）
    tls /path/to/cert.pem /path/to/key.pem
}
```

**启用 HTTP/3：**
```caddyfile
{
    servers {
        protocol {
            experimental_http3
        }
    }
}

example.com {
    root * /var/www/quartz-site
    file_server
}
```

### 3.4 运维管理命令

**常用 systemd 命令：**
```bash
# 启动
sudo systemctl start caddy

# 停止
sudo systemctl stop caddy

# 重启
sudo systemctl restart caddy

# 重载配置（零停机，推荐）
sudo systemctl reload caddy

# 查看状态
sudo systemctl status caddy

# 设置开机自启
sudo systemctl enable caddy
```

**配置验证与管理：**
```bash
# 验证配置文件语法
caddy validate --config /etc/caddy/Caddyfile

# 查看 Caddy 版本
caddy version

# 查看当前配置
caddy adapt --config /etc/caddy/Caddyfile
```

---

## 四、阿里云 ESA 解决端口封禁

### 4.1 原理说明

**问题背景：**
家庭宽带或部分服务器的 80/443 端口被运营商封禁，导致无法使用标准端口访问网站，只能通过 `域名:端口号` 的方式访问，体验不佳。

**ESA 解决方案原理：**
```
用户访问 https://blog.example.com (标准443端口)
        ↓
阿里云 ESA 边缘节点接收请求 (80/443端口)
        ↓
ESA 向源站请求数据 (可使用任意端口，如 8443)
        ↓
将数据返回给用户
```

**核心优势：**
- ✅ 用户使用标准 80/443 端口访问，无感知
- ✅ 源站可使用任意端口，绕过运营商封禁
- ✅ 自动 HTTPS 证书管理
- ✅ 全球边缘加速，访问更快
- ✅ 自带 DDoS 防护和 WAF

### 4.2 开通 ESA 服务

**步骤 1：登录阿里云控制台**
访问 [阿里云官网](https://www.aliyun.com/)，搜索「边缘安全加速 ESA」进入产品页面。

**步骤 2：购买 ESA 服务**
- 个人用户可选择免费版或基础版
- 新用户通常有优惠活动
- 按量付费，成本可控

**步骤 3：进入 ESA 控制台**
购买成功后，在控制台找到「边缘安全加速 ESA」，进入管理界面。

### 4.3 添加站点配置

**步骤 1：开始添加站点**
在左侧菜单点击「站点管理」→ 点击「添加站点」。

**步骤 2：输入站点信息**
- **站点名称**：填写你的主域名，如 `example.com`

**步骤 3：选择区域（关键！）**

| 选项 | 备案要求 | 适用场景 |
|------|----------|----------|
| 中国内地 | **必须备案** | 主要服务国内用户 |
| 全球 | **必须备案** | 服务全球用户 |
| **全球（不包含中国内地）** | **无需备案** | **个人用户首选** |

> ⚠️ **重要提示**：个人用户没有备案的话，一定要选择「**全球（不包含中国内地）**」！

**步骤 4：选择接入方式**
- **CNAME（推荐）**：传统 CDN 模式，在域名 DNS 处添加 CNAME 记录，适合大多数用户
- **NS**：将域名 DNS 服务器改为阿里云 ESA，适合需要完整 DNS 管理的用户

**步骤 5：完成添加**
点击「下一步」完成站点添加。

### 4.4 DNS 解析配置

**步骤 1：进入 DNS 设置**
左侧菜单点击「DNS」→「记录」→ 点击「编辑记录」。

**步骤 2：添加 CNAME 记录**

| 配置项 | 说明 | 示例值 |
|--------|------|--------|
| 记录类型 | 选择 CNAME | CNAME |
| 主机记录 | 你想要的二级域名前缀 | `blog` |
| 代理状态 | **必须开启**（橙色云朵图标） | 开启 ✅ |
| 记录值/源站 | 你的实际源站域名 | `origin.example.com` |
| 回源 HOST | 跟随源站域名 | 跟随源站域名 |
| TTL | 保持自动即可 | 自动 |

**配置示例：**
```
记录类型：CNAME
主机记录：blog
代理状态：开启（代理加速）
记录值/源站：origin.example.com
回源 HOST：跟随源站域名
TTL：自动
```

> 💡 **说明**：
> - `blog.example.com` → 用户访问的域名（通过 ESA 加速）
> - `origin.example.com` → 实际源站域名（指向你的服务器 IP，可用非标准端口）

**步骤 3：在域名服务商添加 CNAME**
1. 按照 ESA 提示，在你的域名 DNS 管理处添加 CNAME 记录
2. 主机记录：`blog`
3. 记录值：ESA 提供的 CNAME 地址

**步骤 4：验证生效**
- 点击「查询」按钮检查 CNAME 记录是否生效
- 生效后点击「我已完成 CNAME 配置」

### 4.5 回源规则配置

这是解决端口封禁的**核心配置**！

**步骤 1：进入回源规则**
左侧菜单点击「规则」→「回源规则」→ 点击「新增规则」。

**步骤 2：配置传入请求类型**
- 选择「自定义规则」
- 条件：主机名 等于 你的加速域名（如 `blog.example.com`）

**步骤 3：配置回源参数**

| 配置项 | 说明 | 推荐值 |
|--------|------|--------|
| 回源 HOST | 回源请求的 HOST 值 | `$host` 或源站域名 |
| 回源协议 | ESA 回源使用的协议 | HTTP 或 HTTPS |
| 回源端口 | 源站实际监听的端口 | **你的自定义端口** |

**配置示例（源站 HTTP，端口 8080）：**
```
传入请求类型：自定义规则
主机名：等于 blog.example.com

回源 HOST：blog.example.com
回源协议：HTTP
回源端口：8080
```

**配置示例（源站 HTTPS，端口 8443）：**
```
回源 HOST：blog.example.com
回源协议：HTTPS
回源端口：8443
```

> 🎯 **关键理解**：
> - 用户访问：`https://blog.example.com`（标准 443 端口）
> - ESA 回源：`http://origin.example.com:8080` 或 `https://origin.example.com:8443`
> - 用户完全感知不到端口差异

### 4.6 SSL 证书配置

**步骤 1：进入证书管理**
左侧菜单点击「SSL/TLS」→「边缘证书」→「概览」。

**步骤 2：申请免费证书**
- 证书颁发机构：`Let's Encrypt（无 SLA）`
- 证书域名：你的加速域名，如 `blog.example.com`
- 支持通配符证书：`*.example.com`

> 💡 每个站点有免费证书配额（通常 5 个），个人用户完全够用。

**步骤 3：自动部署**
申请成功后，证书会自动部署到 ESA 边缘节点：
- ✅ 自动续签
- ✅ 全自动部署
- ✅ 支持多域名

**步骤 4：SSL/TLS 模式设置**
- **灵活（Flexible）**：用户到 ESA 用 HTTPS，ESA 到源站用 HTTP（推荐，源站无需证书）
- **完整（Full）**：用户到 ESA 用 HTTPS，ESA 到源站也用 HTTPS（源站需有证书）
- **完整（严格）**：同完整，但要求源站证书有效

> 💡 **推荐选择「灵活」模式**：源站只需 HTTP 即可，由 ESA 负责 HTTPS 加密，最简单。

### 4.7 验证与测试

**步骤 1：访问测试**
等待几分钟（DNS 生效和证书部署需要时间），然后访问：
```
https://blog.example.com
```

**步骤 2：检查 HTTPS**
- 浏览器地址栏显示 🔒 锁图标
- 点击锁图标查看证书信息，确认是有效证书

**步骤 3：验证端口**
- 用户访问的是标准 443 端口（URL 中不显示端口）
- 源站实际使用的是自定义端口（如 8080/8443）

**步骤 4：速度测试**
- ESA 有全球边缘节点，访问速度应该比直接访问源站更快
- 可以用 ping 或测速工具对比

---

## 五、完整部署流程汇总

### 5.1 整体架构图

```
Obsidian 笔记
    ↓ (npx quartz build)
Quartz 静态站点 (public/)
    ↓ (上传到服务器)
Nginx / Caddy Web 服务器 (监听 8080 端口)
    ↓ (回源)
阿里云 ESA 边缘节点 (监听 80/443 端口)
    ↓
用户访问 (标准端口，HTTPS 加密)
```

### 5.2 快速部署 Checklist

- [ ] 本地安装 Node.js 和 Git
- [ ] 克隆并初始化 Quartz 项目
- [ ] 迁移 Obsidian 笔记到 content 目录
- [ ] 本地预览并调整配置
- [ ] 执行 `npx quartz build` 生成静态文件
- [ ] 购买云服务器（或使用家庭服务器）
- [ ] 安装 Nginx 或 Caddy
- [ ] 配置 Web 服务器监听自定义端口（如 8080）
- [ ] 上传 public 目录内容到服务器
- [ ] 注册域名并添加源站 DNS 解析
- [ ] 开通阿里云 ESA 服务
- [ ] 添加站点（选择全球不含中国内地）
- [ ] 配置 DNS CNAME 记录并开启代理
- [ ] 配置回源规则（设置自定义端口）
- [ ] 申请并部署 SSL 证书
- [ ] 测试访问，确认正常

### 5.3 自动化部署脚本示例

**部署脚本（deploy.sh）：**
```bash
#!/bin/bash

# 配置
SERVER="user@your-server-ip"
REMOTE_PATH="/var/www/quartz-site"
LOCAL_PUBLIC="./public"

echo "🔨 开始构建 Quartz 站点..."
npx quartz build

if [ $? -eq 0 ]; then
    echo "✅ 构建成功"
    echo "📤 正在上传到服务器..."
    
    # 使用 rsync 同步（增量上传，更快）
    rsync -avz --delete $LOCAL_PUBLIC/ $SERVER:$REMOTE_PATH/
    
    if [ $? -eq 0 ]; then
        echo "✅ 部署完成！"
        echo "🌐 访问地址：https://blog.example.com"
    else
        echo "❌ 上传失败"
    fi
else
    echo "❌ 构建失败"
fi
```

---

## 六、常见问题与排错

### 6.1 Quartz 相关问题

**Q: npx quartz build 报错怎么办？**
- 检查 Node.js 版本，建议使用 LTS 版本
- 删除 node_modules 重新安装：`rm -rf node_modules && npm i`
- 检查 content 目录下是否有语法错误的 Markdown 文件

**Q: 图片不显示怎么办？**
- 确认图片路径使用相对路径
- 检查图片是否在正确的 assets 目录中
- Obsidian 中设置附件默认存放位置为「当前文件所在文件夹下的指定子文件夹」

**Q: 双链链接失效怎么办？**
- 确保初始化时选择了 `Treat links as shortest path`
- 检查文件名是否正确，注意大小写
- 避免文件名中有特殊字符

### 6.2 Nginx/Caddy 相关问题

**Q: 访问显示 403 Forbidden？**
- 检查网站目录权限：`sudo chown -R www-data:www-data /var/www/quartz-site`
- 确认目录有执行权限：`chmod +x /var/www/quartz-site`
- 检查 Nginx/Caddy 配置中的 root 路径是否正确

**Q: 静态资源加载失败？**
- 检查文件路径是否正确
- 查看错误日志：`/var/log/nginx/error.log`
- 确认文件已正确上传到服务器

**Q: Caddy 自动 HTTPS 失败？**
- 确认 80 和 443 端口已开放（防火墙/安全组）
- 确认域名已正确解析到服务器 IP
- 检查 DNS 解析是否生效：`dig example.com`

### 6.3 阿里云 ESA 相关问题

**Q: 添加站点提示域名未备案？**
- 选择「全球（不包含中国内地）」区域，无需备案
- 如果选择了中国内地或全球区域，必须备案

**Q: 证书申请一直 pending？**
- 检查 DNS 解析是否生效
- 确认 CNAME 记录配置正确
- 等待 TTL 时间，通常几分钟到几小时

**Q: 访问显示 502 或源站错误？**
- 检查回源规则配置，协议和端口是否正确
- 确认源站服务正常运行
- 检查源站防火墙/安全组是否放行 ESA 回源 IP
- 查看 ESA 日志获取详细错误信息

**Q: 回源 HOST 怎么设置？**
- 如果源站是基于域名的虚拟主机，设置为源站域名
- 如果源站只有一个站点，设置为 `$host` 或源站域名都可以
- 建议设置为用户访问的域名（加速域名）

### 6.4 端口封禁排查

**Q: 如何确认 80/443 端口被封？**
```bash
# 从外网测试端口连通性
telnet your-server-ip 80
telnet your-server-ip 443

# 或使用 nc
nc -zv your-server-ip 80
nc -zv your-server-ip 443
```

**Q: ESA 支持哪些回源端口？**
- ESA 支持大多数常用端口
- 建议使用 8080、8443、8000、8888 等常见替代端口
- 避免使用 22、3389 等管理端口

**Q: 源站只有 HTTP，能用 ESA 吗？**
- 完全可以！
- ESA 负责 HTTPS 加密，回源用 HTTP 即可
- 在 SSL/TLS 设置中选择「灵活」模式

---

## 相关链接

- [[Quartz 官方文档]]
- [[Nginx 官方文档]]
- [[Caddy 官方文档]]
- [[阿里云 ESA 帮助中心]]
- [[Obsidian 官方文档]]

> 📝 **说明**：本教程后续会逐步配上操作截图，方便对照操作。如有问题欢迎交流讨论。
