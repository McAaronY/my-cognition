# Dockerfile 使用说明书



![](https://p3-flow-imagex-sign.byteimg.com/ocean-cloud-tos/image_skill/bfa8e009-5b2c-483b-b703-dd07e178905c_1757923892027825273_origin\~tplv-a9rns2rl98-image-qvalue.jpeg?rk3s=6823e3d0\&x-expires=1789459892\&x-signature=uUkTYSD7B%2FhgFDdcQg6mOFVY%2BB0%3D)

## 一、Dockerfile 基础概念

### 1. 定义

Dockerfile 是一个**文本文件**，包含一系列按顺序执行的指令，用于自动化构建 Docker 镜像。通过 Dockerfile，开发者可以将应用的运行环境、依赖配置、启动命令等 “代码化”，确保镜像构建过程可重复、可追溯，避免手动配置环境的不一致性。

### 2. 核心作用



*   **标准化镜像构建**：统一团队内镜像的构建流程，消除 “本地能跑、线上报错” 的环境差异。

*   **版本控制**：可将 Dockerfile 纳入 Git 等版本控制系统，追踪每一次镜像配置的变更。

*   **自动化集成**：支持与 CI/CD 工具（如 Jenkins、GitLab CI）结合，实现镜像的自动构建、测试与部署。

## 二、Dockerfile 核心指令详解

Dockerfile 指令需遵循 “**大写指令 + 小写参数**” 的规范（指令大小写不强制，但大写可提升可读性），以下是常用核心指令：



| 指令           | 作用                                            | 示例                                         | 注意事项                                                                                                                        |
| ------------ | --------------------------------------------- | ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| `FROM`       | 指定基础镜像（所有指令的起点，必须是 Dockerfile 首条非注释指令）        | `FROM ubuntu:22.04`                        | 优先选择官方镜像（如 alpine、debian），体积更小、安全性更高；避免使用 `latest` 标签（版本不稳定）。                                                               |
| `WORKDIR`    | 设置后续指令（RUN、COPY、CMD 等）的工作目录，类似 `cd` 命令        | `WORKDIR /app`                             | 建议使用绝对路径；多次使用时，后续路径会基于前一次路径拼接（如 `WORKDIR /a` 后再 `WORKDIR b`，最终路径为 `/a/b`）。                                                  |
| `COPY`       | 将宿主机文件 / 目录复制到镜像中（仅复制文件，不处理压缩包等）              | `COPY ./app.py /app/`                      | 源路径是相对于 Dockerfile 所在目录的相对路径；目标路径若不存在，会自动创建。                                                                                |
| `ADD`        | 功能比 `COPY` 更丰富（支持复制压缩包并自动解压、支持 URL 下载）        | `ADD ./app.tar.gz /app/`                   | 若仅需复制文件，优先使用 `COPY`（功能更单一，避免意外行为）；URL 下载场景建议用 `RUN wget`（更灵活，便于控制下载过程）。                                                     |
| `RUN`        | 在镜像构建过程中执行命令（如安装依赖、编译代码），会生成新的镜像层             | `RUN apt update && apt install -y python3` | 尽量将多个命令通过 `&&` 合并（减少镜像层数，降低体积）；避免单独使用 `apt update`（易导致依赖版本不一致）。                                                             |
| `ENV`        | 设置环境变量（构建时和容器运行时均生效，可被后续指令引用）                 | `ENV PYTHONPATH=/app`                      | 可通过 `docker run -e 变量名=新值` 覆盖容器运行时的环境变量；建议用大写字母定义变量名，符合规范。                                                                  |
| `EXPOSE`     | 声明容器运行时监听的端口（仅为文档说明，不实际暴露端口）                  | `EXPOSE 8080`                              | 实际暴露端口需通过 `docker run -p 宿主机端口:容器端口` 实现；可声明多个端口（如 `EXPOSE 80 443`）。                                                         |
| `CMD`        | 指定容器启动时执行的命令（一个 Dockerfile 仅能有一条有效 CMD 指令）    | `CMD ["python3", "/app/app.py"]`           | 若 `docker run` 后指定了命令，会覆盖 CMD 指令；推荐使用 “数组格式”（避免 shell 解析带来的问题）。                                                             |
| `ENTRYPOINT` | 设定容器的 “入口程序”（优先级高于 CMD，不会被 `docker run` 命令覆盖） | `ENTRYPOINT ["python3"]`                   | 常与 CMD 配合使用（ENTRYPOINT 定义固定程序，CMD 传递参数），如 `ENTRYPOINT ["python3"]` + `CMD ["/app/app.py"]`，容器启动时实际执行 `python3 /app/app.py`。 |

## 三、Dockerfile 编写规范与最佳实践

### 1. 编写规范



*   **指令顺序**：按 “基础镜像 → 工作目录 → 复制文件 → 安装依赖 → 环境配置 → 启动命令” 的逻辑排序。

*   **注释清晰**：用 `#` 开头添加注释，说明关键指令的作用（如依赖版本选择原因、路径定义逻辑），便于团队协作。

*   **避免冗余**：不复制无关文件（如 `node_modules`、日志文件），可通过 `.dockerignore` 文件排除不需要的文件（类似 `.gitignore`）。

### 2. 最佳实践

#### （1）减小镜像体积



*   **使用轻量级基础镜像**：如用 `alpine` 镜像（体积通常仅几 MB）替代 `ubuntu`（约 100+ MB），例如 `FROM python:3.11-alpine` 而非 `FROM python:3.11`。

*   **清理构建缓存**：安装依赖后及时清理缓存文件，例如：



```
RUN apt update && apt install -y python3 \\

&#x20;   && rm -rf /var/lib/apt/lists/\*  # 清理 apt 缓存
```



*   **多阶段构建**：对于需要编译的项目（如 Go、Java），用 “构建阶段” 生成产物，再用 “运行阶段” 仅复制产物，减少镜像体积。示例（Go 项目）：



```
\# 构建阶段（使用带编译工具的基础镜像）

FROM golang:1.21 AS builder

WORKDIR /app

COPY . .

RUN go build -o myapp main.go  # 编译生成可执行文件

\# 运行阶段（使用轻量级镜像）

FROM alpine:3.18

WORKDIR /app

COPY --from=builder /app/myapp .  # 仅复制编译产物

CMD \["./myapp"]
```

#### （2）提升安全性



*   **避免使用 root 用户**：在镜像中创建普通用户并切换，减少容器被攻击后的权限风险：



```
RUN adduser -D myuser  # 创建普通用户

USER myuser  # 切换为普通用户
```



*   **及时更新依赖**：基础镜像和安装的依赖需定期更新，修复已知漏洞（如 `FROM ubuntu:22.04` 而非旧版本 `ubuntu:20.04`）。

*   **不存储敏感信息**：禁止在 Dockerfile 中硬编码密码、密钥等敏感数据，可通过 `docker secret` 或环境变量（运行时传入）传递。

## 四、Dockerfile 构建与运行流程

### 1. 构建镜像（`docker build`）

#### 基本命令



```
\# 语法：docker build -t 镜像名:标签 Dockerfile所在目录

docker build -t myapp:v1 .  # 构建镜像，标签为 v1，Dockerfile 在当前目录（. 表示当前目录）
```

#### 常用参数



*   `-t`：指定镜像的 “名称：标签”（标签用于区分版本，如 `v1`、`latest`）。

*   `--no-cache`：不使用构建缓存，强制重新执行所有指令（适用于依赖更新后需要全新构建的场景）。

*   `-f`：指定自定义名称的 Dockerfile（如 Dockerfile 名为 `Dockerfile.dev`）：



```
docker build -t myapp:dev -f Dockerfile.dev .
```

### 2. 运行容器（`docker run`）

基于构建好的镜像启动容器，示例：



```
\# 语法：docker run -p 宿主机端口:容器端口 -d 镜像名:标签

docker run -p 8080:8080 -d myapp:v1  # 后台运行容器，将宿主机 8080 端口映射到容器 8080 端口
```

#### 关键参数



*   `-p`：端口映射（解决 “容器网络隔离” 问题，让外部能访问容器内服务）。

*   `-d`：后台运行容器（避免终端被容器占用）。

*   `-e`：传递环境变量（覆盖 Dockerfile 中 `ENV` 定义的值）：



```
docker run -p 8080:8080 -e PYTHONPATH=/app/config -d myapp:v1
```

## 五、常见问题与解决方案

### 1. 构建镜像时提示 “文件找不到”



*   **原因**：`COPY`/`ADD` 指令的源路径错误（如路径相对于宿主机根目录，而非 Dockerfile 所在目录）。

*   **解决方案**：确保源路径是相对于 Dockerfile 所在目录的相对路径；若文件确实存在，检查是否被 `.dockerignore` 文件排除。

### 2. 容器启动后立即退出



*   **原因**：`CMD`/`ENTRYPOINT` 指令指定的命令执行完毕后退出（如命令是 `echo "hello"`，执行完就结束），或命令报错。

*   **解决方案**：


    *   若需容器长期运行，确保启动命令是 “前台运行” 的服务（如 `python3 ``app.py` 启动 Web 服务）。

    *   查看容器日志排查错误：`docker logs 容器ID`。

### 3. 镜像体积过大



*   **原因**：使用了体积大的基础镜像、未清理构建缓存、复制了冗余文件。

*   **解决方案**：参考 “最佳实践 - 减小镜像体积” 部分，改用轻量级基础镜像、清理缓存、使用多阶段构建。

## 六、示例：完整的 Python Web 项目 Dockerfile

以下是一个 Flask 项目的 Dockerfile 示例，包含多阶段构建和安全性优化：



```
\# 构建阶段：安装依赖并复制代码（使用带 pip 的基础镜像）

FROM python:3.11-slim AS builder

WORKDIR /app

\# 复制依赖清单并安装（先复制 requirements.txt，利用 Docker 缓存，避免代码变更导致依赖重新安装）

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt -t ./deps  # 将依赖安装到指定目录

\# 运行阶段：使用轻量级镜像，仅复制必要文件

FROM python:3.11-alpine

WORKDIR /app

\# 创建普通用户并切换

RUN adduser -D flaskuser

USER flaskuser

\# 复制构建阶段的依赖和项目代码

COPY --from=builder /app/deps ./deps

COPY ./app.py .

\# 设定环境变量（Python 不缓冲输出，便于查看日志）

ENV PYTHONUNBUFFERED=1 PYTHONPATH=/app/deps

\# 声明端口（实际映射通过 docker run -p 实现）

EXPOSE 5000

\# 启动 Flask 服务（前台运行）

CMD \["python3", "app.py"]
```

> （注：文档部分内容可能由 AI 生成）
