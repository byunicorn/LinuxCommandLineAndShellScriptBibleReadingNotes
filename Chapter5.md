####环境变量
 - 全局环境变量 `printenv`
 - 本地环境变量 `set`
 - 设置环境变量(区分大小写)
	- 本地环境变量

            vagrant@lucid32:~$ test=mytest 
vagrant@lucid32:~$ echo $test 
mytest
vagrant@lucid32:~$ test='testing a long string'
            vagrant@lucid32:~$ echo $test
            testing a long string

 - List item

			vagrant@lucid32:~$ bash
vagrant@lucid32:~$ echo $test

	- 设置本地变量后，可以在shell进程的任何地方使用它，但是如果产生了另一个shell(可以用`bash`命令开一个新shell进程)，则不能使用
    - 全局变量`export` 在导出本地环境变量时，不必使用$符号来引用变量名称
	- 删除环境变量`unset` 同样不用$符号

####默认的shell环境变量
![](http://farm4.staticflickr.com/3808/9918175165_26c237fd9a_o.jpg)

####定位系统环境变量
- bash的启动文件依赖于启动bash shell的方法
	- 登陆shell
		1. /etc/profile
	- 交互式shell(比如在CLI中键入bash)
		2. $HOME/.bashrc
####变量数组

    root@lucid32:~# mytest=(one two three)
    root@lucid32:~# echo $mytest
one
root@lucid32:~# echo ${mytest[1]}
    two
    root@lucid32:~# echo ${mytest[*]}
one two three
root@lucid32:~# unset mytest[2]
root@lucid32:~# echo ${mytest[*]}
	one two

####使用命令别名
- `alias -p` 查看活动别名的列表
- `alias ls='ls -il'`建立自己的别名