---
title: windows批处理操作指令
date: 2019-07-02 18:38:12
tags:
    -Windows批处理
---

## Windows环境下运行bat指令

-Windows下 初始化MySQL SQL 文件

```
@echo off　
start  cmd /k   "echo initmysql  &&  title init-mysql  && cd/d  D:\soft\mysql-5.7.26-winx64\bin && mysql -h localhost  -uroot -pnrqzdhlscx2 -Dtest < D:\soft\initmysql.sql"

```

-windows 下，打开多个窗口运行springboot jar包

```
@echo off　
start  cmd /c   "echo Starting graph-eureka-server &&  title graph-eureka-server &&  java -jar   D:\soft\java-service\graph-eureka-server.jar "

start  cmd /c  "echo Starting link-excel &&  title link-excel &&  java -Dfile.encoding=UTF-8  -jar  D:\soft\java-service\link-excel.jar" 

start  cmd /c  "echo Starting link-web  &&  title link-web &&  java -jar   D:\soft\java-service\link-web.jar" 

start  cmd /c  "echo Starting link-graph1  &&  title link-graph1 &&  java -jar D:\soft\java-service\link-graph1.jar"
```

-Windows 下关闭多个Java 虚拟机

```
taskkill /f /t /fi  "imagename eq java.exe" 
```

 ## 备注

 1、echo 命令 
 打开回显或关闭请求回显功能，或显示消息。如果没有任何参数，echo 命令将显示当前回显设置。
语法
echo [{ on|off }] [message]


2、rem 命令，注释，代表此行不会执行

rem 这是一个注释

3、pause 命令
    这是一个暂停指令，按任意键继续

4、start 命令

Start语法
start ["title"] [/dPath] [/min] [/max] [{/separate |/shared}] [{/low | /normal | /high | /realtime | /abovenormal | belownormal}][/wait] [/B] [FileName] [parameters]

启动单独的“命令提示符”窗口来运行指定程序或命令。如果在没有参数的情况下使用，start 将打开第二个命令提示符窗口。

start cmd/c 打开新窗口执行指令后关闭窗口

start cmd/k 打开新窗口执行指令后,保持新窗口

title 新窗口的标题名称

5、taskkill 指令，使用该工具按照进程 ID (PID) 或映像名称终止任务。

```
TASKKILL [/S system [/U username [/P [password]]]]   
         { [/FI filter] [/PID processid | /IM imagename] } [/T] [/F]

```
```
1. /S    system    指定要连接的远程系统。  

2. /U    [domain\]user    指定应该在哪个用户上下文执行这个命令。

3. /P    [password]       为提供的用户上下文指定密码。如果忽略，提示输入。

4. /FI   filter           应用筛选器以选择一组任务。允许使用 "*"。例如，映像名称 eq acme*

5. /PID  processid        指定要终止的进程的 PID。使用 TaskList 取得 PID。

6. /IM   imagename        指定要终止的进程的映像名称。通配符 '*'可用来 指定所有任务或映像名称。

7. /T                     终止指定的进程和由它启用的子进程。

8. /F                     指定强制终止进程。

9. /?                     显示帮助消息

```
例子，杀死对应进程：
taskkill /pid pid  

taskkill /im xxx.exe  

taskkill /fi "imagename eq xxx.exe"  

taskkill /fi "pid eq pid"  