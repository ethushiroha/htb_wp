# Netmon

编号：177

目标ip：`10.10.10.152`

攻击机ip：`10.14.14.2`



## 信息收集

```shell
C:\opt\htb\intranet\Ascension> nmap -Pn 10.10.10.152 -A
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-11 05:19 EDT
Nmap scan report for 10.10.10.152
Host is up (0.23s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE      VERSION
21/tcp  open  ftp          Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-03-19  12:18AM                 1024 .rnd
| 02-25-19  10:15PM       <DIR>          inetpub
| 07-16-16  09:18AM       <DIR>          PerfLogs
| 02-25-19  10:56PM       <DIR>          Program Files
| 02-03-19  12:28AM       <DIR>          Program Files (x86)
| 02-03-19  08:08AM       <DIR>          Users
|_02-25-19  11:49PM       <DIR>          Windows
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp  open  http         Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
|_http-server-header: PRTG/18.1.37.13946
| http-title: Welcome | PRTG Network Monitor (NETMON)
|_Requested resource was /index.htm
|_http-trane-info: Problem with XML parsing of /evox/about
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-04-11T09:21:15
|_  start_date: 2021-04-11T09:17:59

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 133.35 seconds
```



## ftp

从nmap的扫描结果来看，可以匿名登录。

![image-20210411172358418](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210411172358418.png)

匿名登录后，在`/Users/Public/`目录下可以找到user.txt。



## web

访问`http://10.10.10.152/`，发现是`PRTG Network Monitor(NETMON)`。不知道是啥玩意，左下角写了版本号

![image-20210411172717919](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210411172717919.png)

查找一下漏洞`searchsploit PRTG`

![image-20210411172755057](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210411172755057.png)

找到一个rce，看一下用法

![image-20210411172902030](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210411172902030.png)

用到了cookie，应该是需要登录，但是没有用户名和密码。

回到ftp



## ftp 2

在`/ProgramData/Paessler/PRTG Network Monitor`目录下有三个配置文件，全部下载下来

![image-20210411173201612](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210411173201612.png)

最终在`PRTG Configuration.old.bak`中，找到了用户名和密码

```xml
<dbpassword>
    <!-- User: prtgadmin -->
    PrTg@dmin2018
</dbpassword>
```

尝试登录，失败。猜测是修改过密码，尝试把2018改成2019，成功。



## rce

使用刚才发现的漏洞脚本攻击，脚本会创建一个用户，并且加入到`administrator`组中。

![image-20210411173441473](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210411173441473.png)

由于是admin组的，并且目标开启了smb的各种端口。

可以使用`psexec`

![image-20210411173644194](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210411173644194.png)

得到了system权限。

