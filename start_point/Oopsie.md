# Oopsie



-   靶机： `10.10.10.28`
-   攻击机：`10.10.14.88`





## 信息收集



```shell
nmap 10.10.10.28 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-01 19:19 EST
Nmap scan report for 10.10.10.28
Host is up (0.24s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:e4:3f:d4:1e:e2:b2:f1:0d:3c:ed:36:28:36:67:c7 (RSA)
|   256 24:1d:a4:17:d4:e3:2a:9c:90:5c:30:58:8f:60:77:8d (ECDSA)
|_  256 78:03:0e:b4:a1:af:e5:c2:f9:8d:29:05:3e:29:c9:f2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Welcome
```

虽然应该是爆破不出来的，但是看到22端口就习惯性的爆破一下吧，万一就进去了呢

```shell
hydra -l root -P /usr/share/wordlist/rockyou.txt
```



## web

访问一下web服务

![image-20210302082954260](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302082954260.png)

啥也没有，所有的链接都跳转不了。

### 目录信息收集

接下来的突破口有两种：

-   扫目录，遇到web站点扫一下目录，运气好的话能扫出好多有用的东西。真实环境中要注意线程数和访问速度，太快的话遇到waf/cdn会被ban。这个过程可以使用`dirsearch`——[github地址](https://github.com/maurosoria/dirsearch)
-   看前端源代码，js文件中通常会暴露很多路由和接口信息，通过查看js文件，可以收集子域名和目录信息。这个过程可以使用`JSFinder`来做——[github地址](https://github.com/Threezh1/JSFinder)



### 登录窗口

这里使用第二种方法，查看网页源代码之后，在最下面发现了：

![image-20210302083525395](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302083525395.png)

访问`/cdn-cgi/login`:

![image-20210302083603915](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302083603915.png)

对于登录框，接下来要么是sql注入，要么是爆破弱密码（我是真没想到是上一道题的密码啊）

-   如果是sql注入，可以尝试万能密码`'||1||'`，或者直接使用sqlmap跑一下
-   如果是弱密码，burp抓包后，使用爆破模块即可

密码是上题得到的`MEGACORP_4dm1n!!`



### 文件上传

登录之后，选项卡中有一个`upload`，但是需要super admin权限

![image-20210302084039939](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302084039939.png)

试试其他的功能，第一个`Account`，可以看到自己的`Access ID`，是通过传入`id`变量的形式，并且cookie保存的信息就是这个`Access ID`。

![image-20210302084521198](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302084521198.png)



尝试sql注入或者越权访问

-   sql注入：

    -   直接使用sqlmap跑一下，显而易见的，失败了。
    -   手动加入 单引号、双引号、分号等特殊字符，没有报错信息。

-   越权访问：

    -   修改`id`变量的值，id为4的时候出现了另一个用户

        ![image-20210302084649936](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302084649936.png)

        但是他不是super admin，继续。

        直到`id=30`的时候，super admin的`Access ID`出来了

        ![image-20210302084736069](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302084736069.png)

        修改自己的cookie为这个值，访问`uploads`选项卡

        ![image-20210302084933102](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302084933102.png)

        访问到了`uploads`，越权成功。

        ![image-20210302085019626](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302085019626.png)



接下来的文件上传比较简单，没有保护措施

![image-20210302085233423](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302085233423.png)

但是上传的路径不知道

-   靠猜，通常都是`/upload` 或 `/uploads` 或 `/uploadfiles`这种的样子
-   目录扫描的时候扫出类似的目录



咱这里用猜的，访问`/uploads/shell.php`

![image-20210302085426082](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302085426082.png)

[蚁剑](https://github.com/AntSwordProject/AntSword-Loader)连上webshell

![image-20210302085731375](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302085731375.png)



得到第一个flag，在`/home/robert/user.txt`

![image-20210302085857872](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302085857872.png)



## 提权

### robert

在`/var/www/html/login/db.php`中发现密码

![image-20210302091843922](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302091843922.png)

ssh连接

![image-20210302092355879](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302092355879.png)



### root

Linux提权的若干方法：

-   setuid，具有s权限的文件在运行时如果代码中调用了`setuid`函数，那么会改变自己的uid，如果这个程序能打开一个shell，那么可以得到setuid对象的权限

    -   使用命令`find / -perm -u=s -type f 2>/dev/null`

        ![image-20210302090516559](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302090516559.png)

-   sudo免密运行，大致原理同上

-   docker组用户通过docker提权，通过挂载驱动的形式提权

-   内核漏洞，这个就不多说了。

-   计划任务，假设root用户每分钟会运行一次`/tmp/a`，并且该文件可写，那就可以改变文件内容，再次运行就是任意代码执行。



上面找到的`bugtracker`可能是突破口，下载该文件，是一个Linux可执行文件，简单的分析可以直接使用`strings`，或者ida反编译

![image-20210302090926036](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302090926036.png)

这里可以有两种利用方法

-   劫持cat

    ![image-20210302092558352](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302092558352.png)

    通过环境变量的劫持，达到执行命令的效果

-   命令注入

    v6变量直接拼接在`cat /root/report`之后，运行在system函数中，存在命令注入

    ![image-20210302093012476](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302093012476.png)

    但是不能用空格，因为%s遇到空格就会停止，找个空格替代品`${IFS}`就行

    ![image-20210302093359049](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302093359049.png)

flag

![image-20210302093524570](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302093524570.png)

