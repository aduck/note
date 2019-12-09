# ssh代理

> 小tip，截取指定字符左边内容 ${str%char*}，截取指定字符右边内容 ${str#*char}

```bash
-N # 只转发
-f # 后台运行
-v # debug模式运行
-L # 正向代理
-R # 反向代理
-D # socket
```

## 正向代理

在本机通过远端ip访问其它资源

```bash
# 语法 ssh -L 本机ip:本机port:要访问的ip:要访问的port 中间服务器
```

## 反向代理

在其它机器通过远端ip访问内网资源

```bash
# 语法 ssh -R 远端ip:远端port:本机ip:本机port
# 设置反向代理可能需要远端设置/etc/ssh/sshd_config添加配置GatewayPorts yes
```

## socket模式

在本机通过远端ip访问其它资源

```bash
# 语法 ssh -D 本机ip:本机port:远端ip:远端port
# 然后浏览器或者其他应用设置代理localhost:本机port
```
