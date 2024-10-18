### Shell概念

#### Shell解释器

```shell
#!/bin/sh

# 获取当前进程的解释器
# ps -p $$：获取当前进程的详细信息，其中 $$ 是当前脚本的进程 ID
# -o cmd=：只输出命令行部分，省略列标题
# interpreter=$(...)：将命令输出的结果存储在 interpreter 变量中
interpreter=$(ps -p $$ -o cmd=)

echo "Interpreter used: $interpreter"
```



#### Shell变量

变量名和等号之间不能有空格，变量命名规则：

-   只包含数字、字母和下划线，大小写敏感
-   不能以数字开头
-   变量名中不应该包含空格，因为空格通常用于分隔命令和参数

```shell
var="123"
```

使用一个定义过的变量，只要在变量名前面加美元符号即可

```shell
var="123"
echo $var
echo ${var}
```

使用 readonly 命令可以将变量定义为只读变量，只读变量的值不能被改变

```shell
myUrl="https://www.google.com"
readonly myUrl
myUrl="https://www.runoob.com"	# 报错，myUrl: readonly variable
```

使用 unset 命令可以删除变量

```shell
unset variable_name
```

可以使用单引号 **'** 或双引号 **"** 来定义字符串

```shell
my_string='Hello, World!'
my_string="Hello, World!"
```

可以使用 `declare` 或 `typeset` 命令来显式声明一个变量为整数。这种方法确保变量只能存储整数值

```shell
declare -i num=42
# 或者
typeset -i num=42
```

数组可以是整数索引数组或关联数组

```shell
my_array=(1 2 3 4 5)
```

```shell
declare -A associative_array
associative_array["name"]="John"
associative_array["age"]=30
```

获取操作系统或用户设置的特殊变量，用于配置 Shell 的行为和影响其执行环境

```shell
echo $PATH
```

有一些特殊变量在 Shell 中具有特殊含义，例如 **$0** 表示脚本的名称，**$1**, **$2**, 等表示脚本的参数。

**$#**表示传递给脚本的参数数量，**$?** 表示上一个命令的退出状态等

#### Shell 字符串

单引号字符串的限制：

-   单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的
-   单引号字符串中不能出现单独一个的单引号（对单引号使用转义符后也不行），但可成对出现，作为字符串拼接使用

而双引号就没有这些缺点

```shell
your_name="runoob"
str="Hello, I know you are \"$your_name\"! \n"
echo -e $str	# -e选项用于启用解释反斜杠转义序列的功能
```

拼接字符串

```shell
your_name="runoob"
# 使用双引号拼接
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"
echo $greeting  $greeting_1

# 使用单引号拼接
greeting_2='hello, '$your_name' !'
greeting_3='hello, ${your_name} !'
echo $greeting_2  $greeting_3

# output
hello, runoob ! hello, runoob !
hello, runoob ! hello, ${your_name} !
```

获取字符串长度

```shell
string="abcd"
echo ${#string}   # 输出 4
echo ${#string[0]}   # 输出 4
```

提取子字符串

```shell
string="runoob is a great site"
echo ${string:2:1} # 输出 n，从下标2开始截取1个字符
echo ${string:1} # 输出 unoob is a great site
echo ${string::2} # 输出 ru
echo ${string::-1} # 输出 runoob is a great sit
```

查找子字符串

```shell
string="runoob is a great site"
echo `expr index "$string" io`  # 输出 4
```

#### Shell 数组

在 Shell 中，用括号来表示数组，数组元素用"空格"符号分割开。定义数组的一般形式为：

```shell
数组名=(值1 值2 ... 值n)

# 下标定义
array_name[0]=value0
array_name[1]=value1
array_name[2]=value2
```

读取数组元素，使用 **@** 符号可以获取数组中的所有元素

```shell
${数组名[下标]}
echo ${array_name[@]}
```

关联数组，**-A** 选项就是用于声明一个关联数组

```shell
declare -A array_name
```

```shell
declare -A site=(["google"]="www.google.com" ["runoob"]="www.runoob.com" ["taobao"]="www.taobao.com")

# 或者
declare -A site
site["google"]="www.google.com"
site["runoob"]="www.runoob.com"
site["taobao"]="www.taobao.com"
```

使用 **@** 或 ***** 可以获取数组中的所有元素

```shell
my_array[0]=A
my_array[1]=B
my_array[2]=C
my_array[3]=D

echo "数组的元素为: ${my_array[*]}"
echo "数组的元素为: ${my_array[@]}"
```

在数组前加一个感叹号 **!** 可以获取数组的所有键

```shell
declare -A site
site["google"]="www.google.com"
site["runoob"]="www.runoob.com"
site["taobao"]="www.taobao.com"

echo "数组的键为: ${!site[*]}"
echo "数组的键为: ${!site[@]}"
```

获取数组长度的方法与获取字符串长度的方法相同

```shell
my_array[0]=A
my_array[1]=B
my_array[2]=C
my_array[3]=D

echo "数组元素个数为: ${#my_array[*]}"
echo "数组元素个数为: ${#my_array[@]}"
```

#### Shell 注释

以 **#** 开头的行就是注释，会被解释器忽略

多行注释

**:** 是一个空命令，用于执行后面的 Here 文档，**<<'EOF'** 表示开启 Here 文档，COMMENT 是 Here 文档的标识符，在这两个标识符之间的内容都会被视为注释，不会被执行。EOF 也可以使用其他符号

```shell
:<<EOF
注释内容...
注释内容...
注释内容...
EOF
```

#### Shell 传递参数

可以在执行 Shell 脚本时，向脚本传递参数，脚本内获取参数的格式为 **$n**，**n** 代表一个数字，**1** 为执行脚本的第一个参数，**2** 为执行脚本的第二个参数。

```shell
echo "执行的文件名：$0"
echo "第一个参数为：$1"
echo "第二个参数为：$2"
echo "第三个参数为：$3"
echo "参数个数：$#"
echo "所有参数：""$*"
echo "脚本运行的当前进程ID：$$"
echo "后台运行的最后一个进程的ID：$!"
echo "所有参数：""$@"
echo "Shell使用的当前选项：$-"
echo "最后命令的退出状态。0表示没有错误：$?"
```

$* 与 $@ 区别：只有在双引号中体现出来。假设在脚本运行时写了三个参数 1、2、3，则 " * " 等价于 "1 2 3"（传递了一个参数），而 "@" 等价于 "1" "2" "3"（传递了三个参数）

```shell
echo "-- \$* 演示 ---"
for i in "$*"; do
    echo $i
done

echo "-- \$@ 演示 ---"
for i in "$@"; do
    echo $i
done
```

#### Shell 基本运算符

原生bash不支持简单的数学运算，但是可以通过其他命令来实现，例如 awk 和 expr，expr 最常用。

expr 是一款表达式计算工具，使用它能完成表达式的求值操作。假定变量 a 为 10，变量 b 为 20

算术运算符

| 运算符 | 说明                                          | 举例                          |
| :----- | :-------------------------------------------- | :---------------------------- |
| +      | 加法                                          | `expr $a + $b` 结果为 30。    |
| -      | 减法                                          | `expr $a - $b` 结果为 -10。   |
| *      | 乘法                                          | `expr $a \* $b` 结果为  200。 |
| /      | 除法                                          | `expr $b / $a` 结果为 2。     |
| %      | 取余                                          | `expr $b % $a` 结果为 0。     |
| =      | 赋值                                          | a=$b 把变量 b 的值赋给 a。    |
| ==     | 相等。用于比较两个数字，相同则返回 true。     | [ $a == $b ] 返回 false。     |
| !=     | 不相等。用于比较两个数字，不相同则返回 true。 | [ $a != $b ] 返回 true。      |

```shell
a=10
b=20

val=`expr $a + $b`	# 空格一定要有
echo "a + b : $val"

val=`expr $a - $b`
echo "a - b : $val"

val=`expr $a \* $b`
echo "a * b : $val"

val=`expr $b / $a`
echo "b / a : $val"

val=`expr $b % $a`
echo "b % a : $val"

if [ $a == $b ]	# 空格一定要有
then
   echo "a 等于 b"
fi
if [ $a != $b ]
then
   echo "a 不等于 b"
fi
```

关系运算符

| 运算符 | 说明                                                  | 举例                       |
| :----- | :---------------------------------------------------- | :------------------------- |
| -eq    | 检测两个数是否相等，相等返回 true。                   | [ $a -eq $b ] 返回 false。 |
| -ne    | 检测两个数是否不相等，不相等返回 true。               | [ $a -ne $b ] 返回 true。  |
| -gt    | 检测左边的数是否大于右边的，如果是，则返回 true。     | [ $a -gt $b ] 返回 false。 |
| -lt    | 检测左边的数是否小于右边的，如果是，则返回 true。     | [ $a -lt $b ] 返回 true。  |
| -ge    | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | [ $a -ge $b ] 返回 false。 |
| -le    | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | [ $a -le $b ] 返回 true。  |

```shell
a=10
b=20

if [ $a -eq $b ]
then
   echo "$a -eq $b : a 等于 b"
fi
if [ $a -ne $b ]
then
   echo "$a -ne $b: a 不等于 b"
fi
if [ $a -gt $b ]
then
   echo "$a -gt $b: a 大于 b"
fi
if [ $a -lt $b ]
then
   echo "$a -lt $b: a 小于 b"
fi
if [ $a -ge $b ]
then
   echo "$a -ge $b: a 大于或等于 b"
fi
if [ $a -le $b ]
then
   echo "$a -le $b: a 小于或等于 b"
fi
```

布尔运算符

| 运算符 | 说明                                                | 举例                                     |
| :----- | :-------------------------------------------------- | :--------------------------------------- |
| !      | 非运算，表达式为 true 则返回 false，否则返回 true。 | [ ! false ] 返回 true。                  |
| -o     | 或运算，有一个表达式为 true 则返回 true。           | [ $a -lt 20 -o $b -gt 100 ] 返回 true。  |
| -a     | 与运算，两个表达式都为 true 才返回 true。           | [ $a -lt 20 -a $b -gt 100 ] 返回 false。 |

```shell
a=10
b=20

if [ ! [$a == $b] ]
then
   echo "$a != $b : a 不等于 b"
else
   echo "$a == $b: a 等于 b"
fi
if [ $a -lt 100 -a $b -gt 15 ]
then
   echo "$a 小于 100 且 $b 大于 15 : 返回 true"
else
   echo "$a 小于 100 且 $b 大于 15 : 返回 false"
fi
if [ $a -lt 100 -o $b -gt 100 ]
then
   echo "$a 小于 100 或 $b 大于 100 : 返回 true"
else
   echo "$a 小于 100 或 $b 大于 100 : 返回 false"
fi
if [ $a -lt 5 -o $b -gt 100 ]
then
   echo "$a 小于 5 或 $b 大于 100 : 返回 true"
else
   echo "$a 小于 5 或 $b 大于 100 : 返回 false"
fi
```

逻辑运算符

| 运算符 | 说明       | 举例                                       |
| :----- | :--------- | :----------------------------------------- |
| &&     | 逻辑的 AND | [[ $a -lt 100 && $b -gt 100 ]] 返回 false  |
| \|\|   | 逻辑的 OR  | [[ $a -lt 100 \|\| $b -gt 100 ]] 返回 true |

```shell
a=10
b=20

if [[ $a -lt 100 && $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi

if [[ $a -lt 100 || $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi
```

字符串运算符

| 运算符 | 说明                                         | 举例                     |
| :----- | :------------------------------------------- | :----------------------- |
| =      | 检测两个字符串是否相等，相等返回 true。      | [ $a = $b ] 返回 false。 |
| !=     | 检测两个字符串是否不相等，不相等返回 true。  | [ $a != $b ] 返回 true。 |
| -z     | 检测字符串长度是否为0，为0返回 true。        | [ -z $a ] 返回 false。   |
| -n     | 检测字符串长度是否不为 0，不为 0 返回 true。 | [ -n "$a" ] 返回 true。  |
| $      | 检测字符串是否不为空，不为空返回 true。      | [ $a ] 返回 true。       |

```shell
a="abc"
b="efg"

if [ $a = $b ]
then
   echo "$a = $b : a 等于 b"
else
   echo "$a = $b: a 不等于 b"
fi
if [ $a != $b ]
then
   echo "$a != $b : a 不等于 b"
else
   echo "$a != $b: a 等于 b"
fi
if [ -z $a ]
then
   echo "-z $a : 字符串长度为 0"
else
   echo "-z $a : 字符串长度不为 0"
fi
if [ -n "$a" ]
then
   echo "-n $a : 字符串长度不为 0"
else
   echo "-n $a : 字符串长度为 0"
fi
if [ $a ]
then
   echo "$a : 字符串不为空"
else
   echo "$a : 字符串为空"
fi
```

文件测试运算符

| 操作符  | 说明                                                         | 举例                      |
| :------ | :----------------------------------------------------------- | :------------------------ |
| -b file | 检测文件是否是块设备文件，如果是，则返回 true。              | [ -b $file ] 返回 false。 |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。            | [ -c $file ] 返回 false。 |
| -d file | 检测文件是否是目录，如果是，则返回 true。                    | [ -d $file ] 返回 false。 |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | [ -f $file ] 返回 true。  |
| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。            | [ -g $file ] 返回 false。 |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  | [ -k $file ] 返回 false。 |
| -p file | 检测文件是否是有名管道，如果是，则返回 true。                | [ -p $file ] 返回 false。 |
| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。            | [ -u $file ] 返回 false。 |
| -r file | 检测文件是否可读，如果是，则返回 true。                      | [ -r $file ] 返回 true。  |
| -w file | 检测文件是否可写，如果是，则返回 true。                      | [ -w $file ] 返回 true。  |
| -x file | 检测文件是否可执行，如果是，则返回 true。                    | [ -x $file ] 返回 true。  |
| -s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true。     | [ -s $file ] 返回 true。  |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。          | [ -e $file ] 返回 true。  |

```shell
file="./test.sh"
if [ -r $file ]
then
   echo "文件可读"
else
   echo "文件不可读"
fi
if [ -w $file ]
then
   echo "文件可写"
else
   echo "文件不可写"
fi
if [ -x $file ]
then
   echo "文件可执行"
else
   echo "文件不可执行"
fi
if [ -f $file ]
then
   echo "文件为普通文件"
else
   echo "文件为特殊文件"
fi
if [ -d $file ]
then
   echo "文件是个目录"
else
   echo "文件不是个目录"
fi
if [ -s $file ]
then
   echo "文件不为空"
else
   echo "文件为空"
fi
if [ -e $file ]
then
   echo "文件存在"
else
   echo "文件不存在"
fi
```

自增和自减操作符

```shell
# 初始化变量
num=5

echo "初始值: $num"

# 自增
let num++
echo "自增后: $num"

# 自减
let num--
echo "自减后: $num"

# 使用 $(( ))
num=$((num + 1))
echo "使用 $(( )) 自增后: $num"

num=$((num - 1))
echo "使用 $(( )) 自减后: $num"

# 使用 expr
num=$(expr $num + 1)
echo "使用 expr 自增后: $num"

num=$(expr $num - 1)
echo "使用 expr 自减后: $num"

# 使用 (( ))
((num++))
echo "使用 (( )) 自增后: $num"

((num--))
echo "使用 (( )) 自减后: $num"
```

#### Shell echo命令

```shell
#!/bin/bash

# 显示普通字符串
echo "It is a test"
echo It is a test			# 可以省略双引号

# 显示转义字符
echo "\"It is a test\""		# 双引号可以省略

# 显示变量
read name 	# read 命令从标准输入中读取一行,并把输入行的每个字段的值指定给 shell 变量
echo "$name It is a test"

# 显示换行
echo -e "OK! \n" 			# -e 开启转义
echo "It is a test"

# 显示不换行
echo -e "OK! \c" 			# -e 开启转义 \c 不换行
echo "It is a test"

# 显示结果定向至文件
echo "It is a test" > myfile

# 原样输出字符串，不进行转义或取变量(用单引号)
echo '$name\"'

# 显示命令执行结果
echo `date`					# 使用的是反引号 `，反引号内的命令会被执行，并将其输出替换到原位置
```

#### Shell printf 命令

| 序列  | 说明                                                         |
| :---- | :----------------------------------------------------------- |
| \a    | 警告字符，通常为ASCII的BEL字符                               |
| \b    | 后退                                                         |
| \c    | 抑制（不显示）输出结果中任何结尾的换行字符（只在%b格式指示符控制下的参数字符串中有效），而且，任何留在参数里的字符、任何接下来的参数以及任何留在格式字符串中的字符，都被忽略 |
| \f    | 换页（formfeed）                                             |
| \n    | 换行                                                         |
| \r    | 回车（Carriage return）                                      |
| \t    | 水平制表符                                                   |
| \v    | 垂直制表符                                                   |
| \\\   | 一个字面上的反斜杠字符                                       |
| \ddd  | 表示1到3位数八进制值的字符。仅在格式字符串中有效             |
| \0ddd | 表示1到3位的八进制值字符                                     |

```shell
# %s %c %d %f 都是格式替代符
# ％s 输出一个字符串
# ％d 整型输出
# ％c 输出一个字符
# ％f 输出实数，以小数形式输出
# %-10s 指一个宽度为 10 个字符（- 表示左对齐，没有则表示右对齐），任何字符都会被显示在 10 个字符宽的字符内，如果不足则自动以空格填充，超过也会将内容全部显示出来
# %-4.2f 指格式化为小数，其中 .2 指保留 2 位小数
printf "%-10s %-8s %-4s\n" 姓名 性别 体重kg  
printf "%-10s %-8s %-4.2f\n" 郭靖 男 66.1234
printf "%-10s %-8s %-4.2f\n" 杨过 男 48.6543
printf "%-10s %-8s %-4.2f\n" 郭芙 女 47.9876
```

```shell
# format-string为双引号
printf "%d %s\n" 1 "abc"

# 单引号与双引号效果一样
printf '%d %s\n' 1 "abc"

# 没有引号也可以输出
printf %s abcdef

# 格式只指定了一个参数，但多出的参数仍然会按照该格式输出，format-string 被重用
printf %s abc def

printf "%s\n" abc def

printf "%s %s %s\n" a b c d e f g h i j

# 如果没有 arguments，那么 %s 用NULL代替，%d 用 0 代替
printf "%s and %d \n"
```

#### Shell test 命令

Shell中的 test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试

```shell
# 数值测试
num1=100
num2=100
if test $[num1] -eq $[num2]
then
    echo '两个数相等！'
else
    echo '两个数不相等！'
fi

# 字符串测试
num1="ru1noob"
num2="runoob"
if test $num1 = $num2
then
    echo '两个字符串相等!'
else
    echo '两个字符串不相等!'
fi

# 文件测试
cd /bin
if test -e ./notFile -o -e ./bash
then
    echo '至少有一个文件存在!'
else
    echo '两个文件都不存在'
fi
```

#### Shell 流程控制

```shell
if condition1
then
    command1
elif condition2 
then 
    command2
else
    commandN
fi
```

```shell
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
```

```shell
while condition
do
    command
done
```

```shell
until condition
do
    command
done
```

```shell
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;			# 相当于break
模式2)
    command1
    command2
    ...
    commandN
    ;;
esac
```

```shell
break		# 跳出当前循环
continue	# 重新开始本轮循环
```

#### Shell 函数

```shell
[ function ] funname [()]
{
    action;
    [return int;]
}
```

可以带 function fun()定义，也可以直接 fun() 定义,不带任何参数

参数返回，可以显示加：return返回，如果不加，将以最后一条命令运行结果，作为返回值。 return后跟数值 n(0-255)

函数返回值在调用该函数后通过 **$?** 来获得

所有函数在使用前必须定义。这意味着必须将函数放在脚本开始部分，直至shell解释器首次发现它时，才可以使用。调用函数仅使用其函数名即可

```shell
#!/bin/bash

funWithReturn(){
    echo "输入第一个数字: "
    read aNum
    echo "输入第二个数字: "
    read anotherNum
    echo "两个数字分别为 $aNum 和 $anotherNum !"
    return $(($aNum+$anotherNum))
}
funWithReturn
echo "输入的两个数字之和为 $? !"
```

在Shell中，调用函数时可以向其传递参数。在函数体内部，通过 $n 的形式来获取参数的值，例如，$1表示第一个参数，$2表示第二个参数...

```shell
#!/bin/bash

funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```

#### Shell 输入/输出重定向

| 命令            | 说明                                               |
| :-------------- | :------------------------------------------------- |
| command > file  | 将输出重定向到 file。                              |
| command < file  | 将输入重定向到 file。                              |
| command >> file | 将输出以追加的方式重定向到 file。                  |
| n > file        | 将文件描述符为 n 的文件重定向到 file。             |
| n >> file       | 将文件描述符为 n 的文件以追加的方式重定向到 file。 |
| n >& m          | 将输出文件 m 和 n 合并。                           |
| n <& m          | 将输入文件 m 和 n 合并。                           |
| << tag          | 将开始标记 tag 和结束标记 tag 之间的内容作为输入。 |

Here Document 是 Shell 中的一种特殊的重定向方式，用来将输入重定向到一个交互式 Shell 脚本或程序

```shell
# 格式
command << delimiter
    document
delimiter
```

```shell
#!/bin/bash

cat << EOF
hello
world
EOF
```

如果希望执行某个命令，但又不希望在屏幕上显示输出结果，那么可以将输出重定向到 /dev/null

```shell
$ command > /dev/null
```

/dev/null 是一个特殊的文件，写入到它的内容都会被丢弃；如果尝试从该文件读取内容，那么什么也读不到。但是 /dev/null 文件非常有用，将命令的输出重定向到它，会起到"禁止输出"的效果。

```shell
# 屏蔽 stdout 和 stderr
# 0 是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）
# 这里的 2 和 > 之间不可以有空格，2> 是一体的时候才表示错误输出
$ command > /dev/null 2>&1
```

#### Shell 文件包含

和其他语言一样，Shell 也可以包含外部脚本。这样可以很方便的封装一些公用的代码作为一个独立的文件

```shell
. filename   # 注意点号(.)和文件名中间有一空格
# 或
source filename
```

```shell
# test1.sh
var=123
```

```shell
#!/bin/bash

# . ./test1.sh
source ./test1.sh

echo "var=${var}"
```

### Shell实操

```sh
#!/bin/bash

# 一个简单的脚本，接受两个参数并打印它们
print_args() {
	local first_arg=$1 # 直接使用 $1 和 $2 来获取参数值
	local second_arg="$2"

	echo "Function: $0"	# $0 函数名
	echo "First argument: $first_arg"
	echo "Second argument: $second_arg"
}

# 调用函数，传入两个参数
print_args "Hello World" "Goodbye World"

append() {
	local var="$1"
	local value="$2"
	# ${parameter:-word}，参数扩展语法，指定parameter未提供时的默认值
	# $3 未提供，默认未空格
	local sep="${3:- }"
	
	# eval [arguments]，将一个字符串 arguments 解释为命令并执行
	# 可用于 变量赋值给变量  eval "var='$value'"
	# export 命令用于将一个变量导出为环境变量，使其在当前 shell 会话和子进程中可用
	# -- 是一个选项标识符，告诉 export 后面没有选项。这样做可以防止后续参数被误认为是选项
	# :+... 是条件扩展，当前面的变量已经被定义且非空时，则执行...的内容，否则返回空
	eval "export -- \"$var=\${$var:+\${$var}\${value:+\$sep}}\$value\""
}

# 调用 append 函数
my_variable="Hello"           # 定义一个初始变量
echo "Before append: $my_variable"  # 输出: Hello

append my_variable "World" ", "  # 追加 World，使用分隔符 ', '

echo "After append: $my_variable"   # 输出: Hello, World
```

