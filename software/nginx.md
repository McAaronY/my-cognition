# Nginx 使用说明书

## 一、Nginx 基础概念

### 1. 定义

Nginx（发音为 “engine x”）是一款**高性能的 HTTP 和反向代理服务器**，同时也是一个 IMAP/POP3/SMTP 代理服务器。它由俄罗斯程序员 Igor Sysoev 开发，最初旨在解决高并发场景下的服务器性能问题，凭借轻量级、高并发、低内存占用等特性，广泛应用于 Web 服务、负载均衡、反向代理、静态资源托管等场景。

### 2. 核心作用

- **静态资源托管**：高效处理 HTML、CSS、JavaScript、图片、视频等静态文件，支持浏览器缓存策略配置，减轻后端服务器压力。

- **反向代理**：接收客户端请求，将请求转发至后端业务服务器（如 Tomcat、Node.js、Python Flask 服务），隐藏后端服务器真实 IP，提升服务安全性和可维护性。

- **负载均衡**：将大量客户端请求均匀分配到多个后端服务器，避免单点服务器过载，提高服务可用性和并发处理能力。

- **HTTP 缓存与压缩**：通过配置缓存规则，减少重复请求对服务器的访问；支持 Gzip 等压缩算法，减小传输文件体积，提升页面加载速度。

- **SSL/TLS 终止**：集中处理 HTTPS 加密和解密工作，避免每个后端服务器单独处理加密操作，降低后端服务器资源消耗。

## 二、Nginx 安装与环境准备

### 1. 安装前提

- 操作系统：支持 Linux（如 CentOS、Ubuntu）、Windows、macOS，生产环境优先选择 Linux（稳定性和性能更优）。

- 依赖环境：Linux 系统需确保安装`gcc`（编译工具）、`pcre`（正则表达式库，用于 URL 匹配）、`zlib`（压缩库，用于 Gzip 压缩）、`openssl`（加密库，用于 HTTPS）。

### 2. 常见操作系统安装方式

#### （1）CentOS 7/8 安装（YUM 方式，推荐）

```
# 1. 安装依赖（若已安装可跳过）

sudo yum install -y gcc pcre-devel zlib-devel openssl-devel

# 2. 安装Nginx（CentOS默认源无Nginx，需先添加官方源）

# CentOS 7

sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm

# CentOS 8

sudo dnf install -y https://nginx.org/packages/centos/8/x86\_64/RPMS/nginx-1.24.0-1.el8.ngx.x86\_64.rpm

# 3. 安装Nginx

sudo yum install -y nginx  # CentOS 7

# 或

sudo dnf install -y nginx  # CentOS 8

# 4. 启动Nginx并设置开机自启

sudo systemctl start nginx

sudo systemctl enable nginx

# 5. 验证安装（访问服务器IP，若看到Nginx默认页面则成功）

# 或执行命令查看版本

nginx -v  # 输出类似 nginx version: nginx/1.24.0
```

#### （2）Ubuntu 20.04/22.04 安装（APT 方式）

```
# 1. 更新软件源

sudo apt update

# 2. 安装Nginx

sudo apt install -y nginx

# 3. 启动Nginx并设置开机自启

sudo systemctl start nginx

sudo systemctl enable nginx

# 4. 验证安装（同CentOS，访问IP或查看版本）

nginx -v
```

#### （3）Windows 安装（压缩包方式）

1.  访问 Nginx 官方下载页（[https://nginx.org/en/download.html](https://nginx.org/en/download.html)），下载 Windows 版本压缩包（如`nginx-1.24.0.zip`）。

2.  解压压缩包到指定目录（如`D:\nginx-1.24.0`），注意路径中不要包含中文或空格。

3.  启动 Nginx：

- 打开命令提示符（CMD），切换到 Nginx 解压目录：`cd D:\nginx-1.24.0`。

- 执行启动命令：`start nginx`（无明显输出，若进程中存在`nginx.exe`则启动成功）。

1.  验证：打开浏览器访问`http://localhost`，若看到 Nginx 默认页面则成功。

## 三、Nginx 核心配置文件详解

Nginx 的配置文件默认位于`/etc/nginx/`（Linux）或`Nginx解压目录/conf`（Windows），核心配置文件为`nginx.conf`，采用 “块级配置” 语法，支持注释（`#`开头）。

### 1. 配置文件整体结构

```
# 1. 全局块：配置影响Nginx全局的参数

user  nginx;  # Nginx运行用户（Linux下默认nginx，Windows下无需配置）

worker\_processes  auto;  # 工作进程数，建议设置为CPU核心数（auto表示自动检测）

error\_log  /var/log/nginx/error.log warn;  # 错误日志路径及级别（warn/error/info/debug）

pid        /var/run/nginx.pid;  # Nginx进程PID文件路径

# 2. 事件块：配置Nginx与用户的网络连接

events {

   worker\_connections  1024;  # 每个工作进程的最大连接数（默认1024，高并发场景可调整为10000+）

   use epoll;  # 网络I/O模型（Linux推荐epoll，Windows无需配置）

}

# 3. HTTP块：配置HTTP服务器的核心参数，包含全局HTTP配置和虚拟主机配置

http {

   include       /etc/nginx/mime.types;  # 引入MIME类型映射文件（定义文件类型与HTTP响应头的对应关系）

   default\_type  application/octet-stream;  # 默认MIME类型（未匹配时使用）

   # 日志格式配置（main为自定义格式名，可在虚拟主机中引用）

   log\_format  main  '\$remote\_addr - \$remote\_user \[\$time\_local] "\$request" '

                     '\$status \$body\_bytes\_sent "\$http\_referer" '

                     '"\$http\_user\_agent" "\$http\_x\_forwarded\_for"';

   access\_log  /var/log/nginx/access.log  main;  # 访问日志路径及格式

   sendfile        on;  # 开启高效文件传输模式（减少CPU占用，推荐开启）

   # tcp\_nopush     on;  # 配合sendfile使用，提升网络传输效率（可选开启）

   keepalive\_timeout  65;  # 客户端连接超时时间（单位：秒）

   # gzip压缩配置（开启后减小传输文件体积）

   gzip  on;  # 开启gzip压缩

   gzip\_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss;  # 需压缩的文件类型

   # 引入虚拟主机配置文件（推荐将不同站点配置拆分到conf.d目录，便于管理）

   include /etc/nginx/conf.d/\*.conf;

}
```

### 2. 虚拟主机配置（核心功能实现）

虚拟主机（Virtual Host）允许一台 Nginx 服务器托管多个域名站点，通过`server`块配置，默认配置文件位于`/etc/nginx/conf.d/default.conf`（Linux）。

#### （1）静态资源托管配置（示例：托管 HTML 和图片文件）

```
# 虚拟主机配置：托管静态站点（域名：static.example.com）

server {

   listen       80;  # 监听端口（HTTP默认80，HTTPS默认443）

   server\_name  static.example.com;  # 绑定的域名（可配置多个，用空格分隔）

   # 站点根目录（静态文件存放路径）

   root   /usr/share/nginx/static;

   index  index.html index.htm;  # 默认首页文件

   # 访问日志（引用HTTP块中定义的main格式）

   access\_log  /var/log/nginx/static-access.log  main;

   error\_log   /var/log/nginx/static-error.log  warn;

   # 静态文件缓存配置（对图片、CSS、JS设置30天缓存）

   location \~\* \\.(jpg|jpeg|png|gif|css|js)\$ {

       expires 30d;  # 缓存有效期30天

       add\_header Cache-Control "public, max-age=2592000";  # 浏览器缓存控制头

   }

   # 404页面配置（当请求文件不存在时返回自定义404页面）

   error\_page  404              /404.html;

   location = /404.html {

       root   /usr/share/nginx/static;

       internal;  # 仅允许内部跳转访问，不允许直接访问

   }

}
```

#### （2）反向代理配置（示例：代理后端 Node.js 服务）

```
# 虚拟主机配置：反向代理（域名：api.example.com，代理到后端3000端口）

server {

   listen       80;

   server\_name  api.example.com;

   access\_log  /var/log/nginx/api-access.log  main;

   error\_log   /var/log/nginx/api-error.log  warn;

   # 反向代理核心配置：将所有请求转发到后端服务

   location / {

       proxy\_pass http://127.0.0.1:3000;  # 后端服务地址（IP:端口）

       proxy\_set\_header Host \$host;  # 传递客户端请求的Host头到后端

       proxy\_set\_header X-Real-IP \$remote\_addr;  # 传递客户端真实IP到后端

       proxy\_set\_header X-Forwarded-For \$proxy\_add\_x\_forwarded\_for;  # 传递代理链IP

       proxy\_set\_header X-Forwarded-Proto \$scheme;  # 传递请求协议（http/https）

   }

   # 后端服务超时配置

   proxy\_connect\_timeout 5s;  # 与后端建立连接的超时时间

   proxy\_send\_timeout 10s;    # 发送请求到后端的超时时间

   proxy\_read\_timeout 10s;    # 读取后端响应的超时时间

}
```

#### （3）负载均衡配置（示例：分发请求到 3 个后端 Tomcat 服务）

```
# 1. 先在HTTP块中定义负载均衡集群（upstream为集群名，自定义）

http {

   # ... 其他全局配置 ...

   # 负载均衡集群：后端3个Tomcat服务

   upstream tomcat\_cluster {

       server 192.168.1.101:8080 weight=3;  # 权重3（接收30%请求）

       server 192.168.1.102:8080 weight=2;  # 权重2（接收20%请求）

       server 192.168.1.103:8080;           # 默认权重1（接收10%请求）

       # 可选配置：down（标记服务器下线）、backup（备用服务器，仅主服务器故障时启用）

       # server 192.168.1.104:8080 down;

       # server 192.168.1.105:8080 backup;

   }

   # 2. 虚拟主机配置：反向代理到负载均衡集群

   server {

       listen       80;

       server\_name  web.example.com;

       access\_log  /var/log/nginx/web-access.log  main;

       error\_log   /var/log/nginx/web-error.log  warn;

       location / {

           proxy\_pass http://tomcat\_cluster;  # 代理到负载均衡集群（集群名与upstream一致）

           proxy\_set\_header Host \$host;

           proxy\_set\_header X-Real-IP \$remote\_addr;

       }

   }

}
```

#### （4）HTTPS 配置（示例：基于 Let's Encrypt 证书）

```
# 虚拟主机配置：HTTPS服务（需先申请SSL证书，如Let's Encrypt）

server {

   listen       443 ssl;  # 监听HTTPS默认端口443，并开启SSL

   server\_name  https.example.com;

   # SSL证书路径（cert.pem为证书文件，privkey.pem为私钥文件）

   ssl\_certificate     /etc/nginx/ssl/https.example.com/cert.pem;

   ssl\_certificate\_key /etc/nginx/ssl/https.example.com/privkey.pem;

   # SSL优化配置

   ssl\_protocols TLSv1.2 TLSv1.3;  # 支持的TLS协议版本（禁用不安全的SSLv3、TLSv1.0/1.1）

   ssl\_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;  # 加密套件

   ssl\_prefer\_server\_ciphers on;  # 优先使用服务器定义的加密套件

   ssl\_session\_cache shared:SSL:10m;  # SSL会话缓存（提升HTTPS连接速度）

   ssl\_session\_timeout 10m;  # SSL会话超时时间

   # 强制HTTP跳转HTTPS（可选：将80端口请求重定向到443端口）

   # 需单独配置一个80端口的server块

   # server {

   #     listen 80;

   #     server\_name https.example.com;

   #     return 301 https://\$host\$request\_uri;  # 301永久重定向

   # }

   # 站点配置（如静态资源或反向代理，同前所述）

   root   /usr/share/nginx/https;

   index  index.html;

   location / {

       # ... 其他配置 ...

   }

}
```

## 四、Nginx 日常管理命令

### 1. 服务启停与重启（Linux 系统，基于 systemctl）

```
# 启动Nginx

sudo systemctl start nginx

# 停止Nginx

sudo systemctl stop nginx

# 重启Nginx（配置文件修改后需重启生效）

sudo systemctl restart nginx

# 重新加载配置文件（推荐：无需停止服务，平滑加载新配置）

sudo systemctl reload nginx

# 查看Nginx运行状态

sudo systemctl status nginx
```

### 2. 命令行工具（nginx 命令，适用于所有系统）

```
# 检查配置文件语法是否正确（修改配置后必做，避免语法错误导致服务启动失败）

nginx -t  # 输出 "nginx: configuration file /etc/nginx/nginx.conf test is successful" 表示正确

# 查看Nginx版本信息

nginx -v  # 简单版本（如 nginx version: nginx/1.24.0）

nginx -V  # 详细版本（包含编译参数，如 --with-http\_ssl\_module）

# 强制停止Nginx（仅在 systemctl 无法停止时使用）

nginx -s stop

# 优雅停止Nginx（等待当前所有请求处理完成后再停止）

nginx -s quit

# 平滑重新加载配置文件（同 systemctl reload nginx）

nginx -s reload

# 查看Nginx帮助信息

nginx -h
```

### 3. Windows 系统命令（CMD 中执行，需切换到 Nginx 解压目录）

```
# 启动Nginx

start nginx

# 停止Nginx（强制停止）

nginx -s stop

# 优雅停止Nginx

nginx -s quit

# 重新加载配置文件

nginx -s reload

# 检查配置文件语法

nginx -t
```

## 五、常见问题与解决方案

### 1. 启动 Nginx 失败，提示 “端口被占用”

- **原因**：Nginx 监听的端口（如 80、443）已被其他服务占用（如 Apache、IIS、迅雷等）。

- **解决方案**：

1.  查看端口占用情况：

- Linux：`sudo netstat -tulpn | grep 80`（查看 80 端口占用，替换为 443 可查 HTTPS 端口）。

- Windows：`netstat -ano | findstr ":80"`（最后一列数字为占用端口的进程 PID）。

1.  停止占用端口的服务：

- Linux：若为 Apache，执行 `sudo systemctl stop httpd`。

- Windows：打开 “任务管理器”，找到对应 PID 的进程并结束。

1.  或修改 Nginx 监听端口：在`server`块中将`listen 80`改为其他未占用端口（如`listen 8080`），重新加载配置。

### 2. 访问 Nginx 提示 “403 Forbidden”

- **原因**：

1.  站点根目录权限不足：Nginx 运行用户（如`nginx`）无权限读取目录下的文件
