---
title: shell学习
date: 2016-09-22 20:36:16
categories: Shell
tags: [终端命令] 
description: 学习shell
---
# shell学习

echo 输出
chmod +x file 加上执行权限，否则会提示无执行权限

# shell 变量
命名规则
    变量名与等号之间不能有空格
    首个字符必须为字母
    中间不能有空格,可以使用下划线
    不能使用标点符号
    不能属于bash里的关键词(可以help命令查看保留关键字)
    
使用规则
    使用一个定义过的变量,只要在变量名前面加$符号即可a（已定义的变量可以被重新定义）
    
    如: my_name="gy"
        echo $my_name
        or echo ${my_name}
        {}可以理解为Swift中\()
只读变量
    my_name="gY"
    readonly my_name
删除变量
    unset my_name(不能删除制度变量)
变量类型
    1.局部变量
    2.环境变量
      所有程序都能访问,包括shell启动的程序.
    3.shell变量
      shell变量是由程序设置的特殊变量,shell变量中有一部分是环境变量，有一部分是局部变量,这些变        量保证了shell的正常运行

# Shell字符串

单引号
str='this is a string'
单引号字符串的限制:
    * 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的。
    * 单引号子串中不能出现单引号(转义字符也不可以)
双引号
my_name="GY"
双引号的有点:

    * 双引号可以有变量
    * 双引号也可以出现转义字符
获取字符串长度

    echo ${#my_name} （输出2）
    
提取子字符串

    my_name="sunwukong"
    echo ${string:1:4}# 输出unwu
    
查找子字符串:

    echo `expr index "$my_name" un`(Mac中的shell的expr语法是：$((表达式))
    
# Shell数组
定义数组
    在Shell中 用括号来表示数组 空格符号分割开
    arry=(1 2 3 4)
    arry[0]=0
    arry[1]=1

读取数组

    ${arry[i]}
    echo ${arry[@]}(输出数组中的所有元素)

获取数组的长度

    取得数组元素的个数
    length=${#arry[@]} or    
    length=${#arry[*]} 
    取得数组单个元素的长度
    length=${#arry[n]}
    
字符串截取
    假设有变量 var="http://www.baidu.com/1"
    
    1. #号截取,从左边开始删除第一个//字符及左边的所有字符，保留右边字符
       echo ${var#*//} (www.baidu.com/1)
    2. ## 号截取,删除最后一个/及左边的所有字符,保留右边字符
       echo ${var##*/} (1)
    3.%号截取,删除右边字符，保留左边字符
       echo ${var%/*} (从右边删除第一个/及右边的字符)
    4.%%号截取... 从右边开始 删除最后一个/以及右边的字符
    5.从左边第几个字符开始,及字符的个数
       echo ${var:0:5}
    6.从左边第几个字符开始,一直到结束
       echo ${var:7} （包含7）
    7.从右边第几个字符开始,及字符的个数
       echo ${var:0-7:3}
    8.从右边第几个字符开始,一直到结束
       echo ${var:0-7}

    
# Shell传递参数

```
echo "Shell 传递参数实例！";
echo "执行的文件名：$0";
echo "第一个参数为：$1";
echo "第二个参数为：$2";
echo "第三个参数为：$3";
```
为脚本设置可执行权限,并执行脚本

```
$ chmod +x test.sh 
$ ./test.sh 1 2 3
Shell 传递参数实例！
执行的文件名：./test.sh
第一个参数为：1
第二个参数为：2
第三个参数为：3
```



       
        
        










