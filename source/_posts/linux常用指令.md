---
title: linux常用指令
date: 2019-07-04 14:23:45
tags:
    --linux
---



## 一、Linux目录
/系统根目录
/bin 二进制文件目录
/etc 系统配置文件，不建议在次目录存放可执行文件，防火墙文件、网络设置文件、jdk环境，mysqlpeiz wenj 
/usr 应用程序存放文件， jdk,mysql ,redis
/root 系统管理员root家目录 ，相当于Windows的桌面家目录，每个用户都有一个家目录
## 二、磁盘管理命令
ls 列出目录内容
参数 -a 列出所有包括隐藏文件 -l 列出详情 -h友好显示
文件格式
drwxr-xr-x.  3 root root   4096 5月  22 2016 abrt
-rw-r--r--.  1 root root     44 1月   9 13:47 adjtime
-rw-r--r--.  1 root root   1512 1月  12 2010 aliases
10位 drwxr-xr-x 
r 读 w 写 x 可以执行
d 代表目录
- 代表普通文件
l 代表连接

cd  change diroctory 切换目录
 相对路径 绝对路径
cd / 
cd ~ 
cd /root 切换到根目录的root 目录

pwd 显示当前所在目录
mkdir （make dir） 创建目录
mkdir  study
rmdir  （remove dir） 删除空目录
rmdir study

## 三、文件浏览命令

打开查看 日志文件 xml文件 pro 文件

cat 小文件

more 大文件 ，enter下一行，空格下一页 q退出

less 大文件查看分页显示
-m 百分比
-N 行号
less -mN 文件名

tail
tail - 数子 文件名 快速查看文件后多少行的内容
tail -f ：表示持续侦测后面所接的档名，要等到按下[ctrl]-c才会结束tail的侦测

## 四、文件操作命令

1、文件复制 copy

cp 复制文件或目录
复制文件 cp post-install Documents
复制文件夹  -r 表示级联复制 cp -r study Documents/ 

2、文件移动 mv（move）
 mv  文件/目录 移动的位置
 mv -f 文件覆盖  mv -f t5 t3
 mv 改名 mv mv dht.txt dd.txt

3、文件删除 rm (rmove)
-f(force)
-r （reference）级联
删除文件 rm -f 文件名  rm -f dd.txt
删除目录 rm -rf 目录名  rm -rf study
注意 
rm -rf * 删除当前目录所有内容
rm -rf /*  删除Linux 系统根目录下的所有内容


4、查找命令 find 查找文件或目录
find find /root -name 'post*' 查找 以、a\post字符开头的文件或目录
结果
/root/Documents/post-install
/root/post-install.log
/root/post-install

5、文件编辑 vim 
vim 文件名进入一般模式 ，可以浏览 复制
按i 进入插入模式 ，可以编辑删除
按esc 退出到一般模式
在一般模式输入：wq退出保存，按q!强制退出不保存

## 管道命令 | 可以连接多个命令

grep 正则表达式 ，字符串搜索

grep 需要搜索的字符串 搜索的文件 复合返回当前行

管道和grep集合
ll| grep dd
-i（ignore）忽略大小写
grep - i dd 


## 压缩解压 tar

格式 *.tar 多个文件打包为一个文件，大小不会进行压缩
	*.tar.gz 打包并压缩（gzip） 

1、压缩
tar -zcvf 
z 使用gzip压缩
c  建立一个压缩文件
x 解压
v 压缩的过程显示文件
f 使用文档名

2、解压
tar zxvf redis-4.0.2.tar  解压到当前目录
tar -zxvf 压缩包 —C  解压到指定目录

## 系统命令
ps (process status)查看进程
ps -ef 查看系统进程
uid 那个用户打开属于那个用户
pid 进程ID标识进程
cmd 表明进程对应的程序，程序的位置
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 12:37 ?        00:00:01 /sbin/init
root         2     0  0 12:37 ?        00:00:00 [kthreadd]
root         3     2  0 12:37 ?        00:00:00 [migration/0]
root         4     2  0 12:37 ?        00:00:00 [ksoftirqd/0]
root       

ps -ef |grep -i vim

ps au| grep 端口、进程名

## kill 干掉进程
kill - 9 pid 

ifconfig 网络配置

ping 网络连通命令测试

nestat -nap 

halt 直接关机
reboot 重启
setup 网络配置

## 权限授权命令 
chmod(change mode)变更文件或目录权限
drwxr-xr-x. 2 root root  4096 1月  10 12:33 Desktop
drwxr-xr-x. 2 root root  4096 1月  10 13:29 Destop
drwxr-xr-x. 2 root root  4096 1月  10 13
第一位：文件类型 d目录 -文件 l 链接
2-4 位： 所属用户权限 u表示
5-7位：所属组权限 g表示
8-10 其他用户权限 o表示
2-10 所有权限 a表示
r -read 读
w write 写
x excute 执行
- 没有权限

chmod 权限设置 需要更改权限的文件名
chmod u=rw- study
chmod g=rwx study
 chmod a=rwx study
改变study文件夹和文件夹下的权限
chmod -R u=rw- study

r=4,w=2,x=1
rwxrwxrwx  777
chmod 777 文件路径和名


## 安装命令 

rpm 想当于Windows的安装，添加卸载
rpm -ivh 程序名安装
rpm -qa 查看所有程序
rpm -e --nodeps 程序名 程序卸载

yum：相当于可以联网的rpm命令,先下载程序的安装包，在执行rpm安装命令
-y 下载依赖安装
yum -y install gcc gcc-c++ autoconf automake

wget  联网下载安装
-p 级联创建
wget -P /usr/local http://nginx.org/download/nginx-1.12.2.tar.gz

防火墙


## shell脚本 


```
#！告诉系统其后路径所指定的程序，就是解释次脚本文件的shell 程序
#! /bash/shell
echo "hello world"
```
赋予脚本执行权限
chmod +x ./test.sh  #使脚本具有执行权限
./test.sh  #执行脚本

直接写test.sh执行过程linux系统会去path里寻找有没有test.sh的，而path路径里只有
/bin,/sbin,/usr/bin,/usr/sbin,你的当前目录并不在path里，所以写成test.sh是不会找到命令的，需要./test.sh
告诉系统在当前目录找

定义shell变量， 
your_name="runoob.com"，但是变量名和等号之间不能有空格，不能使用bash关键字

意，变量名和等号之间不能有空格，这可能和你熟悉的所有编程语言都不一样。同时，变量名的命名须遵循如下规则：
命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。
中间不能有空格，可以使用下划线（_）。
不能使用标点符号。
不能使用bash里的关键字（可用help命令查看保留关键字）。


## 常用例子
解压文件
tar zxvf redis-4.0.2.tar

移动文件
mv redis-4.0.2  /usr/local/

创建文件
mkdir  /usr/local/redis-4.0.2/bin
复制文件
cp /usr/local/redis-4.0.2/redis.conf /usr/local/redis-4.0.2/etc

查看日志文件
tail -f 文件名

natapp

/Users/ximoyiren/soft/natapp


查看端口占用

netstat -tunlp|grep 13200

lsof -i:13200
 
查看java进程
ps -ef | grep java

ps -ef | grep wecourt-worklog.jar

查看Tomcat进程

ps -ef | grep tomcat6_js13200

找到pid 在kill
