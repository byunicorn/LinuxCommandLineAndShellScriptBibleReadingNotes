###更多结构化命令
####for命令
- 语法

```
for var in list
do
    commands
done
```

```
#!/bin/bash
for test in Alabama Alaska Arizona
do
    echo The next state is $test
done
echo $test 				#退出循环后，仍然可以使用该变量，并可以更改该值
test=Connecticut
echo $test

vagrant@lucid32:~/vimPractice$ ./test17
The next state is Alabama
The next state is Alaska
The next state is Arizona
Arizona
Connecticut
```

- 读取列表中的复杂值

```
#! /bin/bash
for test in I don't know if this'll work
do
    echo "word:$test"
done

vagrant@lucid32:~/vimPractice$ ./test18
word:I
word:dont know if thisll  #shell看到值中的单引号，会试图用它们来定义一个单独的数据值
						  #可以使用转义字符don\'t，或者用双引号来定义引用单引号的值"this'll"
word:work
```

- 为了使for可以更好地区分不同值，可以使用双引号包围每个值，shell不会将双引号作为值的一部分

- 从变量读取列表

```
#!/bin/bash
list="Alabama Alaska Arizona"
list=$list" Arkansas"     #代码可以使用这样的方式往已存在的列表添加一条项目
for state in $list
do
    echo $state
done

vagrant@lucid32:~/vimPractice$ ./test19
Alabama
Alaska
Arizona
Arkansas
```

- 读取命令中的值

```
#! /bin/bash
file="states"
for state in `cat $file`
do
    echo $state
done
```

- 改变字段分隔符
	- 内部字段分隔符(IFS)
	- 默认情况下，bash shell会把以下几种字符看作字段分隔符
		- 空格
		- 制表符
		- 换行符
	- 可在shell脚本中，临时改变IFS的值, 如`IFS=$'\n'`这样shell将只识别换行符
	
	- 需要指定多个字符时，只需要在赋值行中串联起来即可，如`IFS=$'\n':;` (换行符、冒号、分号)
- 通过通配符读取目录


		#! /bin/bash
		for file in /home/vagrant/* /home/vagrant/badtest
		do
		    if [ -d "$file" ]	
    then
        echo "$file is a directory"
	        elif [ -f "$file" ]
    then
        echo "$file is a file"
		    else								#注意else后面就没有then了
                echo "$file doesn't exist"
            fi
		done

- C式的for命令
	- `for (( variable assignment ; condition ; iteration process ))`
	- `for (( a=1 ; a<10; a++ ))`
		1. 变量的赋值可以包含空格
		2. 条件中的变量不以美元符号作为前缀
		3. 迭代处理式不使用expr命令格式 (见第8章)

				#!/bin/bash
				for (( i = 1; i <= 10; i++ ))
				do
				    echo "The next number is $i"
				done

####while命令
- 用法

```
while test commands
do
    other commands
done
```

- 例子

```
#!/bin/bash
var1=10
while [ $var1 -gt 0 ]
do
    echo $var1
    var1=$[ $var1 - 1 ]
done
```

- 多条测试命令: 只按照最后一条测试命令的执行结果来决定循环是否停止

```
while echo $var1
      [ $var1 -ge 0 ]
```

####until命令
- 与while相反，当执行退出状态非0，shell就会执行在循环中的命令，一旦测试返回条件为0，则退出循环
- 用法

```
until test commands
do
    other commands
done
```

####嵌套循环
- 99乘法

```
#!/bin/bash
for (( i = 1; i <=9; i++))
do
    for(( j = 1; j <= i; j++))
    do
        result=$[ $i * $j ]
        echo "$i * $j = $result"
    done
done
```

####文件数据的循环
- 可以通过改变IFS来对文件内容进行操作

```
#!/bin/bash
IFS.OLD=$IFS
IFS=$'\n'
for entry in `cat /etc/passwd`
do
    echo "Values in $entry -"
    IFS=$':'
    for value in $entry
    do
        echo "$value"
    done
done
```

####控制循环
- break命令
	- 多重循环时，break命令会自动终止所在的最里面的循环
	- 跳出外循环 `break n` n表明要跳出的循环级别，如果n设置为2，break命令将停止外循环的下一级循环

```
#!/bin/bash
for (( a = 1; a < 4; a++ ))
do
    echo "Outer loop: $a"
    for (( b = 1; b < 100; b++ ))
    do
        if [ $b -gt 4 ]
        then
            break 2
        fi
        echo "Inner loop: $b"
    done
done

vagrant@lucid32:~/vimPractice$ ./test29
Outer loop: 1
Inner loop: 1
Inner loop: 2
Inner loop: 3
Inner loop: 4
```

- continue命令
	- 和break一样，也可以用`continue n`

```
#!/bin/bash
for (( a = 1; a <= 5; a++ ))
do
    echo "Iteration $a:"
    for (( b = 1; b < 3; b++ ))
    do
        if [ $a -gt 2 ] && [ $a -lt 4 ]
        then
            continue 2
        fi
        var3=$[ $a * $b ]
        echo "The result of $a * $b is $var3"
    done
done

vagrant@lucid32:~/vimPractice$ ./test30
Iteration 1:
The result of 1 * 1 is 1
The result of 1 * 2 is 2
Iteration 2:
The result of 2 * 1 is 2
The result of 2 * 2 is 4
Iteration 3:
Iteration 4:
The result of 4 * 1 is 4
The result of 4 * 2 is 8
Iteration 5:
The result of 5 * 1 is 5
The result of 5 * 2 is 10
```

- 处理循环的输出
	- 可以在done的结尾加上处理语句
		
			#!/bin/bash
			for (( a = 1; a < 10; a++ ))
			do
			    echo "The number is $a"
			done > loop.txt
	
	- 也可以用于将循环的输出管道传送给其他命令(输出会被存到管道里，在loop完之后，echo出来)
			
			#!/bin/bash
			for state in "North Dakota" Connecticut Illinois
			do
    			echo "$state is the next place to go"
			done | sort

			vagrant@lucid32:~/vimPractice$ ./test32
			Connecticut is the next place to go
			Illinois is the next place to go
			North Dakota is the next place to go

