### go 安装以及使用

### 安装

1. window
   - [下载地址](https://golang.google.cn/dl/go1.25.0.windows-amd64.msi)
2. mac

```bash
   brew install go
```

3. linux

```bash
 rm -rf /usr/local/go && tar -C /usr/local -xzf go1.25.0.linux-amd64.tar.gz
 export PATH_$PATH:/usr/local/go/bin
```

### 单个项目例子

1. 新建模块

```bash
## 一下命令会创建一个go.mod 文件
go mod init hellworld

```

2. 新建一个 hellworld.go 文件，编写以下代码

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}

```

运行代码

```bash
go run  .
```

### 增加依赖

```go
import "rsc.io/quote"

func main() {
    fmt.Println(quote.Go())
}

```

执行命令

```bash

go mod tidy

```

也可以通过以下方式，添加依赖

```bash
 go get golang.org/x/example/hello/reverse
```

### 创建工作空间

命令

```bash
## 一下命令会创建一个go.work 文件
  go work init  ""

```

### go 打包应用

mac 以及 Linux 系统打包命令

```bash
go env -w CGO_ENABLED=0 GOOS=linux FOARCH=amd64
go env -w CGO_ENABLED=0 GOOS=windows FOARCH=amd64
go env -w CGO_ENABLED=0 GOOS=darwin FOARCH=amd64
```

window

```bash
go env -w CGO_ENABLED=0 GOOS=linux FOARCH=amd64
go env -w CGO_ENABLED=0 GOOS=darwin3 FOARCH=amd64
```
