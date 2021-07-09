- [shell in practice](#shell-in-practice)
  - [date](#date)
  - [随机字符串生成](#随机字符串生成)
    - [`tr` 命令](#tr-命令)
    - [生成随机数字](#生成随机数字)
  - [处理文件内容并排序](#处理文件内容并排序)
  - [统计正在连接的IP地址](#统计正在连接的ip地址)
  - [PS查看进程，并过滤grep](#ps查看进程并过滤grep)
  - [如何shell添加多行字符或者变量值](#如何shell添加多行字符或者变量值)
  - [grep 常用](#grep-常用)
  - [declare用法](#declare用法)
  - [打印数字seq](#打印数字seq)
  - [script交互](#script交互)
  - [grep和Egrep的用法](#grep和egrep的用法)

# shell in practice


## date

[devops@localhost ~]$ next_day=$(date -d "1 day" +%Y%m%d)
[devops@localhost ~]$ echo $next_day
20210629
[devops@localhost ~]$ start_day=20200322
[devops@localhost ~]$ next_old_day=$(date -d "1 day ${start_day}" +%Y%m%d)
[devops@localhost ~]$ echo $next_old_day
20200323

## 随机字符串生成
>使用循环在 /test 目录下创建 10 个 txt 文件，要求文件名称有 6 位随机小写字母加固定字符串（_gg）组成，例如 pzjebg_gg.txt

* /dev/random 依赖系统中断生成随机字符串，可以保证数据的随机性但生成数据慢，会占用系统进程资源
* /dev/urandom 不依赖系统中断生成随机字符串，生成速度快，但数据随机性不足（一般使用 /dev/urandom）

### `tr` 命令

`tr` 命令可以对来自标准输入的字符进行替换、压缩和删除。它可以将一组字符变成另一组字符。
* -c： 取代所有不属于第一字符集的字符
* -d：删除所有属于第一字符集的字符

例如：从输入文本中，把不在字符集中的字符删除
```bash
[devops@localhost ~]$ echo "aa..,+1 b2c /* $dd 3 ls 4" | tr -dc '0-9 \n'
1 2   3  4
```

```bash
#!/bin/bash

if [ ! -d /test ]; then
    mkdir /test
fi

cd /test

for((i=0;i<10;i++));do
    filename=$(tr -dc 'A-Za-z0-9' < /dev/urandom | head -c 6)
    touch ${filename}_gg.txt
done

```

### 生成随机数字
* $RANDOM # 此系统变量可以默认随机生成 0～32767 的数字
  
```bash
[devops@localhost ~]$ echo $RANDOM
12017
[devops@localhost ~]$ echo $(($RANDOM%1000))  //1000 以内的随机数
621
```

## 处理文件内容并排序

有一个 a.txt 文本（内容如下），要求将所有域名截取出来， 并统计重复域名出现的次数

```
http://www.baidu.com/index.html
https://www.atguigu.com/index.html
http://www.sina.com.cn/1024.html
https://www.atguigu.com/2048.html
http://www.sina.com.cn/4096.html
https://www.atguigu.com/8192.html

```

```bash
[devops@localhost ~]$ cat a.txt  | cut -d '/' -f 3 | sort | uniq -c | sort -nr
      3 www.atguigu.com
      2 www.sina.com.cn
      1 www.baidu.com
```

* 命令解释
  * cut -d "/" -f 3 用 "/" 作为分隔符，获取第三字段
  * sort 第一次排序
  * uniq -c 显示改行重复次数
  * sort -nr 按照数值从大到小排序（默认从小到大，-r 参数反转

## 统计正在连接的IP地址

> 统计当前服务器正在连接的 ip 地址，并按连接次数排序

```bash
[devops@localhost ~]$ netstat -an | grep ESTABLISHED | awk '{print $5}' | cut -d ':' -f 1 | sort -n | uniq -c | sort -nr
      2 10.14.5.30
```

## PS查看进程，并过滤grep

```bash
[devops@localhost ~]$ ps aux | grep  ssh
root       1182  0.0  0.0 112900  4336 ?        Ss   Jun28   0:00 /usr/sbin/sshd -D
root      40737  0.0  0.0 158852  5708 ?        Ss   Jun30   0:00 sshd: devops [priv]
devops    40740  0.0  0.0 159248  2912 ?        S    Jun30   0:00 sshd: devops@pts/0
root      41579  0.0  0.0 158852  5708 ?        Ss   07:28   0:00 sshd: devops [priv]
devops    41582  0.0  0.0 158852  2444 ?        S    07:28   0:00 sshd: devops@pts/1
devops    41663  0.0  0.0 112808   964 pts/1    S+   07:35   0:00 grep --color=auto ssh
```
所以 grep 要 grep -v grep 过滤掉 「grep」关键字，还要 grep -v $$ 或者 grep -v $0 过滤掉 bash 命令本身 

```bash
#!/bin/bash
#

this_pid=$$

ps -ef | grep ssh | grep -v grep | grep -v $this_pid  &> /dev/null

if [ $? -eq 0 ];then
    echo "sshd service is running well"
else
    /etc/init.d/sshd restart
    echo "sshd service is down, starting it ..."
fi
```

## 如何shell添加多行字符或者变量值

```bash
# possibility 1:
echo "line 1" >> greetings.txt
echo "line 2" >> greetings.txt

# possibility 2:
echo "line 1
line 2" >> greetings.txt

# possibility 3:
cat <<EOT >> greetings.txt
line 1
line 2
EOT

# possibility 4:
echo "line 1" | sudo tee -a greetings.txt > /dev/null

# possibility 3:
sudo tee -a greetings.txt > /dev/null <<EOT
line 1
line 2
EOT
```


## grep 常用

```bash

egrep -A 5 keyword txt   //向下5行
egrep -B 5 keyword txt   //向上5行
egrep -C 5 keyword txt   //显示上下5行

grep -w keyword  //精确匹配
grep -v keyword  //反向匹配

```

## declare用法

*选项*
* +/-："-"可用来指定变量的属性，"+"则是取消变量所设的属性；
* -f：仅显示函数；
* r：将变量设置为只读；
* x：指定的变量会成为环境变量，可供shell以外的程序来使用；
* i：[设置值]可以是数值，字符串或运算式。

```bash
declare test='ywnz.com'    #定义并初始化shell变量

echo $test
```

## 打印数字seq

```bash
[root@101]# for i in `seq 01 09`;do echo $i;done
1
2
3
4
5
6
7
8
9
[root@101]# for i in `seq -w 1 23`;do echo $i;done
01
02
03
04
05
06
07
08
09
10
11
12
13
14
15
16
17
18
19
20
21
22
23
```
## script交互

```bash
#!/bin/bash

read -r -p "Are You Sure? [Y/n] " input

case $input in
    [yY][eE][sS]|[yY])
		echo "Yes"
		;;

    [nN][oO]|[nN])
		echo "No"
       	;;

    *)
		echo "Invalid input..."
		exit 1
		;;
esac
```

```bash
#!/bin/bash
# init
function pause(){
   read -p "$*"
}
 
# ...
# call it
pause 'Press [Enter] key to continue...'
# rest of the script
# ...
```

## grep和Egrep的用法


* egrep和grep -E是等效的，egrep相比grep对正则表达式有了一些扩展支持，具体包括一下几点（其实这些特性grep是可以用的，只不过要在元字符前面加上转义符，比如用到+时，应敲入\+）：
```
+：匹配一个或多个先前的字符。如：'[a-z]+able’，匹配一个或多个小写字母后跟able的串，如loveable,enable,disable等。

?：匹配零个或多个先前的字符。如：’gr?p’匹配gr后跟一个或没有字符，然后是p的行。

a|b|c :匹配a或b或c。如：grep|sed匹配grep或sed

():分组符号，如：love(able|rs)ov+匹配loveable或lovers，匹配一个或多个v。

x{m},x{m,},x{m,n}:作用同x\{m\},x\{m,\},x\{m,n\}
```

* grep还支持一些POSIX字符类，也一并记录如下吧，虽然平时应该不大可能用到：
```
[:alnum:]：文字数字字符

[:alpha:]：文字字符

[:digit:]：数字字符

[:graph:]：非空字符（非空格、控制字符）

[:lower:]：小写字符

[:cntrl:]：控制字符

[:print:]：非空字符（包括空格）

[:punct:]：标点符号

[:space:]：所有空白字符（新行，空格，制表符）

[:upper:]：大写字符

[:xdigit:]：十六进制数字（0-9，a-f，A-F）
```