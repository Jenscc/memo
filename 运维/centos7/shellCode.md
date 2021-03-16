# shell code

## 命令行

### 链接文件

#### 软链接 sl

```bash
ln -s data_file sl_data_file
```

#### 硬链接 hl

```bash
ln code_file hl_code_file
```



### 查看文件

#### 查看整个文件 -n(添加行号)

##### cat

适合短文件

##### less(more)

适合长文件，需要终端能够识别上下键

#### 查看部分文件 -n(修改显示行数，默认10行)

##### head/tail



## shell脚本

### 执行数学运算

#### expr

![image-20210220193350522](https://gitee.com/jenscc/picgo/raw/master/img/image-20210220193350522.png)

尽管标准操作符在expr命令中工作得很好,但在脚本或命令行上使用它们时仍有问题出现。
许多expr命令操作符在shell中另有含义(比如星号) 。当它们出现在在expr命令中时,会得到一些诡异的结果。

> $ expr 5 * 2
>
> expr: syntax error
>
> $

要解决这个问题,对于那些容易被shell错误解释的字符,在它们传入expr命令之前,需要使
用shell的转义字符(反斜线)将其标出来。

> $ expr 5  \\* 2



```bash
# !/bin/bash
# An example of using the expr command
var1 = 10
var2 = 20
var3 = $(expr $var2 / $var1)
echo The result is $var3
```

要将一个数学算式的结果赋给一个变量,需要使用命令替换来获取expr命令的输出:

> $ chmod u+x test.sh
>
> $ ./test.sh
>
> The result is 2
>
> $



#### 使用方括号

在bash中，在将一个数学运算结果赋给某个变量时，可以用$和[]将数学表达式围起来。

```bash
#! /bin/bash
var1=100
var2=50
var3=45
var4=$[$var1*($var2-$var3)]
echo The final result is $var4
```



#### 浮点解决方法

使用内建的bash计算器bc

浮点运算是由内建变量scale控制的。必须将这个值设置为你希望在计算结果中保留的小数位数，否则无法得到期望的结果。

##### 命令替换

```bash
#!/bin/bash
var1=100
var2=45
var3=$(echo "scale=4; $var1 / $var2" | bc)
echo The answer for this is $var3
```

##### 内联输入重定向

```bash
#!/bin/bash

var1=10.46
var2=43.67
var3=33.2
var4=71

var5=$(bc << EOF
scale = 4
a1 = ($var1 * $var2)
b1 = ($var3 * $var4)
a1 + b1
EOF
)

echo The final answer for this mess is $var5
```



### 状态码

```bash
echo $?
```

![image-20210220202654967](https://gitee.com/jenscc/picgo/raw/master/img/image-20210220202654967.png "Linux退出状态码")



### 结构化命令

#### if-then

###### if-then(-else)

```bash
if command
then 
	commands
(else
	commands)
fi
```

bash shell的if语句会运行if后面的那个命令。如果该命令的退出状态码是0(该命令成功运行),位于then部分的命令就会被执行。如果该命令的退出状态码是其他值, then部分的命令就不会被执行,bash shell会继续执行脚本中的下一个命令。 fi 语句用来表示if-then语句到此结束。

###### if-then嵌套

```bash
if command
then 
	commands
(elif command
then	
	commands)
fi
```



#### test命令

test <font color="grey">*condition*</font>

如果test命令中列出的条件成立，test命令就会退出并返回状态码0.这样if-then语句配合test命令就可以起到其他编程语言中的if-then语句的作用

bash shell提供了另一种条件测试方法

```bash
if [ condition ]
then
	commands
fi
```

方括号定义了测试条件。注意，第一个方括号之后和第二个方括号之前必须加上一个空格

##### 文件比较

![image-20210221195326502](https://gitee.com/jenscc/picgo/raw/master/img/image-20210221195326502.png "test命令的文件比较功能")

###### 1.检查目录

-d会检查指定的目录是否存在于系统中

将文件写入目录或准备切换到某个目录中，先进行测试

```bash
#!/bin/bash
jump_directory=/home/jens
if [ -d $jump_directory ]; then
    echo "The $jump_directory directory already exists"
    cd $jump_directory
    ls
else
    echo "The $jump_directory directory does not exist"
fi
```

###### 2.检查对象是否存在

-e检查文件或目录是否存在

```bash
#!/bin/bash
# Check if either a directory or file exists
location=$HOME
file_name=".zshrc"

if [ -e $location ]
then # Directory does exist
    echo "Now on the $location directory"
    echo "Now checking on the file, $file_name"

    if [ -e $location/$file_name ]
    then # File exists
        echo "$file_name exists"
    else
        echo "$file_name does not exist"
    fi
else
    echo "$location does not exist"
fi
```



#### 复合条件测试

[ condition1 ] && [ codition2 ]

[ condition1 ] || [ codition2 ]



#### if-then高级特性

##### 1.双括号高级表达式

![image-20210221201741058](https://gitee.com/jenscc/picgo/raw/master/img/image-20210221201741058.png)

##### 2.双方括号

双方括号里的expression使用了test 命令中采用的标准字符串比较。但它提供了test命
令未提供的另一个特性——模式匹配/正则

```bash
#!/bin/bash
# using pattern matching
#
if [[ $USER == r* ]]
then
echo "Hello $USER"
else
echo "Sorry, I do not know you"
fi
```



#### case命令

case命令会采用列表格式检查单个变量的多个值

case *variable* in
*pattern1* | *pattern2*) *commands1*;;
*pattern3*) *commands2*;;
*) *default commands*;;
esac

```bash
#!/bin/bash
# Using the case command
#
case $USER in
c* | b*)
    echo "Welcome, $USER"
    echo "Please enjoy your visit";;
testing)
    echo "Special testing account";;
j*)
    echo "root account";;
*)
    echo "something wrong"
esac
```



#### for命令

for var in *list*

do

​	*commands*

done

##### 列表的复杂值

特殊字符需要用转义字符(\\)

for循环假定每个值都是用空格分割的。遇到包含空格的值需要用双引号("")圈起

##### 从变量读取列表

```bash
#!/usr/bin/env bash
# using variable to hold the list
#
list="Alabama Alaska Arkanss Colorado"
list=$list" Connecticut"

for state in $list; do
    echo "Have you ever visited $state?"
done
```

##### 从命令读取值

```bash
#!/bin/bash
# reading values from a file

file="states"

for state in $(cat $file)
do
	echo "Visit beautiful $state"
done
```



#### C语言风格的for命令

```bash
#!/bin/bash
# c-style loop

for (( i = 1; i <= 10; i++ ))
do
	echo "The next number is $i"
done
```



#### while命令

while test *commands*

do

​	*other commands*

done

```bash
#!/bin/bash

var1=10
while [ $var1 -gt 0 ]
do
	echo $var1
	var1=$[ $var1 - 1 ]
done
```



while命令允许在while语句行定义多个测试命令，但只有**最后一个**测试命令的退出状态码会被用来决定什么时候结束循环。

在包含多个命令的while语句中，在每次迭代中所有的测试命令都会被执行，包括测试命令失败的最后一次迭代



#### until命令

until命令和while命令的工作方式完全相反。until命令要求你指定一个通常返回非零退出状态码的测试命令。只有测试命令的退出状态不为0,bash shell才会执行循环中列出的命令。一旦测试命令返回了退出状态码0,循环就结束了。

until test *commands*

do

​	*other commands*

done

























