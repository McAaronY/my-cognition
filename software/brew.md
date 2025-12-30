### brew 使用

### 配置文件

```bash
    ~/.zshrc
    /etc/profile
    ~/.bash_profile
    ~/.bash_zshrc
```

### 下载

```bash
 Https://mirrors.aliyun.com/homebrew

```

### 替换国内镜像

```bash
>> 更换源
## 查询
export HOMEBREW_API_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles/api
## 下载地址
export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles

>> 重置官方源
git remote set


```

### 基本命令

```bash
    ## 更新
    brew update
    ## 查找
    brew search <包名>
    ## 安装
    brew install <包名>
    ## 查看安装包
    brew list
    ## 清理过期的软件包
    brew cleanup
    ##  查看包的信息
    brew info
    ## 卸载
    brew uninstall 


```
