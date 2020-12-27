## shell版本

当前系统支持的`shell`在 `/etc/shells`文件中查看，`linux`预设的是`bash(Bourbe Again SHell)`

## 命令行使用

### 命令换行

只需要在末尾添加`\`并回车就可以在下一行继续敲命令

`\`的作用只是转义下一个字符，因此`\`后不能有其它字符

```bash
[root@JJZ local]# ls \
> /home/
```

## 环境变量

### 常见变量

```bash
echo $HOME 	# 也可以这样写 echo ${HOME}
echo $USER
echo $MAIL
echo $SHELL
echo $$ 	# 当前使用shell的pid
```

### 查看环境变量

```bash
env # 查看全局变量(使用export导出的变量)
set # 查看局部变量(没用export导出的变量)
export
```

### 定义环境变量

子进程只能使用父进程的全局变量，不能使用局部变量

```bash
name=JJZ 		 # 子进程不可见
export name=JJZ  # 子进程可见
export var=$(ls -a) # 取出命令结果赋值给环境变量

declare -x name  # 把局部变量变为全局变量
declare +x name  # 把全局变量变为局部变量
```

### 转义

```bash
name=JJZ
myname="my name is $name" # my name is JJZ
myname='my name is $name' # my name is $name
```

### 删除环境变量

```bash
unset name
```

### 环境变量文件

```bash
# 系统级别
- `/etc/environment`
- `/etc/profile`
- `/etc/profile/*.sh`

# 用户级别
- `~/.bash_profile`
- `~/.bashrc`
- `~/.profile`
```



## 命令别名

```bash
# 起别名
alias lla='ll -a'
# 取消别名
unalias lla
# 列出所有别名
alias
```

## 显示信息配置

### 主机名

```bash
[root@JJZ ~]# 
# 在/etc/hostname中修改@后面的名字
```

### 修改命令提示字符

也就是这个`[root@JJZ ~]# `

```bash
# 修改环境变量PS1即可, 规则如下
# 默认 PS1='[\u@\h \W]\$ '
\d ：可显示出『星期 月 日』的日期格式，如："Mon Feb 2" 
\H ：完整的主机名	# 在/etc/hostname中修改
\h ：仅取主机名在第一个小数点之前的名字
\t ：显示时间，为 24 小时格式的『HH:MM:SS』
\T ：显示时间，为 12 小时格式的『HH:MM:SS』
\A ：显示时间，为 24 小时格式的『HH:MM』
\@ ：显示时间，为 12 小时格式的『am/pm』
\u ：目前使用者的账号名称，如『dmtsai』；
\v ：BASH 的版本信息
\w ：完整的工作目录名称，由根目录写起的目录名称。但家目录会以 ~ 取代；
\W ：利用 basename 函数取得工作目录名称，所以仅会列出最后一个目录名。
\# ：下达的第几个指令。
\$ ：提示字符，如果是 root 时，提示字符为 # ，否则就是 $
```

### 登录bash欢迎信息

在`/etc/issue, /etc/motd`中修改

```bash
[root@JJZ ~]# cat /etc/issue
\S
Kernel \r on an \m
# 对应信息如下
\d 本地端时间的日期； 
\l 显示第几个终端机接口； 
\m 显示硬件的等级 (i386/i486/i586/i686...)； 
\n 显示主机的网络名称； \O 显示 domain name； 
\r 操作系统的版本 (相当于 uname -r) \t 显示本地端时间的时间； 
\S 操作系统的名称；
\v 操作系统的版本

[root@JJZ ~]# cat /etc/motd
Welcome to Alibaba Cloud Elastic Compute Service !
```

## 通配符

```bash
*
?
[abcd] 	# 匹配abcd任意一个
[0-9]	# 匹配0到9任意一个, 多个区间用 , 隔开

```

## 管道操作

```bash
sort # 按行排序
cat /etc/passwd | sort -t ':' -k 3 -r # 按:分隔后的第三个区间降序排序, 
lp:x:     4  :7:lp:/var/spool/lpd:/sbin/nologin
adm:x: 	  3  :4:adm:/var/adm:/sbin/nologin
daemon:x: 2  :2:daemon:/sbin:/sbin/nologin
bin:x:	  1  :1:bin:/bin:/sbin/nologin
root:x:	  0  :0:root:/root:/bin/bash

uniq # 删除重复的行

wc # 用于列出结果数量
ls / | wc -l # 按行计算
		  -w # 按单词计算
		  -m # 按char计算
		  -c # 按byte计算
```

## shell脚本

### 执行shell

```bash
# 在当前shell中执行
. profile.sh
source profile.sh
# 在子shell中执行
./profile.sh	# 需要x权限
bash profile.sh
```

### 快捷键及基本命令

```bash
^D # 退出shell
^A # 移动光标到行首
^E # 移动光标到行尾
^U # 删除光标前的字符
^K # 删除光标后的字符
^S # 锁定屏幕显示
^Q # 解锁
^Z # 当前进程暂停运行, 通过fg拉回来
sleep 3000 &		# 后台执行, 用户退出后程序也退出
nohup sleep 3000 &	# 后台执行, 用户退出仍然执行
nohup sleep 3000 1>nohup.out 2>&1 &
nohup sleep 3000 &>nohup.out &

read # 读取键盘输入
read -p "提示信息: " variable
read -t <timeout>   variable
read -n <char_nums> variable
```

### 特殊符号

```bash
~  # 家目录
!  # 执行历史命令
	!!		# 执行上一条命令
	!number # 执行第number条命令
	!string # 匹配最近的一条命令
	!$ 		# 上一个命令的最后一个参数
	
$  # 取环境变量值
	$? # 上一条命令的返回值
		0   # 成功, true
		1   # 失败, false
	$1 # 取第一个参数的值 $2..以此类推
	$! # 上一个进程的pid
	$$ # 当前进程的pid
	$# # 参数的个数
	$0 # 脚本名
	$* # 所有参数
	$@ # 所有参数
&  # 后台执行
*  # 匹配任意字符
?  # 匹配一个字符
[] # 匹配其中一个
	[abc] [a-z] [0-9,a-z]
	[^abcd]	# 匹配非abcd的任意一个
{} # 集合
	mkdir /home/{aaa,bbb}
	touch /home/{1..5}.txt
	cp /root/{1.txt,2.txt} # cp /root/1.txt /root/2.txt
() # 在子shell中执行
;  # 一行多条命令用;隔开
&& # 和;的区别是满足短路求值原则
|| # 同上
\  # 转义
|  # 管道
`` # 解析为命令并提前执行 echo "hello `echo world`" ==> hello world
$()# 同上
"" # 解释变量
'' # 不解释变量

()	  # 子shell中执行命令
(())  # 数值比较 C风格
$()   # 执行命令
$(()) # 算数运算
[]    # 条件测试
[[]]  # 条件测试, 支持正则
${}	  # 取值
$[]   # 算数运算

>
>>
2>
2>>
2>&1
<
```

### echo

```bash
echo -e "a\nb"  # 转义\n
echo -e "\e[1;31mtext\e[0m"	# 红
echo -e "\e[1;32mtext\e[0m"	# 绿
echo -e "\e[1;33mtext\e[0m" # 黄
echo -e "\e[1;34mtext\e[0m" # 蓝
```

### 运算

```bash
# 算数运算
ans=`expr 1 + 2` # 必须有空格
ans=$((1 + 2))
ans=$[1 + 2]
let ans=2+3		 # 不能有空格

# 长度, 删除, 替换
url=www.baidu.com
echo ${#url}	 # 获取长度 13
# 从前面开始删除 *为通配符 原url的值不变
echo ${url#www.} # 从前面删除www.  ==> baidu.com
echo ${url#*.}   # 删到第一个.的位置 ==> baidu.com
echo ${url##*.}  # 删到最后一个.的位置 ==> com
# 从后面开始删除
echo ${url%.com} # 删除.com		==> www.baidu
echo ${url%.*}   # 删到第一个.的位置 ==> www.baidu
echo ${url%%.*}  # 删到最后一个.的位置 ==> www
# 按索引取值
echo ${url:4:5}  # 从索引4开始取5个 ==> baidu
# 替换
echo ${url/old/new} # 替换一个
echo ${url//old/new} # 替换所有
# 默认值 - :-
echo ${url-www.baidu.com} # 如果url没有定义则赋一个默认值www.baidu.com
echo ${url:-www.baidu.com} # 如果url没有定义或为空值则赋一个默认值www.baidu.com
```

### 逻辑判断

```bash
### 判断对象主要分为3种: 
`- 文件`
`- 数值`
`- 字符串`

-a : # &&用于 [ 外
-o : # ||用于 [ 外
!

# 文件判断
# 判断是不是目录
if [ -d $dir ]; then
-f # 普通文件
-e # 文件存在, 此处文件是各种文件的统称
-r # 当前用户有读权限
-w # 当前用户有写权限
-x # 当前用户有执行权限
-L # 链接文件
-b # 块设备文件
-c # 字符设备文件

# 数值比较
[ $UID -eq 0 -a $age -gt 18 ]


# 字符串比较
[ $USER = 'root' ] # ==也行
!=
-z	# 长度为0返回true
-n	# 长度不为0返回true
```

