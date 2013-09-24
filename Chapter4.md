####监控程序
- 进程查看
	- `ps -el`
		1. UID: 负责启动进程的用户
		2. PID: 进程的ID
		3. PPID: 父进程的PID
		4. C: 进程存续期的处理器利用率
		5. STIME: 进程启动时的系统时间
		6. TTY: 进程从中启动的终端设备
		7. TIME: 运行进程所需的累计CPU时间
		8. CMD: 启动程序的名称
	- `top` 实时进程
	- 停止进程
		1. `kill PID` 发送TERM信号给PID的进程
		2. `killall`

| 信号       | 名称           |  描述           | 
| ------------- |:-------------:|:-------------:|
| 1      | HUP |挂起|
| 2      | INT |中断|
| 3      | QUIT|停止运行|
| 9      | KILL|强制终止|
| 11     | SEGV|段违例|
| 15     | TERM|条件终止|
| 17     | STOP|强制停止，但未终止|
| 18     | TSTP|停止或暂停，但继续在后台运行|
| 19     | CONT|STOP或TSTP之后恢复执行|      
              

- 监控磁盘空间
	- `mount` 挂载介质，mount可以显示系统当前挂载的介质设备列表
		1. 介质的设备位置
		2. 介质的虚拟目录中的挂载点
		3. 文件系统类型
		4. 已挂载介质的访问状态
	- `mount -t type device directory`
	- `mount -t iso -o loop xxxxx-DVD32.iso mnt` -o在文件系统中添加特定选项: ro只读，rw读写，loop挂载文件系统，选项之间用逗号分隔
	- `umount [directory|device]`
	- `df -h` 查看已挂载磁盘的使用情况
	- `du` 显示当前目录下所有的文件、目录和子目录，并显示占用的空间
- 操作数据文件
	- 数据排序
		1. `sort`
	- 数据搜索
		1. `grep [options] pattern [file]` -v 反转，不输出有该pattern的数据行 -n 行号 -c count -e 多个pattern

                root@lucid32:/# du|sort -t ' ' -k 1 -n|tail #以空格为分隔符，取第一个分段的数据，按照数值来排序
                90728   ./lib/modules/2.6.32-38-generic
                90732   ./lib/modules
                103360  ./usr/lib
                119376  ./usr/local/go
                119476  ./usr/local
                126984  ./var
                135116  ./lib
                192024  ./usr/share
                458696  ./usr
                860416  .
	- 压缩数据(压缩，解压，显示内容)
		1. bzip2: `bzip2`  `bunzip2` `bzcat` 
		2. gzip:  `gzip`  `gunzip`  `zcat`  
		3. zip: `zip` `unzip` `zipnote` `zipsplit`
	- 归档数据
		1. `tar function [options] object1 object2 ...` -c 创建新的tar归档文件 -x 解压
![](http://farm3.staticflickr.com/2833/9918113436_a1ece988a2_o.jpg)
