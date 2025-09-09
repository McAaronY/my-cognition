### go,gin,docker,go-sqlite3 本地开发部署

### 项目目录如下图

![如图所示](/McAaronY/my-cognition/refs/heads/main/images/go-docker.png)

### dockerfile 脚本

```bash
# 使用 Go 1.25 官方镜像作为构建环境
FROM golang:1.25-alpine3.22 AS builder

## 替换国内镜像
RUN echo "http://mirrors.aliyun.com/alpine/v3.22/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/v3.22/community/" >> /etc/apk/repositories

## 编译需要使用的系统工具
RUN apk update && \
    apk add --no-cache \
    gcc \
    g++ \
    musl-dev \
    make \
    curl && \
    rm -rf /var/cache/apk/*  # 清理缓存，减少镜像体积
## 设置go的依赖下载地址
ENV GOPROXY https://goproxy.cn,direct
## 配置go的打包环境 1 需要动态连接库，0 不需要动态连接库
ENV CGO_ENABLED=1
ENV GOOS=linux
ENV GOARCH=amd64

## 创建打包镜像的工作目录
WORKDIR /app



# 复制依赖文件并下载
COPY go.mod go.sum ./
RUN go mod download

# 复制源代码
COPY . .

# # 编译时添加链接信息，便于排查
RUN go build -ldflags "-X main.buildTime=$(date +%Y%m%d%H%M%S)" -o main .

# 运行和使用更小的运行时镜像
FROM alpine:3.22

### 运行环境的工作目录，即程序的工作目录
WORKDIR /root/

FROM alpine:3.22

# 关键步骤：替换为阿里云 Alpine 源（覆盖官方 repositories 文件）
RUN echo "https://mirrors.aliyun.com/alpine/v3.22/main/" > /etc/apk/repositories && \
    echo "https://mirrors.aliyun.com/alpine/v3.22/community/" >> /etc/apk/repositories && \
    # 可选：添加测试源（如需尝鲜新版本依赖，一般生产环境不建议）
    # echo "https://mirrors.aliyun.com/alpine/v3.22/testing/" >> /etc/apk/repositories && \
    # 更新源缓存并安装常用工具（按需调整）
    apk update && \
    apk add --no-cache \
    curl \
    wget \
    bash \
    tzdata && \
    # 配置时区（避免日志时间错乱，可选）
    ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    # 清理缓存（减少镜像体积）
    rm -rf /var/cache/apk/*

# 调整数据库权限
RUN touch /root/server.db && chmod 664 /root/server.db

# 复制编译好的二进制 也就是打包后的/app/main 复制到 /root/
COPY --from=builder /app/main .

# 暴露端口
EXPOSE 8080

# 添加健康检查
HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl -f http://localhost:8080/health || exit 1

# 启动命令添加日志输出
CMD ["sh", "-c", "echo 'Starting application...' && ./main"]

```

### docker-compose 、

```yml
 version: "3.9"

services:
  web:
    build: .
    ports:
      - "8080:8080"
    volumes:
      - .:/app
    restart: always

```

### 注意事项

1. 要选对 Linux 的 docker 镜像，部分镜像不能使用，特别是使用了 sqlite3 这类需要动态连接系统 dll 的资源库
2. 需要正确编写 dockerfile
3. 必须要对 docker 的构建要有了解
4. 按规范创建 go 项目的文件格式
