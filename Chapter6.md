###文件权限
####Linux安全性
- /etc/passwd文件

		root@lucid32:~# cat /etc/passwd #登陆用户名;密码;UID;GID;注释字段;HOME的位置;默认shell
		root:x:0:0:root:/root:/bin/bash
		daemon:x:1:1:daemon:/usr/sbin:/bin/sh
		bin:x:2:2:bin:/bin:/bin/sh
		sys:x:3:3:sys:/dev:/bin/sh
		...bulabula...
		vagrant:x:1000:1000:vagrant,,,:/home/vagrant:/bin/bash
		vboxadd:x:999:1::/var/run/vboxadd:/bin/false

	- 500以下的UID保留作为系统账户
- /etc/shadow 管理账户密码
- 添加新用户`useradd`
	- 要查看设置的默认值


			root@lucid32:~# /usr/sbin/useradd -D
			GROUP=100 #新用户将被添加到GID为100的公共用户组中
			HOME=/home #将在/home/loginname目录下创建HOME账户
			INACTIVE=-1 #密码过期不会禁用账户
			EXPIRE=
			SHELL=/bin/bash #设置默认bash
			SKEL=/etc/skel #将/etc/skel目录中的内容复制到用户的HOME目录
			CREATE_MAIL_SPOOL=no
	
- 删除用户`userdel` 注意 -r会删除/home/loginname目录
- 修改用户
	- `usermod` 编辑用户账户字段
		1. -l 修改登录名
		2. -L 锁定账户(无法登陆，但不删除其内容)
		3. -p 修改密码
		4. -U 解除锁定
	- `passwd`修改密码
	- `finger by`快速查找用户信息
- 使用用户组
	- 允许多用户共享某个对象(文件、目录、设备)的公共权限集
	- /etc/group
	- 创建新用户组时，系统默认不会为它分配任何用户。要添加新用户，需要使用`usermod`命令`usermod -G bulabula loginname`
- 修改用户组
	- `groupmod` 允许修改已有用户组的GID(-g)或用户组名称(-n)

####解码文件权限
- 使用文件权限符号
	- `ls -l`输出清单中第一个字段是描述文件和目录的权限的代码
		1. `-`文件 `d`目录 `l`链接 `c`字符设备 `b`块设备 `n`网络设备
		2. `r`读 `w`写 `x`执行
		3. 每三位对应: 对象所有者，对象用户组，系统上的其他人
- 默认文件权限
	- `umask`
	- 设置在/etc/profile启动文件中
	- 文件的完整权限是模式666，目录的完整权限是777(没有x就无法`cd`进入)，掩码就是反过来减去

####修改安全设置
- 修改权限
	- `chmod [ugoa] [+-=] [rwx]` u 用户, g 组, o 其他, a 所有
- 修改所有者
	- `chown options owner[.group] file`
	- `chgrp groupname file` 改变用户组

####共享文件
- Linux为每个文件和目录都存储了3个额外的信息位
	1. set user id(SUID): 当文件由用户执行时，程序将在文件所有者的权限下运行
	2. set group id(SGID): 对于文件，程序将在文件用户组的权限下运行。对于目录，目录中创建的新文件使用目录用户组作为默认用户组
	3. 粘着位: 进程结束后，文件仍保留在内存中
	4. 可以用`chmod`来设置SUID/SGID/粘着位，也是对应的3位二进制，在开头

			root@lucid32:/home/vagrant/testdir# chmod 6666 testfile
			root@lucid32:/home/vagrant/testdir# ls -l
			total 0
			-rwSrwSrw- 1 root shared 0 2013-08-20 06:57 testfile
	5. `chmod g+s file` 设置SGID