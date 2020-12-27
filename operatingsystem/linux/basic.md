## 目录结构

![image-20201219225044144](https://gitee.com/p8t/picbed/raw/master/imgs/20201219225045.png)

```bash
/usr：
usr 是 unix shared resources(共享资源) 的缩写，
用户应用程序放在此目录下，类似于windows下的program files目录。

/usr/bin：
普通用户使用的命令

/usr/sbin：
超级用户使用的命令 s可以认为是system或super

/bin：
软链接，指向/usr/bin

/sbin：
软链接，指向/usr/sbin

/home：
用户家目录

/root：
root家目录, 与/home同性质

/boot：
与系统启动有关

/dev：
dev 是 Device(设备) 的缩写, 与外部设备有关

/etc：
etc 是 Etcetera(等等) 的缩写, 存放配置文件

/lib：
lib 是 Library(库) 的缩写这个目录里存放着系统最基本的动态连接共享库，其作用类似于 Windows 里的 DLL 文件。几乎所有的应用程序都需要用到这些共享库。

/opt：
opt 是 optional(可选) 的缩写，这是给主机额外安装软件所摆放的目录。比如你安装一个ORACLE数据库则就可以放到这个目录下。默认是空的。

/proc：
proc 是 Processes(进程) 的缩写，/proc 是一种伪文件系统（也即虚拟文件系统），存储的是当前内核运行状态的一系列特殊文件，
这个目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访问这个目录来获取系统信息。
这个目录的内容不在硬盘上而是在内存里，我们也可以直接修改里面的某些文件

/lost+found：
这个目录一般情况下是空的，当系统非法关机后，这里就存放了一些文件。

/media：
linux 系统会自动识别一些设备，例如U盘、光驱等等，当识别后，Linux 会把识别的设备挂载到这个目录下。

/mnt：
系统提供该目录是为了让用户临时挂载别的文件系统的，我们可以将光驱挂载在 /mnt/ 上，然后进入该目录就可以查看光驱里的内容了。

/selinux：
这个目录是 Redhat/CentOS 所特有的目录，Selinux 是一个安全机制，类似于 windows 的防火墙，但是这套机制比较复杂，这个目录就是存放selinux相关的文件的。

/srv：
该目录存放一些服务启动之后需要提取的数据。

/sys：
这是 Linux2.6 内核的一个很大的变化。该目录下安装了 2.6 内核中新出现的一个文件系统 sysfs 。
sysfs 文件系统集成了下面3种文件系统的信息：针对进程信息的 proc 文件系统、针对设备的 devfs 文件系统以及针对伪终端的 devpts 文件系统。
该文件系统是内核设备树的一个直观反映。
当一个内核对象被创建的时候，对应的文件和目录也在内核对象子系统中被创建。

/tmp：
tmp 是 temporary(临时) 的缩写这个目录是用来存放一些临时文件的。

/usr/src：
内核源代码默认的放置目录。

/var：
var 是 variable(变量) 的缩写，这个目录中存放着在不断扩充着的东西，
我们习惯将那些经常被修改的目录放在这个目录下。包括各种日志文件。

/run：
是一个临时文件系统，存储系统启动以来的信息。当系统重启时，这个目录下的文件应该被删掉或清除。如果你的系统上有 /var/run 目录，应该让它指向 run。
```

## 一、文件管理

### 文件分类

```bash
# 通过第一位判断 -是普通文件 d是文件夹
-rwxrwxrwx # 普通文件
drwxrwxrwx # 目录文件
l...	# 链接文件
b...	# block 块设备文件
c...	# character 字符设备文件
s...	# socket 套接字文件
p...	# pipe 命名管道文件
```

### 文件操作

```bash
# 创建文件夹
mkdir -p a/b/c # 递归创建
	  -p --parents
mkdir {a,b} # 同时创建a和b

# 复制
cp -r <src> <dest> # 递归复制
   -r --recursive

# 删除
rm -rf <file>
	# ? 匹配一个
	# * 匹配任意
```

### 查看文件

```bash
cat <file>
tac <file> # 倒着显示
    
head -10 <file> # 查看前10行

tail -10 <file> # 查看后10行
	-f # 刷新追加的文件内容, 用于实时监控文件变化, 看日志变化很方便

more <file> # 对于行数过多的文件, cat直接跳到最后面显示, 
			# more可以翻页显示: 回车下一行, 空格翻页
less <file> # more不支持往前翻页, less支持, PageUp
# 在more或less状态输入 /text 可以搜索text
```

## 二、用户和用户组

### 相关文件

`/etc/passwd`文件，7列，用于存放用户基本信息

```bash
root  : x : 0 : 0 : root :  /root :  /bin/bash
# root: 用户名, 登录系统的名字
# x: 密码占位符, 也就是密码信息不在这
# 0: uid, 用户ID
#	 0: root账号
#	 1~499: 系统账号
# 	 1000+: 普通用户账号
# 0: gid, 用户主组ID
# root: description, 描述
# /root: HOME, 用户家目录
# /bin/bash: 用户登录使用的shell
```

`/etc/shadow`，9列，用于存放用户密码信息

```bash
root : $6$HSH.l/JBs$DSQ.ttLQwHc.Q6qdD/ : 18573 : 0 : 99999 : 7 :::
# 1. root: 用户名
# 2. $6$HSH.l...: 加密后的密码
	  # $6开头表示SHA512加密
	  # * 表示账号被锁定, 不能登录
	  # !! 表示密码过期
# 3. 18573: 最后一次修改时间, 从1970.01.01开始的天数
# 4. 0: 两次密码修改时间间隔. 如果为5则说明5天内只能修改一次密码
# 5. 99999: 密码有效期, 超过此时间需要修改密码
# 6. 7: 警告时间, 密码到期前7天告诉用户需要修改密码了
# 7. 不活动时间, 用户不活跃达到天数后,冻结
# 8. 失效时间, 账号达到天数就失效
# 9. 保留
```

`/etc/group`，4列，用于存放用户组信息

```bash
root:x:0:
# 1. root: 组名
# 2. x: 组密码占位符
# 3. 0: GID, 组id
# 4. 组成员
```

### 相关命令

```bash
# 我是谁
whoami

# 【以下针对root操作】

# 添加新用户, 不指定主组则同时为它创建一个用户主组
useradd <username>
	-u <uid> # 指定uid
	-d <dir> # 指定家目录
# 修改密码 (弹出对话), 不加参数修改自己密码
passwd <user> # <user> 代表可以是用户名, 也可以是uid
# 查询用户组信息
id <user>
# 删除用户
userdel -r <user>
	-r # 移除家目录和邮件目录: /var/spool/mail/<username>
# 修改用户属性
usermod <user>
	-g # 修改基本组
	-G # 修改附加组, 加上-a参数追加附加组

# 添加用户组
groupadd <groupname>
	-g # 指定gid
# 删除用户组
groupdel <group>
# 修改用户组信息
groupmod <group>
	-g # 指定gid
# 组内添加或移除用户
gpasswd <group>
	-a # 添加用户到组内 --add
	-d # 从组内移除用户 --delete
```

## 三、权限

### 用户提权

在`/etc/sudoers`文件，记录了哪些用户（以用户组的形式配置）拥有哪些特权命令，普通用户通过命令前面加上`sudo`来执行这些特权命令

```bash
# 登录到其它账号
su - <user> # 加 - 切换环境变量
# 执行特权命令
sudo <command>
```

### 文件基本权限（UGOA）

文件默认644，目录默认755

对于目录rwx的含义如下

- r：列出该目录下的文件，没有这个权限w没意义
- w：添加，删除，重命名文件。文件内容的修改则和文件权限有关
- x：进入工作目录，也就是cd，没有这个权限上面两个没意义

```bash
d   rwx  r-x   r-x.   4    root  root    58  Dec 20 10:39 file
#	属主  属组  其它  链接数  属主   属组  文件大小
#	User Group Other

# 修改文件权限
chmod 777  1.txt # UGA对该文件权限均为rwx
chmod u=rw 1.txt # U权限为rw
chmod u+r  1.txt # U添加r权限
chmod u-r  1.txt # U失去r权限
chmod ug+r 1.txt # UG添加r权限
chmod +r   1.txt # UGO添加r权限
chmod a=rw 1.txt # UGA权限均为rw a代表all
	-R # 递归修改权限
	
# 高级权限suid
suid是针对文件所设置的一个特别的权限
功能: 调用文件的用户, 临时具备属主的能力(对该文件的权限和属主一样)
chmod u+s 1.txt # 如果有x权限显示s; 如果没有x权限显示S

# 修改文件所属
chown <user> 1.txt			# 修改文件属主
chown :<group> 1.txt		# 修改文件属组
chown <user>:<group> 1.txt 	# 修改文件属主及属组
```

### 文件特殊权限

```bash
# 增加特殊权限
chattr +i 1.txt # i表示不能对该文件进行任何操作
# 列出特殊权限
lsattr 1.txt
```

### 权限掩码

```bash
# 文件创建时都有默认权限, 影响权限值的是umask

# 查询权限掩码
umask  # 默认022, 对于目录文件就是755, 对于文件需要-x
# 设置权限掩码
umask 000
```

## 四、进程管理

```bash
ps aux
	a # 所有程序
	u # 以用户为主的格式来显示
	x # 不以终端机来区分
	--sort=+%cpu # 以占用cpu升序排列
	--sort=-%cpu # 降序排列
USER PID %CPU %MEM VSZ    RSS  TTY  STAT  START  TIME  COMMAND
root 1   0.0  0.3  128392 5932  ?   Ss    14:24  0:03  /usr/lib/systemd/systemd
# root:  	运行该程序的用户
# PID:	 	进程标识
# VSZ:   	占用的虚拟内存(磁盘)
# RSS:   	占用实际内存
# TTY:   	进程运行的终端
# STAT:  	进程状态
	R-running
	T-stop
	S-sleep
	Z-zombie # 程序终止却无法移出内存
# START: 	进程启动时间
# TIME:		进程占用CPU的总时间
# COMMAND:	进程对应的程序文件

# 查看父子进程关系
ps -ef
UID  PID  PPID  C  STIME  TTY  TIME      CMD
root  1    0    0  19:19   ?   00:00:01  /usr/lib/systemd/systemd
# PPID: 父进程PID, 0代表没有. 当一个进程杀不死, 可以杀它爹

# 显示指定列
ps axo user,pid,ppid,cmd

top -d 3 -p <...pid> # 3秒刷新一次, 显示指定进程
PID USER  PR NI  VIRT RES SHR  S  %CPU %MEM TIME+ COMMAND
PR, NI: # 优先级
S: # 状态
VIRT,RES,SHR: # 内存
```

### 使用信号控制进程

```bash
# 列出支持的信号
kill -l

kill <sign> <pid>
	 -9  # 立刻杀死进程
	 -15 # 正常关闭进程
```

### 进程优先级

进程优先级分为两种，NI和PR。我们只能调整NI，范围是[-20, 19]，数字越小优先级越高。调整完后会映射到系统的PR优先级，会把我们设置的值再加上20得到对应的PR优先级，PR优先级我们不能调整。这样做的意义是，无论我们把进程优先级调的多高也不会影响系统运行。换句话说，我们能调整的优先级范围只是系统优先级范围的一小部分。

```bash
nice -n <num> <command> # 进程启动时设置优先级
renice -n <num> <pid>   # 重新设置优先级
```

### /proc/

这是一个虚拟文件系统，是系统内存的映射，可以通过它获取系统信息

```bash
/proc/cpuinfo
/proc/meminfo
/proc/cmdline # 系统内核 目前为3.10.0
```

## 五、管道和重定向

### FD

File Descriptors，文件描述符/文件句柄。进程使用文件描述符来管理打开的文件

| FD    | 通道名   | 描述         | 默认连接 | 用途  |
| ----- | -------- | ------------ | -------- | ----- |
| 0     | stdin    | 标准输入     | 键盘     | 只读  |
| 1     | stdout   | 标准输出     | 终端     | 只写  |
| 2     | stderr   | 标准错误输出 | 终端     | 只写  |
| 3~255 | filename | 其它文件     | none     | 读/写 |

可以在 `/proc/<pid>/fd`中查看进程使用的FD

### 重定向

```bash
date > time.txt 相当于 date 1> time.txt # 意思是调用值为1的FD输出到time.txt

date 2> time.txt # 引导错误输出到time.txt
date &> time.txt # 正确输出和错误输出都会输入到time.txt, 这个&和后台运行的&不一样
date 1> info.txt 2> err.txt # 分开输出到文件

date > time.txt  # 覆盖内容
date >> time.txt # 追加内容

# 想要重定向, 但又不想输入到文件就可以重定向到下面位置
/dev/null
```

### 管道

命令1的输出作为命令2的输入

```bash
cat /etc/passwd | grep root | tail -1

# tee会把前一个结果输入到文件, 同时传给下一个, 也就是三通管道
# 不加文件名就会输出到屏幕
cat /etc/passwd | tee info.txt | tail -5

# 对于rm等不接受前面信息的命令, 可以使用xargs让它接受
# 下面这条命令就会把del.txt文件内记录的字符串 当作rm要删除的文件路径
cat del.txt | xargs rm -rfv
```

## 六、存储管理

> 环境：我在虚拟机上新增了一块5G的磁盘，对应的文件为/dev/sdb

### 磁盘抽象描述文件

```bash
/dev/sda
	s: # SATA, 串口
	d: # 代表磁盘
	a: # 第一块
/dev/sdb # 第二块
```

### 分区方式

```bash
MBR: 最大磁盘容量2T, 最多4个主分区
GPT:
```

### 新增磁盘使用步骤

也就是从插上磁盘到能正常使用需要的步骤

- 1、分区（MBR或GPT）
- 2、格式化（文件系统）
- 3、挂载（mount）

### 查看磁盘信息

```bash
# 查看磁盘的抽象文件及分区
ll /dev/sd*
brw-rw----. 1 root disk 8,  0 Dec 22 15:13 /dev/sda
brw-rw----. 1 root disk 8,  1 Dec 22 15:13 /dev/sda1
brw-rw----. 1 root disk 8,  2 Dec 22 15:13 /dev/sda2
brw-rw----. 1 root disk 8, 16 Dec 22 15:13 /dev/sdb # 新增的5G磁盘
# 第一个b代表块设备文件
# sda1和sda2是磁盘sda的不同分区; sdb是另一块磁盘

# 列出所有可用块设备的信息, 分区, 挂载信息
lsblk # list block
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   20G  0 disk 
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   19G  0 part 
  ├─centos-root 253:0    0   17G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
sdb               8:16   0    5G  0 disk 
├─sdb1            8:17   0    2G  0 part 
└─sdb2            8:18   0    1G  0 part 
sr0              11:0    1  4.4G  0 rom  
```

### 新增磁盘到能用过程实操

```bash
# 第一步 选择需要分区的磁盘, 然后会弹出对话框, 进行分区信息的填写
-------------------------------------------------------------------------
fdisk /dev/sdb   # 会弹出对话框
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0xabb78855.

Command (m for help): n # 输入n

Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p # 选择p

Partition number (1-4, default 1): 1 # 选择1, 结果体现为sdb1

First sector (2048-10485759, default 2048): # 直接回车, 会默认挨着上一个分区分配结束的位置开始分配
# 起始扇区, 单位是512B(一个扇区大小), 因此5G磁盘最大10485759范围
# 开始的0~2047个扇区用于记录分区信息

Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-10485759, default 10485759): +2G
# 结束扇区, 可以直接写大小, 然后自动转换成结束扇区; 直接回车则全部划分

Partition 1 of type Linux and of size 2 GiB is set # 到目前为止, 操作没有真正生效
Command (m for help): w # write 开始执行分区操作

The partition table has been altered!
Calling ioctl() to re-read partition table.
Syncing disks.
-------------------------------------------------------------------------

# 刷新分区信息
partprobe /dev/sdb # 后面参数可以不加, 全部刷新

# 显示分区信息
fdisk -l /dev/sdb
Disk /dev/sdb: 5368 MB, 5368709120 bytes, 10485760 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xabb78855

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048     4196351     2097152   83  Linux

lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   20G  0 disk 
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   19G  0 part 
  ├─centos-root 253:0    0   17G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
sdb               8:16   0    5G  0 disk 
└─sdb1            8:17   0    2G  0 part /mnt/sdb1_disk
sr0              11:0    1  4.4G  0 rom  

# 第二步 格式化, 选择文件系统
-------------------------------------------------------------------------
mkfs.xfs /dev/sdb1 # 选择刚刚的分区进行格式化, 使用xfs文件系统
mkfs -t xfs /dev/sdb1 # 也可以这样写
# mkfs : Make File Sytem
meta-data=/dev/sdb1        isize=512    agcount=4, agsize=131072 blks
         =                 sectsz=512   attr=2, projid32bit=1
         =                 crc=1        finobt=0, sparse=0
data     =                 bsize=4096   blocks=524288, imaxpct=25
         =                 sunit=0      swidth=0 blks
naming   =version 2        bsize=4096   ascii-ci=0 ftype=1
log      =internal log     bsize=4096   blocks=2560, version=2
         =                 sectsz=512   sunit=0 blks, lazy-count=1
realtime =none             extsz=4096   blocks=0, rtextents=0
-------------------------------------------------------------------------

# 第三步 挂载mount
-------------------------------------------------------------------------
mkdir /mnt/sdb1_disk
mount -t ext4 /dev/sdb1 /mnt/sdb1_disk # 把前面这块分区挂载到后面这个文件夹内
	-t # 指明文件系统
# 从现在起, 往/mnt/sdb1_disk里存放的东西都在sdb这块磁盘的sdb1分区里
-------------------------------------------------------------------------

df -hT # 查看磁盘分区使用情况
	-h # 人性化显示单位
	-T # 显示文件系统类型
Filesystem  Size  Used Avail Use% Mounted on
/dev/sdb1   2.0G  6.0M  1.8G   1% /mnt/sdb1_disk
```

### 创建交换分区

也就是虚拟内存

```bash
# 第一步 分区类型选择 82 Linux swap  也可以不改
fdisk /dev/sdb # 按照帮助信息修改分区类型  l列出类型  t选择类型  选择82
# 第二步 格式化分区
mkswap /dev/sdb2
# 第二步 挂载分区
swapon /dev/sdb2

# 刷新分区信息
partprobe /dev/sdb # 后面参数可以不加, 全部刷新
```

### 分区卸载

```bash
umount /dev/sdb1
# 数据还在里面, 但是访问不到了; 本来是通过一个目录进去的, 现在目录没了相当于连接断了
# 重新挂载就可以继续访问里面的文件了
```

### 永久挂载

上面操作过程只是临时挂载，系统重启后挂载信息就消失了，但分区信息还在。想要永久挂载就需要把挂载信息写入系统相关配置文件，这个文件是`/etc/fstab`

```bash
UUID=b71695cc-04ef-4f88-8c65-ee76c714f97e /mnt/sdb1_disk xfs defaults 0 0
# 重点在前三个参数
# 1、分区的UUID或分区路径(/dev/sdb1), 通过lsblk查看
# 2、挂载点, 也就是挂载到哪个目录下
# 3、分区的文件系统
```

配置好后，系统重启就会自动挂载这些分区

### 删除分区

与分区相关的操作都可以通过`fdisk`命令完成

## 七、链接文件

### 软链接

相当于快捷方式，指向的是指向文件的索引。删除原文件时，会删除指向文件的索引，因此软链接也失效。说简单点就是软链接只记录了源文件的绝对路径。

![image-20201222205000284](https://gitee.com/p8t/picbed/raw/master/imgs/20201222205002.png)

```bash
ln -s <src_file> <link_file> # link_file 指向了 src_file

ln -s 1.txt 1.txt-link
-rw-r--r--. 1 root root 4 Dec 22 19:10 1.txt
lrwxrwxrwx. 1 root root 5 Dec 22 19:11 1.txt-link -> 1.txt
# 注意第一个 l 代表链接文件
```

### 硬链接

指向的是文件，删除原文件时，不影响硬链接的指向，硬链接不失效

![image-20201222205016482](https://gitee.com/p8t/picbed/raw/master/imgs/20201222205017.png)

```bash
ln <src_file> <link_file>

echo 123 > 1.txt
ln 1.txt 2.txt
ln 1.txt 3.txt
ll
-rw-r--r--. 3 root root 4 Dec 22 20:46 1.txt
-rw-r--r--. 3 root root 4 Dec 22 20:46 2.txt
-rw-r--r--. 3 root root 4 Dec 22 20:46 3.txt
# 这个3代表文件硬链接数量, 可知当硬链接数量为0时, 文件才会被真正删除
```

## 八、环境变量

> 参考文章：
>
> [How to Add a Directory to Your $PATH in Linux](https://www.howtogeek.com/658904/how-to-add-a-directory-to-your-path-in-linux/)
>
> [How to set your $PATH variable in Linux](https://opensource.com/article/17/6/set-path-linux)
>
> [Adding a Path to the Linux PATH Variable](https://www.baeldung.com/linux/path-variable)

系统默认环境变量配置为

```bash
/usr/bin		# 基本系统命令
/usr/sbin		# 特殊系统命令, root可使用
/usr/local/bin  # 用户后来加入的
/usr/local/sbin # 用户后来加入的
```



系统级别

- `/etc/environment`
- `/etc/profile`
- `/etc/profile/*.sh`

用户级别

- `~/.profile`
- `~/.bashrc`
- `~/.bash_profile`

配置方式

```bash
# 每个路径变量用 : 分隔
# $PATH取之前的路径变量, 如果不加, 读取到这里就会覆盖之前的路径变量, 只保留/my/dir
export PATH=$PATH:/my/dir # 把当前路径变量放最后面
export PATH=/my/dir:$PATH # 把当前路径变量放最前面
# 配置完成后重启或者使用下面重新加载
source /etc/profile
. /etc/profile

# 打印已配置的路径变量
echo $PATH
```



## 九、查找和压缩

### which

用于查找命令（可执行文件），从`PATH`环境变量中查找

### whereis

在`which`基础上列出帮助文档位置

### find

```bash
# 指定路径下按名称查找
find <path> -name <file> [action]
# 忽略大小写
find <path> -iname <file>
# 查找文件大于5M的
find /etc/ -size +5M
# 最多查找4级
find /usr/ -maxdepth 4 -name 'ifcfg*'

# 查找完后列出详细信息
find ./ -name '1*' -ls
# 查找后删除
find ./ -name '1*' -delete
# 查找后复制 把2.txt复制到当前目录22.txt
find ./ -name 2.txt -ok cp -rf {} ./22.txt \; # 注意末尾
```

### tar

```bash
-c	# create 压缩
-x	# extract 解压
-z	# gzip/ungzip 通过gzip指令压缩/解压缩文件,文件名最好为*.tar.gz
-v	# verbose 显示操作过程
-f	# file 指定备份文件

tar -xzvf <file.tar.gz>				# 解压
	-C # 解压到指定目录, 但是不能重命名
tar -czvf <file.tar.gz> <file..>	# 压缩
```

### locate

## 十、软件管理

### 包分类

#### RPM包

`RPM: RedHat Package Manager`

`RPM`是二进制包，不需要编译，可以直接使用

`RPM`包命名规范，以`httpd-2.4.6-95.el7.centos.x86_64.rpm`为例

- `httpd`：软件包名，上面那个是全包名
- `2.4.6`：软件版本号
- `95`：发行次数
- `el7`：发行商，`el`代表`Red Hat Enterprise Linux`，因此是红帽发布的，适用于`RedHat7.x`和`CentOS7.x`
- `centos`：适用操作系统
- `x86_64`：适用处理器架构
- `rpm`：后缀

#### 源码包

从名字就可以看出来，需要经过编译后才能使用

### 包管理

#### RPM包管理

##### 1、YUM

`YUM: Yellow dog Updater, Modified`

yum 是改进型的 RPM 软件管理器，它很好的解决了 RPM 所面临的软件包依赖问题。yum 在服务器端存有所有的 RPM 包，并将各个包之间的依赖关系记录在文件中，当管理员使用 yum 安装 RPM 包时，yum 会先从服务器端下载包的依赖性文件，通过分析此文件从服务器端一次性下载所有相关的 RPM 包并进行安装。

**如何使用`YUM`**

- 配置`yum`源
- 使用

`yum`源配置在`/etc/yum.repos.d/*.repo`文件中，可以有多个，它会自动读取

```bash
yum list gcc # 查看包
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.ustc.edu.cn
 * centos-sclo-rh: mirrors.ustc.edu.cn
 * centos-sclo-sclo: mirrors.ustc.edu.cn
 * extras: mirrors.ustc.edu.cn
 * updates: mirrors.ustc.edu.cn
Installed Packages
gcc.x86_64    4.8.5-44.el7    @base  # @开头代表已安装

yum search gcc  # 搜索
yum install gcc # 安装
	-y # 确认提示自动回答yes
yum remove gcc  # 移除
yum update gcc  # 更新
yum check-update gcc # 检查更新

# 在/etc/hostname修改@后面的名字
```

##### 2、RPM

```bash
# 查询
rpm -q <package>	# 查询是否安装	
rpm -qi <package>	# 详细信息
rpm -qa # 查询所有
rpm -ql <package> 	# 查询安装位置
rpm -qc <package> 	# 查询配置文件
```

#### 源码包

```bash
# 使用步骤

- 获取源码
- `./configure`：`--prefix=/usr/local/xxx`指定安装路径
- `make`：生成`Makefile`
	如果这一步有报错, make clean 清空中间文件，然后解决错误
- `make install`：
```



## 十一、服务管理

> [The Story Behind ‘init’ and ‘systemd’: Why ‘init’ Needed to be Replaced with ‘systemd’ in Linux](https://www.tecmint.com/systemd-replaces-init-in-linux/#:~:text=Systemd%20and%20Distro%20Integration%20%20%20Linux%20Distribution,%20%20Yes%20%205%20more%20rows%20)

从`CentOS7`开始，`systemd`代替了`SystemV(init)`来管理系统服务，相关命令只需要`systemctl`

相关配置

- `/usr/lib/systemd/system`
- `/run/systemd/system`
- `/etc/systemd/system`

```bash
# 服务状态
active(running)
active(exited)
active(waiting)
inactive

enabled		# 开机自启
disabled	# 开机不自启

static 	  	# 开机不自启, 但是可能被其它自启动服务启动
mask		# 不能被启动

systemctl start <service>
	stop
	kill
	restart
	enable  # 开机自启
	disable # 开机不自启
	status
	is-enabled
	mask    # 无法启动
	unmask  # 取消
```

## 十二、系统资源查看

```bash
# 系统资源, 包括CPU, 内存等
vmstat 1 3 # 监听3次, 每次间隔1秒

# 查看系统启动信息, 可以筛选出各种硬件信息
dmesg

# 查看内存信息
free -h
cat /proc/meminfo

# 查看CPU信息
cat /proc/cpuinfo

# 查看系统负载, top命令的第一行
uptime
16:58:16 up 33 days, 18:04,  2 users,  load average: 0.01, 0.02, 0.05
# 信息显示依次为：现在时间、系统已经运行了多长时间、目前有多少登录用户、
# 系统在过去的1分钟、5分钟和15分钟内的平均负载。

# 查看已登陆用户
w
who

# 查看内核信息
uname
	-a # 所有信息
	-r # 内核版本
	-s # 内核名称
	-m # 硬件架构
	
# 查看发行版本
lsb_release -a

# 查看进程使用了哪些文件
lsof -p <pid>
	 -c <name>

# 查看用户登录历史
last
lastlog # 查看每个用户上一次登录时间
lastb	# 查看登录失败
# 网络
netstat -a # 显示所有socket状态
		-t # tcp
		-u # udp
		-n # 不解析信息, 对于特殊数字, 比如22端口不会显示为ssh
		-l # 显示正在监听的socket
		-p # 显示pid和进程名
netstat -atunp  # 显示所有状态socket
netstat -ltunp 	# 显示listening状态socket
netstat -rn	   	# 查看路由表
```

## 十三、文本检索

### grep

```bash
# grep按行截取
grep -i abc  # 忽略大小写
grep -v '^#' # 排除指定行, 可用于过略注释, 
			 # ^表示以#开头才过滤, 否则只要行内有#就会被过滤
	 -E	# 使用扩展正则, 如果不加-E, 使用正则需要转义(){}
	 	# 或者直接使用egrep
grep -Ev "^#|^$" # 过滤注释和空行

# cut按列截取		 
cut
	-d # 指定分隔符
	-f # 指定截取第几列
cut -d ":" -f 2,3 /etc/passwd # 以:分隔, 截取2,3列
		   -f 2-5	# 截取2到5列
		   
# 格式化输出命令
printf
	%ns		# 字符串, n指定长度, 可省略
	%ni		# 整数, n指定长度, 可省略
	%m.nf	# 浮点数, 一共m位, 小数占n位

# awk中含有printf和print, 区别是后者有换行,
# 注意第一个不是上面那个命令, 而是awk中的
# 默认以空白符分隔
awk '{print $3}'	# $3代表第三列
	-F ":"	# 指定分隔符
awk 'OFS="\t" {print $3,$4}'	# 指定输出分隔符
https://www.howtogeek.com/562941/how-to-use-the-awk-command-on-linux/
# 下面这个可以抓取当前已经使用的内存
free -m | grep "^Mem" | awk '{print $3}' | cut -d "M" -f 1

# 修改检索结果
https://www.howtogeek.com/666395/how-to-use-the-sed-command-on-linux/
# 打印第二行, 如果不加-n, 则除了输出第二行外还会把原结果再输出一遍,
# 因为sed本身就会输出, 而p的含义也是输出
sed -n '2p' /etc/passwd 
# 删除第2到4行内容, 这里就不需要加-n了
tail -10 /etc/passwd | sed '2,4d'
# 在第二行之后添加一行数据 hello world
head -5 /etc/passwd | sed '2a hello world'
# 在第二行之前添加两行数据 hello world, 注意那个\, 换到下一行输入结果也会换行
head -5 /etc/passwd | sed '2i hello \
world'
# 把第二行替换为haha, c的作用是替换整行
head -5 /etc/passwd | sed '2c haha'
# 把第二行所有root替换为rt, 不指定行数则替换所有行, 用法参考vim
head -5 /etc/passwd | sed '2s/root/rt/g'
```



## 十四、正则

```bash
^	# 从行首开始匹配
	grep "^root" /etc/passwd
$	# 从行尾开始匹配
	grep "/bin/bash$" /etc/passwd
.	# 匹配任意一个字符
	grep "r..t" /etc/passwd
()	# 标记为一个表达式
*	# 匹配前一个表达式0到多次
+	# 匹配前一个表达式1到多次
?	# 匹配前一个表达式0到1次
.*	# 匹配任意
[]	# 匹配其中一个
[^]	# 不匹配其中任意一个
\<	# 词首定位符
\> 	# 词尾定位符
```

## 十五、定时任务

### 一次性任务

是否有权执行一次性任务在以下文件配置

- `/etc/at.allow`
- `/etc/at.deny`

每个定时任务都会在`/var/spool/at`下有一个对应的任务脚本

```bash
at now +2min	# 输入要执行的命令, 然后^D完成
atq		# 查询任务
at -d 5 # 删除5号任务
# 定时任务会在/var/spool/at下生成一个对应的脚本文件, 删除该文件与at -d删除等效
at -c 5 # 查看5号任务, 实际上就是查看/var/spool/at下对应任务的脚本文件

at now +2min -f t.sh  # 指定脚本

# 下面是时间写法
at 12:00	# 如果今天过了12点, 则明天12点执行
at 12:00 2020-12-28
at 12:00 +3days # minutes months...
```

### 循环任务

是否有权执行循环任务在以下文件配置

- `/etc/cron.allow`
- `/etc/cron.deny`

循环任务的执行依赖`crond.service`服务，因此必须保证该服务启动

crond.service服务会默认从以下地方读取循环任务

- `/etc/crontab`
- `/ec/cron.d/*`
- `/var/spool/cron/*`

用户通过`crontab -e`命令发起的所有循环任务都在`/var/spool/cron`下的一个文件中，文件名就是用户名

```bash
crontab -e	# 编辑任务
crontab -l	# 查看任务
		-u	# root用户指定其它用户名字, 可以查看别人的任务
crontab -r	# 移除所有任务, 相当于把/var/spool/cron/<login>文件删了


# 执行crontab -e后, 需要编辑定时任务文件, 格式如下
 *      *       *     *      *    COMMAND
 分     时      日     月     周
0-59   0-23   1-31   1-12   0-7
-----------------------------------------------------------
5  2  5  2  * # 代表每个2月5号的02:05都会执行
5  2  5  *  * # 代表每个5号的02:05都会执行
5  2  *  *  * # 代表每个02:05都会执行
5  *  *  *  * # 代表每小时的5分都会执行  00:05 01:05 02:05...
*  *  *  *  * # 代表每分钟都会执行
0  *  *  *  * # 代表每小时都会执行

0  2  *  *  5 # 代表每个星期五 02:00执行

*/5 */2 * * * # 代表每隔2小时5分钟执行一次

* 3,6 * * *	  # 代表每天03:00和06:00都会执行
* 3-6 * * *	  # 代表每天03:00,04:00,05:00,06:00都会执行
```

## 十六、日志

产生系统日志的服务是`rsyslogd`，此外第三方进程也会有自己的日志

系统日志绝大多数都在`/var/log`下

### 常见日志

```bash
/var/log/messages 	# 记录了系统重要信息, 比如启动了某一个服务或者系统发生了错误
/var/log/secure		# 记录了账号登录,切换账号信息
/var/log/cron
/var/log/dmesg
/var/log/yum.log
/var/log/lastlog	# 每个用户上次登录信息 lastlog命令
/var/log/btmp		# 登录失败日志 lastb命令
/var/log/wtmp		# 用户登录历史 last命令
/var/log/utmp		# 当前登录用户信息 w who users命令
```

### 配置文件

```bash
/etc/rsyslog.conf		# 主配置文件
/etc/sysconfig/rsyslog	# 
/etc/logrotate.d/syslog	# 日志轮转相关
```

### 日志级别

```bash
# 从上往下越来越严重
debug
info
notice
warning
err
crit
alert
emerg
```

### 日志轮转

配置文件

- `/etc/logrotate.conf`
- `/etc/logrotate.d/*`

```bash
# rpm包安装的默认加入了系统日志轮转
# 源码包或其它安装的程序如果没有日志轮转功能,可以加入系统日志轮转功能
/var/log/btmp {	# 写上第三方程序日志路径
    weekly		# 每周进行一次日志轮替操作
    dateext		# 按天切片
    rotate 7	# 保留7份, 按天切片就是保留7个
    create 		# 由于管理的日志文件会被改名为日期后缀, 因此需要额外创建一个和原文件名一样的备份
}
```

```bash
logrotate -v /etc/logratate.conf 	# 查看日志轮转信息
logrotate -f /etc/logrotate.conf	# 立即执行日志轮转
```

## 杂项

```bash
# 让程序后台运行
<command> &
# 查看后台程序及其作业号 使用 kill -9 %num 按作业号杀死进程
jobs
# 把后台程序拉到前台
fg <num> # foregroud
# 把进程扔到后台, Ctrl+Z先扔到后台然后用bg唤醒
bg <num> # backgroud
# 虽然&可以让程序后台运行, 但是用户退出登录后, 程序也会停止
# 此时就需要使用nohup脱机运行
nohup <command> & # 脱机后台运行

# 远程文件复制
scp -r local_dir user@host:remote_dir	# 上传
scp -r user@host:remote_dir local_dir	# 下载
	-r	# 递归复制

# 通过url下载文件
wget <url>

# 关机
shutdown -h now
halt
poweroff
init 0
# 重启
shutdown -r now
reboot
init 6

# 系统级别, systemd不使用该机制
0	# 关机
1	# 单用户, 只允许root用户登录
2	# 不完全多用户, 不含NFS服务: Network File System
3	# 完全多用户, 正常模式
4	# 未分配
5	# 带有3的图形界面
6	# 重启
runlevel	# 查询运行级别
```

