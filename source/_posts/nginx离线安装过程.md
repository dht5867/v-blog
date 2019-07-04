---
title: nginx离线安装过程
date: 2019-07-02 19:02:39
tags:
    -NGINX
---

# nginx 离线安装过程 
1、安装gcc gcc++ 依赖

a、首先现在了nginx的最新版本nginx-1.10.0.tar.gz，上传到服务器(/usr/local/src/nginx)目录

b、解压tar -zxvf nginx-1.10.0.tar.gz

c、进入解压目录执行./configure, 这里用到了gcc、pcre、zlib库，如果没有安装会出现C compiler cc is not found等错误

gcc依赖库下载地址：http://download.csdn.net/detail/yidragon88xx/9903875

## 1)、安装gcc库
仓库地址 http://vault.centos.org/6.5/os/x86_64/Packages/ 包仓库
rpm -ivh mpfr-2.4.1-6.el6.x86_64.rpm
rpm -ivh mpfr-2.4.1-6.el6.x86_64.rpm
rpm -ivh ppl-0.10.2-11.el6.x86_64.rpm
rpm -ivh cloog-ppl-0.15.7-1.2.el6.x86_64.rpm
rpm -ivh cpp-4.4.7-17.el6.x86_64.rpm
rpm -Uvh libgcc-4.4.7-17.el6.x86_64.rpm
rpm -Uvh libgomp-4.4.7-17.el6.x86_64.rpm
rpm -ivh glibc-2.12-1.192.el6.x86_64.rpm
rpm -ivh glibc-headers-2.12-1.192.el6.x86_64.rpm
rpm -ivh glibc-devel-2.12-1.192.el6.x86_64.rpm
rpm -ivh gcc-4.4.7-17.el6.x86_64.rpm

## 2）、安装pcre库

pcre下载地址：http://download.csdn.net/detail/yidragon88xx/9903904

rpm -ivh pcre-devel-7.8-7.el6.x86_64.rpm

## 3）、安装zlib库

zlib下载地址：http://download.csdn.net/detail/yidragon88xx/9903920

rpm -ivh zlib-devel-1.2.3-3.x86_64.rpm

如果安装过程中还出现其他库没有安装的情况，可以从如下网址中搜索：

https://centos.pkgs.org
http://rpm.pbone.net/
http://www.rpm-find.net/
安装如果报错
error while loading shared libraries: libpcre.so.1: cannot open shared object file: No such file or directory，意思是找不到libpcre.so.1这个模块，而导致启动失败。

解决方法如下
如果是32位系统

[root@lee ~]#  ln -s /usr/local/lib/libpcre.so.1 /lib

如果是64位系统

[root@lee ~]#  ln -s /usr/local/lib/libpcre.so.1 /lib64


d、依赖库都安装完成之后然后重新执行

./configure

编译过程中会出现很多信息有些是not found信息，这些不用关心，只要在最后出现，表示编译成功

Configuration summary
+ using system PCRE library
+ OpenSSL library is not used
+ md5: using system crypto library
+ sha1 library is not used
+ using system zlib library 

e、执行make

f、执行make install

g、nginx就安装完成了，然后进入/usr/local/目录发现生成新文件夹nginx表示安装成了

h、进入nginx根目录的sbin下执行./nginx启动nginx

i、查看启动情况

ps -ef|grep nginx

## 4）配置NGINX 环境变量

export HGINX_HOME=/usr/local/nginx
export PATH=$PATH:$HGINX_HOME/sbin
 
## 5）配置NGINX开机自启动

	1.写自启动脚本
	2.设置权限
	chmod 777 /etc/init.d/nginx
	3.设置开机默认启动
	chkconfig --add nginx //添加系统服务
	chkconfig --level 345 nginx on //设置开机启动,启动级别
	chkconfig --list nginx //查看开机启动配置信息
## nginx控制命令

service nginx start   #开启
service nginx stop    #停止
service nginx restart #重启
service nginx reload  #重新加载
