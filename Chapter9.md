###使用结构化命令
####使用if-then语句
```
if command
then
	commands
fi
```

- shell脚本中的if语句后面跟的不是true or false, 而是一些语句，如果语句的返回状态为0，则执行then后面的内容
	
		#!/bin/bash
		if date
		then
	    	echo "it worked"
		fi
####使用if-then-else语句
```
#!/bin/bash
testuser=vagrant
if grep $testuser /etc/passwd
then
    echo The bash files for user $testuser are:
    ls -a /home/$testuser/.b*
else
	echo The user $testuser do not exist
fi
```

####嵌套if语句
```
if command1
then
	bulabula
elif command2
then
	bulabula
elif command3
then
	bulabula
fi
```

####test命令
- test命令提供一种检测if-then语句中不同条件的方法，如果test命令中列出的条件评估值为true，test命令以0状态码退出
- 使用格式 `test condition`
- 另一种在if-then语句中声明test的方法

		if [ condition ]
		then
			commands
		fi
- 数值比较
	1. `n1 -eq n2`等于 `n1 -le n2`小于等于 `n1 -ge n2`大于等于 `n1 -lt n2`小于 `n1 -gt n2`大于 `n1 -ne n2`不等于
	2. bash shell只能处理整数数字，我们只是欺骗shell把浮点值作为字符串值存储于一个变量，如果只是echo没有问题，但是在面向数值的函数中不起作用
- 字符串比较
	1. `str1 = str2`等于 `str1 != str2`不等 `str1 < str2`小于 `str1 > str2`大于 `-n str1`字符串长度是否大于0 `-z str1`字符串长度是否等于0 
	2. 字符串顺序
		- 大于和小于符号一定要转义，否则就重定向了
				
				#!/bin/bash
				val1=baseball
				val2=hockey
				if [ $val1 \> $val2 ]
				then
				    echo "$val1 is greater than $val2"
				else
				    echo "$val1 is less than $val2"
				fi
		- 大于和小于顺序与在sort命令中的顺序不同，test命令中，大写字母小于小写字母(ASCII码)，而sort使用系统当前语言设置定义的排列顺序，对英语来说，小写字母在大写字母之前
- 字符串大小
	- 注意：如不确定变量内容，在数值比较或者字符串比较前，最好使用-n 或 -z测试一下是否有值。空变量和没有初始化的变量可能会对shell脚本测试产生灾难性的影响
```
vagrant@lucid32:~/vimPractice$ cat test13
#!/bin/bash
val1=testing
val2=''
if [ -n $val1 ]
then
    echo "the string '$val1' is not empty"
else
    echo "the string '$val1' is empty"
fi

if [ -z $val2 ]
then
    echo "the string '$val2' is empty"
else
    echo "the string '$val2' is not empty"
fi
if [ -z $val3 ]
then
    echo "the string '$val3' is empty"
else
    echo "the string '$val3' is not empty"
fi
#注意，val3没有在shell脚本中定义，所以字符串长度为0
vagrant@lucid32:~/vimPractice$ ./test13
the string 'testing' is not empty
the string '' is empty
the string '' is empty
```

- 文件比较
	- `-e file`既适用于目录也适用于文件，要确定该路径是个文件，需用`-f file`
	- `-nt`和`-ot`的使用，要先确定比较的两个文件存在
| 比较       | 描述           | 
| ------------- |:-------------:|
|`-d file`| 检查file是否存在并且是一个目录 |
|`-e file`| 检查file是否存在|
|`-f file`| 检查file是否存在并且是一个文件|
|`-r file`| 检查file是否存在并且可读|
|`-s file`| 检查file是否存在并且不为空|
|`-w file`| 检查file是否存在并且可写 |
|`-x file`| 检查file是否存在并且可执行 |
|`-O file`| 检查file是否存在并且被当前用户拥有|
|`-G file`| 检查file是否存在并且默认组是否为当前用户组|
|`file1 -nt file2`| 检查file1是否比file2新|
|`file1 -ot file2`| 检查file1是否比file2旧|
         

- 复合条件检查
	- `[ condition1 ] && [ condition2 ]`
	- `[ condition1 ] || [ condition2 ]`


- if-then的高级特性
	- 双圆括号表示数学表达数 `(( expression ))`, 双圆括号里的< >不用转义
![](http://farm6.staticflickr.com/5337/9918247035_1c07f37e26_o.jpg)

			#! /bin/bash
			val1=10
			if (( $val1 ** 2 > 90 ))
			then
			    (( val2 = $val1 ** 2 ))
		    	echo "The square of $val1 is $val2"
			fi
	- 双方括号表示高级字符串处理函数，允许模式匹配

			#! /bin/bash
			if [[ $USER == v* ]]
			then
			    echo "Hello $USER"
			else
			    echo "Sorry, I don't know you"
			fi

####case命令
```
case variable in
pattern1 | pattern2) commands1;;
pattern3) commands2;;
*) default commands;;
esac
```
	