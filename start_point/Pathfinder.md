# Pathfinder



-   靶机： `10.10.10.30`
-   攻击机：`10.10.14.88`



## 信息收集



```shell
└─# nmap 10.10.10.30 -A                                                                                                                                                    4 ⚙
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-02 01:11 EST
Nmap scan report for 10.10.10.30
Host is up (0.22s latency).
Not shown: 989 closed ports
PORT     STATE SERVICE           VERSION
53/tcp   open  domain            Simple DNS Plus
88/tcp   open  kerberos-sec      Microsoft Windows Kerberos (server time: 2021-03-02 14:21:46Z)
135/tcp  open  msrpc             Microsoft Windows RPC
139/tcp  open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: MEGACORP.LOCAL0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap              Microsoft Windows Active Directory LDAP (Domain: MEGACORP.LOCAL0., Site: Default-First-Site-Name)
3269/tcp open  globalcatLDAPssl?
```

nmap扫描出了Kerberos、LDAP等，大致可以断定为是一台KDC

**由于萌新初次接触域渗透，本机子基本是照着wp复现一遍，学习一下工具的使用和原理什么的。**



## 先写写自己做了啥

大多都是无用功



### 尝试使用impacket

https://github.com/ropnop/impacket_static_binaries/releases/tag/0.9.22.dev-binaries

-   `psexec`

    久仰大名，但是使用需要目标域成员能访问`admin$`共享，即域控的`administrator`组，一般域成员使用不了。上个靶机中得到的**sandra**用户只是一个普通的域成员，无权使用

    ![image-20210302142632173](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302142632173.png)

-   `wmiexec`

    也是久仰大名了，总觉得他俩差不多，在这个环境中也用不了，没有权限（🤷‍♂️），悲

    ![F9658923E26596D4D4B082154CFDAE3D](https://gitee.com/ethustdout/pics/raw/master/uPic/F9658923E26596D4D4B082154CFDAE3D.jpg)

-   `smbexec`

    这几个的参数都差不多你敢信，万一人家就是一样的代码呢（滑稽）

    ![C3A4C2056AEDF963BA1F9A0A975764C1](https://gitee.com/ethustdout/pics/raw/master/uPic/C3A4C2056AEDF963BA1F9A0A975764C1.jpg)

-   `smbclient`

    这个能登录，但是只能读文件啊喂

    ![image-20210302142751947](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302142751947.png)



### 尝试通过上一台机子连接DC

在上一台机子的cs上加入一个target

![image-20210302142903239](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302142903239.png)

然后右键->jump

![image-20210302143007960](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302143007960.png)

使用sandra的凭证登录，都不行

获取域内用户hash，失败

![image-20210302144033652](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302144033652.png)



## 在写一下复现了啥

-   使用**BloodHound**工具，知道了存在`svc_bes`用户，并且该用户是域管理员

-   使用`GetNPUsers`，得到了`svc_bes`用户的TGT，并使用john解密

    ![image-20210302144708936](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302144708936.png)

    `Sheffield19      ($krb5asrep$23$svc_bes@MEGACORP.LOCAL)`

-   使用`evil-winrm`登录，得到命令执行权限，cs上线

-   提权

    -   使用`secretsdump_linux_x86_64`，得到`administrator`用户的hash值
    -   `psexec`使用`pass the hash`攻击



## 最后写写自己又干了啥

在拿到**svc_bes**用户cs上线之后，使用**梼杌**的获取域内全部用户的hash模块

<img src="https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302151354457.png" alt="image-20210302151354457" style="zoom:50%;" />

得到了**administrator**用户的hash值

![image-20210302150620403](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302150620403.png)

在用cs自带的`psexec`使用`pass the hash`攻击

![image-20210302151325897](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302151325897.png)

拿到的权限就是system

![image-20210302150933120](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302150933120.png)

为什么直接是system权限呢

>   psexec 会启动服务，服务（service）的权限是system 
>
>   你看看psexec会操作的详细步骤
>
>   ​																												——lhaihai学长

提上日程（新建选项卡）





## 反思

小萌新略微学习学习了cs的使用方法，对于`blood-hound`这个工具，还是一脸懵逼



小萌新在脚本小子的路上越走越远

