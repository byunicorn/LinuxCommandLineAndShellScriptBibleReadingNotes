###创建函数
####基本脚本函数
- 定义函数

```
#方法1
function name{
	commands
}
#方法2
name(){
	commands
}
```
- 使用函数
	- 直接使用name就能调用，定义需要在调用之前，函数名必须唯一，重新定义的话，会覆盖原有的定义，而不会报错

#### 返回值
- 默认情况下，退出状态是函数最后一条命令返回的退出状态，函数执行完毕后，可以用`$?`(返回最近已执行命令的退出状态)来确定函数的退出状态
- 使用return命令：在函数完成后尽快提取返回值；退出状态的取值范围是0~255

```
#!/bin/bash
function dbl { #注意：functionName后面一定要有一个空格
    read -p "Enter a value: " value
    echo "doubling the value"
    return $[ $value * 2 ]
}

dbl
echo "The new value is $?"
```

- 使用函数输出，函数的输出也可以捕获并存放到shell变量中

```
#!/bin/bash
function dbl {
    read -p "Enter a value: " value
    echo $[ $value * 2 ]
}
result=`dbl`
echo "The new value is $result"
```

####在函数中使用变量
- 向函数传递参数，和普通调用脚本一样
- 如果想要调用脚本的参数，`./test1 10 15`，在函数中不能直接调用$1和$2来得到10和15，需要在使用函数时显式地传入`functionName $1 $2`

```
#!/bin/bash
function addem {
    if [ $# -eq 0 ] || [ $# -gt 2 ]
    then
        echo -1
    elif [ $# -eq 1 ]
    then
        echo $[ $1 + $1 ]
    else
        echo $[ $1 + $2 ]
    fi
}
echo -n "Adding 10 and 15: "
value=`addem 10 15`
echo $value
echo -n "Let's try adding just one number: "
value=`addem 10`
echo $value
```

- 在函数中处理变量：全局变量和局部变量（local关键字）

```
#!/bin/bash
function dbl {
    value=$[ $value * 2 ]
}
read -p "Enter a value: " value
dbl
echo The new value is: $value
```

```
#!/bin/bash
function funcl {
    temp=$[ $value + 5 ] #函数体里定义的也能在外部访问到
    result=$[ $temp * 2 ] #如果想定义局部变量，需要使用local关键字
}
value=6
funcl
echo "The result is $result"
echo "The temp is $temp"
```

####数组变量和函数
- 向函数传递数组：函数只会提取数组变量的第一个值，如果想解决这个问题，必须将数组拆开，传给函数后，再在函数体内重组

```
#!/bin/bash
function testit {
    local newarray
    newarray=(`echo "$@"`)
    echo "The new array value is: ${newarray[*]}"
}
myarray=(1 2 3 4 5)
echo The original array is ${myarray[*]}
#这里如果用testit myarray，则只会传递第一个元素给函数
testit ${myarray[*]}
```

- 从函数返回数组：和传入数组时一样，用`echo ${innerArray[*]}`来返回数组，用`result=(${funcl argument})`来重组数组


####函数递归
```
#!/bin/bash
function factorial {
    if [ $1 -eq 1 ]
    then
        echo 1
    else
        local temp=$[ $1 - 1 ]
        local result=`factorial $temp`
        echo $[ $result * $1 ]
    fi
}
read -p "Enter value: " value
result=`factorial $value`
echo "The factorial of $value is: $result"
```

####库文件
- shell函数的作用域，仅在其定义所处的shell会话中有效，如果从shell命令界面运行一个库文件，那么shell将会打开一个新的shell，并在新shell中运行这个脚本，这将为新shell定义函数。
- `source`命令，source命令在当前运行的shell环境中执行，`source`有个短小的别名`.`

```
#!/bin/bash
. ./myfuncs #如果不使用source命令，则会新开一个shell，下面将无法调用myfuncs里的函数

value1=10
value2=5
result1=`addem $value1 $value2`
result2=`multem $value1 $value2`
result3=`divem $value1 $value2`
echo "The result of adding them is: $result1"
echo "The result of multiplying them is: $result2"
echo "The result of dividing them is: $result3"
```

####在命令行中使用函数
- 在命令行创建函数，如果定义在一行内，则每条命令的结尾必须包括分号
- 用命令行创建函数时需要小心

```
vagrant@lucid32:~/linuxAndShell/chapter14$ function divem { echo $[ $1 / $2 ];}
vagrant@lucid32:~/linuxAndShell/chapter14$ divem 100 5
20

vagrant@lucid32:~/linuxAndShell/chapter14$ function multem {
> echo $[ $1 * $2 ]
> }
vagrant@lucid32:~/linuxAndShell/chapter14$ multem 2 5
10
```

- 在`.bashrc`文件中定义函数
	- 可以在主目录下的.bashrc文件的最后面直接定义函数
	- 也可以用`source`命令将库文件的函数包含进`.bashrc`脚本
