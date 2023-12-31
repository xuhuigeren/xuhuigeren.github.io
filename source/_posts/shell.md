---
title: shell
date: 2021-03-15 15:45:40
tags: [Algorithm, Shell]
categories: Algorithm
description: 整理牛客和力扣上的shell编程题
---

## nc_01统计文件的行数

写一个 bash脚本以输出一个文本文件 nowcoder.txt中的行数
示例:
假设 nowcoder.txt 内容如下：

```
#include <iostream>
using namespace std;
int main()
{
    int a = 10;
    int b = 100;
    cout << "a + b:" << a + b << endl;
    return 0;
}
```

你的脚本应当输出：
9

```shell
cat nowcoder.txt | wc -l

wc -l nowcoder.txt | awk '{print $1}'

awk '{print NR}' nowcoder.txt | tail -n1

awk 'END{print NR}' nowcoder.txt

#sed 统计行
sed -n '$=' nowcoder.txt

#使用 grep 搜索 ""，然后利用 grep 自带的功能统计行
grep -c "" nowcoder.txt

grep -n "" nowcoder.txt  | awk -F ":" '{print $1 }' | tail -n 1
```




## nc_02打印文件的最后5行

经常查看日志的时候，会从文件的末尾往前查看，于是请你写一个 bash脚本以输出一个文本文件 nowcoder.txt中的最后5行
示例:
假设 nowcoder.txt 内容如下：

```
#include<iostream>
using namespace std;
int main()
{
int a = 10;
int b = 100;
cout << "a + b:" << a + b << endl;
return 0;
}
```

你的脚本应当输出：
```
int a = 10;
int b = 100;
cout << "a + b:" << a + b << endl;
return 0;
}
```



```shell
tail -n5 nowcoder.txt

awk 'NR>=4{print $0}' nowcoder.txt
```

```
NR ||　NF
NR代表的是这个文本文件的行数（记录数）
NF代表的是一个文本文件中一行（一条记录）中的字段个数/列数
September 2003               # NR=1;NF=2
Su Mo Tu We Th Fr Sa         # NR=2;NF=7
    1  2  3  4  5  6         # NR=3;NF=6
07 08 09 10 11 12 13         # NR=4;NF=7
14 15 16 17 18 19 20         # NR=5;NF=7
21 22 23 24 25 26 27         # NR=6;NF=7
28 29 30                     # NR=7;NF=3
```



## nc_03输出7的倍数

写一个 bash脚本以输出数字 0 到 500 中 7 的倍数(0 7 14 21...)的命令



```shell
#!/bin/bash
for num in {0..500..7}; do
  echo "${num}"
done
```


```
# seq 用于生成从一个数到另一个数之间的所有整数。
# 用法：seq [选项]... 尾数
# 或：seq [选项]... 首数 尾数
# 或：seq [选项]... 首数 增量 尾数
seq 0 7 500
```



## nc_04输出第五行内容

写一个 bash脚本以输出一个文本文件 nowcoder.txt 中第5行的内容

示例:
假设 nowcoder.txt 内容如下：
```
welcome
to
nowcoder
this
is
shell
code
```
你的脚本应当输出：
is




```shell
# p 打印 通常 p 会与参数 sed -n 一起运行～
sed -n '5p' nowcoder.txt

head n 5 nowcoder.txt | tail -n 1

awk 'NR==5{print $0}' nowcoder.txt

awk '{if(NR==5){print $0}}' nowcoder.txt
```




## nc_05打印空行的行号

写一个 bash脚本以输出一个文本文件 nowcoder.txt中空行的行号,可能连续,从1开始

示例:
假设 nowcoder.txt 内容如下：

```
a
b

c

d

e


f
```

你的脚本应当输出：

```
3
5
7
9
10
```







```shell
^：开始

$：结尾

^$：表示空行

#!/bin/bash
awk '/^$/ {print NR}' nowcoder.txt

awk '{if($0 == "") {print NR}}' nowcoder.txt

awk '{if (NF==0) print NR}' nowcoder.txt

sed -n '/^$/=' nowcoder.txt

grep -n '^$' nowcoder.txt | awk -F: '{print $1}'
```





##  nc_06去掉空行

写一个 bash脚本以去掉一个文本文件 nowcoder.txt中的空行
示例:
假设 nowcoder.txt 内容如下：

```
abc

567


aaa
bbb



ccc
```

你的脚本应当输出：

```
abc
567
aaa
bbb
ccc
```



```shell
# -v 显示不包含匹配文本的所有行
grep -v '^$' nowcoder.txt
# -e 指定字符串做为查找文件内容的样式
grep -e '\S'

awk '{if($0!="") {print $0 }}' nowcoder.txt
awk '!/^$/ {print $NF}'
# cat 输出文本内容，然后通过管道符交由 awk 做非空校验然后输出
cat nowcoder.txt | awk NF

sed '/^$/d' nowcoder.txt
```



## nc_07打印字母数小于8的单词

写一个 bash脚本以统计一个文本文件 nowcoder.txt中字母数小于8的单词。

示例:假设 nowcoder.txt 内容如下：

```
how they are implemented and applied in computer 
```



你的脚本应当输出：

```
how
they
are
and
applied
in
```
说明:不要担心你输出的空格以及换行的问题



```shell
cat nowcoder.txt | awk '{
for (i=1;i<=NF;i++){
        if (length($i) < 8)
                print $i
}
}'
```



## nc_08统计所有进程占用内存大小的和

假设 nowcoder.txt 内容如下：

```
root         2  0.0  0.0      0     0 ?        S    9月25   0:00 [kthreadd]
root         4  0.0  0.0      0     0 ?        I<   9月25   0:00 [kworker/0:0H]
web       1638  1.8  1.8 6311352 612400 ?      Sl   10月16  21:52 test
web       1639  2.0  1.8 6311352 612401 ?      Sl   10月16  21:52 test
tangmiao-pc       5336   0.0  1.4  9100240 238544   ??  S     3:09下午   0:31.70 /Applications
```

上内容是通过`ps aux | grep -v 'RSS TTY' `命令输出到`nowcoder.txt`文件下面的
请你写一个脚本计算一下所有进程占用内存大小的和:



```shell
awk '{a+=$6}END{print a}'
对第一列数字求和： awk '{a+=$1}END{print a}'
对第二列数字求和： awk '{a+=$2}END{print a}'
```



## nc_09统计每个单词出现的次数

写一个 bash脚本以统计一个文本文件 nowcoder.txt 中每个单词出现的个数。

为了简单起见，你可以假设：
nowcoder.txt只包括小写字母和空格。
每个单词只由小写字母组成。
单词间由一个或多个空格字符分隔。

示例:
假设 nowcoder.txt 内容如下：

```
welcome nowcoder
welcome to nowcoder
nowcoder
```

你的脚本应当输出（以词频升序排列）：

```
to 1 
welcome 2 
nowcoder 3 
```

说明:
不要担心个数相同的单词的排序问题，每个单词出现的个数都是唯一的。



```shell
cat nowcoder.txt | xargs -n1 | sort | uniq -c | sort -n | awk '{print $2,$1}'
```

对于nowcoder.txt文件进行词频统计，首先要做的事情就是把nowcoder.txt文件当中的每一个单词分割出来，分割出每一个单词可以使用以下两种方式：

**使用awk命令：**


```shell
awk '{for(i=1;i<=NF;i++){print $i}}' nowcoder.txt 
其中NF表示当前记录的字段数（即列数）
$i 文件中每行以间隔符号分割的不同字段
如果对awk命令不熟悉，可以参考之前分享的一篇文章学习：
号称三剑客之首的awk，开始秀！
```
**使用xargs命令：**

```shell
cat nowcoder.txt | xargs -n1
xargs命令是用于给其他命令传递参数的一个过滤器，也是组合多个命令的一个工具。
-n1，指定 输出时每行输出的 1列
welcome
nowcoder
welcome
to
nowcoder
nowcoder

-n2，指定 输出时每行输出的 2列
welcome nowcoder
welcome to
nowcoder nowcoder
```

可以在xargs的基础之上使用一些shell小工具来得到每个单词出现的次数。sort 工具及 uniq 工具，这里仅介绍解决问题使用的参数，关于小工具（grep、cut、sort、uniq、tee、diff、past、tr）可以参考文章：[Shell编程之文本处理工具与bash的特性](https://mp.weixin.qq.com/s/7pfE3S-uDSLOG1AZSj3D1A)

```shell
sort工具用于排序，它将文件的每一行作为一个单位，从首字母向后按照ASCII码值进行比较，默认将他们升序输出。
nowcoder
nowcoder
nowcoder
to
welcome
welcome


uniq用去取出连续的重复行

-c ：统计重复行的次数
     3 nowcoder
     1 to
     2 welcome

-r : 降序排列

-n : 以数字排序，默认是按照字符排序的。
      1 to
      2 welcome
      3 nowcoder

最后我们仅需要对上面的结果进行排序啦，很简单的使用sort就可以啦！
```



## nc_10第二列是否有重复

给定一个 `nowcoder.txt`文件，其中有3列信息，如下实例，编写一个`shell`脚本来检查文件第二列是否有重复，且有几个重复，并提取出重复的行的第二列信息：
实例：

```
20201001 python 99
20201002 go 80
20201002 c++ 88
20201003 php 77
20201001 go 88
20201005 shell 89
20201006 java 70
20201008 c 100
20201007 java 88
20201006 go 97
```

结果：

```
2 java
3 go
```





```shell
awk '{print $2}' nowcoder.txt | sort | uniq -cd | sort -n
```



```shell
awk '{print $2}' nowcoder.txt

python
go
c++
php
go
shell
java
c
java
go
```

```shell
awk '{print $2}' nowcoder.txt | sort

c
c++
go
go
go
java
java
php
python
shell
```

```shell
awk '{print $2}' nowcoder.txt | sort | uniq -cd

3 go
2 java
```

```shell
uniq [-c/d/D/u/i] [-f Fields] [-s N] [-w N] [InFile] [OutFile]
```

> - -c: 在每列旁边显示该行重复出现的次数。
> - -d: 仅显示重复出现的行列，显示一行。
> - -D: 显示所有重复出现的行列，有几行显示几行。
> - -u: 仅显示出一次的行列。
> - -i: 忽略大小写字符的不同。
> - -f Fields: 忽略比较指定的列数。
> - -s N: 忽略比较前面的N个字符。
> - -w N: 对每行第N个字符以后的内容不作比较。
>
> [InFile]: 指定已排序好的文本文件。如果不指定此项，则从标准读取数据；
> [OutFile]: 指定输出的文件。如果不指定此选项，则将内容显示到标准输出设备（显示终端）

```shell
sort [-b/d/f/g/i/M/n/r] [InFile]
```

> - -b: ignore-leading-blanks，忽略前面空格符部分
> - -d: data-order，仅考虑空格和字母数字字符
> - -f: ignore-case，忽略大小写
> - -g: general-numeric-sort，根据一般数值进行排序
> - -i: ignore-nonprinting，忽略不可打印的字符，比如换行符、回车符
> - -M: month-sort，以月份进行排序
> - -n: numeric-sort，根据字符串数值进行排序-r: reverse，反向输出排序结果



## nc_11转置文件中的内容

写一个 bash脚本来转置文本文件`nowcoder.txt`中的文件内容。

为了简单起见，你可以假设：
你可以假设每行列数相同，并且每个字段由空格分隔

示例:
假设 `nowcoder.txt `内容如下：

```
job salary
c++ 13
java 14
php 12
```

你的脚本应当输出（以词频升序排列）

```
job c++ java php
salary 13 14 12
```





```shell
awk '{ 
    # NF表示列数,NR表示当前行数
    for (i=1; i<=NF; i++){
        if(NR==1){ 
            # 处理第一行时,将第i列的值($i)存入arr[i],i为数组的下标,数组不用定义可以直接使用
            arr[i]=$i;   
        }
        else{
            # 不是第一行时，将该行对应i列的值拼接到arr[i]
            arr[i]=arr[i] " " $i
        }
    }
}
END{
    # 每行处理完以后,输出数组
    for (i=1; i<=NF; i++){
        print arr[i]
    }
}' nowcoder.txt
```



## shell基本语法

**AWK 是一种处理文本文件的语言，是一个强大的文本分析工具**

**语法**

```bash
awk [选项参数] 'script' var=value file(s)
或
awk [选项参数] -f scriptfile var=value file(s)
```

**选项参数**

- -F fs or --field-separator fs
  指定输入文件折分隔符，fs是一个字符串或者是一个正则表达式，如-F:
- -v var=value or --asign var=value
  赋值一个用户定义变量。
- -f scripfile or --file scriptfile
  从脚本文件中读取awk命令。
- -mf nnn and -mr nnn
  对nnn值设置内在限制，-mf选项限制分配给nnn的最大块数目；-mr选项限制记录的最大数目。这两个功能是Bell实验室版awk的扩展功能，在标准awk中不适用。
- -W compact or --compat, -W traditional or --traditional
  在兼容模式下运行awk。所以gawk的行为和标准的awk完全一样，所有的awk扩展都被忽略。
- -W copyleft or --copyleft, -W copyright or --copyright
  打印简短的版权信息。
- -W help or --help, -W usage or --usage
  打印全部awk选项和每个选项的简短说明。
- -W lint or --lint
  打印不能向传统unix平台移植的结构的警告。
- -W lint-old or --lint-old
  打印关于不能向传统unix平台移植的结构的警告。
- -W posix
  打开兼容模式。但有以下限制，不识别：/x、函数关键字、func、换码序列以及当fs是一个空格时，将新行作为一个域分隔符；操作符**和**=不能代替^和^=；fflush无效。
- -W re-interval or --re-inerval
  允许间隔正则表达式的使用，参考(grep中的Posix字符类)，如括号表达式[[:alpha:]]。
- -W source program-text or --source program-text
  使用program-text作为源代码，可与-f命令混用。
- -W version or --version
  打印bug报告信息的版本。



**sed**

**sed 命令是利用脚本来处理文本文件。**

**语法**

```shell
sed [-hnV][-e<script>][-f<script文件>][文本文件]
```

**参数说明**

- -e<script>或--expression=<script> 以选项中指定的script来处理输入的文本文件。
- -f<script文件>或--file=<script文件> 以选项中指定的script文件来处理输入的文本文件。
- -h或--help 显示帮助。
- -n或--quiet或--silent 仅显示script处理后的结果。
- -V或--version 显示版本信息。

**动作说明**：

- a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)
- c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
- d ：删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
- i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
- p ：打印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
- s ：取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法！例如 1,20s/old/new/g 就是啦！



** wc**

**wc命令的功能为统计指定文件中的字节数、字数、行数, 并将统计结果显示输出。**

**参数说明**

- -c 统计字节数 chars byres
- -l 统计行数      lines
- -w 统计词数    words