####深入理解Linux内核
#####Linux内核4个功能：
* 系统内存管理
* 软件程序管理
* 硬件管理
* 文件系统管理

#####系统内存管理
* `cat /proc/meminfo`  可以查看Linux系统中虚拟内存的当前状态
* `awk '/Mem/ {printf "%s: %d MB\n", $1,$2/1024.0}' /proc/meminfo` 物理内存和没使用的空间
* `ipcs -m`查看系统当前的共享内存分页
######软件程序管理
* 正在运行的程序 - 进程
* 第一个进程 - 初始进程（init process）
	* 进程表：`/etc/inittabs`
    * 对于ubuntu之类的，启动/关闭进程的脚本在`/etc/init.d/`文件夹下面，对应的运行级别在`/etc/rcX.d`下
    * 运行级别1只启动基本系统进程，单用户模式，只允许一个人登陆
    * 标准运行级别是3，支持大部分应用软件
    * 常用运行级别是5，图形化界面 X window
    * 查看进程：`ps ax`
```
vagrant@web:~$ ps ax | awk '$3 ~ /R/ ||NR == 1 {print $0}' #正在运行状态的进程
  PID TTY      STAT   TIME COMMAND #第三列：R运行，S睡觉，SW睡觉并等待
 1509 pts/0    R+     0:00 ps ax
```
#####硬件管理
* 识别：设备文件
* 三种：字符，块，网络
* 查看：`cd /dev` `ls -al sda* ttyS*`
* 不同Linux发行版使用不同设备名称处理设备，sda是第一个SCSI硬盘，ttyS是标准的IBM PC COM端口
* 第5列是主设备节点号，sda设备都是节点8，第6列是次设备节点号，每个设备有唯一的次设备节点号
```
brw-rw---- 1 root disk    8,  0 2013-08-14 05:46 sda
brw-rw---- 1 root disk    8,  1 2013-08-14 05:46 sda1
brw-rw---- 1 root disk    8,  2 2013-08-14 05:46 sda2
brw-rw---- 1 root disk    8,  5 2013-08-14 05:46 sda5
crw-rw---- 1 root dialout 4, 64 2013-08-14 05:46 ttyS0
crw-rw---- 1 root dialout 4, 65 2013-08-14 05:46 ttyS1
crw-rw---- 1 root dialout 4, 66 2013-08-14 05:46 ttyS2
crw-rw---- 1 root dialout 4, 67 2013-08-14 05:46 ttyS3
```

######文件系统管理
* Linux用虚拟文件系统（VFS）与每个文件系统进行连接

####GNU（GNU's not Unix）实用程序
* 核心GNU实用程序
	* 处理文件的实用程序
	* 操作文本的实用程序
	* 管理进程的实用程序
* shell是一个特殊的交互式实用程序
* 图形界面
	* x window
	* KDE
	* GNOME
* Linux发行版
	* 三种类型：完整核心Linux发行版，特定发行版，LiveCD测试发行版
	* 核心：内核、图形桌面、几乎所有的Linux应用程序，预编译的内核
	* 特定：以一个主要的发行版为基础，包含对特定领域有用的应用程序子集