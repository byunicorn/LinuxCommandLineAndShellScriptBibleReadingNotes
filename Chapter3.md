####启动Shell
* shell启动时，会自动执行用户主目录的.bashrc文件中的命令
* `/etc/bashrc` 该文件设置各种应用程序中使用的环境变量

####Shell提示符
* 控制命令行提示符的环境变量：
	* PS1:默认命令行提示符的格式
	* PS2:第二层命令行提示符的格式

```
vagrant@web:/etc$ echo $PS1
${debian_chroot:+($debian_chroot)}\u@\h:\w\$ 
#\u当前用户的用户名,\h本地主机名,\w当前目录
vagrant@web:/etc$ echo $PS2
>
```

####bash手册
* man bash
####文件系统导航
* Linux在一个目录结构中存储文件，该目录成为虚拟目录
* 安装在Linux PC中的第一个硬盘称为**根驱动器**，其他目录都是从这里开始创建的
* 在根驱动器中，Linux创建一些名为**挂载点**的特殊目录。挂载点是虚拟目录中用于分配额外存储设备的目录          
![](http://farm6.staticflickr.com/5488/9917989976_3068219384_o.jpg)
* 上图中，Disk1与虚拟目录的根关联（/），其他硬盘可以挂载到虚拟目录结构的任何地方，Disk2挂载到了/home，这也是用户目录所在的位置       
        
**通用Linux目录名称**

| 目录       | 用法           | 
| ------------- |:-------------:|
| /      | 虚拟目录的根目录，通常此处没有文件 |
| /bin   | 二进制目录，存储许多GNU用户级别的实用程序 |
| /boot  | 引导目录，存储引导文件 |
| /dev   | 设备目录，Linux在该目录中创建设备节点 |
| /etc   | 系统配置文件目录 |
| /home  | 主目录，Linux在该目录中创建用户目录 |
| /lib   | 库目录，存储系统和应用程序库文件 |
| /media | 媒体目录，可移动媒体设备常用的挂载点 |
| /mnt   | 挂载目录，另一个可移动设备常用的挂载点 |
| /opt   | 可选目录，常用于存储可选软件包 |
| /root  | 根主目录 |
| /sbin  | 系统二进制目录，存储许多GNU管理级别的实用程序 |
| /tmp   | 临时目录 |
| /usr   | 用户安装软件的目录 |
| /var   | 可变目录，用于经常更改的文件，如日志文件 |
             

* 文件和目录列表
	* `ls -F`可以查看文件类型，/表文件夹，*表可执行文件
	* `cp -R` 递归复制
	* `ls -R` 会循环列出该文件夹下的文件、文件夹，一直到叶子节点
	* 链接文件，如果需要在系统中维护同一个文件的两个（或以上）副本，不一定需要使用两个物理副本，可以使用一个物理副本和多个虚拟副本，这种虚拟副本称为链接`ln` `cp -l` `cp -s`
	* 硬链接文件使用的索引节点编号与源文件相同，在移除了最后一个链接文件之前，硬链接文件将一直维护索引节点编号，并保留数据，但是对于软链接，底层文件不在了，那么链接的指向内容自然也消失了
	* `touch`除了创建新文件外，还可以修改访问时间和修改时间，不改变文件内容`touch -a` `touch -m`

```
vagrant@lucid32:~$ ls -F
go1.1.1.linux-386.tar.gz  postinstall.sh*  vimPractice/
vagrant@lucid32:~/vimPractice$ ls -Ali
total 8
786448 -rw-r--r-- 2 vagrant vagrant 26 2013-08-17 16:17 test1
786448 -rw-r--r-- 2 vagrant vagrant 26 2013-08-17 16:17 test2	#硬链接
786450 lrwxrwxrwx 1 vagrant vagrant  5 2013-08-17 16:21 test3 -> test1	#软链接
vagrant@lucid32:~/vimPractice$ rm test1
vagrant@lucid32:~/vimPractice$ ls -Ali
total 4
786448 -rw-r--r-- 1 vagrant vagrant 26 2013-08-17 16:17 test2
786450 lrwxrwxrwx 1 vagrant vagrant  5 2013-08-17 16:21 test3 -> test1
vagrant@lucid32:~/vimPractice$ cat test3
cat: test3: No such file or directory
```

* 查看文件内容
	* 查看文件统计数据
		1. `stat test2` 提供文件系统中文件状态的完整摘要
		2. `file test2` 查看文件内部并确定文件的类型：文本文件、可执行文件、数据文件
	* 查看文件内容
		1. `cat -n` 行号 `cat -b` 有文本的行号
		2. `more`可以用H看help `less`
	* 查看部分文件
		1. `tail`
		2. `head`
