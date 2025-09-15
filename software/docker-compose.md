# Docker Compose 使用说明书

## 一、Docker Compose 基础概念

### 1. 定义

Docker Compose 是 Docker 官方推出的**多容器编排工具**，通过一个 YAML 格式的配置文件（默认名为 `docker-compose.yml`），定义多个容器的服务配置、网络连接、数据卷挂载等关系，实现 “一键启动 / 停止 / 管理” 整个应用集群，避免手动逐个操作容器的繁琐流程。

### 2. 核心作用

- **简化多容器管理**：无需分别执行 `docker run` 启动每个容器，通过单条命令即可管理所有关联服务（如 Web 服务、数据库、缓存等）。

- **统一环境配置**：配置文件可纳入版本控制，确保开发、测试、生产环境的容器启动参数一致，解决 “环境不一致” 问题。

- **自动维护依赖关系**：支持定义容器启动顺序（如先启动数据库，再启动依赖数据库的 Web 服务），自动创建共享网络和数据卷。

## 二、Docker Compose 安装与环境准备

### 1. 安装前提

- 已安装 Docker 引擎（Docker Compose 需依赖 Docker 运行环境，确保 `docker --version` 可正常输出）。

### 2. 安装方式（以 Linux 为例）

#### 方式 1：通过 Docker 官方脚本安装（推荐）

```
\# 下载 Docker Compose 二进制文件（适用于 x86\_64 架构，其他架构需替换 URL 中的架构标识）

sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-\$(uname -s)-\$(uname -m)" -o /usr/local/bin/docker-compose

\# 赋予执行权限

sudo chmod +x /usr/local/bin/docker-compose

\# 验证安装（输出版本号即成功）

docker-compose --version
```

#### 方式 2：通过 pip 安装（适用于已安装 Python 的环境）

```
pip install docker-compose
```

### 3. 验证环境

执行 `docker-compose --version`，若输出类似 `docker-compose version 1.29.2, build 5becea4c` 的信息，说明安装成功。

## 三、核心配置文件：docker-compose.yml 详解

`docker-compose.yml` 是 Docker Compose 的核心，采用 YAML 语法，需注意**缩进规范（2 个空格，不允许用 Tab）**。以下是配置文件的核心字段及示例。

### 1. 基础结构

一个完整的 `docker-compose.yml` 包含 4 个顶层字段：

- `version`：指定 Compose 文件格式版本（需与 Docker Compose 版本兼容，推荐使用 `3.8` 及以上，适配主流 Docker 版本）。

- `services`：定义多个容器服务（如 `web`、`db`、`redis`），每个服务对应一个容器配置。

- `networks`：定义自定义网络，实现容器间的隔离与通信。

- `volumes`：定义数据卷，实现容器数据持久化或容器间数据共享。

### 2. 核心字段详解（以 “Flask + MySQL + Redis” 应用为例）

```
\# 1. 版本声明（兼容 Docker 19.03.0+）

version: "3.8"

\# 2. 服务定义（每个服务对应一个容器）

services:

&#x20; \# 服务 1：Flask Web 应用

&#x20; web:

&#x20;   \# 构建镜像（两种方式：1. 基于 Dockerfile 构建；2. 直接使用已有镜像）

&#x20;   build: ./web  # 表示从 ./web 目录下的 Dockerfile 构建镜像

&#x20;   \# image: python:3.11-slim  # 若无需构建，直接指定已有镜像（与 build 二选一）

&#x20;  &#x20;

&#x20;   \# 容器名称（自定义，避免自动生成随机名称）

&#x20;   container\_name: flask-web

&#x20;  &#x20;

&#x20;   \# 端口映射（宿主机端口:容器端口）

&#x20;   ports:

&#x20;     \- "8080:5000"  # 宿主机 8080 端口映射到容器 5000 端口（Flask 默认端口）

&#x20;  &#x20;

&#x20;   \# 环境变量（两种方式：1. 直接指定；2. 从文件读取）

&#x20;   environment:

&#x20;     \- FLASK\_ENV=production  # 直接指定环境变量

&#x20;     \- DB\_HOST=db  # 数据库服务名（容器间可通过服务名访问，无需 IP）

&#x20;   env\_file:

&#x20;     \- ./web/.env  # 从文件读取环境变量（文件格式：KEY=VALUE）

&#x20;  &#x20;

&#x20;   \# 依赖服务（启动顺序：先启动 db 和 redis，再启动 web）

&#x20;   depends\_on:

&#x20;     \- db

&#x20;     \- redis

&#x20;  &#x20;

&#x20;   \# 挂载数据卷（宿主机目录/数据卷:容器目录，实现数据持久化）

&#x20;   volumes:

&#x20;     \- ./web/app:/app  # 宿主机 ./web/app 目录挂载到容器 /app（代码实时同步，开发环境常用）

&#x20;     \- web-logs:/app/logs  # 挂载自定义数据卷 web-logs 到容器 /app/logs（持久化日志）

&#x20;  &#x20;

&#x20;   \# 加入自定义网络（实现服务间通信）

&#x20;   networks:

&#x20;     \- app-network

&#x20; \# 服务 2：MySQL 数据库

&#x20; db:

&#x20;   \# 使用官方 MySQL 镜像

&#x20;   image: mysql:8.0

&#x20;   container\_name: mysql-db

&#x20;  &#x20;

&#x20;   \# 端口映射（可选，生产环境建议不暴露端口，仅内部通信）

&#x20;   ports:

&#x20;     \- "3306:3306"

&#x20;  &#x20;

&#x20;   \# 环境变量（MySQL 必要配置：root 密码、数据库名、编码）

&#x20;   environment:

&#x20;     \- MYSQL\_ROOT\_PASSWORD=root123

&#x20;     \- MYSQL\_DATABASE=flask\_db

&#x20;     \- MYSQL\_CHARSET=utf8mb4

&#x20;  &#x20;

&#x20;   \# 数据卷挂载（持久化 MySQL 数据，避免容器删除后数据丢失）

&#x20;   volumes:

&#x20;     \- mysql-data:/var/lib/mysql  # 自定义数据卷 mysql-data 挂载到 MySQL 数据目录

&#x20;     \- ./db/init.sql:/docker-entrypoint-initdb.d/init.sql  # 初始化 SQL 脚本（容器启动时自动执行）

&#x20;  &#x20;

&#x20;   networks:

&#x20;     \- app-network

&#x20; \# 服务 3：Redis 缓存

&#x20; redis:

&#x20;   image: redis:7.0

&#x20;   container\_name: redis-cache

&#x20;   ports:

&#x20;     \- "6379:6379"

&#x20;   volumes:

&#x20;     \- redis-data:/data  # Redis 数据持久化目录

&#x20;   networks:

&#x20;     \- app-network

\# 3. 自定义网络（默认会创建一个名为 "项目名\_default" 的网络，自定义网络更灵活）

networks:

&#x20; app-network:

&#x20;   driver: bridge  # 网络驱动类型（默认 bridge，适用于单机容器通信）

\# 4. 自定义数据卷（全局数据卷，可被多个服务挂载）

volumes:

&#x20; web-logs:  # 日志数据卷

&#x20; mysql-data:  # MySQL 数据卷

&#x20; redis-data:  # Redis 数据卷
```

### 3. 关键字段说明

| 字段             | 作用                                                                         |
| ---------------- | ---------------------------------------------------------------------------- |
| `build`          | 指定 Dockerfile 所在目录，用于构建自定义镜像（与 `image` 二选一）。          |
| `image`          | 直接使用 Docker Hub 或本地已有的镜像（如 `mysql:8.0`）。                     |
| `container_name` | 自定义容器名称，避免 Compose 自动生成随机名称。                              |
| `ports`          | 端口映射，格式为 `宿主机端口:容器端口`（如 `8080:5000`）。                   |
| `environment`    | 设置环境变量，支持数组（`- KEY=VALUE`）或字典（`KEY: VALUE`）格式。          |
| `env_file`       | 从外部文件读取环境变量（避免硬编码敏感信息，如密码、密钥）。                 |
| `depends_on`     | 定义服务依赖关系，控制容器启动顺序（但不等待服务 “就绪”，仅等待容器启动）。  |
| `volumes`        | 数据卷挂载，支持 “宿主机目录：容器目录” 或 “数据卷名称：容器目录” 两种格式。 |
| `networks`       | 将服务加入自定义网络，实现服务间通信（同一网络内可通过服务名访问）。         |

## 四、Docker Compose 核心命令

所有命令需在 `docker-compose.yml` 所在目录执行，核心命令如下：

### 1. 启动服务（默认前台运行，加 `-d` 后台运行）

```
\# 后台启动所有服务（推荐，容器在后台运行，不占用终端）

docker-compose up -d

\# 前台启动（实时输出容器日志，按 Ctrl+C 停止服务）

docker-compose up

\# 启动时重新构建镜像（适用于 Dockerfile 或代码修改后）

docker-compose up -d --build
```

### 2. 停止并删除服务（容器、网络会被删除，数据卷默认保留）

```
\# 停止服务并删除容器、网络（数据卷不删除）

docker-compose down

\# 停止服务并删除容器、网络、数据卷（谨慎使用，数据会丢失）

docker-compose down -v

\# 停止服务并删除容器、网络、镜像（删除通过 build 构建的镜像）

docker-compose down --rmi all
```

### 3. 查看服务状态

```
\# 查看所有服务的运行状态（Up 表示运行中，Exited 表示已停止）

docker-compose ps

\# 查看指定服务的状态（如查看 web 服务）

docker-compose ps web
```

### 4. 查看容器日志

```
\# 查看所有服务的日志（实时输出，按 Ctrl+C 退出）

docker-compose logs

\# 查看指定服务的日志（如查看 web 服务）

docker-compose logs web

\# 实时跟踪日志（加 -f 参数，类似 tail -f）

docker-compose logs -f web

\# 查看最近 10 行日志并实时跟踪

docker-compose logs -f --tail=10 web
```

### 5. 启动 / 停止 / 重启指定服务

```
\# 启动指定服务（如启动 db 服务）

docker-compose start db

\# 停止指定服务

docker-compose stop db

\# 重启指定服务

docker-compose restart db

\# 强制停止指定服务（类似 kill 命令）

docker-compose kill db
```

### 6. 查看数据卷和网络

```
\# 查看 Compose 管理的数据卷

docker-compose volume ls

\# 查看 Compose 管理的网络

docker-compose network ls
```

## 五、实战案例：基于 Compose 部署 Wordpress 博客

### 1. 需求

通过 Compose 部署 “Wordpress + MySQL” 应用，实现：

- MySQL 数据库持久化存储。

- Wordpress 依赖 MySQL 服务，自动按顺序启动。

- 端口映射到宿主机，外部可访问。

### 2. 编写 docker-compose.yml

```
version: "3.8"

services:

&#x20; # Wordpress 服务

&#x20; wordpress:

&#x20;   image: wordpress:latest

&#x20;   container\_name: wordpress

&#x20;   ports:

&#x20;     - "80:80"  # 宿主机 80 端口映射到 Wordpress 80 端口（浏览器直接访问宿主机 IP 即可）

&#x20;   environment:

&#x20;     - WORDPRESS\_DB\_HOST=mysql  # 数据库服务名（无需 IP）

&#x20;     - WORDPRESS\_DB\_USER=root   # 数据库用户名

&#x20;     - WORDPRESS\_DB\_PASSWORD=root123  # 数据库密码（需与 MySQL 配置一致）

&#x20;     - WORDPRESS\_DB\_NAME=wordpress\_db  # 数据库名

&#x20;   depends\_on:

&#x20;     - mysql  # 依赖 MySQL 服务

&#x20;   volumes:

&#x20;     - wordpress-data:/var/www/html  # 持久化 Wordpress 数据（主题、插件、上传文件）

&#x20;   networks:

&#x20;     - wp-network

&#x20; # MySQL 服务

&#x20; mysql:

&#x20;   image: mysql:8.0

&#x20;   container\_name: wp-mysql

&#x20;   environment:

&#x20;     - MYSQL\_ROOT\_PASSWORD=root123  # 与 Wordpress 配置的密码一致

&#x20;     - MYSQL\_DATABASE=wordpress\_db  # 与 Wordpress 配置的数据库名一致

&#x20;   volumes:

&#x20;     - mysql-data:/var/lib/mysql  # 持久化 MySQL 数据

&#x20;   networks:

&#x20;     - wp-network

\# 自定义网络

networks:

&#x20; wp-network:

&#x20;   driver: bridge

\# 数据卷

volumes:

&#x20; wordpress-data:

&#x20; mysql-data:
```

### 3. 部署步骤

1.  在任意目录创建 `docker-compose.yml`，粘贴上述配置。

2.  执行 `docker-compose up -d`，等待镜像拉取和容器启动（首次启动较慢，需耐心等待）。

3.  打开浏览器，访问宿主机 IP（如 `http://192.168.1.100`），按照 Wordpress 引导页面完成安装。

4.  停止服务：执行 `docker-compose down`（数据卷保留，下次启动数据不丢失）。

## 六、常见问题与解决方案

### 1. 服务启动失败，提示 “端口被占用”

- **原因**：`ports` 配置的宿主机端口已被其他程序占用（如 80 端口被 Nginx 占用）。

- **解决方案**：修改 `ports` 中的宿主机端口（如将 `80:80` 改为 `8080:80`），重新执行 `docker-compose up -d`。

### 2. 依赖服务启动后，主服务仍连接失败（如 Web 服务连不上数据库）

- **原因**：`depends_on` 仅保证容器启动顺序，不保证服务 “就绪”（如 MySQL 容器启动后，数据库服务还需 1-2 秒初始化，Web 服务此时连接会失败）。

- **解决方案**：

1.  开发环境：手动延迟启动主服务（如 `docker-compose start web` 延迟 5 秒执行）。

2.  生产环境：在主服务中添加 “服务就绪检测”（如使用 `wait-for-it` 脚本，等待数据库端口可连接后再启动应用）。

### 3. 数据卷挂载后，容器内无法读取宿主机文件

- **原因**：

1.  宿主机目录路径错误（需使用相对路径或绝对路径，相对路径基于 `docker-compose.yml` 所在目录）。

2.  权限问题（宿主机目录权限不足，容器内用户无法读取）。

- **解决方案**：

1.  检查路径：确保 `volumes` 中的宿主机目录存在（如 `./web/app` 需真实存在）。

2.  调整权限：执行 `chmod -R 755 宿主机目录`（如 `chmod -R 755 ./web/app`），赋予读写权限。

### 4. 执行 `docker-compose` 命令提示 “命令不存在”

- **原因**：Docker Compose 未安装成功或未添加到系统 PATH 路径。

- **解决方案**：重新执行安装命令（参考 “二、安装方式”），或手动指定 `docker-compose` 路径（如 `/usr/local/bin/docker-compose up -d`）。

## 七、最佳实践

1.  **拆分配置文件**：

- 开发环境：`docker-compose.dev.yml`（开启代码实时挂载、调试模式）。

- 生产环境：`docker-compose.prod.yml`（关闭调试、不暴露不必要端口、使用更安全的环境变量）。

- 启动时指定配置文件：`docker-compose -f docker-compose.prod.yml up -d`。

1.  **避免硬编码敏感信息**：

- 不直接在 `environment` 中写密码、密钥，改用 `env_file` 引用外部文件（如 `.env`），并将 `.env` 加入 `.gitignore`，避免提交到代码仓库。

1.  **数据卷管理**：

- 优先使用自定义数据卷（如 `mysql-data`），而非宿主机目录挂载（减少环境依赖，跨机器迁移更方便）。

- 定期备份数据卷：执行 `docker run --rm -v 数据卷名:/source -v 宿主机备份目录:/backup alpine tar -czf /backup/volume-backup.tar.gz -C /source .`。

1.  **服务健康检查**：

- 在 `services` 中添加 `healthcheck` 配置，检测服务是否就绪（如 MySQL 端口是否可连接），示例：

```
services:

&#x20; db:

&#x20;   image: mysql:8.0

&#x20;   healthcheck:

&#x20;     test: \["CMD", "mysqladmin", "ping", "-h", "localhost", "-uroot", "-proot123"]

&#x20;     interval: 10s  # 每 10 秒检查一次

&#x20;     timeout: 5s    # 检查超时时间

&#x20;     retries: 5     # 重试 5 次失败后，标记服务不健康
```

> （注：文档部分内容可能由 AI 生成）
