---
title: shell入门
date: 2022-11-26 00:14:09
author: 三岁浪迹天涯
categories: linux
tags:
  - shell
  - 基础
---

## 基础知识

在 shell 脚本，`#!` 告诉系统其后路径所指定的程序即是解释此脚本文件的 Shell 解释器

```bash
#!/bin/bash
```

执行shell脚本的方式：

```shell
sh /path/to/script.sh
bash /path/to/script.sh
source /path/to/script.sh
./path/to/script.sh
```

使用`#！`标识后，如果文件具有可执行权限，可以直接通过`path/to/test.sh`执行

## 输出

### echo	

```shell
echo -e "YES\c" # -e 开启转义 \c 不换行
echo "NO"
#  Output:
#  YESNO
```

### printf

```shell
printf "%s %s %s\n" a b c d e f g h i j
#  Output:
#  a b c
#  d e f
#  g h i
#  j
```

## 字符串

shell 字符串可以用单引号 `''`，也可以用双引号 `“”`，也可以不用引号。

- 单引号的特点
  - 单引号里不识别变量
  - 单引号里不能出现单独的单引号（使用转义符也不行），但可成对出现，作为字符串拼接使用。
- 双引号的特点
  - 双引号里识别变量
  - 双引号里可以出现转义字符

### 单引号和双引号

当局部变量和环境变量包含空格时，它们在引号中的扩展要格外注意。随便举个例子，假如我们用`echo`来输出用户的输入：

```shell
INPUT="A string  with   strange    whitespace."
echo $INPUT   ### A string with strange whitespace.
echo "$INPUT" ### A string  with   strange    whitespace.
```

调用第一个`echo`时给了它 5 个单独的参数 —— `$INPUT` 被分成了单独的词，`echo`在每个词之间打印了一个空格。第二种情况，调用`echo`时只给了它一个参数（整个$INPUT 的值，包括其中的空格）。

```shell
# 使用单引号拼接
name1='white'
str1='hello, '${name1}''
str2='hello, ${name1}'
echo ${str1}_${str2}
# Output:
# hello, white_hello, ${name1}

# 使用双引号拼接
name2="black"
str3="hello, "${name2}""
str4="hello, ${name2}"
echo ${str3}_${str4}
# Output:
# hello, black_hello, black
```

### 字符串常见操作

```shell
# 获取字符串长度
text="12345"
echo ${#text}
# Output:
# 5

# 截取子字符串
text="12345"
echo ${text:2:2}
# Output:
# 34

# 查找子字符串
text="hello"
echo `expr index "${text}" ll`

# Execute: ./str-demo5.sh
# Output:
# 3
```

## map

map创建方式

```bash
# 方式一
declare -A myMap
myMap["my03"]="03"

# 方式二
declare -A myMap=(["my01"]="01" ["my02"]="02")
```

## 数组	

bash只支持一维数组

### 创建数组

```shell
# 创建数组的不同方式
nums=([2]=2 [0]=0 [1]=1)
colors=(red yellow "dark blue")
```

### 输出数组所有元素

```shell
echo ${colors[*]}
# Output: red yellow dark blue

echo ${colors[@]}
# Output: red yellow dark blue

printf "+ %s\n" ${colors[*]}
# Output:
# + red
# + yellow
# + dark
# + blue

printf "+ %s\n" "${colors[*]}"
# Output:
# + red yellow dark blue

printf "+ %s\n" "${colors[@]}"
# Output:
# + red
# + yellow
# + dark blue
```

### 数组常见操作

```shell
# 访问数组部分元素
echo ${nums[@]:0:2}

# 访问数组长度
echo ${#nums[*]}
# Output:
# 3

# 向数组中添加元素
colors=(white "${colors[@]}" green black)
echo ${colors[@]}
# Output:
# white red yellow dark blue green black

# 删除元素
unset nums[0]
echo ${nums[@]}
# Output:
# 1 2
```

## 运算符

### 关系运算符

关系运算符只支持数字，不支持字符串，除非字符串的值是数字。

| 运算符 | 说明                                                  | 举例                         |
| ------ | ----------------------------------------------------- | ---------------------------- |
| `-eq`  | 检测两个数是否相等，相等返回 true。                   | `[ $a -eq $b ]`返回 false。  |
| `-ne`  | 检测两个数是否相等，不相等返回 true。                 | `[ $a -ne $b ]` 返回 true。  |
| `-gt`  | 检测左边的数是否大于右边的，如果是，则返回 true。     | `[ $a -gt $b ]` 返回 false。 |
| `-lt`  | 检测左边的数是否小于右边的，如果是，则返回 true。     | `[ $a -lt $b ]` 返回 true。  |
| `-ge`  | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | `[ $a -ge $b ]` 返回 false。 |
| `-le`  | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | `[ $a -le $b ]`返回 true。   |

### 布尔运算符

| 运算符 | 说明                                                | 举例                                       |
| ------ | --------------------------------------------------- | ------------------------------------------ |
| `!`    | 非运算，表达式为 true 则返回 false，否则返回 true。 | `[ ! false ]` 返回 true。                  |
| `-o`   | 或运算，有一个表达式为 true 则返回 true。           | `[ $a -lt 20 -o $b -gt 100 ]` 返回 true。  |
| `-a`   | 与运算，两个表达式都为 true 才返回 true。           | `[ $a -lt 20 -a $b -gt 100 ]` 返回 false。 |

### 逻辑运算符

| 运算符 | 说明       | 举例                                            |
| ------ | ---------- | ----------------------------------------------- |
| `&&`   | 逻辑的 AND | `[[ ${x} -lt 100 && ${y} -gt 100 ]]` 返回 false |

### 字符串运算符

| 运算符 | 说明                                       | 举例                       |
| ------ | ------------------------------------------ | -------------------------- |
| `=`    | 检测两个字符串是否相等，相等返回 true。    | `[ $a = $b ]` 返回 false。 |
| `!=`   | 检测两个字符串是否相等，不相等返回 true。  | `[ $a != $b ]` 返回 true。 |
| `-z`   | 检测字符串长度是否为 0，为 0 返回 true。   | `[ -z $a ]` 返回 false。   |
| `-n`   | 检测字符串长度是否为 0，不为 0 返回 true。 | `[ -n $a ]` 返回 true。    |
| `str`  | 检测字符串是否为空，不为空返回 true。      | `[ $a ]` 返回 true。       |

### 文件测试运算符

| 操作符  | 说明                                                         | 举例                        |
| :------ | ------------------------------------------------------------ | --------------------------- |
| -b file | 检测文件是否是块设备文件，如果是，则返回 true。              | `[ -b $file ]` 返回 false。 |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。            | `[ -c $file ]` 返回 false。 |
| -d file | 检测文件是否是目录，如果是，则返回 true。                    | `[ -d $file ]` 返回 false。 |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | `[ -f $file ]` 返回 true。  |
| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。            | `[ -g $file ]` 返回 false。 |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  | `[ -k $file ]`返回 false。  |
| -p file | 检测文件是否是有名管道，如果是，则返回 true。                | `[ -p $file ]` 返回 false。 |
| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。            | `[ -u $file ]` 返回 false。 |
| -r file | 检测文件是否可读，如果是，则返回 true。                      | `[ -r $file ]` 返回 true。  |
| -w file | 检测文件是否可写，如果是，则返回 true。                      | `[ -w $file ]` 返回 true。  |
| -x file | 检测文件是否可执行，如果是，则返回 true。                    | `[ -x $file ]` 返回 true。  |
| -s file | 检测文件是否为空（文件大小是否大于 0），不为空返回 true。    | `[ -s $file ]` 返回 true。  |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。          | `[ -e $file ]` 返回 true。  |

## 控制语句

[]和[[]]的区别：

优先考虑使用[[]]，它是内置在shell中的一个命令，支持字符串的模式匹配等，比如[[ hello == hell? ]]，结果为真。

### 条件语句

if语句

```shell
x=10
y=20
if [[ ${x} > ${y} ]]; then
   echo "${x} > ${y}"
elif [[ ${x} < ${y} ]]; then
   echo "${x} < ${y}"
else
   echo "${x} = ${y}"
fi
# Output: 10 < 20
```

case语句

```shell
exec
case ${oper} in
  "+")
    val=`expr ${x} + ${y}`
    echo "${x} + ${y} = ${val}"
  ;;
  "-")
    val=`expr ${x} - ${y}`
    echo "${x} - ${y} = ${val}"
  ;;
  "*")
    val=`expr ${x} \* ${y}`
    echo "${x} * ${y} = ${val}"
  ;;
  "/")
    val=`expr ${x} / ${y}`
    echo "${x} / ${y} = ${val}"
  ;;
  *)
    echo "Unknown oper!"
  ;;
esac
```



### for循环

for循环的几种使用方法

```shell
for arg in elem1 elem2 ... elemN
do
  ### 语句
done

for (( i = 0; i < 10; i++ )); do
  echo $i
done

DIR=/home/zp
for FILE in ${DIR}/*.sh; do
  mv "$FILE" "${DIR}/scripts"
done
# 将 /home/zp 目录下所有 sh 文件拷贝到 /home/zp/scripts
```

### while循环

```shell
while [[ ${x} -lt 10 ]]; do
  echo $((x * x))
  x=$((x + 1))
done
```

### select循环

```shell
select answer in elem1 elem2 ... elemN
do
  ### 语句
done
```

`select`会打印`elem1..elemN`以及它们的序列号到屏幕上，之后会提示用户输入。

```shell
PS3="Choose the package manager: "
select ITEM in bower npm gem pip
do
echo -n "Enter the package name: " && read PACKAGE
case ${ITEM} in
  bower) bower install ${PACKAGE} ;;
  npm) npm install ${PACKAGE} ;;
  gem) gem install ${PACKAGE} ;;
  pip) pip install ${PACKAGE} ;;
esac
break # 避免无限循环
done
```

运行这个脚本，会得到如下输出：

```shell
$ ./my_script
1) bower
2) npm
3) gem
4) pip
Choose the package manager: 2
Enter the package name: gitbook-cli
```

## 函数

```shell
calc(){
  PS3="choose the oper: "
  select oper in + - \* / # 生成操作符选择菜单
  do
  echo -n "enter first num: " && read x # 读取输入参数
  echo -n "enter second num: " && read y # 读取输入参数
  exec
  case ${oper} in
    "+")
      return $((${x} + ${y}))
    ;;
    "-")
      return $((${x} - ${y}))
    ;;
    "*")
      return $((${x} * ${y}))
    ;;
    "/")
      return $((${x} / ${y}))
    ;;
    *)
      echo "${oper} is not support!"
      return 0
    ;;
  esac
  break
  done
}
calc
echo "the result is: $?" # $? 获取 calc 函数返回值
```

💡 说明：

1. 函数定义时，`function` 关键字可有可无。
2. 函数返回值 - return 返回函数返回值，返回值类型只能为整数（0-255）。如果不加 return 语句，shell 默认将以最后一条命令的运行结果，作为函数返回值。
3. 函数返回值在调用该函数后通过 `$?` 来获得。
4. 所有函数在使用前必须定义。这意味着必须将函数放在脚本开始部分，直至 shell 解释器首次发现它时，才可以使用。调用函数仅使用其函数名即可。

### 位置参数

**位置参数**是在调用一个函数并传给它参数时创建的变量。

位置参数变量表：

| 变量           | 描述                           |
| -------------- | ------------------------------ |
| `$0`           | 脚本名称                       |
| `$1 … $9`      | 第 1 个到第 9 个参数列表       |
| `${10} … ${N}` | 第 10 个到 N 个参数列表        |
| `$*` or `$@`   | 除了`$0`外的所有位置参数       |
| `$#`           | 不包括`$0`在内的位置参数的个数 |
| `$FUNCNAME`    | 函数名称（仅在函数内部有值）   |

```shell
#!/usr/bin/env bash

x=0
if [[ -n $1 ]]; then
  echo "第一个参数为：$1"
  x=$1
else
  echo "第一个参数为空"
fi

y=0
if [[ -n $2 ]]; then
  echo "第二个参数为：$2"
  y=$2
else
  echo "第二个参数为空"
fi

paramsFunction(){
  echo "函数第一个入参：$1"
  echo "函数第二个入参：$2"
}
paramsFunction ${x} ${y}
```

执行结果：

```
$ ./function-demo2.sh
第一个参数为空
第二个参数为空
函数第一个入参：0
函数第二个入参：0

$ ./function-demo2.sh 10 20
第一个参数为：10
第二个参数为：20
函数第一个入参：10
函数第二个入参：20
```

### 函数处理参数

另外，还有几个特殊字符用来处理参数：

| 参数处理 | 说明                                             |
| -------- | ------------------------------------------------ |
| `$#`     | 返回参数个数                                     |
| `$*`     | 返回所有参数                                     |
| `$       | 参数处理                                         |
| $!       | 后台运行的最后一个进程的 ID 号                   |
| `$@`     | 返回所有参数                                     |
| $-       | 返回 Shell 使用的当前选项，与 set 命令功能相同。 |
| $?       | 函数返回值                                       |
