---
title: Shell脚本学习Day1
date: 2023-11-21 12:49:50
categories:
- 学习
tags:
- shell
- 脚本
- 运维
- linux
---

## Shell脚本是什么、它是必需的吗?
答:一个Shell脚本是一个文本文件，包含一个或多个命令。作为系统管理员，我们经常需要使用多个命令
来完成一项任务，我们可以添加这些所有命令在一个文本文件(Shell脚本)来完成这些日常工作任务。
## 什么是默认登录shell，如何改变指定用户的登录shell
答:在Linux操作系统，“/bin/bash”是默认登录shell，是在创建用户时分配的。使用chsh命令可以改变默
认的shell。示例如下所示:
```bash
# chsh <用户名> -s <新shell>
# chsh linuxtechi -s /bin/sh
```
## 可以在shell脚本中使用哪些类型的变量?
答：在shell脚本，我们可以使用两种类型的变量：
- 系统定义变量
- 用户定义变量

系统变量是由系统系统自己创建的。这些变量通常由大写字母组成，可以通过“set”命令查看。  
用户变量由系统用户来生成和定义，变量的值可以通过命令“echo $<变量名>”查看。
## 如何使脚本可执行 ?
答：使用chmod命令来使脚本可执行。例子如下：
```bash
chmod a+x myscript.shs
```
##  “#!/bin/bash”的作用 ?
答：#!/bin/bash是shell脚本的第一行，称为释伴（shebang）行。这里#符号叫做hash，而! 叫做
bang。它的意思是命令通过 /bin/bash 来执行。
## 如何调试shell脚本 ?
答：使用'-x'参数（sh -x myscript.sh）可以调试shell脚本。另一个种方法是使用‘-nv’参数( sh -nv
myscript.sh)。
## 如何将标准输出和错误输出同时重定向到同一位置?
答：这里有两个方法来实现：  
- 方法一：
    ```bash
    2>&1 (如# ls /usr/share/doc > out.txt 2>&1 )
    ```
- 方法二：
  ```bash
  &> (如# ls /usr/share/doc &> out.txt )
  ```
## Shell脚本中“if”语法如何嵌套?
答：基础语法如下：
```bash
if [ 条件 ]
then
命令1
命令2
…..
else
if [ 条件 ]
then
命令1
命令2
….
else
命令1
命令2
…..
fi
fi
```
## Shell脚本中“$?”标记的用途是什么？
答：在写一个shell脚本时，如果你想要检查前一命令是否执行成功，在if条件中使用“$?”可以来检查前一
命令的结束状态。简单的例子如下：
```bash
root@localhost:~# ls /usr/bin/shar
/usr/bin/shar
root@localhost:~# echo $?
0
```
如果结束状态是0，说明前一个命令执行成功。
```bash
root@localhost:~# ls /usr/bin/share
ls: cannot access /usr/bin/share: No such file or directory
root@localhost:~# echo $?
2
```
如果结束状态不是0，说明命令执行失败。
## 在shell脚本中，如何写入注释 ?
答：注释可以用来描述一个脚本可以做什么和它是如何工作的。每一行注释以#开头。例子如下：
```bash
#!/bin/bash
# This is a command
echo “I am logged in as $USER
```
## 在shell脚本中如何比较两个数字 ?
答：在if-then中使用测试命令（ -gt 等）来比较两个数字，例子如下：
```bash
#!/bin/bash
x=10
y=20
if [ $x -gt $y ]
then
echo “x is greater than y”
else
echo “y is greater than x”
fi
```
## Shell脚本中break命令的作用 ?
答：break命令一个简单的用途是退出执行中的循环。我们可以在while和until循环中使用break命令跳
出循环。
## Shell脚本中continue命令的作用 ?
答：continue命令不同于break命令，它只跳出当前循环的迭代，而不是整个循环。continue命令很多
时候是很有用的，例如错误发生，但我们依然希望继续执行大循环的时候。
## Shell脚本中Case语句的语法 ?
答：基础语法如下：
```bash
case 变量 in
值1)
命令1
命令2
...
最后命令
;;
值2)
命令1
命令2
...
最后命令
;;
esac
```
## Shell脚本中while循环语法 ?
答：如同for循环，while循环只要条件成立就重复它的命令块。不同于for循环，while循环会不断迭
代，直到它的条件不为真。基础语法：
```bash
while [ 条件 ]
do
命令...
done
```
## Shell脚本中for循环语法 ?
答：for循环的基础语法：
```bash
for 变量 in 循环列表
do
命令1
命令2
...
最后命令
done
```
## Shell脚本如何比较字符串?
答：test命令可以用来比较字符串。测试命令会通过比较字符串中的每一个字符来比较。
## Bourne shell(bash) 中有哪些特殊的变量 ?
答：下面的表列出了Bourne shell为命令行设置的特殊变量。
```
内建变量 解释
$0      命令行中的脚本名字
$1      第一个命令行参数
$2      第二个命令行参数
...     ...
$9      第九个命令行参数
$#      命令行参数的数量
$*      所有命令行参数，以空格隔开
```
## 在shell脚本中，如何测试文件 ?
答：test命令可以用来测试文件。基础用法如下表格：
```
Test       用法
-d 文件名   如果文件存在并且是目录，返回true
-e 文件名   如果文件存在，返回true
-f 文件名   如果文件存在并且是普通文件，返回true
-r 文件名   如果文件存在并可读，返回true
-s 文件名   如果文件存在并且不为空，返回true
-w 文件名   如果文件存在并可写，返回true
-x 文件名   如果文件存在并可执行，返回true
```
