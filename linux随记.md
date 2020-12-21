# Linux随记

> 记录不知道的linux命令

## cat/more/less/tail命令

`cat` 直接输出文件所有内容  
`more` 分页输出文件内容，支持空格或者f翻页，b返回上一页，内容读取完成自动退出，会加载整个文件  
`less` 最常用的读取文件内容命令，支持分页查看，支持`/xx`或者`?xx`搜索，不会一次加载整个文件  
`tail` 用来实时查看文件变更，默认展示最后10行  

```bash
tail
# exp
tail -f messages.log # 实时查看文件
tail +20 messages.log # 显示20行到结尾
# 常用参数
-f # 实时监测，直至ctrl+c结束
-n n # 查看最后n行内容
-c n # 显示n字节内容
```

## sort

用来处理输出的排序

```bash
# exp
sort -k 1 -r # 按照第一列降序排列
# 常用参数
-k pos1[,pos2] # 按照某列排序
-r # 降序，默认升序
-f # 忽略大小写
-u # 去除重复行
-o # 将结果导出文件
```

## ps

查看进程和线程情况

```bash
# exp
ps -eo pmem,pcpu,rss,vsize,args | sort -k 1 -r # 查看进程占用情况降序
ps -ef # 展示UID,PID,PPID,C,STIME,TTY,TIME,CMD等信息
ps -aux # 展示USER,PID,%CPU,%MEM,VSZ,RSS,TTY,STAT,START,TIME,COMMAND等信息
# 常用参数
-A/-e # 显示所有(用户)进程
-f # 格式化输出UID,PID,PPID,C,STIME,TTY,TIME,CMD
-o # 指定显示哪些信息
```

## top

实时查看cpu、内存等占用情况

```bash
# shift+m 按照内存使用情况排序
# f 编辑显示的列
```

## history

查看命令操作记录

```bash
history # 显示所有记录
history 10 # 显示最近10条记录
history | less # 分页查看记录
history -c # 清空记录
!! # 执行上次命令
!xx # 执行上次以xx开头的命令
```

## upgrade忽略更新

执行apt-get upgrade时有时候想要忽略某些包，可以使用`apt-mark`命令，忽略`apt-mark hold package`，恢复`apt-mark unhold package`

```bash
apt-mark hold package1 package2
apt-mark unhold package1 package2
```

## find命令

> find target [params]

```bash
# 基础
find / -name demo # 查找根目录下所有名为demo的文件/夹
find / -name *.txt # 查找txt结尾的文件
# 按用户/用户组查找
find / -user root # 查找所有人为root的文件
find / -group mysql # 查找属于mysql用户组的文件
find / -user root -o -group mysql # 查找所有人为root或者用户组为mysql的文件
find / -user root -a -group mysql # 查找所有人为root并且用户组为mysql的文件
find / -not -user root # 取反
# 查找层级
find / -mindepth 1 -maxdepth 2 -name passwd # 查找最小层级为1最大层级为2的名为passwd的文件
# 按大小查找
find / -size 20K # 查找大小为20K的文件
find / -size -20K # 查找小于20K的文件
find / -size +20K # 查找大于20K的文件
# 按类型查找
find / -type f # f普通文件 d目录 b块设备 s套接字 c字符设备 l链接 p管道
# 按权限查找
find / -perm 444
# 按修改时间查找
find / -ctime 10 # 查找更新距离现在10分钟的文件
find / -ctime +10 # 查找更新距离现在超过10分钟的文件
find / -ctime -10 # 查找更新距离现在小于10分钟的文件
# 查找后续操作
find / -name test.txt -exec chmod +w {} \; # 查找名为test.txt的文件后添加可写权限
```
