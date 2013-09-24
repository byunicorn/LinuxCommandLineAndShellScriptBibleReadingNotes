###基本脚本编译
####创建脚本文件
- 第一行须指明使用的shell: `#!/bin/bash`
- 普通脚本文件中，#后的是注释，但是shell脚本文件的第一行是特例
```
vagrant@lucid32:~/vimPractice$ ./test1
-bash: ./test1: Permission denied
vagrant@lucid32:~/vimPractice$ ls -l test1
-rw-r--r-- 1 vagrant vagrant 22 2013-08-22 15:40 test1
vagrant@lucid32:~/vimPractice$ chmod u+x test1
vagrant@lucid32:~/vimPractice$ ./test1
Thu Aug 22 15:42:12 CEST 2013
vagrant  pts/0        2013-08-22 15:39 (10.0.2.2)
```

####显示消息
- `echo` 默认情况下不需要用引号来标记想要显示的字符串，但如果字符串中使用了引号，就有可能出现问题
- `echo -n` 不换行

####使用变量
- 环境变量
	- `echo "User info for userid: $User"` 引号内的也可以转换
	- `echo "The cost of the item is \$15"`
- 反引号
	- 将shell命令的输出赋值给变量
```
testing=`date`
```
- 输出重定向
	- `ls > test6`
	- `date >> test6` 添加到文件后面
- 输入重定向
	- 将文件的内容重定向到一条命令中: `command < inputfile`
	- 内置输入重定向: `<<`允许在命令行中而非文件中为输入重定向指定数据，须指定一个文本标记来说明输入数据的开始和结尾

```
#wc命令对数据中的文本计数: 文本行数，单词数，字节数
vagrant@lucid32:~/vimPractice$ wc < test1
 5 17 92
#内置输入重定向符号是两个小于号<<
#除了这个符号，还必须指定一个文本标记来说明输入数据的开始和结尾。
vagrant@lucid32:~$ wc << EOF
> test string 1
> test string 2
> test string 3
> EOF
 3  9 42
```

- 管道
	- `|`将一个命令的输出发送至另一个命令的输入 `ls -l|more`
- 数学计算
	- `expr`
	- 使用括号 `var1=$[1+5]`

```
root@lucid32:~# expr 5 * 2
10
root@lucid32:~# var4=$[1+5]
root@lucid32:~# echo $var4
6
root@lucid32:~# var5=$[$var4 * 2]
root@lucid32:~# echo $var5
12
root@lucid32:~# cat test5
#! /bin/bash
var1=100
var2=50
var3=45
var4=$[$var1*($var2-$var3)]
echo The final result is $var4
```

- 浮点解决方案
	1. `bc` 
	2. 要退出bash计算器，必须输入quit
			
				root@lucid32:~# bc -q
				var1=10
				var1 * 4
				40
				var2 = var1/5
				print var2
				2
				quit
	3. 在脚本中使用bc

				root@lucid32:~# cat test9
				#! /bin/bash
				var1=`echo "scale=4;3.44/5"|bc`
				echo The answer is $var1

				root@lucid32:~# cat test12
				#! /bin/bash
				var1=10.46
				var2=43.67
				var3=33.2
				var4=71

				var5=`bc<<EOF
				scale=4
				a1=($var1 * $var2)
				b1=($var3 * $var4)
				a1+b1
				EOF
				`
				echo the final answer for this mess is $var5

- 退出脚本
	- 核对退出状态
		1. `$?`保存最后一条命令执行结束的退出状态，如想核对一条命令的退出状态，须在该命令执行之后立刻使用`$?`
		2. 成功完成的退出状态是0，如执行错误，则会返回一个正整数
	- 退出命令
		1. `exit`命令允许在脚本结束时，返回一个退出状态
		2. 状态码范围: 0-255

		