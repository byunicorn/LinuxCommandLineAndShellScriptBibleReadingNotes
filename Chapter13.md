###脚本控制
####处理信号
- Linux信号回顾
	- bash shell会忽略它收到的(3),(15)信号，以防止交互的shell意外终止
	- bash shell会处理它收到的(1),(2)信号

![](http://farm8.staticflickr.com/7311/9918443114_2debcfaebe_o.jpg)

- 生成信号
	- 中断进程`ctrl+c`
	- 暂停进程`ctrl+z`，方括号里的是shell分配的作业编号，可以用`ps au`命令来查看停止的作业，STAT为T
	- 如果想在作业停止时退出shell，需要键入两次exit，知道停止作业的PID，可以用`kill`命令来终止它

			vagrant@lucid32:~/linuxAndShell/chapter13$ vim test1

			[1]+  Stopped                 vim test1

			vagrant@lucid32:~/linuxAndShell/chapter13$ ps au
			USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
			root       625  0.0  0.1   1788   560 tty4     Ss+  16:09   0:00 /sbin/getty -8
			...
			vagrant    876  0.1  0.8   4816  3340 pts/0    Ss   16:10   0:00 -bash
			vagrant    912  0.0  0.9   9080  3524 pts/0    T    16:11   0:00 vim test1
			vagrant    930  0.0  0.2   2652  1016 pts/0    R+   16:14   0:00 ps au

			vagrant@lucid32:~/linuxAndShell/chapter13$ kill -9 912
			[1]+  Killed                  vim test1

- 捕获信号
	- `trap commands signals` 列出shell执行的命令，以及希望捕获的信号列表(空格分隔)
	- 捕获脚本退出，向trap命令添加`EXIT`信号
	- 移除捕获，`trap - EXIT`

```
#每次使用Ctrl+C组合键时，脚本执行在trap命令中的echo语句，而不是忽略信号并允许shell停止脚本
vagrant@lucid32:~/linuxAndShell/chapter13$ cat test1
#!/bin/bash
trap "echo Haha" SIGINT SIGTERM
trap "echo byebye" EXIT
echo "This is a test program"
count=1
while [ $count -le 2 ]
do
    echo "Loop #$count"
    sleep 10
    count=$[ $count + 1 ]
done
echo "This is the end of the test program"

vagrant@lucid32:~/linuxAndShell/chapter13$ ./test1
This is a test program
Loop #1
Loop #2
This is the end of the test program
Byebye
```


####以后台模式运行脚本
- 在后台模式中，进程运行时与终端会话的STDIN、STDOUT和STDERR无关
- 以后台模式运行shell脚本，只要在脚本后面加上一个`&`符号
- 退出终端，每个后台进程都连着一个终端会话(pts/0)

```
vagrant@lucid32:~/linuxAndShell/chapter13$ ./test1 &
[1] 1282		#作业编号和PID
vagrant@lucid32:~/linuxAndShell/chapter13$ This is a test program
Loop #1
ls -l
total 4
-rwxr--r-- 1 vagrant vagrant 243 2013-09-09 14:35 test1
vagrant@lucid32:~/linuxAndShell/chapter13$ Loop #2
```

####在不使用控制台的情况下运行脚本
- 有时需要从终端会话启动脚本，然后让脚本结束之前以后台模式运行，可以用`nohup`命令实现
- `nohup`会阻塞发送到进程的任何SIGHUP信号，可以防止在退出终端会话时退出进程
- 因为`nohup`将进程与终端断开，所以进程没有STDOUT和STDERR输出连接，这些输出会被重定向到nohup.out的文件内(同一目录多个命令运行时须注意，都会重定向到同一个nohup.out文件里)

```
vagrant@lucid32:~/linuxAndShell/chapter13$ nohup ./test1 &
[1] 1292
vagrant@lucid32:~/linuxAndShell/chapter13$ nohup: ignoring input and appending o
utput to `nohup.out'
```

####作业控制
- 查看作业：带加号的作业被视为默认作业，如果命令行没有作业编号，则它应该是任何作业控制命令引用的作业，带减号的是在处理完当前默认作业后会变为默认作业的作业。某一时间点，只能有一个带加号的作业，也只能有一个带减号的作业

```
vagrant@lucid32:~/linuxAndShell/chapter13$ ./test2
This is a test program
Loop #1      #这里Ctrl+Z停止

[1]+  Stopped                 ./test2
vagrant@lucid32:~/linuxAndShell/chapter13$ ./test1 > test1.out &
[2] 1323
vagrant@lucid32:~/linuxAndShell/chapter13$ jobs
[1]+  Stopped                 ./test2
[2]-  Running                 ./test1 > test1.out &
```

- 重新启动停止的作业: 后台模式`bg jobNumber`, 前台模式`fg jobNumber`

####变得更好
- 默认情况下，Linux系统上的调度优先级(内核相对于其他进程分配给某一进程的CPU时间量)都相同，从-20(最高优先级)~20(最低优先级)，默认情况为0
- `nice -n 优先级 commands`
- 提高优先级可能会失败，安全特性，防止用户以高优先级启动所有命令
- renice可以修改进程优先级 `renice 优先级 -p PID`

```
vagrant@lucid32:~/linuxAndShell/chapter13$ nice -n 5 ./test1 &
[3] 1333
```

####准确无误地运行

- at和atd(前台和后台)命令,`at [ -f filename ] time` ,time参数指定希望Linux运行作业的时间
- 时间格式
	- `Now + 25 mininutes`
	- `10:15PM tomorrow`
	- `MMDDYY` `MM/DD/YY` `DD.MM.YY`
	- 文本日期格式 `Jul 4` `Dec 25`
- at命令会把作业提交到作业队列中，有26种不同的作业队列可用于不同的优先级(a-z)，默认at作业都提交到作业队列a，如果希望以较低的优先级运行作业，则可以使用-q参数指定字母
- 作业在Linux系统运行时，没有与该作业相关联的监视器，Linux使用提交作业的用户的电子邮件地址作为STDOUT和STDERR
- `atq` 能够查看系统中排队的作业
- `atrm jobNumber` 移除作业

- `batch` 安排脚本在系统使用率时运行，如果平均负载低于0.8，它将运行任何在作业队列中等待的作业，还可以指定尝试运行作业的最早时间 `batch [-f filename] [time]`

- 调度定期脚本
	- cron表格，格式如下`min hour dayofmonth month dayofweek command` /etc/crontab
	- `crontab -l`
	- 如果在使用`cron`调度作业时Linux处于关闭状态，作业将无法运行，`anacron`程序使用时间戳确定调度的作业是否在正确的时间间隔运行。如果确定某个作业错过了调度的运行时间，将自动尽快运行作业。
	- 要使用 anacron 服务，你必须安装了 anacron RPM 软件包， anacron 服务必须在运行；要判定该软件包是否被安装，使用`rpm -q anacron` `dpkg -l` 命令；要判定该服务是否在运行，使用`/sbin/service anacron status`命令

#### 从头开始
- 让脚本在Linux系统启动时或者用户启用新bash shell会话时自动运行
- 启动时启动脚本
![](http://farm3.staticflickr.com/2854/9918536943_470522e8b3_o.jpg)
- 随shell一起启动
	- .bash_profile (特定用户)
	- .bashrc		(特定用户)
	- /etc/bashrc	(系统每个用户)
