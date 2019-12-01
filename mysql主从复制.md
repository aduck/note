# mysql主从复制

## 为了解决什么问题

> 在腾讯云有个服务器，为了数据安全，同步保留一份在本地nas上，可以实时备份

## 主从复制原理

> 主服务器开启一个IO线程，从服务器开启IO线程和sql线程，通过binlog文件解析执行的sql，然后近似实时的在从服务器执行，binlog只会保存增删改的操作，不保存查询等

## 如何配置

> 预设主服务器ip 10.0.0.2 从服务器ip 10.0.0.3 端口都是默认3306

### master（主）服务器配置

* 设置server_id并开启binlog

```bash
vim /etc/mysql/mysql.conf.d/mysqld.cnf
# 在[mysqld]下添加
server-id               = 1
log_bin                 = /var/log/mysql/mysql-bin.log
# 重启mysql
```

* 查看是否已经开启

```bash
mysql> show variables like 'server_id'; # 1
mysql> show variables like 'log_bin'; # ON
```

* 建立从服务复制用户

```bash
mysql> grant replication slave on *.* to 'slaver'@'%' identified by '密码';
# 刷新权限
mysql> flush privileges;
```

* 锁表备份并记录`position`和`file`

```bash
# 锁表，为了防止备份时写入
mysql> flush table with read lock; # 这一步是选做
# 备份 -A表示全部 部分部分用-B 数据库1 数据库2 -E 包含事件触发 ，这一步是选做
mysqldump -uroot -p'密码' -B xx xx -E > mysql.bak #推荐
mysqldump -uroot -p'密码' -A > mysql.bak # 不推荐备份所有数据库，会把mysql.user等信息一并同步
# 查看master状态，记录File和Position字段
mysql> show master status;
# 待从服务器还原完成后解锁
mysql> unlock tables;
```

### slave（从）服务器配置

* 设置server_id并关闭binlog

```bash
# 还原备份(先拷贝一份bak文件到从服务器rsync -avzP xx@xx.xx.xx.xx:/home/xx/mysql.bak ./)
mysql -Pport -uroot -p'密码' < mysql.bak # 这一步是选做
# 更改mysqld配置文件[mysqld]后添加
server-id       = 2
skip-log-bin
# 重启mysql服务
```

* 配置复制参数

```bash
mysql> CHANGE MASTER TO
MASTER_HOST='10.0.0.2',
MASTER_PORT=3306,
MASTER_USER='slaver',
MASTER_PASSWORD='xxx',
MASTER_LOG_FILE='mysql-bin.000001',
MASTER_LOG_POS=857;
```

* 打开复制开关

```bash
mysql> start slave;
```

## 一些问题

### ubuntu18.04安装mysql8.0

```bash
# 从官网下载apt源包
wget https://dev.mysql.com/get/mysql-apt-config_0.8.13-1_all.deb
# 安装，弹窗选择安装版本和安装项，然后mysql的apt源就添加完成
dpkg -i mysql-apt-config_0.8.13-1_all.deb
# 更新
apt-get update
# 安装
apt-get install mysql-server
```

* 可能会出现依赖错误的问题，需要手动执行`apt-get install mysql-community-server`再`apt-get install mysql-server`

* 更改mysql默认端口`vim /etc/mysql/mysql.conf.d/mysqld.cnf`添加`port = xxx`重启服务即可

* 更改（忘记）密码

```bash
# 更改配置文件添加 skip-grant-tables
# 重启mysql
# 刷新权限
mysql> flush privileges;
# 更改密码
> alter user 'root'@'localhost' IDENTIFIED BY '新密码'; # 注意不能再用update mysql.user set xxx=password()这个方法了，会报语法错误
# 配置文件注释skip-grant-tables
# 重启mysql服务
```

* 开启远程访问

> 最好不要开启root账号的远程权限，可以登录root用户创建新用户`CREATE USER '用户'@'%' IDENTIFIED BY '密码';`，然后刷新权限即可，已经存在的可以`update mysql.user set host = '%' where user = 'xxx';flush privileges;`

### 主从同步错误处理

```bash
# 更改mysqld配置文件[mysqld]后添加
# 跳过所有错误
slave-skip-errors=all
# 跳过指定类型的错误
# slave-skip-errors=1146,1053
```
