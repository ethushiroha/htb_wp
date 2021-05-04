# Legacy

目标 ip ： `10.10.10.4`

攻击机 ip : `10.10.16.218`



## 信息收集

```shell
Nmap scan report for 10.10.10.4
Host is up (0.44s latency).
Not shown: 997 filtered ports
PORT     STATE  SERVICE       VERSION
139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_clock-skew: mean: 5d00h30m31s, deviation: 2h07m13s, median: 4d23h00m33s
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:11:d6 (VMware)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2021-05-09T08:11:35+03:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
```

目标是一台 **windows xp** 靶机。



## smb

扫描结果说开启了 guest 帐户，于是想要去连接一下。发现并不得行。

![2ulHY8](https://gitee.com/ethustdout/pic2/raw/master/uPic/2ulHY8.png)



还以为是要加域名，结果也还是不行，只能换条路了。

说到 Windows xp 的漏洞， 第一反应想到的就是 `ms17_010`



## ms17_010

直接上 msf 的， 失败了

![image-20210504132608387](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210504132608387.png)

只能用于 x64 的系统，但是目标是 Windows xp，只有 x86



Google 一下，找了一篇文章：https://ivanitlearning.wordpress.com/2019/02/24/exploiting-ms17-010-without-metasploit-win-xp-sp3/

里面提到了这个仓库：https://github.com/helviojunior/MS17-010

模仿他操作，成功得到一个 meterpreter shell

![U4XmsS](https://gitee.com/ethustdout/pic2/raw/master/uPic/U4XmsS.png)

在 `C:\cuments and Settings` 下的 John 和 Administrator 的 Desktop 中有 flag

 