## Redis 介绍

### redis 是由key-value作为类型的内存系统,支持单线程,分布式的的存储方式,对于实时比较强的数据存储具有高性能的的查询和存储优势.并且对于低数据量的存储
具有较好的数据一致性.

### 安装教程

1. windows 
  1.1 下载地址 https://github.com/dmajkic/redis/downloads

  1.2 启动服务
  
  ```bash
   redis-server.exe redis.conf
  ```
  1.3 连接客户端
  ```bash
     redis-cli.exe -h 127.0.0.1 -p 6379
  ```
2. linux 

  2.1 下载地址 : http://www.redis.net.cn/download/

  ```bash
   wget http://download.redis.io/releases/redis-2.8.17.tar.gz
   tar xzf redis-2.8.17.tar.gz
   cd redis-2.8.17
   make
  ```

  2.2 启动服务

  ```bash
   ./redis-server redis.conf
  ```

  2.3 连接客户端

  ```bash
     ./redis-cli.exe
  ```

3. docker-componse 安装

```bash
services:
  redis:
    image: redis:7.2
    container_name: redis
    restart: always
    ports:
      - "6387:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5

```

## 配置(redis.conf)

1.1 通过命令获取配置信息

```bash

  config get *

```

1.2 配置说明

| 参数 | 说明 |
| --- | ---  |
| daemonize no|默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程|
| pidfile /var/run/redis.pid | ---  |
| port 6379 | ---  |
| bind 127.0.0.1| -- |
| timeout 300 | ---  |
| loglevel verbose | ---  |
| logfile stdout | ---  |
| databases 16 | ---  |
| bind 127.0.0.1 | ---  |
| bind 127.0.0.1 | ---  |













