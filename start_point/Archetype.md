# Archetype



-   靶机： `10.10.10.27`
-   攻击机：`10.10.14.34`



## 信息收集

```shell
┌──(kali㉿kali)-[~/hack_the_box]
└─$ nmap 10.10.10.27 -A           
Nmap scan report for 10.10.10.27
Host is up (0.29s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE      VERSION
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
1433/tcp open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM
| ms-sql-ntlm-info: 
|   Target_Name: ARCHETYPE
|   NetBIOS_Domain_Name: ARCHETYPE
|   NetBIOS_Computer_Name: ARCHETYPE
|   DNS_Domain_Name: Archetype
|   DNS_Computer_Name: Archetype
|_  Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2021-02-24T22:44:23
|_Not valid after:  2051-02-24T22:44:23
|_ssl-date: 2021-02-25T08:37:30+00:00; +1h20m38s from scanner time.
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h56m37s, deviation: 3h34m40s, median: 1h20m37s
| ms-sql-info: 
|   10.10.10.27:1433: 
|     Version: 
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| smb-os-discovery: 
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: Archetype
|   NetBIOS computer name: ARCHETYPE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-02-25T00:36:53-08:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-02-25T08:36:55
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 130.65 seconds
```



扫描结果说明了：

1.  目标是`Windows Server 2019`
2.  smb可以使用guest用户登录
3.  目标使用了`mssql`



## smbclient

以guest用户登录目标机器

![image-20210225164220367](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210225164220367.png)

发现目标下存在backups文件夹可以访问。

![image-20210225164154992](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210225164154992.png)

查看一下下载下来的文件

![image-20210225164413390](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210225164413390.png)

里面写到了mssql的用户名和密码

不用猜都知道肯定是要用mssql执行系统命令拿shell



## mssql

由于对mssql缺乏认知和了解，不知道怎么连接上去，就尝试了一下`msfconsole`

看到有一个`mssql_exec`，就决定是你了小火龙

![image-20210225164550486](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210225164550486.png)

设置参数，这里有一个注意点：

>   SQL Server的身份验证方式有两种：操作系统身份验证和数据库身份验证，此处的参数值应该是操作系统身份验证方式，其前半部分ARCHETYPE是靶机的主机名，后半部分sql_svc是具有数据库登录权限的操作系统用户名。

卡了我好久

![image-20210225164644591](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210225164644591.png)

所以要把`use_windows_authent`设置为`true`，并且按照他的要求设置`domain`(图上未标出)，然后`exploit`

![image-20210225164957307](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210225164957307.png)

可以看到`whoami`命令已经执行成功了

该用户的flag在`C:\Users\sql_svc\Desktop\user.txt`，使用`type` 命令获取

![image-20210225165118035](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210225165118035.png)

这就是第一个flag



## 提权

第一次遇到Windows，要提权emmm，不知道该怎么办。

查了查wp

>   可以发现sql_svc同时是操作系统普通用户、数据库和数据库服务用户，值得去检查频繁访问的文件或已执行的命令，使用如下命令来访问PowerShell历史记录文件：

```powershell
type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

![image-20210225165415264](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210225165415264.png)

然后就懂了，这里暴露出来的肯定是用户名和密码

但是这也没开3389，咋登录windows呢

wp上给了一个工具`psexec.py`

登录上去，拿到最后的flag

![image-20210225165607949](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210225165607949.png)





## 反思

-   对于Windows的渗透，技术不到位，有很多情况不知道解决方案（mssql，提权等）
-   太过于依赖工具和脚本，对原理性的知识了解不深入

