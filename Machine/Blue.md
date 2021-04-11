# Blue



## 信息收集

```shell
C:\home\kali> nmap -Pn 10.10.10.40 -A               
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-11 09:45 EDT
Nmap scan report for 10.10.10.40
Host is up (0.18s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -19m57s, deviation: 34m37s, median: 0s
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-04-11T14:48:05+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-04-11T13:48:05
|_  start_date: 2021-04-11T13:45:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 146.53 seconds
```



## Ms17_010

目标系统 `Windows7 sp1`，应该是有永恒之蓝漏洞的。

直接使用msf的相关模块，如下设置参数

![image-20210411221115337](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210411221115337.png)

尝试攻击

![image-20210411221154833](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210411221154833.png)

攻击成功

![image-20210411221246494](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210411221246494.png)

直接是system权限

![image-20210411221329652](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210411221329652.png)



