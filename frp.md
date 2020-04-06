# frp内网穿透工具

> 版本为0.32.1

## 目标

1. 通过`ssh -oPort=6000 user@公网ip`访问内网的ssh服务

2. 通过在浏览器打开`http://test.123.com`访问内网`8080`端口的web服务

3. ...

## 需要准备

1. 有公网ip的服务器（用来部署frps服务）

2. 内网机器（用来部署frpc服务）

3. [frp文档](https://github.com/fatedier/frp/blob/master/README_zh.md)

## 操作步骤

### github上[下载源码](https://github.com/fatedier/frp/releases)

  ```bash
  # 下载时请找对应系统版本，并且服务端和内网版本号要保持一致
  # 以下只是个例子，url自行替换！！！
  curl https://github.com/fatedier/frp/releases/download/v0.32.1/frp_0.32.1_linux_amd64.tar.gz
  ```

### 使用sftp工具将文件分别拷贝指定位置

  ```bash
  # frps和frps.ini文件拷贝到公网服务器
  tar -zxvf frp_xxx_xxx.tar.gz
  cd frp_xxx_xxx
  scp ./frps user@公网ip:/srv/frp/
  scp ./frps.ini user@公网ip:/srv/frp/

  # frpc和frpc.ini文件拷贝到内网服务器
  # ...
  ```

### 配置frps.ini并启动frps服务（公网服务器端）

#### frps.ini

  ```ini
  [common]
  bind_port = 8000
  ```

#### 启动frps服务

  ```bash
  ./frps -c ./frps.ini
  ```

### 配置frpc.ini并启动frpc服务（内网服务器端）

#### frpc.ini

  ```ini
  [common]
  server_addr = 公网服务器ip
  server_port = 8000
  # 至此启动./frpc -c ./frpc.ini公网和内网就可以建立数据通道
  # 配置ssh
  [ssh]
  type = tcp
  local_ip = 127.0.0.1
  local_port = 22
  remote_port = 6000
  # 使用内网服务器的端口22与公网端口6000通信，访问公网ip:6000即可访问内网ip:22
  ```

#### 启动frpc服务

  ```bash
  ./frpc -c ./frpc.ini
  ```

## 配置案例

### `frps.ini`

```yml
[common]
bind_port = 8000
# 开启http(s)服务
vhost_http_port = 80
vhost_https_port = 443
# 开启二级域名
subdomain_host = 123.com
# 开启udp用来p2p连接
# bind_udp_port = 8001
# 开启kcp
kcp_bind_port = 8000
# 连接池最大连接数
max_pool_count = 50
# 启用dashboard
dashboard_port = 8500
dashboard_user = admin
dashboard_pwd = admin
```

### `frpc.ini`

```yml
[common]
server_addr = 公网服务器ip
server_port = 8000
# 启用kcp(udp)协议
protocol = kcp
# 启用tls
tls_enable = true
# 预创建连接数
pool_count = 5

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
#use_encryption = true
use_compression = true

#[p2p_ssh]
# p2p穿透需要在本机再开一个frpc服务，sk输入下方的sk，不能100%穿透，至少我没成功
#type = xtcp
# sk一致才能访问
#sk = abcdefg
#local_ip = 127.0.0.1
#local_port = 22

[web]
type = http
local_port = 8080
# 二级域名
subdomain = test
```

## docker里部署frps服务

> 假设/srv/frp文件夹里有`frps`,`frps.ini`文件

### 新建`Dockerfile`文件

```Dockerfile
FROM ubuntu
# 指定工作目录，不存在会自动创建
WORKDIR /app
# 创建conf目录用来存放frps.ini
RUN mkdir conf
# 复制文件
COPY ./frps .
# 暴露端口
EXPOSE 8000 8500
# 挂载文件，需要在挂载位置放入frps.ini文件
VOLUME /app/conf
# 开启服务
CMD ["./frps", "-c", "./conf/frps.ini"]
```

### 新建`docker-compose.yml`文件

```yml
version: 3.1
services:
  frp_server:
    container_name: frps
    image: frp/server:latest
    ports:
      - "8000:8000/udp"
      - "80:80"
      - "443:443"
      - "8500:8500"
      - "6000:6000"
    volumes:
      # 记得在/var/docker/frp里放入frps.ini文件
      - /var/docker/frp:/app/conf
```

### 设置开机自启动

```bash
cd /lib/systemd/system/
touch frps.service
nano frps.service
# 写入以下内容
# [Unit]
# Description=Docker service deploy
# After=network-online.target

# [Service]
# ExecStart=/usr/local/bin/docker-compose -f /docker-compose位置/docker-compose.yml up -d
# ExecReload=/bin/kill -s HUP $MAINPID

# [Install]
# WantedBy=multi-user.target
systemctl daemon-reload
systemctl enable frps.service
```
