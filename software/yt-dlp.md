# yt-dlp 命令大全

yt-dlp 是一款功能强大的视频下载工具（youtube-dl 分支），支持 1000+ 视频平台，包含下载、格式选择、音频提取、批量操作等核心功能。以下是 分类整理的常用命令大全，涵盖基础到高级用法，方便快速查询：

一、基础下载命令（核心常用）

1. 直接下载（默认最佳格式）

```bash
  yt-dlp [视频 URL] # 自动选择分辨率最高的视频+音频合并格式（优先 mp4）
```

示例：

```bash
yt-dlp https://www.youtube.com/watch?v=XXXXXXX

```

2. 下载并指定保存文件名

```bash

yt-dlp -o "文件名.%(ext)s" [URL] # %(ext)s 自动填充文件格式（mp4/m4a 等）
```

示例（自定义文件名 + 路径）：

```bash
yt-dlp -o "~/Downloads/我的视频.%(ext)s" https://www.bilibili.com/video/XXXXXXX
```

3. 下载播放列表（整个列表 / 指定范围）

```bash


# 下载整个播放列表

yt-dlp [播放列表 URL]

# 下载播放列表第 1-5 集（索引从 1 开始）

yt-dlp --playlist-items 1-5 [播放列表 URL]

# 下载播放列表第 1、3、5 集

yt-dlp --playlist-items 1,3,5 [播放列表 URL]

# 跳过前 2 集，下载后续所有

yt-dlp --playlist-start 3 [播放列表 URL]
```

4. 断点续传（恢复中断的下载）

```bash

yt-dlp -c [URL] # 自动检测未完成文件，继续下载
```

二、格式选择命令（视频 / 音频质量控制）
yt-dlp 支持分离视频流（无音频）和音频流，可手动组合或选择预设格式，核心参数 -f（--format）。

1. 查看可用格式（关键前提）
   先查询目标视频的所有支持格式（分辨率、编码、大小等）：

```bash
   yt-dlp -F [URL] # 列出所有可用格式，格式码（format code）是选择依据
```

输出示例（关键列）：
|format| code| ext | resolution | note|
|--- |--- |--- |--- |--- |
|248| webm| 1080p |视频流|（无音频）|
|140| m4a |audio |only |音频流（高质量）|
|18| mp4| 720p| 视频 + 音频（合并）| 2. 选择指定格式下载

```bash
   运行

# 按格式码选择（推荐，精准）

yt-dlp -f 18 [URL] # 下载 720p mp4（视频+音频合并）

# 组合视频流+音频流（分离流合并，画质更优）

yt-dlp -f "248+140" [URL] # 1080p 视频流 + 高质量音频流

# 按格式类型选择（简化）

yt-dlp -f "bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]" [URL]

# 含义：优先 mp4 视频+mp4 音频，否则选最佳 mp4 合并格式
```

3. 限制分辨率 / 大小

```bash


# 下载不超过 720p 的最佳格式

yt-dlp -f "bestvideo[height<=720]+bestaudio/best[height<=720]" [URL]

# 下载文件大小不超过 50MB 的格式

yt-dlp -f "best[filesize<=50M]" [URL]

```

4. 仅下载音频（提取音乐）

```bash


# 直接提取音频（默认 m4a 格式）

yt-dlp -x [URL]

# 提取音频并转换为 mp3 格式（需安装 ffmpeg）

yt-dlp -x --audio-format mp3 [URL]

# 提取高质量音频（指定音频码率）

yt-dlp -x --audio-quality 0 [URL] # 0=最高质量（FFmpeg 音质等级）
```

5. 仅下载视频（无音频）

```bash
yt-dlp -f "bestvideo" [URL] # 下载分辨率最高的纯视频流
```

三、批量下载与批量处理

1. 从文件读取 URL 批量下载
   将多个视频 URL 存入文本文件（如 urls.txt，每行一个 URL），执行：

```bash
   yt-dlp -a urls.txt # -a = --batch-file
```

2. 下载多个 URL（直接传入）

```bash
   yt-dlp [URL1] [URL2] [URL3] # 空格分隔多个 URL
```

3. 批量重命名（播放列表统一命名）

```bash

# 播放列表文件命名格式：[序号]-[标题].ext

yt-dlp -o "%(playlist_index)s-%(title)s.%(ext)s" [播放列表 URL]

# 包含作者名：[作者]-[标题].ext

yt-dlp -o "%(uploader)s-%(title)s.%(ext)s" [URL]
```

常用变量（可组合）：

1.  %(title)s：视频标题
2.  %(uploader)s：上传者 / 作者名
3.  %(playlist)s：播放列表名称
4.  %(playlist_index)s：播放列表中的序号
5.  %(id)s：视频 ID（避免重名） 4. 批量下载字幕（见下文 “字幕相关”）

四、字幕与弹幕下载

1. 下载视频字幕（自动匹配语言）

```bash


# 下载所有可用字幕（默认 .vtt 格式）

yt-dlp --all-subs [URL]

# 下载指定语言字幕（如中文、英文）

yt-dlp --sub-langs zh-CN,en [URL] # 中文（简体）+英文

# 下载字幕并嵌入视频（需 ffmpeg，仅 mp4 格式支持）

yt-dlp --embed-subs --sub-langs zh-CN [URL]

# 字幕格式转换（如 vtt 转 srt）

yt-dlp --sub-langs zh-CN --convert-subs srt [URL]
```

2. 下载弹幕（B 站等平台支持）
   B 站弹幕（xml 格式）下载：

```bash
运行
yt-dlp --write-sub --sub-langs danmaku [B 站视频 URL]

# 弹幕文件为 .xml，可用工具转换为字幕格式
```

3. 仅下载字幕（不下载视频）

```bash
   运行
   yt-dlp --write-sub --sub-langs zh-CN -f "none" [URL]
```

五、高级功能（代理、限速、登录、去水印等）

1. 代理设置（科学上网 / 内网代理）

```bash
   运行

# HTTP 代理

yt-dlp --proxy http://127.0.0.1:7890 [URL]

# SOCKS5 代理（如 Clash）

yt-dlp --proxy socks5://127.0.0.1:7891 [URL]
```

2. 限速下载（避免占用全部带宽）

```bash
运行
yt-dlp --limit-rate 1M [URL] # 限制速度为 1MB/s（支持 K/M/G）
```

3. 登录认证（下载需登录的视频，如 B 站大会员、YouTube 会员）

```bash
运行

# 账号密码登录（不推荐，明文风险）

yt-dlp -u 用户名 -p 密码 [URL]

# 浏览器 cookie 登录（推荐，免明文密码）

# 1. 用浏览器插件导出 cookie 为 Netscape 格式（如 Cookie-Editor）

# 2. 执行命令：

yt-dlp --cookies "cookie.txt" [URL]
```

4. 下载隐藏 / 私有视频（需权限）

```bash
运行

# YouTube 私有视频（需登录账号有访问权限）

yt-dlp --cookies "cookie.txt" [私有视频 URL]

# B 站加密视频（需携带登录 cookie）

yt-dlp --cookies "cookie.txt" [加密视频 URL]
```

5. 去水印（部分平台支持）
   部分短视频平台（如抖音、快手）的水印可通过 --extract-audio 或指定格式规避，或使用专用参数：

```bash
运行

# 抖音去水印（需最新版 yt-dlp，自动识别无水印流）

yt-dlp [抖音视频 URL] # 新版默认优先下载无水印格式
```

6. 多线程下载（加速）

```bash
运行
yt-dlp -N 4 [URL] # -N = --no-part，使用 4 线程下载（默认单线程）
```

7. 跳过已下载文件（避免重复）

```bash
运行
yt-dlp -w [URL] # -w = --no-overwrites，跳过已存在的文件
```

8. 下载后自动转换格式（需 ffmpeg）

```bash
运行

# 下载后转换为 MP4 格式（强制）

yt-dlp --recode-video mp4 [URL]

# 下载后转换为 MKV 格式

yt-dlp --recode-video mkv [URL]
```

9. 自定义 HTTP 请求头（突破反爬）

```bash
运行
yt-dlp --add-header "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/114.0.0.0" [URL]
```

六、常用参数速查表（按功能分类）
|功能 |参数 |示例|
|---|---|---|
|基础下载 | 无参数 | yt-dlp [URL] |
|自定义文件名| -o "名称.%(ext) s" |yt-dlp -o "视频 1.%(ext) s" [URL]|
|断点续传| -c / --continue| yt-dlp -c [URL]|
|查看格式列表| -F / --list-formats |yt-dlp -F [URL]|
|选择格式| -f [格式码 / 规则]| yt-dlp -f 18 [URL]|
|仅音频| -x / --extract-audio | yt-dlp -x [URL]|
|音频转| MP3 -x --audio-format mp3 |yt-dlp -x --audio-format mp3 [URL]|
|批量下载| -a [文件]| yt-dlp -a urls.txt|
|播放列表范围 | --playlist-items 1-5 |yt-dlp --playlist-items 1-5 [URL]|
|下载字幕| --sub-langs zh-CN| yt-dlp --sub-langs zh-CN [URL]|
|嵌入字幕| --embed-subs| yt-dlp --embed-subs --sub-langs zh-CN [URL]|
|代理 |--proxy [代理地址] |yt-dlp --proxy socks5://127.0.0.1:7891 [URL]|
|限速| --limit-rate 1M | yt-dlp --limit-rate 1M [URL]|
|跳过已下载| -w / --no-overwrites |yt-dlp -w [URL]|
|强制覆盖| -O / --overwrites| yt-dlp -O [URL]|
|静默模式（少输出）| -q / --quiet |yt-dlp -q [URL]|
|详细日志| -v / --verbose |yt-dlp -v [URL]
七、安装与依赖

1. 安装 yt-dlp
   1. Windows：下载 exe 文件（官网），添加到环境变量。
   2. macOS/Linux：

```bash
# 用 pip 安装（推荐，自动更新）

pip install yt-dlp

# 或直接下载二进制文件

curl -L https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -o /usr/local/bin/yt-dlp
chmod a+rx /usr/local/bin/yt-dlp
```

2. 必备依赖（核心功能支持）
   1. ffmpeg：用于视频 / 音频合并、格式转换、字幕嵌入（必装！）
      1. Windows：下载 ffmpeg 并添加到环境变量。
      2. macOS：brew install ffmpeg
      3. Linux：sudo apt install ffmpeg（Debian/Ubuntu）
         八、注意事项
1. 遵守版权协议：仅下载自己拥有版权或允许下载的视频，请勿用于商业用途。
1. 平台限制：部分平台（如 Netflix、Disney+）有 DRM 加密，yt-dlp 无法下载。
1. 版本更新：yt-dlp 频繁更新以适配平台变化，建议定期更新：

```bash
yt-dlp -U # 自动更新（pip 安装则用 pip install -U yt-dlp）
```

4. 问题排查：若下载失败，先更新 yt-dlp + ffmpeg，再用 -v 参数查看日志定位问题。
