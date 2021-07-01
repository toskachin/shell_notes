- [shell in practice](#shell-in-practice)
	- [date](#date)
	- [随机字符串生成](#随机字符串生成)
		- [`tr` 命令](#tr-命令)
		- [生成随机数字](#生成随机数字)
	- [处理文件内容并排序](#处理文件内容并排序)
	- [统计正在连接的IP地址](#统计正在连接的ip地址)
	- [PS查看进程，并过滤grep](#ps查看进程并过滤grep)

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

