---
title: 玄机-第一章 应急响应-Linux日志分析
published: 2025-03-15 16:24:02

tags: [玄机,应急响应]
category: 靶机
draft: false
---
# 第一章 应急响应-Linux日志分析

> https://xj.edisec.net/challenges/24
> 

### 有多少IP在爆破主机ssh的root帐号，如果有多个使用","分割

`ssh`登录日志一般存储在`auth.log`中，那么爆破不应该就是第一次就成功的，那么肯定会存在失败的记录所以直接搜索存在`Failed`的语句

```bash
cat auth.log.1 | grep -a "Failed password for root" | awk '{print $11}' | sort | uniq -c
      4 192.168.200.2
      1 192.168.200.31
      1 192.168.200.32
```

```bash
flag{192.168.200.2,192.168.200.31,192.168.200.32}
```

### ssh爆破成功登陆的IP是多少，如果有多个使用","分割

成功是`Accept`

```bash
cat auth.log.1 | grep -a "Accepted" | awk '{print $11}' | uniq
192.168.200.2
```

```bash
flag{192.168.200.2}
```

### 爆破用户名字典是什么？如果有多个使用","分割

根据登陆失败的记录来截取

去掉`from`因为不是用户名，是语句截取的问题

```scheme
cat auth.log.1 | grep -a "Failed" | awk -F 'for invalid user|for' '{print $2}' | awk '{print $1}'| sort |uniq -c | sort -nr
      6 root
      5 user
      5 hello
      5 from
      1 test3
      1 test2
      1 test1
```

`flag`很玄学，不知道按什么排序的

```bash
flag{user,hello,root,test3,test2,test1}
```

### 登陆成功的IP共爆破了多少次

我们知道成功爆破的ip是`192.168.200.2`

```bash
cat auth.log.1 | grep -a "Failed password for root" | awk '{print $11}' | sort | uniq -c
      4 192.168.200.2
      1 192.168.200.31
      1 192.168.200.32
```

### 黑客登陆主机后新建了一个后门用户，用户名是多少

```scheme
root@ip-10-0-10-1:/var/log# cat auth.log.1 | grep -a "new user"
Aug  1 07:50:45 linux-rz useradd[7551]: new user: name=test2, UID=1000, GID=1000, home=/home/test2, shell=/bin/sh
Aug  1 08:18:27 ip-172-31-37-190 useradd[487]: new user: name=debian, UID=1001, GID=1001, home=/home/debian, shell=/bin/bash
```

```scheme
root@ip-10-0-10-1:/var/log# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:101:102:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
systemd-network:x:102:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:103:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:104:105::/nonexistent:/usr/sbin/nologin
unscd:x:105:109::/var/lib/unscd:/usr/sbin/nologin
ntp:x:106:112::/nonexistent:/usr/sbin/nologin
sshd:x:107:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
test2:x:1000:1000::/home/test2:/bin/sh
debian:x:1001:1001:Debian:/home/debian:/bin/bash
```

明显`test2`