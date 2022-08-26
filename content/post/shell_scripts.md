---
layout: post
title: Bash Shell脚本常用语法及操作
date: 2022-08-25
author: opso
tags:
- shell
- bash
---

在linux下使用Go开发，或是做docker服务编排过程中，经常会用到 `shell`脚本，比如比较常见的管理程序启停重载配置这样的管理脚本，下面总结列出一些个人比较常用的语法和操作。

## bash和sh的区别

> `sh`（Bourne Shell）是一种由 POSIX 标准描述的编程语言。它有许多实现（ksh88、Dash、...）。 `bash` 也可以被认为是 `sh` 的实现

> `bash`（Bourne-Again Shell）最初是一个与 `sh` 兼容的实现（尽管它比 `POSIX` 标准早了几年），但随着时间的推移，它已经获得了许多扩展。许多这些扩展可能会改变有效 `POSIX shell` 脚本的行为，因此 `Bash` 本身并不是一个有效的 `POSIX shell`。相反，它是 `POSIX shell` 语言的一种方言。

> **POSIX**表示可移植操作系统接口（Portable Operating System Interface of UNIX，缩写为 POSIX ）。POSIX标准意在期望获得源代码级别的软件可移植性。换句话说，为一个POSIX兼容的操作系统编写的程序，应该可以在任何其它的POSIX操作系统上编译执行。

sh 遵循POSIX规范：“当某行代码出错时，不继续往下解释”。bash 就算出错，也会继续向下执行

`bash --posix` 可以让bash遵守posix模式模拟sh

## 脚本第一行

第一行的内容指定了shell脚本解释器的路径，如果路径不存在或者没有生命，运行时候会指定一个默认的脚本解释器。

```bash
#!/bin/bash
```

## 变量

```bash
name="opso"
echo $name
echo ${name}
```

## 条件判断

```bash
if condition
then
    # do something
fi

if condition1; then
    # do something
esif condition2; then
    # do something
else
    # do something
fi
```

## if判断

### 单中括号[ ]

|  表达式   | 说明  |
|  ----  | ----  |
| !  | false |
| -n string | 字符串长度大于0 |
| -z string | 字符串长度等于0 |
| A == B | 字符串A是否等于B |
| A != B | 字符串A不等于B |
| A -eq B | 数字A等于B |
| A -gt B | 数字A大于B |
| A -lt B | 数字A小于B |
| bool -a bool | 逻辑与And |
| bool -o bool | 逻辑或Or |
| -d dir | 目录是否存在 |
| -e/-f file | 文件是否存在 |
| -s file | 文件存在且大小不为0 |
| -r file | 文件是否可读 |
| -w file | 文件是否可写 |
| -x file | 文件是否有执行权限 |

单中括号可以看作是`test`命令的变体，详细参数说明也可以使用 `man test` 查看。

> 注意下面的格式，中括号/表达式/变量之间必须使用空格隔开

```bash
a=10
b=20
if [ $a == $b ]
then
   echo "a 等于 b"
elif [ $a -gt $b ]
then
   echo "a 大于 b"
elif [ $a -lt $b ]
then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi
```

### 双中括号[[ ]]

双中括号判断是支持通配符的，且支持 `&&`,`||`,`<`,`>` 等比较常用的字符做判断。

```bash
a=10
b=20
if [ $a == 10 && $b > $a ]
then
   echo "a 等于 10 且 b 大于 "
fi

c="release-1.2"
if [[ "$c" == release* ]]; then
    echo "字符串c以release开头"
fi
```

## for

### 循环

```bash
for i in {1,2,3,4,6}
do
   echo $i
done
# 1
# 2
# 3
# 4
# 6

names=(
   "foo bar"
   "foobar tom"
)

for i in ${names[@]}; do
   echo "$i"
done
# foo
# bar
# foobar
# tom
```

### range

```bash
# 输出 1 2 3
for value in {1..3}
do
    echo $value
done

# 输出 1 3 5
for value in {1..6..2}
do
    echo $value
done
```

## case

```bash
case <variable> in
<pattern 1>)
    <commands>
    ;;
<pattern 2>)
    <other commands>
    ;;
esac
```

例子：

```bash
# callable
case $1 in
    start)
        start
        ;;
    stop)
        stop
        ;;
    status)
        status
        ;;
    restart)
        restart
        ;;
    reload)
        reload
        ;;
    *)
        echo "USAGE: $0 {start|stop|status|reload|restart}"
        exit 0
esac
```

## \$符号

- \$`数字` 表示函数或者脚本传参位置对应的值
- \$? 表示上一个命令段运行后的错误码，0表示正常，1表示异常

```bash
#!/bin/sh
echo $0
echo $1
echo $2

a=10
b=20
test $a == $b
echo $?

$ sh test.sh status
# test.sh
# test
# status
# 1
```

## 字符串操作

可以使用`{string+特定符号+find}`

```bash
str="date-date-20220101"

# 替换第一个匹配项
echo "${str/date/hello}"
# 删除第一个匹配项
echo "${str#date}"
# 删除最后一个匹配项
echo "${str%01}"

# date-date-20220101
# hello-date-20220101
# -date-20220101
# date-date-202201
```

## 其他

### cd命令隔离

```bash
cd /home/opso
pwd # /home/opso
mkdir log
(
   cd log
   pwd # /home/opso/log
)
rm -r log
pwd # /home/opso
```

### 显示执行的命令

脚本执行过程中显示正在执行的命令

```bash
set -x
```

### 切换到脚本所在目录

```bash
curr_path=$(cd "$(dirname "$0")";pwd)
cd $curr_path
```

### 格式输出文本到文件

```bash
str="hello world"
tee test.php <<EOF
<?php
   print_r("${str}");
EOF

sql=$(cat <<EOF
SELECT foo, bar FROM db
WHERE foo='baz'
EOF
)

$ cat <<EOF > print.sh
#!/bin/bash
echo \$PWD
echo $PWD
EOF
```

### macOS下命令兼容问题

在`macOS`下部分内置的命令和`linux`中不同，比如`echo`，不支持参数比如`-n`/`-e`，可以从输出内容中入手用`\r`/`\c` 实现相同功能。

```bash
if [ "$(uname)" = "FreeBSD" ]; then
    ECHO="echo -e"
elif [ "$(uname)" = "Darwin" ]; then
    ECHO="echo"
else
    ECHO="/bin/echo -e"
fi

echo_success() {
   $ECHO "\\033[60G\\033[32m [  OK  ] \\033[0m\r\c" # 在行尾60字符处输出绿色的 [ OK ]
   echo "service start"
}

echo_failure() {
   $ECHO "\\033[60G\\033[31m [FAILED] \\033[0m\r\c" # 在行尾60字符处输出红色的 [ FAILED ]
   echo "service start"
}

echo_success
echo_failure
# service start                                               [  OK  ] 
# service start                                               [FAILED]
```

`sed`命令