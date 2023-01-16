---
title: linux常用命令
date: 2022-12-02 11:32:31
author: 三岁浪迹天涯
categories: linux
tags:
  - linux
  - 基础
---

## 文件操作

### 常见命令

```
cat			chmod		diff		cut			mv
tee			cp			which		whereis		awk
scp			sed			tr			sort		head
tail		less		more		unzip		tar
vi			wc			ln			uniq		nl
xargs
```

### 具体案例

#### 复制文件夹

cp -r 文件夹

#### 统计文件有多少行

```bash
wc testfile           # testfile文件的统计信息  
3 92 598 testfile       # testfile文件的行数为3、单词数92、字节数598
```

- -c或--bytes或--chars 只显示Bytes数。
- -l或--lines 显示行数。
- -w或--words 只显示字数。

####  **建立软链接(快捷方式)，以及硬链接的命令。**

 软链接： ln -s slink source
 硬链接： ln link source 

#### which、whereis、find、locate的区别

#### sort命令

- -n 依照数值的大小排序。
- -r 以相反的顺序来排序
- [-k field1[,field2]] 按指定的列进行排序。

#### uniq命令用法

用于检查及删除文本文件中重复出现的行列，一般与 sort 命令结合使用

- -c或--count 在每列旁边显示该行重复出现的次数。
- -d或--repeated 仅显示重复出现的行列。

当文件中重复行不是连续的，uniq是不起作用的，所以要结合sort使用

```bash
# 统计文件中不重复的行
sort test.txt | uniq
# 统计各行在文件中出现的次数
sort testfile1 | uniq -c
# 在文件中找出重复的行
sort testfile1 | uniq -d
```

#### xargs

xargs 用作替换工具，读取输入数据重新格式化后输出。

```bash
# 多行转换为单行输出# 
cat test.txt | xargs
cat test.txt | xargs
# 每行三个数据进行展示
cat test.txt | xargs -n3
```



## 磁盘管理

### 常见命令

```
df		du		cd		mkdir		ls
rmdir	pwd
```

### 具体案例

####  **du 和 df 的定义，以及区别？**

 **答案：** 

 du 显示目录或文件的大小 

 df 显示每个<文件>所在的文件系统的信息，默认是显示所有文件系统。
 （文件系统分配其中的一些磁盘块用来记录它自身的一些数据，如 i 节点，磁盘分布图，间接块，超级块等。这些数据对大多数用户级的程序来说是不可见的，通常称为 Meta Data。） du 命令是用户级的程序，它不考虑 Meta Data，而 df 命令则查看文件系统的磁盘分配图并考虑 Meta Data。
 df 命令获得真正的文件系统数据，而 du 命令只查看文件系统的部分情况。 

```
df -h查看当前目录
df -h /usr/  查看指定目录
du --max-depth=1 -h 查看当前目录每个文件夹的情况
du --max-depth=1 -h  /usr/ 指定目录
du -sh /usr/ 计算文件夹大小
```

#### 创建多级目录

mkdir -p

## 网络管理

### 常见命令

```
nc			ifconfig		netstat
```

### 具体案例

#### netstat命令参数

列出所有端口 netstat -a
列出所有 tcp 端口 netstat -at
列出所有 udp 端口 netstat -au

## 系统管理

### 常见命令

```
date		kill		ps			top			pstree
tree		shutdown	uname		who			free
crontab		yum			job			history
```

### 具体案例

#### 查看linux系统版本和内核版本

**查看内核版本**

cat /proc/version

uname -a

**查看Linux系统版本的命令**

lsb_release -a					  有的版本支持

cat /etc/redhat-release 	适合redhat系的linux，如centos

cat /etc/issue					  适合所有版本

#### ps aux命令中常见状态

![image-20221119164738929](C:\Users\Administrator\blog\newblog\source\picture\image-20221119164738929.png)

#### free命令

​	用于显示内存状态。

​	会显示内存的使用情况，包括实体内存，虚拟的交换文件内存，共享内存区段，以及系统核心使用的缓冲区等。

#### top命令

 top命令用于实时显示 process 的动态。

[Linux top命令详解：持续监听进程运行状态 (biancheng.net)](http://c.biancheng.net/view/1065.html)

top -b -n 1 > /root/top.log		\#让top命令只执行一次，然后把执行结果保存到top.log文件中，这样就能看到所有的进程了

- -b：使用批处理模式输出。一般和"-n"选项合用，用于把 top 命令重定向到文件中；
- -n 次数：指定 top 命令执行的次数。一般和"-"选项合用；
- -p 进程PID：仅查看指定 ID 的进程；

top命令可以按shift+f设置排序规则和展示内容

#### 查看某个进程下线程信息

top -H -p <PID>

ps -T -p <PID>

另外pstree -p <PID>可以查看这些进程的父子关系

#### ps -ef和ps aux的区别

ps -ef包含进程和父进程

ps aux包含内存CPU的占用情况

#### 查看后台任务?

job -l
