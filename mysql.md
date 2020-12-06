# 记录使用mysql的一些问题

> 当前系统ubuntu 18.04, mysql版本5.7

## 更改mysql数据存储路径

1. 关闭mysql服务 `systemctl stop mysql.service`

2. 复制原目录到目标路径 `cp -ar /var/lib/mysql /mnt/opt/mysql`

3. 更改mysql配置文件 `nano /etc/mysql/mysql.conf.d/mysqld.cnf`，ctrl+w搜索datadir，更改路径

4. 更改后重启mysql报错，提示没有权限，原因可能是apparmor的问题，需要更改`/etc/apparmor.d/usr.sbin.mysqld`文件，找到`Allow data dir access`更改为目标路径，重启apparmor服务又出现报错解析失败，可能需要安装`snapd`，再次运行`systemctl restart apparmor.service`成功，重启mysql服务`systemctl restart mysql.service`

## 开启远程访问

root账号无法开启远程访问，不知道为啥

1. 新增远程服务账号admin `GRANT ALL ON *.* TO admin@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;`

2. 刷新权限 `FLUSH PRIVILEGES;`

3. 修改配置文件 `nano /etc/mysql/mysql.conf.d/mysqld.cnf`，更改`bind_address = 0.0.0.0`

## load data权限问题

1. 导入提示`The MySQL server is running with the --secure-file-priv option so it cannot execute this statement`，修改mysql配置文件，找到或者新增`secure_file_priv = /mnt/opt/mysql-files`修改目录，值为空表示不限制

2. 运行`load data infile xxx`提示文件不存在没有权限，推测可能又是`apparmor`，`nano /etc/apparmor.d/usr.sbin.mysqld`找到`Allow data files dir access`更改为导入文件存放目录，重启`apparmor`和`mysql`解决