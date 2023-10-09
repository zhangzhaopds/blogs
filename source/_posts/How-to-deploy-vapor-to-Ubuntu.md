---
title: How to deploy Vapor to Ubuntu
date: 2023-10-08 17:56:41
tags:
---
## 如何将Vapor项目部署到Ubuntu？

#### 前言
###### Vapor官方文档
	https://legacy.docs.vapor.codes/3.0/

###### Swift下载地址
	https://www.swifwt.org/download/#using-downloads

###### 环境：
	
	Ubuntu 22.04

	Swift 5.9,	x86_64
	https://download.swift.org/swift-5.9-release/ubuntu2204/swift-5.9-RELEASE/swift-5.9-RELEASE-ubuntu22.04.tar.gz

##### 备注：

文中涉及到的文件路径，可根据自己喜好修改调整
```
Swift安装包：/usr/src/

Vapor Toolbox： /usr/local/projects/vap

Hello项目：usr/local/projects/vap
```


#### 1、安装Swift

```
sudo apt update && sudo apt upgrade
```

```
apt-get install \
          binutils \
          git \
          gnupg2 \
          libc6-dev \
          libcurl4-openssl-dev \
          libedit2 \
          libgcc-9-dev \
          libpython3.8 \
          libsqlite3-0 \
          libstdc++-9-dev \
          libxml2-dev \
          libz3-dev \
          pkg-config \
          tzdata \
          unzip \
          zlib1g-dev
```
 
```
# 导入 PGP
wget -q -O - https://swift.org/keys/all-keys.asc | gpg --import -
```

```
cd /usr/src/
```

```
apt install -y wget
```

```
# 下载Swift压缩包
wget https://download.swift.org/swift-5.9-release/ubuntu2204/swift-5.9-RELEASE/swift-5.9-RELEASE-ubuntu22.04.tar.gz
```

```
# 解压
tar xzf swift-5.9-RELEASE-ubuntu22.04.tar.gz
```

```
# 添加软连接
export PATH=/usr/src/swift-5.9-RELEASE-ubuntu22.04/usr/bin:"${PATH}"
```

```
# 校验
swift --version

# 输出下面内容，表示安装成功
Swift version 5.9 (swift-5.9-RELEASE)
Target: x86_64-unknown-linux-gnu
```

#### 2、安装Vapor Toolbox
	
```
cd /usr/local/projects/vap
```

```
git clone https://github.com/vapor/toolbox.git
```
	
```
cd toolbox
```
	
```
git checkout 18.7.1
```
	
```
swift build -c release --disable-sandbox
```
	
```
mv .build/release/vapor /usr/local/bin
```
	

```
# 校验
vapor --help
```

#### 3、新建Vapor项目
		
```
# 进入文件夹
cd /usr/local/projects/vap
```

```
# 新建项目
vapor new Hello	
```
	
```
# 构建
cd Hello
swift build
```
	
```
# 运行
swift run

# 或者
swift run App serve --env production
swift run App serve --env development
swift run App serve --env testing
```

#### 4、安装nginx
	
使用反向代理，将Vapor 服务器绑定到 80 端口
```
# 安装
apt-get update
apt-get install nginx
```

```
# 修改站点配置
vim /etc/nginx/sites-enabled/default
```
	
```
# 具体修改如下：
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /usr/local/projects/vap/Hello/Public/;

    # nginx 直接处理所有静态资源文件的请求，其余请求则回落 (fallback) 到 Vapor 应用
    location / {
        try_files $uri @proxy;
    }

    location @proxy {
        proxy_pass http://127.0.0.1:8080;
        proxy_pass_header Server;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_connect_timeout 3s;
        proxy_read_timeout 10s;
    }
}
```


```
# 运行
service nginx start
```

在Hello项目运行的前提下，访问 http://你的IP地址, 一切正常的话，将输出 It works!

#### 5、安装supervisor
	
使项目保持后台运行
```
# 安装
sudo apt-get install supervisor
```

```
# 进入配置目录
cd /etc/supervisor/conf.d/
```

```
# 为项目添加配置
touch Hello.conf
```

```
# 编辑配置
vim Hello.conf
```

```
# 具体编辑内容如下：
[program:hello]
	command=/usr/src/swift-5.9-RELEASE-ubuntu22.04/usr/bin/swift run App serve --env production
 directory=/usr/local/projects/vap/Hello/
 autorestart=true
 user=root
 stdout_logfile=/var/log/supervisor/%(program_name)-stdout.log
 stderr_logfile=/var/log/supervisor/%(program_name)-stderr.log
```

```
# 重启
systemctl restart supervisor
```

```
# 重新加载配置
sudo supervisorctl reread
```

```
# 添加需要运行的配置
sudo supervisorctl add hello
```

```
# 启动配置
sudo supervisorctl start hello
```
	
```
# vap项目更新后，需重启
sudo supervisorctl restart all
```

```
# Supervisor状态
systemctl status supervisor
```

```
# 查看运行中的服务， 输入exit退出
supervisorctl
```

👉👉👉👉  End 👈👈👈👈