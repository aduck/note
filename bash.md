# bash脚本

## 各种括号

```bash
() # 创建子shell
$() # 命令替换，等同于``
$(()) # 整数运算，等同于$[]和`expr 1+1`
${} # 字符替换，等同于$var
[] # 相当于test，注意空格啊 [ $a -eq 1 ]
(()) # 高级test，可以运算
[[]] # 高级test，可以组合命令
```

## 变量

* 赋值时=前后不能有空格
* readonly var设置只读变量
* unset var删除变量
* 单引号原样输出，双引号里可以有变量和转义符
* 获取字符串长度 ${#str}
* 获取数组长度 ${#arr[@]} / ${#arr[*]}

## 参数

```bash
$n # 传入的第几个参数 如./a.sh 1 2 则$1=1
$* # 以单引号方式输出所有参数'$1 $2'
$@ # 以"$1" "$2"的形式返回
$# # 返回传入的参数个数
$? # 最后的命令退出状态，返回0没错误 其它代表有错误
$$ # 脚本运行的当前进程ID号
```

## 流程控制

> -eq相等 -ne不相等 -gt大于 -lt小于 -ge大于等于 -le小于等于

* if...else

	```bash
	if test 1 -lt 2
	then
		echo "hello"
	elif test 2 -lt 3
	then
		echo "world"
	else
		echo "."
	fi
	```

* case

	```bash
	read -p "输入" num
	case $num in
		1|2|3|4|5) echo "输入了$num"
		;;
		*) echo "不合法"
		;;
	esac
	```

## 循环

* for...in

	```bash
	for item in 1 2 3 4
	do
		echo $item
	done
	```

* while

	```bash
	i=0
	while [ $i -lt 5 ]
	do
		echo $i
		let "i++"
	done
	```

## 函数

```bash
# 带返回值
demo(){
	read -p "输入第一个值" num1
	read -p "输入第二个值" num2
	return $(($num1+$num2))
}
demo
echo "和为 $?"
# 带参数
demo2(){
	echo "参数1：$1"
	echo "参数2：$2"
	echo "总参数个数：$#"
}
demo2 1 2 3
```

## 其它

* 引入文件 . ./a.sh或者source ./a.sh
* 读取输入 read -p "tip" str
* 菜单选择

	```bash
	select menu in 中国 美国 日本 韩国
	do
		case $REPLY in
		1|3|4)
			echo "亚洲"
			;;
		2)
			echo "北美洲"
			;;
		*)
			echo "error"
			;;
		esac
	done
	```

## 问题总结

* 获取uuid

	```bash
	cat /proc/sys/kernel/random/uuid
	```

* 变量赋值

	```bash
	# 相当于js中a = b || c
	a=${b:-c}
	```

* 获取当前时间

	```bash
	# 返回2019-03-01 00:00:00格式
	date "+%Y-%m-%d %H-%M-%S"
	# 只返回日期
	date +%d
	# 返回秒数
	date +%s
	# 返回前一分钟的分钟数
	date -d "-1 min" +%M
	```

* 判断文件夹是否存在，不存在就创建
	> -e name判断文件是否存在，-d name 判断是否存在并且为文件夹，另外还有-r(文件存在并可读) -w(文件存在并可写) -x(文件存在并可执行) -s(文件存在且不为空文件)

	```bash
	if [[ ! -d $dir ]]
	then
		mkdir -p $dir
	else
		echo "文件夹已存在"
	fi
	```

* 查看内置变量

	```bash
	env
	export
	```

## awk使用

```bash
# 作为管道使用
ls -la | awk 'BEGIN{print "Count"} /patten/ {action} END{print $1}'
#获取container_name，打印匹配到行的最后一列/patten/{action}
docker container ls | awk '/xxx/{print $NF}'
```
