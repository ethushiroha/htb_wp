# Vaccine



-   靶机： `10.10.10.46`
-   攻击机：`10.10.14.88`





## 信息收集



```shell
└─# nmap 10.10.10.46 -A                                                                                                                                                    1 ⚙
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-01 20:46 EST
Nmap scan report for 10.10.10.46
Host is up (0.23s latency).
Not shown: 977 closed ports
PORT      STATE    SERVICE       VERSION
21/tcp    open     ftp           vsftpd 3.0.3
22/tcp    open     ssh           OpenSSH 8.0p1 Ubuntu 6build1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c0:ee:58:07:75:34:b0:0b:91:65:b2:59:56:95:27:a4 (RSA)
|   256 ac:6e:81:18:89:22:d7:a7:41:7d:81:4f:1b:b8:b2:51 (ECDSA)
|_  256 42:5b:c3:21:df:ef:a2:0b:c9:5e:03:42:1d:69:d0:28 (ED25519)
80/tcp    open     http          Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: MegaCorp Login
```

老样子，遇到ftp和ssh先爆破一波，web服务访问也是登录，也爆破

![image-20210302095757633](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302095757633.png)

一顿操作猛如虎，一看，啥都没有。

结果ftp的密码居然是上一道题`/root/.config/filezilla`下的`filezilla.xml`

`ftpuser/mc@F1l3ZilL4`吐了



## ftp

登录上去发现backup.zip

![image-20210302095011014](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302095011014.png)

下载下来，解压需要密码

使用`fcrackzip`爆破密码，[参数和使用教程](https://blog.csdn.net/weixin_43272781/article/details/100751375)

![image-20210302095159837](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302095159837.png)

解压后得到`index.php`

```php
<?php
session_start();
  if(isset($_POST['username']) && isset($_POST['password'])) {
    if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3") {
      $_SESSION['login'] = "true";
      header("Location: dashboard.php");
    }
  }
?>
```

破解一下hash

![image-20210302095706364](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302095706364.png)



## web

得到了用户名和密码之后就可以登录了，登录之后有一个搜索框，这回总是sql注入了吧

果不其然，直接堆叠注入。

![image-20210302095959505](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302095959505.png)

看一下版本

```sql
select version();
PostgreSQL 11.5 (Ubuntu 11.5-1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 9.1.0-9ubuntu2) 9.1.0, 64-bit
```

这数据库没有注过啊。

-   sqlmap。 --os-shell

    居然能用

    ![image-20210302102217607](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302102217607.png)

    传过去一个nc，连上反弹shell

    ![image-20210302102920598](/Users/shirohaethu/Library/Application Support/typora-user-images/image-20210302102920598.png)

    ![image-20210302102938607](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302102938607.png)

user.txt

![image-20210302103249535](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302103249535.png)



-   万能的百度

![image-20210302100939553](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302100939553.png)

文件读取

![image-20210302101000333](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302101000333.png)

文件写失败了。。。



## 提权

在`/var/www/html/dashboard.php`中，有当前用户`postgres`的密码

![image-20210302103407792](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302103407792.png)

ssh建立连接

![image-20210302103509012](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302103509012.png)

`sudo -l`发现可以免密运行`/bin/vi /etc/postgresql/11/main/pg_hba.conf`，vi 是可以得到一个`/bin/bash`的

![image-20210302103649670](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302103649670.png)

![image-20210302103724396](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302103724396.png)

得到root权限

flag:

![image-20210302103748836](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302103748836.png)

