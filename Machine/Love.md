# Love

-   攻击机ip：`10.10.16.218`
-   目标ip：`10.10.10.239`





## 信息收集

```shell
Nmap scan report for 10.10.10.239
Host is up (0.42s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE      VERSION
80/tcp   open  http         Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1j PHP/7.3.27)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
|_http-title: Voting System using PHP
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
443/tcp  open  ssl/http     Apache httpd 2.4.46 (OpenSSL/1.1.1j PHP/7.3.27)
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
|_http-title: 403 Forbidden
| ssl-cert: Subject: commonName=staging.love.htb/organizationName=ValentineCorp/stateOrProvinceName=m/countryName=in
| Not valid before: 2021-01-18T14:00:16
|_Not valid after:  2022-01-18T14:00:16
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp  open  microsoft-ds Windows 10 Pro 19042 microsoft-ds (workgroup: WORKGROUP)
3306/tcp open  mysql?
| fingerprint-strings: 
|   DNSStatusRequestTCP, Kerberos, LANDesk-RC: 
|_    Host '10.10.16.218' is not allowed to connect to this MariaDB server
5000/tcp open  http         Apache httpd 2.4.46 (OpenSSL/1.1.1j PHP/7.3.27)
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
|_http-title: 403 Forbidden
```



## Web

80 端口，访问之后是一个登陆界面

![image-20210731235944611](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210731235944611.png)

尝试目录扫描

```shell
──(kali㉿kali)-[/opt/dirsearch]
└─$ python3 dirsearch.py --url http://10.10.10.239/ -e php                           

  _|. _ _  _  _  _ _|_    v0.4.1
 (_||| _) (/_(_|| (_| )

Extensions: php | HTTP method: GET | Threads: 30 | Wordlist size: 8884

Error Log: /opt/dirsearch/logs/errors-21-07-31_10-33-11.log

Target: http://10.10.10.239/

Output File: /opt/dirsearch/reports/10.10.10.239/_21-07-31_10-33-12.txt

[10:33:13] Starting:      
[10:33:16] 301 -  339B  - /plugins  ->  http://10.10.10.239/plugins/
[10:33:41] 301 -  337B  - /ADMIN  ->  http://10.10.10.239/ADMIN/                  
[10:33:41] 301 -  337B  - /Admin  ->  http://10.10.10.239/Admin/       
[10:33:49] 403 -  302B  - /Trace.axd::$DATA                                                
[10:33:57] 301 -  337B  - /admin  ->  http://10.10.10.239/admin/                                     
[10:33:57] 200 -    6KB - /admin%20/
[10:33:58] 301 -  338B  - /admin.  ->  http://10.10.10.239/admin./
[10:33:58] 200 -    6KB - /admin/            
[10:33:58] 200 -    6KB - /admin/?/login      
[10:33:59] 302 -   16KB - /admin/home.php  ->  index.php                                                    
[10:33:59] 200 -    6KB - /admin/index.php
[10:33:59] 302 -    0B  - /admin/login.php  ->  index.php
[10:34:16] 301 -  348B  - /bower_components  ->  http://10.10.10.239/bower_components/                            
[10:34:17] 200 -    7KB - /bower_components/                             
[10:34:19] 403 -  302B  - /cgi-bin/                     
[10:34:29] 301 -  336B  - /dist  ->  http://10.10.10.239/dist/                    
[10:34:29] 200 -    1KB - /dist/                
[10:34:39] 302 -    0B  - /home.php  ->  index.php               
[10:34:41] 301 -  338B  - /images  ->  http://10.10.10.239/images/ 
[10:34:41] 200 -    2KB - /images/
[10:34:42] 301 -  340B  - /includes  ->  http://10.10.10.239/includes/
[10:34:42] 200 -    2KB - /includes/              
[10:34:42] 200 -    4KB - /index.php                                                                           
[10:34:43] 200 -    4KB - /index.php/login/                                                                  
[10:34:50] 302 -    0B  - /login.php  ->  index.php                                                     
[10:34:51] 302 -    0B  - /logout.php  ->  index.php
[10:35:06] 200 -    2KB - /plugins/                  
```

发现有admin/路由，访问之后是一样的登录界面，`username` 字段存在sql注入

![image-20210801000154686](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210801000154686.png)



## sql注入

尝试dump数据解密，失败了

```shell
$2y$10$4E3VVe2PWlTMejquTmMD6.Og9RmmFN.K5A1n99kHNdQxHePutFjsC
```

在手动测试sql注入的时候有泄漏路径

![image-20210801000402893](/Users/shirohaethu/Library/Application Support/typora-user-images/image-20210801000402893.png)

尝试写webshell。用sqlmap的 --os-shell参数

![image-20210801000503471](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210801000503471.png)

成功。上传一句话木马之后用蚁剑连接

## webshell

![image-20210801000613366](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210801000613366.png)

上传msf后门，启动msf，得到msf的shell

![image-20210801000652544](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210801000652544.png)

## 提权

用本地提权建议模块查看信息，发现存在`exploit/windows/local/always_install_elevated`可利用模块

![image-20210801000736497](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210801000736497.png)

使用该模块攻击，得到了system权限

![image-20210801000807807](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210801000807807.png)



![image-20210801000821565](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210801000821565.png)
