# Shocker



-   攻击ip：`10.10.16.218`
-   目标ip：`10.10.10.56`



## 信息收集

```shell
Nmap scan report for 10.10.10.56 (10.10.10.56)
Host is up (0.75s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
```

目标开放 80/2222 端口



## web

访问之后

![image-20210801233340085](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210801233340085.png)

看不出啥东西，进行目录扫描

```shell
http://10.10.10.56:80/cgi-bin 				==> 403
http://10.10.10.56:80/cgi-bin/user.sh		==> 200
```

apache cgi-bin 存在shellshock漏洞

![image-20210801233703086](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210801233703086.png)

使用脚本进行攻击，得到shell

![image-20210801233902038](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210801233902038.png)



## 提权

上传Linux exploit suggester

https://github.com/mzet-/linux-exploit-suggester

运行之后把结果下载到本地

发现可以免密运行`/usr/bin/perl`

![image-20210801234041270](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210801234041270.png)

使用perl 提权

![image-20210801234056372](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210801234056372.png)

