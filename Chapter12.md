###显示数据
####了解输入和输出
- 标准文件描述符
	- STDIN
		1. 标准输入是键盘，使用输入重定向符号(<)时，就将使用重定向引用的文件替换标准的输入`cat < testfile`
	- STDOUT
		1. 标准输出是终端监视器，使用输出重定向符号(>)时，就将输出重定向到指定的文件中 	`who >> test2`
	- STDERR
		1. 默认情况下，STDERR文件描述符和STDOUT文件描述符指向相同的位置
	- 重定向
		- `ls -al 'test1' 1>out 2>err`
		- `ls -al 'test1' &>outAndErr`将err和out重定向到同一个输出文件，需要使用&符号

####在脚本中重定向输出
- 临时重定向
- 如果想要故意在脚本中生成错误信息，可以将单个输出行重定向到STDERR，重定向到某个文件描述符时，须在文件描述符编号前添加&号

```
#!/bin/bash
echo "This is an error" >&2
echo "This is normal output"

#如果像往常一样运行脚本，不会有任何不同
vagrant@lucid32:~/linuxAndShell/chapter12$ ./test1
This is an error
This is normal output
#如果使用重定向STDERR，则第一句话会被重定向
vagrant@lucid32:~/linuxAndShell/chapter12$ ./test1 2> test1.err
This is normal output
```

- 永久重定向
- 如在脚本中重定向许多数据，重定向每个echo语句太不方便了，可以使用`exec`命令通知shell在脚本执行期间重定向特定的文件描述，`exec`启动一个新shell，并将STDOUT文件描述符重定向到一个文件

```
#!/bin/bash
exec 2>$(basename $0).error
echo "This is the start of the script"
echo "now redirecting all output to another location"

exec 1>$(basename $0).output
echo "This is output should go to the testout file"
echo "but this should go to the testerr file" >&2

vagrant@lucid32:~/linuxAndShell/chapter12$ ./test2
This is the start of the script
now redirecting all output to another location
```

- 在脚本中重定向输入

```
#!/bin/bash
exec 0< testfile
count=1
while read line
do
    echo "Line #$count: $line"
    count=$[ $count + 1 ]
done
```

####创建自己的重定向
- 不局限于3种默认的文件描述符，shell最多可以有9个打开的文件描述符，其他6个标号从3-8
- 创建输出文件描述符

```
#!/bin/bash
exec 3>$(basename $0).3.out
echo "This should display on the monitor"
echo "and this should be stored in the file" >&3
echo "Then back on the monitor again"
```

- 重定向文件描述符

```
#!/bin/bash
exec 3>&1 #任何发送到文件描述符3的输出将出现到监视器上
exec 1>$(basename $0).out #将STDOUT重定向到一个文件
echo "This should store in the output file" #这两行都会被记录到文件里
echo "along with this line."
exec 1>&3 #将STDOUT重定向到文件描述符3的位置，即监视器
echo "Now things should be back to normal"
```

- 创建输入文件描述符

```
#!/bin/bash
exec 6<&0  #文件描述符保存STDIN的位置
exec 0<testfile  #STDIN重定向到文件
count=1
while read line
do
    echo "Line #$count: $line"
    count=$[ $count + 1 ]
done
exec 0<&6   #恢复STDIN原来的位置
read -p "Are you done now?" answer
case $answer in
    Y|y) echo "Goodbye";;
    N|n) echo "Sorry, this is the end.";;
esac
```

- 创建读取/写入文件描述符

```
#!/bin/bash
exec 3<>testfile #将3分配给文件testfile的输入输出操作
read line <&3 #从3中读出一行
echo "Read: $line"
echo "This is a test line" >&3 #将这行写入3指向的文件中
```

- 关闭文件描述符

```
#!/bin/bash
exec 3>$(basename $0).out
echo "This is a test line of data" >&3
exec 3>&- #关闭文件描述符
echo "This won't work">&3 #尝试写入文件将会出错

vagrant@lucid32:~/linuxAndShell/chapter12$ ./test8
./test8: line 5: 3: Bad file descriptor
```

- 列出开放文件描述符
	- `lsof`列出整个linux系统上所有的开放文件描述符，包括后台运行的进程，以及所有登陆到系统的用户账户，许多linux系统都隐藏了该命令，要用普通账户运行它，需要使用完整路径
	- `-p`参数可以指定进程ID，`-d`可以指定要显示的文件描述符编号，为了定位当前PID，可以使用特殊环境变量`$$`，该变量设置为当前PID，`-a`选项可以连接其他两个选项结果
		
			vagrant@lucid32:~/linuxAndShell/chapter12$ lsof -a -p $$ -d 0,1,2
			COMMAND  PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
			bash    1397 vagrant    0u   CHR  136,2      0t0    5 /dev/pts/2
			bash    1397 vagrant    1u   CHR  136,2      0t0    5 /dev/pts/2
			bash    1397 vagrant    2u   CHR  136,2      0t0    5 /dev/pts/2

- 禁止命令输出
	- 可以将STDERR重定向到空文件，输出到空文件的内容会全部丢失
	- `ls -al > /dev/null` `./badfile 2> /dev/null`
	- 可以用它快速将数据从现有文件移除
			
			vagrant@lucid32:~/linuxAndShell/chapter12$ cat /dev/null > test8.out
			vagrant@lucid32:~/linuxAndShell/chapter12$ cat test8.out
			vagrant@lucid32:~/linuxAndShell/chapter12$

- 使用临时文件
	- `/tmp`目录处理不需要永久保留的文件，大部分linux发行版会在启动时删除`/tmp`里的文件
	- 任何账户都可以访问`/tmp`
	- 创建本地临时文件
		- `mktemp`在本地目录创建临时文件
				
				vagrant@lucid32:~/linuxAndShell/chapter12$ mktemp testing.XXXXXX
				vagrant@lucid32:~/linuxAndShell/chapter12$ ls -al testing.*
				-rw------- 1 vagrant vagrant 0 2013-09-06 06:20 testing.4Yf6NE

				#!/bin/bash
				tempfile=`mktemp $(basename $0).XXXXXX`
				exec 3>$tempfile
				echo "This script writes to temp file $tempfile"
				echo "This is the first line" >&3
				echo "This is the second line" >&3
				echo "This is the last line" >&3
				exec 3>&-
				echo "Done creating temp file. The contents are:"
				cat $tempfile
				rm -f $tempfile 2>/dev/null

- 在`/tmp`中创建临时文件
	- `-t`选项强迫`mktemp`在系统的临时文件夹下创建文件，此方法返回临时文件的完整路径名 `mktemp -t testing.XXXXXX`

- 创建临时目录
	- `-d`选项让`mktemp`命令创建一个临时目录，而不是一个文件
	- `mktemp -d dir.XXXXXX`

####记录消息
- 有时候需要将输出同时发送到监视器和文件，这种情况下需要两次重定向，也可以使用`tee`命令，可以将STDIN的数据同时分发到两个目的地，一个是STDOUT，另一个是`tee`指定的文件名
- `date | tee testfile` 默认情况下，`tee`会覆盖输出文件，如果要添加内容，需要加上`-a`选项 `date |tee -a testfile`
