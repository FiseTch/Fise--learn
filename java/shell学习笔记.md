#### shell

##### shell 概述

* shell是一个命令行解释器，他接受应用程序/用户命令，然后调用操作系统内核
* 功能强大的编程语言，易编写，调试，灵活性强

##### shell解析器

* `linux` 提供的解析器主要用的有两种
  1. /bin/sh
  2. /bin/bash
* ==`sudo`代表拥有了root权限，`linux`系统快捷键为`Tab` ,双击`Tab`会显示提示信息==
* ==| grep 对显示的信息进行筛选==
* 查询默认的shell解析器 ：`echo $SHELL`

##### shell脚本入门

1. 脚本入门 
   * 一般以#！/bin/bash开头
2. 脚本书写
3. 脚本执行方式
   * sh  文件名
   * bash 文件名
   * ./ 文件名（需要权限）
   * /文件名（需要权限）

##### shell常用系统变量

###### 系统变量

* $HOME
* $PWD
* $SHELL
* $USER

###### 自定义变量

`A=2` 等号两边不允许有空格

###### 撤销变量

`unset 变量`

###### 声明静态变量

`readonly 变量 `

###### 将变量升级为全局变量 

`export 变量名`

###### $n

允许脚本执行时，输入参数 其中$0代表脚本自身。其他的代表参数

> `$* ` 获取命令行中所有参数，整体获取
>
> `$@`获取命令行中所有参数，把每个参数区分对待
>
> `$?` 确认上一条命令是否正确执行

###### 注意

* 变量名称可以有字母，数字，下划线组成，但是不能以数字开头，环境变量名建议大写
* 等号两侧不允许有空格
* 变量默认都是字符串类型，无法进行数值计算
* 如果变量的值有空格，需要用双引号包含

##### 运算符

1. 基本语法：`$((运算符))` 或者`$[运算符]` 

2. expr 加减乘除，取余 ==注意：expr运算符中需要有空格==

   >例子：` expr `expr 3 + 2` \* 4` 结果为20
   >

   
##### 条件判断

* 基本语法`[ condition ]` ==condition前后必须有空格==

  >1. 整数比较
  >2. 字符串比较
  >3. 文件权限判断
  >4. 文件判断

  * 根据`echo $?` 来判断语句是否执行正确 $返回值为0时语句为真，否则为假$

##### 函数

* 系统函数

  >`dirname` 从给定的文件中返回绝对路径

* 自定义函数

  >shell脚本中使用函数，必须先声明
  >
  >格式： function 函数名(){
  >
  >}

##### shell工具

###### cut工具（并没有改变源文件）

* 用法 `cut -d " " -f 1 cut.txt`

* `cat cut.txt | grep guan | cut -d " " -f 1`

###### sed工具（并没有改变源文件）

* `sed "2a mei nv" sed.txt`  在sed.txt中第三行添加mei nv 

* `sed "/wo/d" sed.txt`  将包含`wo`的行删除掉

* `sed "s/wo/ni/g sed.txt"`将sed.txt文件中`wo` 替换成`ni`,其中g代表全部替换

* `sed -e "2d" sed.txt` 删除第二行

* `sed -e "2d" -e "s/wo/ni/g sed.txt"`     将`sed.txt` 文件中的第二行删除并将`wo`替换成`ni`

* 常用选项

  >-e 直接在指令列模式上进行sed的动作编辑

###### awk工具（强大的文本分析工具）

* 用法：`awk -F : '/^root/ {print $7}' passwd`  搜索passwd文件以root关键字开头的所有行，并输出该行的第7列
* `awk -F : 'BEGIN {print "user shell"} {print $1","$7}'END{print "dahaige,bin/zuishuai"}  passwd` 只显示/etc/passwd的第一列和第七列，以逗号分隔，且在所有行前面添加列名user，shell在最后一行添加 "dahaige,/bin/zuishuai"

###### sort

##### echo(display a line of text)