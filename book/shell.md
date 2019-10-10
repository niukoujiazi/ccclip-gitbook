SHELL
====

# 参考
[egrep 命令](https://www.ibm.com/support/knowledgecenter/zh/ssw_aix_71/com.ibm.aix.cmds2/egrep.htm)

[Shell特殊变量：Shell $0, $# , $*, $@, $?, $$和命令行参数](https://blog.csdn.net/u011341352/article/details/53215180)

[Shell 运算符：Shell 算数运算符、关系运算符、布尔运算符、字符串运算符等](http://wiki.jikexueyuan.com/project/shell-tutorial/shell-operator.html)

[Shell脚本编程30分钟入门](https://github.com/qinjx/30min_guides/blob/master/shell.md)

[Shell 基本运算符](http://www.runoob.com/linux/linux-shell-basic-operators.html)
# 运算符参考
# 特殊变量
```
$0        当前脚本的文件名
$n        传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。
$#         传递给脚本或函数的参数个数。
$*        传递给脚本或函数的所有参数。
$@        传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同，下面将会讲到。
$?        上个命令的退出状态，或函数的返回值。一般情况下，大部分命令执行成功会返回 0，失败返回 1。
$$        当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。
 
$* 和 $@ 的区别
$* 和 $@ 都表示传递给函数或脚本的所有参数，不被双引号(" ")包含时，都以"$1" "$2" … "$n" 的形式输出所有参数。
但是当它们被双引号(" ")包含时，"$*" 会将所有的参数作为一个整体，以"$1 $2 … $n"的形式输出所有参数；"$@" 会将各个参数分开，以"$1" "$2" … "$n" 的形式输出所有参数。
```
# 运算符
算术运算符
下表列出了常用的算术运算符，假定变量 a 为 10，变量 b 为 20：

运算符	说明	举例
+	加法	`expr $a + $b` 结果为 30。
-	减法	`expr $a - $b` 结果为 -10。
*	乘法	`expr $a \* $b` 结果为  200。
/	除法	`expr $b / $a` 结果为 2。
%	取余	`expr $b % $a` 结果为 0。
=	赋值	a=$b 将把变量 b 的值赋给 a。
==	相等。用于比较两个数字，相同则返回 true。	[ $a == $b ] 返回 false。
!=	不相等。用于比较两个数字，不相同则返回 true。	[ $a != $b ] 返回 true。
注意：条件表达式要放在方括号之间，并且要有空格，例如: [$a==$b] 是错误的，必须写成 [ $a == $b ]。


关系运算符
关系运算符只支持数字，不支持字符串，除非字符串的值是数字。

下表列出了常用的关系运算符，假定变量 a 为 10，变量 b 为 20：

运算符	说明	举例
-eq	检测两个数是否相等，相等返回 true。	[ $a -eq $b ] 返回 false。
-ne	检测两个数是否不相等，不相等返回 true。	[ $a -ne $b ] 返回 true。
-gt	检测左边的数是否大于右边的，如果是，则返回 true。	[ $a -gt $b ] 返回 false。
-lt	检测左边的数是否小于右边的，如果是，则返回 true。	[ $a -lt $b ] 返回 true。
-ge	检测左边的数是否大于等于右边的，如果是，则返回 true。	[ $a -ge $b ] 返回 false。
-le	检测左边的数是否小于等于右边的，如果是，则返回 true。	[ $a -le $b ] 返回 true。

布尔运算符
下表列出了常用的布尔运算符，假定变量 a 为 10，变量 b 为 20：

运算符	说明	举例
!	非运算，表达式为 true 则返回 false，否则返回 true。	[ ! false ] 返回 true。
-o	或运算，有一个表达式为 true 则返回 true。	[ $a -lt 20 -o $b -gt 100 ] 返回 true。
-a	与运算，两个表达式都为 true 才返回 true。	[ $a -lt 20 -a $b -gt 100 ] 返回 false。

逻辑运算符
以下介绍 Shell 的逻辑运算符，假定变量 a 为 10，变量 b 为 20:

运算符	说明	举例
&&	逻辑的 AND	[[ $a -lt 100 && $b -gt 100 ]] 返回 false
||	逻辑的 OR	[[ $a -lt 100 || $b -gt 100 ]] 返回 true

字符串运算符
下表列出了常用的字符串运算符，假定变量 a 为 "abc"，变量 b 为 "efg"：

运算符	说明	举例
=	检测两个字符串是否相等，相等返回 true。	[ $a = $b ] 返回 false。
!=	检测两个字符串是否相等，不相等返回 true。	[ $a != $b ] 返回 true。
-z	检测字符串长度是否为0，为0返回 true。	[ -z $a ] 返回 false。
-n	检测字符串长度是否为0，不为0返回 true。	[ -n "$a" ] 返回 true。
$	检测字符串是否为空，不为空返回 true。	[ $a ] 返回 true。

# egrep
用途
搜索文件获得模式。

语法
egrep [ -h ] [ -i ] [ -p[ Separator ] ] [ -s ] [ -u ] [ -v ] [ -w ] [ -x ] [ -y ] [ [ -b ] [ -n ] | [ -c | -l | -q ] ] { { -ePattern | -fStringFile } ... |Pattern } [ File ... ]

描述
egrep 命令会在输入文件（缺省值为标准输入）中搜索与用 Pattern 参数指定的模式相匹配的行。这些模式是完整的正则表达式就像在 ed 命令中的那样（除了 \（反斜杠）和 \\（双反斜杠））。以下规则也应用于 egrep 命令:

一个正则表达式后面带一个 ＋ （加号）会匹配一个或多个的正则表达式。
一个正则表达式后面带一个 ？ （问号）会匹配零个或一个该正则表达式。
由 | （竖线）或者换行符隔开的多个正则表达式会匹配与任何一个正则表达式所匹配的字符串。
一个正则表达式可以被包括在“()”（括号）中进行分组。
换行符将不会被正则表达式匹配。

运算符的优先顺序是 [, ], *, ?, +, 合并, | 和换行符。

注：egrep 命令与 grep 命令带 -E 标志是一样的，除了错误消息和使用情况消息不同以及 -s 标志的功能不同之外。
egrep 命令会显示包含该匹配行的文件，如果您指定了多于一个 File 参数的话。对 shell 有特殊含义的字符 （$, *, [, |, ^, (, ), \ ) 出现在 Pattern 参数中时必须带双引号。如果 Pattern 参数不是简单字符串，通常必须用单引号将整个模式括起来。在诸如 [a-z]之类的表达式中，减号意味着按照当前整理顺序“直到”。整理顺序可以定义等价的类以供在字符范围中使用。它使用了快速确定性的算法，有时需要外部空间。


# shell
# expect自动交互
参考：[Shell脚本交互之：自动输入密码](https://blog.csdn.net/zhangjikuan/article/details/51105166)

[Ubuntu使用Spawn和expect实现ssh自动登陆](https://www.jianshu.com/p/d573ac55c24c)

[Expect中的timeout设定](https://www.cnblogs.com/super119/archive/2010/12/18/1909963.html)

[linux expect自动登录ssh](https://www.cnblogs.com/muhe221/articles/4127718.html)

需要安装`except`才能使用

```
$ apt-get install tcl tk expect
```
要运行`expect`脚本必须使用

```
$ expect filename.sh
```
# 语法
这是一个设置自动修改postgresql密码的脚本

```
# !/bin/expect
spawn psql
expect "postgres=# "
send "\\password postgres\r"
expect "Enter new password:"
send "123456\r"
expect "Enter it again:"
send "123456\r"
expect "postgres=# "
send "\\q\r"
expect eof
```

`!/bin/expect` 指明这是个expect脚本

`spawn` 后面接续指令，开始自动交互，

`expect` 期待控制台输出的结果，符合这个结果就执行下面的`send`

`send`当` expect`获得预期结果时将会执行本语句，记得加上`\r`来模拟回车

`interact` 交互模式,用户会停留在远程服务器上面.

`set timeout -1` 没有超时时间
`set timeout 30`设置超时时间为30秒
# 使用echo指令向文件写入内容
[Linux学习笔记——如何使用echo指令向文件写入内容](https://blog.csdn.net/xukai871105/article/details/35834703)

# 覆盖文件内容

```
echo "text" > text.txt
```
使用`>`将会覆盖文件夹原内容写入，如果文件不存在则创建文件
# 追加文件内容

```
echo "text" >> text.txt
```
使用`>>`向文件追加内容，不会改动原内容
