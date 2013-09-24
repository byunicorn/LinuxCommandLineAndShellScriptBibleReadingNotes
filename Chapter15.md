
###在脚本中添加颜色
####创建文本菜单
- 创建菜单布局
	- `echo -e "1.\tDisplay disk space"` -e选项允许打印制表符和换行符

```
#!/bin/bash
clear
echo
echo -e "\t\t\tSys Admin Menu\n"
echo -e "\t1. Display disk space"
echo -e "\t2. Display logged on users"
echo -e "\t3. Display memory usage"
echo -en "\t\tEnter option: " #-n 不换行
read -n 1 option #-n 1 表示只读一个字符，用户不用按enter
```

- 创建菜单函数
	- 可以为未实现的函数创建stub函数(尚未包含任何命令的函数)，为了可以顺利地进行调试
```
function diskspace {
    clear
    echo "This is where the diskspace commands will go"
}
```

- select命令
	- select命令允许从单命令行创建菜单，然后获取输入的答案并自动处理它
```
select variable in list
do
    commands
done

PS3="Enter option: " #PS3 定义选择的提示符
select option in "Display disk space" "Display logged on users" "Display memory
usage" "Exit program"
do
    case $option in
    "Exit program")  #注意，存储在变量中的结果值是整个文本字符串，不是与该菜单相关联的数字
        break;;
    "Display disk space")
        diskspace;;
    "Display logged on users")
        whoseon;;
    "Display memory usage")
        memusage;;
    *)
        clear
        echo "Sorry, wrong selection";;
    esac
done
clear
```

####添加颜色
- ANSI转义码以控制序列指示器(CSI)开头，后面跟表示要在显示器上执行的操作的数据
- 有些ANSI转义码可以用于将光标定位在显示器上的指定位置，擦除部分显示，以及控制显示的格式。这就必须使用Select Graphic Rendition， SGR转义码，格式为`CSIn[;k]m` m表示转义码，n和k参数定义所使用的显示控制，中间用分号隔开
- 要显示设置为使用斜体和闪烁，可以发送代码`CSI3;5m`

![](http://farm6.staticflickr.com/5467/9918470926_7091d16431_o.jpg)

- 要指定背景黑色，前景红色，可使用代码`CSI31;40m` (3表示前景，1表示红色，4表示背景)

![](http://farm4.staticflickr.com/3687/9918516344_24165592e7_o.jpg)

- 显示ANSI转义码，问题是创建CSI字符，该字符通常是一个两字符序列：`ESC ASCII`值，后跟左方括号字符，需要在脚本中插入ESC值时，就需要用`Ctrl-V`组合键后跟ESC值，输入此组合键时，字符^[出现。
```
#下面的命令将输入红底白字的This is a test，输出之后又恢复为普通模式，如果不恢复，则ANSI颜色控制会保持有效
vagrant@lucid32:~/linuxAndShell/chapter15$ echo ^[[41mThis is a test^[[0m
This is a test
#在一个echo中放置两个转义控制码时，需要用引号将代码字符串起来，否则会无法解析，进而报错
vagrant@lucid32:~/linuxAndShell/chapter15$ echo "^[[33;44mThis is a test^[[0m"
This is a test
```

####制作窗口
- dialog软件包
```
dialog --widget parameters
```

![](http://farm4.staticflickr.com/3817/9918439305_145f2e7161_o.jpg)

- 每个对话框小部件均已两种形式提供输出: 1.STDERR 2.使用退出代码状态