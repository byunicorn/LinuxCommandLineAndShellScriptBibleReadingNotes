###sed和gawk介绍

####文本处理
- sed称为流编辑器，将根据编辑器处理数据之前事先提供的规则集编辑数据流
- 每次读入一行数据，将该数据与所提供的编辑器命令进行匹配，根据命令修改数据流中的数据，然后将新数据输出到STDOUT，接着读取下一行数据
- `sed options script file`

![](http://farm6.staticflickr.com/5464/9918515854_7423e5c302_o.jpg)

- 在命令行中定义编辑器命令

```
vagrant@lucid32:~/linuxAndShell/chapter16$ echo "This is a test"|sed 's/test/big test/'
This is a big test
```

- 多个命令需要使用`-e`，命令用分号隔开，命令结尾和分号之间不能有空格

```
vagrant@lucid32:~/linuxAndShell/chapter16$ cat data1
The quick brown fox jumps over the lazy dog.
The quick brown fox jumps over the lazy dog.
vagrant@lucid32:~/linuxAndShell/chapter16$ sed -e 's/brown/green/; s/dog/cat/' data1
The quick green fox jumps over the lazy cat.
The quick green fox jumps over the lazy cat.
```

- 从文件读取编辑器命令

```
vagrant@lucid32:~/linuxAndShell/chapter16$ cat script1
s/brown/green/
s/fox/elephant/
s/dog/cat/

vagrant@lucid32:~/linuxAndShell/chapter16$ sed -f script1 data1
The quick green elephant jumps over the lazy cat.
The quick green elephant jumps over the lazy cat.
```

- awk程序，提供了一种编程语言而不仅仅是编辑器命令
- `awk options program file`

![](http://farm3.staticflickr.com/2849/9918607363_1ecc3357d3_o.jpg)

- awk命令行假定脚本是单文本字符串，所以须将脚本包括在单引号内

```
vagrant@lucid32:~/linuxAndShell/chapter16$ awk '{print "Hello world!"}'
This is a test
Hello world!
hello
Hello world!
#要结束awk程序，必须信号说明数据流已经结束，bash中提供的生成End-of-File字符的组合键：Ctrl+D
```

- 使用数据字段变量
	- `$0` 整行文本， `$1` 文本行中第一个数据字段, `$n` 文本行中第n个数据字段
	- `awk -F: '{print $1}' /etc/passwd`
	- 多个命令可以用分号隔开，或者用每次输入一行(多行)

			vagrant@lucid32:~/linuxAndShell/chapter16$ echo "My name is Rich"|awk '{$4="Dave";print $0}'
			My name is Dave
			vagrant@lucid32:~/linuxAndShell/chapter16$ awk '{
			> $4="testing"
			> print $0}'
			This is not a good test.
			This is not testing good test.

	- 从文件读取程序

			vagrant@lucid32:~/linuxAndShell/chapter16$ vim script2
			vagrant@lucid32:~/linuxAndShell/chapter16$ awk -F: -f !$ /etc/passwd
			awk -F: -f script2 /etc/passwd
			root's userid is root
			daemon's userid is daemon
			...
			vagrant@lucid32:~/linuxAndShell/chapter16$ cat script2
			{print $5 "'s userid is " $1}
			
    - 在处理数据之前/之后运行脚本
        - awk允许指定运行脚本的时间，可能需要在处理数据之前运行脚本`awk 'BEGIN {print "title"} {print $0} END {print "end"}`

```
vagrant@lucid32:~/linuxAndShell/chapter16$ cat script4
BEGIN {
    print "The latest list of users and shells"
    print "Userid       Shell"
    print "------       ------"
    FS=":"
}
{
    print $1 "  " $7
}
END {This\tline\tcontains\t\ttabs.$
		
    print "This concludes the listing"
}
```

#### sed编辑器基础知识
- 更多替换选项：`s/pattern/replacement/flag`
    - 数字：表示新文本替换的模式，指定替换哪一次的实例
    - g：表示用新文本替换现有文本的全部实例
    - p：表示打印原始行的内容
    - w file：将替换的结果写入文件
    
            # -n 选项禁止sed编辑器的输出。然而，p替换标记会输出所有已修改的行，二者结合使用仅生成替换命令已更改的那些行的输出。
            vagrant@lucid32:~/linuxAndShell/chapter16$ sed -n 's/test/trail/p' data5
            This is a trail line.
            vagrant@lucid32:~/linuxAndShell/chapter16$ cat data5
            This is a test line.
            This is a different line.
            
    - P.S.替换某个文件中的路径名可能会很难，例`sed 's/\/bin\/bash\/\bin\/csh/' /etc/passwd`，因为正斜杠要用作字符串界定符，所以如果正斜杠出现在模式文本中，必须使用反斜杠使其转义，但这样很容易混淆，为了解决这个问题，sed允许为替换命令中的字符串界定符选择一个不同的字符，例`sed 's!/bin/bash!/bin/csh!' /etc/passwd`
    
- 使用地址
    - 如果仅想将某个命令应用于某一特定的文本数据行或一组文本数据行，必须使用行寻址
        1. 行的数值范围
        2. 筛选行的文本模式
        
                #两种形式使用相同的格式指定地址
                [address]command
                #多条命令的用法
                address { 
                    command1 
                    command2
                }
                #数字式行寻址，首行行号为1，每个新行的行号顺延
                sed '2s/dog/cat/' data1 #只替换第二行的dog
                sed '2,3s/dog/cat/' data1 #换2,3行的dog
                sed '2,$s/dog/cat/' data1 #从第二行到最后一行
                
                #使用文本模式筛选器
                /pattern/command
                #如果只想为用户rich更改默认的shell
                sed '/rich/s/bash/csh/' /etc/passwd
                
                #组合命令，某一行上执行多个命令
                sed '2{
                s/fox/elephant/
                s/dog/cat/
                }' data1
        
    - 删除行，`d`命令，将删除与所提供的寻址模式一致的所有文本行
    
            sed '3d' data1 #按行号删除第三行
            sed '/number 1/d' data1 #删除包含字符串number 1所在的行
            
            #还可以使用两个文本模式删除若干行，第一个模式将“打开”行删除，第二个模式将“关闭”行删除，sed编辑器将删除指定的这两行（包括指定的行）之间所有的文本行，如果发现了开始匹配的模式，没有找到匹配的结束模式，那么整个数据流都会被删除
            sed '/1/,/3/d' data1
            
    - 插入和附加文本，插入命令`i`(指定行之前添加新的一行)，附加命令`a`(在指定行之后添加新的一行)
    
            #不能在单命令行上使用这两个命令，必须单独指定要插入/附加的行
            sed '[address]command\
            new line'
        
            vagrant@lucid32:~/linuxAndShell/chapter16$ echo "testing"|sed 'i\
            > This is a test'
            This is a test
            testing
            
            #在第二行后面插入一行
            vagrant@lucid32:~/linuxAndShell/chapter16$ sed '2i\
            > This is an inserted line.' data1
            The quick brown fox jumps over the lazy dog.
            The quick brown fox jumps over the lazy dog.
            This is an inserted line.
            The quick brown fox jumps over the lazy dog.
            
            #插入多行
            vagrant@lucid32:~/linuxAndShell/chapter16$ echo "testing"|sed '$a\
            > This is one line.\
            > This is another line.'
            testing
            This is one line.
            This is another line.
            
    - 更改行，命令`c`
    
            vagrant@lucid32:~/linuxAndShell/chapter16$ sed '/number 3/c\
            > This is a changed line of text.' data6
            This is line number 1.
            This is line number 2.
            This is a changed line of text.
            This is line number 4.
            
    - 变换命令，`y`对单个字符进行操作的sed编辑命令
    
            [address]y/inchars/outchars/
            vagrant@lucid32:~/linuxAndShell/chapter16$ sed 'y/123/789/' data6
            This is line number 7.
            This is line number 8.
            This is line number 9.
            This is line number 4.
    
    - 打印命令温习
        - p：表示打印原始行的内容
        
                vagrant@lucid32:~/linuxAndShell/chapter16$ sed -n '/3/{
                > p
                > s/line/test/p
                > }' data6
                This is line number 3.
                This is test number 3.
                
        - =: 打印行号
                
                vagrant@lucid32:~/linuxAndShell/chapter16$ sed -n '/number 4/{
                > =
                > p
                > }' data6
                4
                This is line number 4.

                vagrant@lucid32:~/linuxAndShell/chapter16$ sed -n 'l' data8 #\033是Esc键的ASCII码的八进制值
		
		`vagrant@lucid32:~/linuxAndShell/chapter16$ echo ^[[41m"test"^[[0m|sed 'l'`
		`\033[41mtest\033[0m$`
		`test`
                
        - l (小写L): 列出行，允许打印数据流中的文本和不可打印的ASCII字符
        - 将文件用于sed
            - 写文件，[address]w filename
            - 从文件读取数据，[address]r filename，对于读命令，不能使用地址范围，只能指定单一的行号或文本模式地址，sed编辑器在该地址之后插入文件中的文本
            
                    vagrant@lucid32:~/linuxAndShell/chapter16$ sed '1,2w test' data6
                    This is line number 1.
                    This is line number 2.
                    This is line number 3.
                    This is line number 4.
                    vagrant@lucid32:~/linuxAndShell/chapter16$ cat test
                    This is line number 1.
                    This is line number 2.
                    vagrant@lucid32:~/linuxAndShell/chapter16$ sed '3r data7' data6
                    This is line number 1.
                    This is line number 2.
                    This is line number 3.
                    This is an added line.
                    This is the second added line.
                    This is line number 4.
