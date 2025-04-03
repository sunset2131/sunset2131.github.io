---
layout: config.default_layout
title: 玄机-第二章 日志分析-apache日志分析
date: 2025-04-04 00:36:35
updated: 2025-04-03 16:36:22
comments: true
tags: [玄机,应急响应,日志分析]
categories: 靶机
---

# 第二章 日志分析-apache日志分析

用的很笨的方法去做的

1. 提交当天访问次数最多的IP，即黑客IP：
    
    ```bash
    root@ip-10-0-10-5:/var/log/apache2#  cat access.log.1 | awk -F '-' '{print $1}'|sort|uniq -c|sort
          1 
          1 192.168.200.211 
          1 192.168.200.48 
         29 ::1 
          5 192.168.200.38 
       6555 192.168.200.2 
    ```
    
    可以看到最多的IP是`192.168.200.2` 既：
    
    ```bash
    flag{192.168.200.2}
    ```
    
2. 黑客使用的浏览器指纹是什么，提交指纹的md5：
    
    指纹指的应该是`UA` ，很奇怪这个不对
    
    ```bash
    root@ip-10-0-10-5:/var/log/apache2#  cat access.log.1 | grep 192.168.200.2 | head -n 1
    192.168.200.2 - - [03/Aug/2023:08:24:29 +0000] "GET / HTTP/1.1" 200 1945 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:109.0) Gecko/20100101 Firefox/115.0"
    ```
    
    再查看一下，下面还有`safari`的UA
    
    ```bash
    192.168.200.2 - - [03/Aug/2023:08:46:38 +0000] "GET / HTTP/1.1" 200 1945 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"
    192.168.200.2 - - [03/Aug/2023:08:46:38 +0000] "GET / HTTP/1.1" 200 1944 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"
    192.168.200.2 - - [03/Aug/2023:08:46:38 +0000] "GET /lts7Km HTTP/1.1" 404 492 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"
    192.168.200.2 - - [03/Aug/2023:08:46:38 +0000] "GET /UoQqyZ HTTP/1.1" 404 492 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"
    192.168.200.2 - - [03/Aug/2023:08:46:38 +0000] "GET /.ace8Cy HTTP/1.1" 404 492 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"
    192.168.200.2 - - [03/Aug/2023:08:46:38 +0000] "GET /lzz02M/ HTTP/1.1" 404 492 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"
    192.168.200.2 - - [03/Aug/2023:08:46:38 +0000] "GET /CXDj5h.php HTTP/1.1" 404 492 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"
    192.168.200.2 - - [03/Aug/2023:08:46:38 +0000] "GET /RL8s5f.aspx HTTP/1.1" 404 492 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"
    192.168.200.2 - - [03/Aug/2023:08:46:38 +0000] "GET /2oGGLR.jsp HTTP/1.1" 404 492 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"
    192.168.200.2 - - [03/Aug/2023:08:46:38 +0000] "GET /u9Mnnt.html HTTP/1.1" 404 492 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"
    192.168.200.2 - - [03/Aug/2023:08:46:38 +0000] "GET /r02mMk.js HTTP/1.1" 404 492 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"
    192.168.200.2 - - [03/Aug/2023:08:46:38 +0000] "GET /!.gitignore HTTP/1.1" 404 492 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36
    ```
    
    ```bash
    root@ip-10-0-10-5:/var/log/apache2# echo -n "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36" | md5sum
    2d6330f380f44ac20f3a02eed0958f66  
    ```
    
    ```bash
    flag{2d6330f380f44ac20f3a02eed0958f66}
    ```
    
3. 查看包含index.php页面被访问的次数，提交次数：
    
    ```bash
    root@ip-10-0-10-5:/var/log/apache2# cat access.log.1 |grep 192.168.200.2 | grep /index.php | wc -l
    	27
    ```
    
    ```bash
    flag{27}
    ```
    
4. 查看黑客IP访问了多少次，提交次数：
    
    ```bash
    root@ip-10-0-10-5:/var/log/apache2# cat access.log.1 |grep 192.168.200.2 | wc -l
    6556
    ```
    
    ```bash
    flag{6555}
    ```
    
5. 查看2023年8月03日8时这一个小时内有多少IP访问，提交次数:
    
    ```bash
    root@ip-10-0-10-5:/var/log/apache2# cat access.log.1 | grep 03/Aug/2023:08 | awk -F '-' '{print $1}'| sort | uniq | wc -l
    5
    ```
    
    ```bash
    flag{5}
    ```