###处理用户输入
####命令行参数
- 读取参数
	- `$0`为程序名称

	- `$1`为第一个参数以此类推

			#!/bin/bash
			factorial=1
			for (( number = 1; number <= $1; number++ ))
			do
    			factorial=$[ $factorial * $number ]
			done
			echo The factorial of $1 is $factorial
	
	- 参数靠空格分隔，所以如果一个参数包含有空格，需要使用引号`ls -l 'by tool'`
	- 9个变量之后，必须使用大括号将变量括起来，如`${10}`       
      
    

- 读取程序名称

```
#传递给$0的其实是路径，而不仅仅是名称
vagrant@lucid32:~/linuxAndShell/chapter11$ cat test2
#!/bin/bash
echo The command entered is: $0
vagrant@lucid32:~/linuxAndShell/chapter11$ ./test2
The command entered is: ./test2
vagrant@lucid32:~/linuxAndShell/chapter11$ ~/linuxAndShell/chapter11/test2
The command entered is: /home/vagrant/linuxAndShell/chapter11/test2

# basename命令只返回程序名称，不带路径
#!/bin/bash
name=`basename $0`
echo The command entered is: $name
```

- 测试参数
	- 不确定参数是否存在时，在使用前，可以先测试它 `if [ -n "$1"]` (是否存在参数1)

- 特殊的参数变量
	1. `$#`参数个数

	2. `if [ $# -ne 2 ]` 判断参数个数

	3. 获取最后一个参数，`${!#}`， 而不是`${$#}`
- 获取所有数据
	- `$@`将命令行中提供的所有参数作为同一个字符串的多个单词处理
	- `$*`将命令行中提供的所有参数作为一个单词处理

```
#!/bin/bash
count=1
for param in "$*"
do
    echo "\$* paramter #$count = $param"
    count=$[ $count + 1 ]
done
count=1
for param in "$@"
do
    echo "\$@ paramter #count = $param"
    count=$[ $count + 1 ]
done

vagrant@lucid32:~/linuxAndShell/chapter11$ ./test4 1 2 3
$* paramter #1 = 1 2 3
$@ paramter #count = 1
$@ paramter #count = 2
$@ paramter #count = 3
```

- 移位
	- `shift`左移

			#!/bin/bash
			echo The original paramters: $*
shift 2
echo "Here's the new first parameter: $1"

			vagrant@lucid32:~/linuxAndShell/chapter11$ ./test5 1 2 3 4 5
			The original paramters: 1 2 3 4 5
			Here's the new first parameter: 3

- 处理选项
	- 对于既需要使用选项又需要使用参数的情况，可以通过特殊字符码`--`将两者分开，它代表选项的结束
	- 处理带值的的选项，则要shift两次（下面示例中的-b选项
	- getopt命令
		1. 命令格式`getopt options optstring parameters`
		2. 选项字符串optstring，定义命令行中有效选项字母，还定义哪些选项字母需要参数值，先列出将在脚本中用到的每个命令行选项字母，需要参数值的选项字母后面放置一个冒号
		3. `getopt ab:cd -a -b test1 -cd test2 test3` `-a -b test1 -c -d -- test2 test3`
		4. 上一条命令解析时自动将-cd选项分割成两个不同的选项，并插入双破折号来分隔行中的额外参数
	- 脚本中使用getopt
		- set命令的一个选项是双破折号，表示将命令行参数变量替换为set命令的命令行中的值，即，将原始脚本命令行参数送给getopt，然后将getopt解析的结果送给set命令
	- 更高级的getopts命令 `getopts optstring variable`，如果要禁止输出错误消息，那么使选项字符串冒号开头，getopts命令使用两个环境变量，`OPTARG`包含需要参数值的选项要使用的值，环境变量`OPTIND`包含的值表示getopts停止处理时在参数列表的位置，getopts命令解析命令行选项时，会将打头的破折号去掉，所以在case定义中不需要破折号
```
#!/bin/bash
while [ -n "$1" ]
do
     case "$1" in
        -a) echo "found -a option";;
        -b) param=$2
            echo "found the -b option, with param value $param"
            shift;;
        -c) echo "found -c option";;
        --) shift
            break;;
        *) echo "$1 is not an option";;
        esac
        shift
done
count = 1
for param in $@
do
		echo "paramter #$count: $param"
		count=$[ $count + 1 ]
done
```
```
vagrant@lucid32:~/linuxAndShell/chapter11$ cat test8
#!/bin/bash
set -- `getopt -q ab:c "$@"`
while [ -n "$1" ]
do
    case "$1" in
        -a) echo "Found the -a option";;
        -b) param="$2"
            echo "Found the -b option, with parameter value $param"
            shift;;
        -c) echo "Found the -c option";;
        --) shift
            break;;
        *) echo "$1 is not an option";;
        esac
        shift
done
count=1
for param in "$@"
do
    echo "Parameter #$count: $param"
    count=$[ $count + 1 ]
done
```

```
#!/bin/bash
while getopts :ab:c opt
do
    case "$opt" in
        a) echo "Found the -a option";;
        b) echo "Found the -b option, with value $OPTARG";;
        c) echo "Found the -c option";;
        *) echo "Unknown option: $opt";;
    esac
done

vagrant@lucid32:~/linuxAndShell/chapter11$ ./test9 -ab test1 -c
Found the -a option
Found the -b option, with value test1
Found the -c option
```
- 标准化选项

![](http://farm8.staticflickr.com/7291/9918341636_79610223cd_o.jpg)

- 获取用户输入
	- 基本读取
		1. `read`命令接受标准输入 (-n抑制字符串结尾的新行符，允许用户在字符串后面立刻输入数据，而不是在下一行中输入，使脚本看起来更整齐
		2. `read -p` p选项允许read命令行中直接指定一个提示
		3. `read -p "Please enter your name:" first last`
				
				#!/bin/bash
				echo -n "Enter your name: " 
				read name
				echo "Hello $name, welcome to my program."
	- 计时
		1. `read -t 5`可以指定一个计时器，指定等待输入的秒数，当计时器计时器数满时，read命令返回一个非零退出状态
		2. 除了计时，还可以设置read命令计数输入的字符，当输入字符数目达到预定数目时自动退出，并将输入的数据赋值给变量`read `
				
				#!/bin/bash
				read -n 1 -p "Do you want to continue [Y/N]?" answer
				case $answer in
				Y | y) echo
        			echo "fine, continue on...";;
				N | n) echo
        			echo OK, goodbye
        			exit;;
				esac
				echo "This is the end of the script"
	- 默读
		- `read -s`能够使该命令输入的数据不显示在监视器上
		- `read -s -p "Enter your password: " pass`
	- 读取文件

			#!/bin/bash
			count=1
			cat test|while read line
			do
		    	echo "Line $count: $line"
		    	count=$[ $count + 1 ]
			done
			echo "Finished processing the file"