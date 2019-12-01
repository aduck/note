# NAS

> 一个小提示，创建一个不能登录的账户 useradd -s /sbin/nologin -M xxx

## 迅雷远程下载

```bash
# 通过docker实现
docker run -d -v ~/download/:~/app/TDDOWNLOAD yinheli/docker-thunder-xware
# 通过docker logs containerid获取激活码
# 远程下载绑定设备 http://yuancheng.xunlei.com/
```

## aria2下载器

```bash
docker pull xujinkai/aria2-with-webui:latest
```

## samba服务器

* 安装samba

  ```bash
  sudo apt-get install samba
  ```

* 创建共享目录

  ```bash
  sudo mkdir -p /home/path/share
  cd /home/path/
  chmod 777 /home/path/share
  ```

* 修改配置/etc/samba/smb.conf

  ```bash
  # 添加
  [share]
  comment = share nas files
  path = /home/path/share
  available = yes
  browseable = yes
  force user = nobody
  force group = nogroup
  writable = yes
  public = no
  create mask = 0777
  directory mask = 0777
  valid users = xxx
  ```

* 创建samba账号

  ```bash
  cd /etc/samba/
  touch smbpasswd
  smbpasswd -a xxx # Failed to add entry for user xxx 出现这个提示，需要新增用户 useradd -s /sbin/nologin -M xxx
  systemctl enable smbd # 设置开机启动
  systemctl restart smbd # 重启服务
  ```

* 可以访问

  ```bash
  \\123.234.22.213\share # win
  smb://123.223.42.435/share # mac
  ```

## rsync同步服务器

> 请使用绝对路径

### 服务端

* 查看是否安装rsync

  ```bash
  dpkg -l | grep rsync
  ```

* 修改配置

  ```bash
  vi /etc/rsyncd.conf
  ### 配置如下
  uid = rsync
  gid = rsync
  use chroot = no
  max connections = 200
  timeout = 300
  pid file = /var/run/rsyncd.pid
  lock file = /var/run/rsync.lock
  log file = /var/log/rsyncd.log
  ignore errors
  read only = false
  list = false
  hosts allow = 172.16.1.0/24
  hosts deny = 0.0.0.0/32
  auth users = rsync_user
  secrets file = /etc/rsync.password
  [backup]
  comment = "backup dir"
  path = /var/backup
  ```

* 创建rsync用户

  ```bash
  id rsync # 查看是否存在rsync账号
  useradd -s /sbin/nologin -M rsync
  ```

* 创建备份目录并修改权属

  ```bash
  mkdir /var/backup
  chown -R rsync.rsync /var/backup
  ```

* 创建rsync密码文件，并修改权限

  ```bash
  touch /etc/rsync.password
  vi /etc/rsync.password
  # 账号:密码
  rsync_user:123456
  chmod 600 /etc/rsync.password
  ```

* 启动rsync服务

  ```bash
  rsync --daemon # 启动
  systemctl enable rsync # 设置开机启动
  ```

### 客户端

* 配置密码文件并修改权限

  ```bash
  touch /etc/rsync.password
  vi /etc/rsync.password
  # 只需要密码
  123456
  chmod 600 /etc/rsync.password
  ```

* 开始同步

  ```bash
  rsync -avzP /test rsync_user@172.16.1.41::backup --password-file=/etc/rsync.password # -a归档递归方式传输并保存文件属性相当于-rtopgDl -v显示详细输出 -z压缩 -P显示进度
  ```

## nas配套

### ubuntu配置固定ip（ubuntu 18.04）

> 这步最好在路由器配置

```bash
cd /etc/netplan # 查看配置文件名称
vi 01-network-manager-all.yaml # 配置
network:
  ethernets:
    enp0s3: # 网卡名称
      addresses: [192.168.199.101/24] # 本机ip及掩码
      dhcp4: no #dhcp4关闭
      dhcp6: no # dhcp6关闭
      gateway4: 192.168.199.1 # 网关
      nameservers:
        addresses: [114.114.114.114, 8.8.8.8] # 设置dns
  version: 2
netplan apply # 使配置生效
```

### ubuntu挂载新硬盘

* 查看硬盘和分区

  ```bash
  fdisk -l
  ```

* 硬盘分区（大于2T用parted）

  ```bash
  fdisk /dev/sdb # 注意名称是从第一步来的
  -> m获取帮助
  -> n创建新分区 -> p主分区 -> 1只分一个区
  -> m获取帮助
  -> w保存并退出
  ```

* 格式化分区

  ```bash
  mkfs -t ext4 /dev/sdb1 # 注意分区名是从上一步来的，格式化成ext4
  ```

* 设置挂载点挂载并永久保存

  ```bash
  mkdir -p /media/storage # 位置和名称自取
  ls -l /dev/disk/by-uuid/ # 查看分区uuid
  vim /etc/fstab # 写入配置 UUID=xxx /media/storage ext4 defaults 0 2
  ```

### wake on lan

> 为了延长硬盘寿命tt

* 设置bios（根据具体型号来）

  ```bash
  Power Management Setup
  -> PME Event Wake Up 设置 enable
  ```

* 查看是否支持wol

  ```bash
  apt-get install ethtool
  ethtool ens33 # 通过ip address获取对应网卡
  ```

* 查看mac地址和ip地址

  ```bash
  ip address
  ```

* 路由器设置端口映射（外网唤醒需要）

### 磁盘健康状态（也可以使用smartctl）

* 坏道检测

  ```bash
  badblocks -vs -o xxx /dev/sda # -v显示执行详细情况 -s显示进度 -o输出到文件或者直接>
  ```

* smartctl磁盘健康管理

  ```bash
  # 安装
  apt-get install smartmontools
  # 查看服务运行状态
  systemctl status smartmontools
  # 检查磁盘smart功能是否启用
  smartctl -i /dev/sdb1
  # 启用/禁用smart磁盘功能
  smartctl -s on/off /dev/sdb1
  # 查看磁盘详细smart信息
  smartctl -a /dev/sdb1 # IED类型
  smartctl -a -d ata /dev/sdb1 # SATA类型
  # 显示磁盘总体健康状态
  smartctl -H /dev/sdb1
  # 测试硬盘（分为long测试和short测试）
  smartctl --test=long/short /dev/sdb1
  # 查看测试结果
  smartctl -l selftest /dev/sdb1
  # 显示错误日志
  smartctl -l error /dev/sdb1
  ```

### 温度监测

> GUI工具Psensor，可以查看cpu硬盘温度和使用率，依赖lm-sensors hddtemp

```bash
apt-get install lm-sensors hddtemp # 安装lm-sensors hddtemp
sensors-detect # 检测传感器
sensors # 测试运行
apt-get install psensor # 安装psensor
```

### crontab定时任务

* 语法

  ```bash
  crontab [-u user] { -l | -r | -e } # -l查看crontab的内容 -r移除当前用户的所有任务 -e编辑任务内容
  ```

* 说明

  ```bash
  #用户级的任务建议crontab -e，系统级的任务直接vi /etc/crontab
  #crontab -e设置的任务会存档在/var/spool/cron/xxx，修改请使用crontab -e
  #crontab操作的会被记录到/var/log/cron中，可以通过vi /etc/rsyslog.d/50-default.conf cron.* 前的#删掉然后重启rsyslog服务和cron服务
  #优先级/etc/cron.allow>/etc/cron.deny，用来配置哪些用户(不)可以启动crontab，都不存在则只有root可以
  ```

* 书写格式

  ```bash
  分(0-59) 时(0-23) 日(1-31) 月(1-12) 周(0-7) 脚本
  *(所有) ,(与) -(范围) /n(每隔)
  ```

## 用户/用户组相关

* 查看

  ```bash
  cat /etc/shadow # 查看安全用户账号信息
  cat /etc/passwd # 查看账号信息
  cat /etc/group # 查看组账号信息
  cat /etc/gshadow # 查看安全组账号信息
  ```

* 添加

  ```bash
  # 添加用户
  useradd [选项] 用户名 # -g指定主组 -G指定附加组 -m创建home目录 -M不创建home目录 -s指定登录的shell
  # 添加组
  groupadd [选项] 组名
  ```

* 更新

  ```bash
  # 更新用户
  usermod [选项] 用户名 # -a添加到附加组 -s登录shell -d指定home目录 -m移动到新home目录
  ```

* 删除

  ```bash
  # 删除用户
  userdel [选项] 用户名 # -f强制 -r同时删除home目录
  # 删除组
  groupdel [选项] 组名
  ```

* 更改密码

  ```bash
  passwd [用户名]
  ```

* 切换用户

  ```bash
  su username
  ```

## 防火墙配置

> 存在于外网与内网之间ufw

### 常用命令

```bash
ufw status # 查看防火墙状态
ufw enable/disable/reload # 打开/关闭/重载防火墙
ufw allow/deny 137,138/udp|222|139,445/tcp # 允许/不允许udp端口137和138，tcp和udp端口222，tcp端口139,445
ufw insert num allow 80/tcp # 添加规则并设置num
ufw delete rule/num # 通过rule或num删除规则
ufw allow 3306/tcp comment 'mysql' # 添加注释
```

### app配置

> 目录/etc/ufw/application.d

* 新增(需要新建app文件)/修改/删除

  ```bash
  # 文件内容格式
  [Name]
  title=xxx
  description=xxx
  ports=xxx
  # 然后执行
  ufw app update Name/all
  ```
