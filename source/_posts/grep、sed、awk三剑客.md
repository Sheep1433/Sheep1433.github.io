---
title: grep、sed、awk三剑客
date: 2022-12-01 22:02:56
author: 三岁浪迹天涯
categories: linux
tags:
  - 三剑客
  - 基础
---

## grep

文本搜索工具，支持使用正则表达式搜索文本，全称为Global Regular Expression Print。

### 参数选项

**[options]主要参数：**

> **-c：只输出匹配行的计数。 -i：不区分大 小写(只适用于单字符)。**
> -h：查询多文件时不显示文件名。
> -l：查询多文件时只输出包含匹配字符的文件名。
> **-n：显示匹配行及 行号。**
> -s：不显示不存在或无匹配文本的错误信息。
> **-v：排除，不显示过滤的字符串的行；显示不包含匹配文本的所有行**
> -e : 指定字符串做为查找文件内容的样式，可以指定多个
> **-E ：将样式为延伸的正则表达式来使用。**
> **-o ：输出精确匹配的字符而不是默认的整行**
> -f ：指定规则文件，其内容含有一个或多个规则样式
> 		让grep查找符合规则条件的文件内容，格式为每行一个规则样式

**Context control：**

> -B 除了显示匹配的一行之外，并显示该行之前的num行
> -A 除了显示匹配的一行之外，并显示该行之后的num行
> -C 除了显示匹配的一行之外，并显示该行之前后各num行

**pattern正则表达式主要参数：**

> \：忽略正则表达式中特殊字符的原有含义。
> .：所有的单个字符。
> *：有字符，长度可以为0。^：匹配正则表达式的开始行。
> $: 匹配正则表达式的结束行。
> \<：从匹配正则表达 式的行开始。
> \>：到匹配正则表达式的行结束。
> [ ]：单个字符，如 [Gg]rep 匹配Grep和grep。
> [ - ]：范围，如[A-Z]，即A、B、C一直到Z都符合要求。
> [^]：匹配一个不在指定范围内的字符
> 		如：'[^A-FH-Z]rep'匹配不包含A-F和H-Z的一个字母开头，紧跟rep的行
> x\{m\}：重复字符x，m次，如：'0\{5\}'匹配包含5个0的行
> x\{m,\}：重复字符x,至少m次，如：'0\{5,\}'匹配至少有5个0的行
> x\{m,n\}：重复字符x，至少m次，不多于n次，如：'0\{5,10\}'匹配5 -- 10个0的行

### 常用示例

```bash
# 查找指定进程个数
ps -ef |grep -c sshd
# 查找test.txt中符合文件test2.txt的规则的内容，test2.txt有多行
cat test.txt | grep -f test2.txt
# 查找test.txt中符合文件test2.txt的规则的内容，并显示行号
cat test.txt | grep -nf test2.txt
# 从多个文件中查找关键词
grep 'linux' test.txt test2.txt
# 查找以w开头的行内容
cat test.txt | grep '^w'
# 查找以w结尾的行内容
cat test.txt | grep 'w$'
# 查找非u开头的行内容
cat test.txt | grep ^[^u]
# 查找有有连续两个uu以上的行内容
cat test.txt |grep -E 'u{2,}' grep -v '^$' test.txt
cat test.txt |grep 'u\{2,\}'
# 查找包含ab或者cd的行内容
cat test.txt |grep -E 'ab|cd'
cat test.txt |grep 'ab\|cd'
# 查找当前目录以txt结尾的文件中所有包含每个字符串至少有7个连续小写字符的字符串的行
grep -E '[a-z]{7,}' *.txt
# 查找包含ab的行内容并显示接下来的10行
grep 'ab' -A 10 test.txt
# 过滤空行内容并输出
 grep -v '^$' test.txt
```

## sed

### 参数选项

**参数说明**：

> -e 以选项中指定的script来处理输入的文本文件。
>
> -f以选项中指定的script文件来处理输入的文本文件。
>
> -h或--help 显示帮助。
>
> -n或--quiet或--silent 仅显示script处理后的结果。
>
> -V或--version 显示版本信息。

**动作说明**：

> a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
>
> c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
>
> d ：删除，因为是删除啊，所以 d 后面通常不接任何东东；
>
> i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
>
> **p ：打印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～**
>
> s ：取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正则表达式！例如 1,20s/old/new/g 就是啦！

### 常用示例

```bash
# 在test文件的第四行后添加一行，并将结果输出到标准输出
sed '4a newline' test.txt
# 将test文件的内容列出并列出行号，同事将2-5行删除
nl test.txt | sed '2,5d'
# 在第二行前插入
nl test.txt | sed '2i newline'
# 在第二行后加入两行字(使用反斜杠)
nl test.txt | sed '2a newline\
newline_2'
# 搜索test中有关键字oo的行
nl test.txt | sed -n '/oo/p'
# 将test.txt文件中每行第一次出现的oo用kk替换
sed -e 's/oo/kk/' test.txt
# 将test.txt所有的oo用kk替换，并更新文件
sed -i 's/oo/kk/g' test.txt
```

## awk

### 参数选项

- -F fs or --field-separator fs
  指定输入文件折分隔符，fs是一个字符串或者是一个正则表达式，如-F:。

### 常用示例

```bash
# 每行按空格或TAB分割，输出文本中的1、4项
awk '{print $1,$4}' log.txt
# 使用","分割
awk -F, '{print $1,$2}'   log.txt
```

