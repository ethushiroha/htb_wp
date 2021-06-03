# Devel



目标ip：`10.10.10.5`

攻击机ip：`10.10.16.218`



## 信息收集

```shell
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-31 00:38 EDT
Nmap scan report for 10.10.10.5
Host is up (0.57s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|specialized|phone
Running (JUST GUESSING): Microsoft Windows 2008|7|Vista|Phone|8.1 (91%)
OS CPE: cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_vista::sp2 cpe:/o:microsoft:windows_7::sp1 cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows cpe:/o:microsoft:windows_8.1
Aggressive OS guesses: Microsoft Windows Server 2008 R2 (91%), Microsoft Windows Embedded Standard 7 (91%), Microsoft Windows Server 2008 (90%), Microsoft Windows Vista SP2, Windows 7 SP1, or Windows Server 2008 (90%), Microsoft Windows 8.1 Update 1 (85%), Microsoft Windows Phone 7.5 or 8.0 (85%), Microsoft Windows 7 or Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 R2 or Windows 8.1 (85%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (85%), Microsoft Windows 7 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

TRACEROUTE (using port 21/tcp)
HOP RTT       ADDRESS
1   583.65 ms 10.10.16.1
2   583.75 ms 10.10.10.5

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 76.72 seconds
```



## ftp

nmap 扫描信息得到可以使用匿名登录 ftp

![image-20210603105447207](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210603105447207.png)

看一眼目录

![image-20210603105528921](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210603105528921.png)

结合 80 页面的 iis 默认界面可知，这是 web 目录。

尝试上传一个冰蝎的 aspx shell

![image-20210603105953426](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210603105953426.png)

上传成功，尝试连接

![image-20210603110019943](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210603110019943.png)

可以连接上



## 提权

这里为了使用提权信息比对，用冰蝎弹一个 msf 的 shell

![image-20210603110404120](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210603110404120.png)

点击 给我连  就能得到一个 msf shell

使用 `post/multi/recon/local_exploit_suggester` 模块尝试提权

![image-20210603110732612](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210603110732612.png)

首先排除 bypassuac 的 poc， 然后就是`exploit/windows/local/ms10_015_kitrap0d` 模块

设置好参数 直接使用

![image-20210603111138213](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210603111138213.png)

成功。权限是 system

![image-20210603111159147](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210603111159147.png)
