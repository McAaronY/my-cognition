
 ## yt-dlp 安装与 SSL 问题完整指南（CentOS 7 + Python 3.10）

 >>> 说明：
 >>>本文档整理了在 CentOS 7 上从零安装 Python 3.10、
>>> 配置 OpenSSL、解决 yt-dlp SSL 问题及全局命令设置的完整步骤


 1. 安装 Python 3.10

 1.1 安装依赖包
 yum install -y gcc make zlib-devel bzip2-devel \
 openssl-devel ncurses-devel sqlite-devel \
 readline-devel tk-devel libffi-devel xz-devel

 1.2 下载 Python 3.10 源码
 cd /usr/local/src
 curl -O https://www.python.org/ftp/python/3.10.13/Python-3.10.13.tgz
 tar xf Python-3.10.13.tgz
 cd Python-3.10.13

 1.3 下载并安装 OpenSSL 1.1.1（Python 编译依赖）
 cd /usr/local/src
 curl -L -o openssl-1.1.1w.tar.gz https://www.openssl.org/source/openssl-1.1.1w.tar.gz
 tar xf openssl-1.1.1w.tar.gz
 cd openssl-1.1.1w
 ./config --prefix=/usr/local/openssl-1.1.1 --openssldir=/usr/local/openssl-1.1.1 shared zlib
 make -j$(nproc)
 make install
 /usr/local/openssl-1.1.1/bin/openssl version

 1.4 编译安装 Python 3.10（不使用 --enable-optimizations 避免 PGO 崩溃）
 cd /usr/local/src/Python-3.10.13
 make clean
 ./configure --prefix=/usr/local/python3.10 ...
     --with-openssl=/usr/local/openssl-1.1.1 \
     LDFLAGS="-Wl,-rpath=/usr/local/openssl-1.1.1/lib" \
     CPPFLAGS="-I/usr/local/openssl-1.1.1/include"
 make -j$(nproc)
 make install

 1.5 验证 Python
 /usr/local/python3.10/bin/python3.10 -V
 /usr/local/python3.10/bin/python3.10 - << 'EOF'
 import ssl
 print(ssl.OPENSSL_VERSION)
 EOF

 2. 安装 yt-dlp

 2.1 官方单文件安装（推荐）
 cd /usr/local/bin
 curl -L -o yt-dlp https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp
 chmod +x yt-dlp
 yt-dlp --version

 2.2 使用 Python pip 安装
 /usr/local/python3.10/bin/pip3.10 install -U yt-dlp
 或使用国内镜像：
 /usr/local/python3.10/bin/pip3.10 install -U yt-dlp -i https://pypi.tuna.tsinghua.edu.cn/simple

 2.3 安装 ffmpeg（音视频合并必备）
 cd /usr/local/bin
 curl -L -o ffmpeg-release-amd64-static.tar.xz https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-amd64-static.tar.xz
 tar xf ffmpeg-release-amd64-static.tar.xz
 cp ffmpeg-*-static/ffmpeg ffmpeg-*-static/ffprobe /usr/local/bin/
 chmod +x /usr/local/bin/ffmpeg /usr/local/bin/ffprobe
 ffmpeg -version

 3. 配置 yt-dlp 全局命令

 3.1 确认 /usr/local/bin 在 PATH
 echo $PATH
 export PATH=/usr/local/bin:$PATH

 3.2 永久 PATH（全局生效）
 vi /etc/profile.d/yt-dlp.sh
 添加：
 export PATH=/usr/local/bin:$PATH
 source /etc/profile.d/yt-dlp.sh

 3.3 或创建软链接
 ln -s /usr/local/python3.10/bin/yt-dlp /usr/local/bin/yt-dlp

 3.4 验证
 yt-dlp --version
 which yt-dlp

 4. SSL 证书验证失败问题

 4.1 安装 / 更新系统根证书
 yum install -y ca-certificates
 update-ca-trust force-enable

 4.2 安装 certifi 并配置 Python 使用
 /usr/local/python3.10/bin/pip3.10 install certifi
 export SSL_CERT_FILE=$(python3 -m certifi)

 4.3 永久生效
 echo 'export SSL_CERT_FILE=$(python3 -m certifi)' >> ~/.bashrc
 source ~/.bashrc

 4.4 验证
 yt-dlp <URL>

 4.5 临时绕过 SSL（不推荐）
 yt-dlp --no-check-certificate <URL>

 5. 总结

 - 安装 Python 3.10 必须使用 OpenSSL ≥1.1.1
 - Python SSL 证书问题可通过 certifi 和更新 ca-certificates 解决
 - yt-dlp 全局可用：放 /usr/local/bin 或配置软链接/修改 PATH
 - 不推荐使用 --no-check-certificate 做长期方案