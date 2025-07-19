[TOC]



## 第一章：docker环境搭建

### 一，获取VPN代理

https://xn--cp3a08l.com/#/register?code=dGEFYisq

进入上面的网站后，注册并登录；

在该网站里选择“**一瓶盖可乐**”（最经济实惠），并支付后，点击使用文档，下滑找到windows系统教程，进行vpn代理相关软件的安装和配置。

![](E:\渗透\docker环境搭建\1.png)

根据这个使用文档，配置好自己的vpn代理。

![](E:\渗透\docker环境搭建\3.png)

### 二，配置Ubuntu系统

1，激活root用户，并配置root用户的ssh远程连接

```
# 在普通用户界面进入root用户界面
ning@ning:~$ sudo -i
[sudo] ning 的密码：      # 输入普通用户的密码
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

root@ning:~# passwd
新的密码：                  # 设置root用户的密码
重新输入新的密码：           # 再次输入rroot用户的密码

root@ning:~# vim /etc/ssh/sshd_config       # 将该配置文件中的root登录开启
PermitRootLogin yes

root@ning:~# systemctl restart ssh      # 重启ssh服务
```

2，安装docker，并配置docker配置文件

```
root@ning:~# apt-get install docker.io docker-compose

root@ning:~# docker -v     # 查看docker的版本
Docker version 27.5.1, build 27.5.1-0ubuntu3~24.04.2

root@ning:~# cd /etc/systemd/system
root@ning:/etc/systemd/system# mkdir docker.service.d   # 创建docker配置文档路径
root@ning:/etc/systemd/system# cd docker.service.d/
root@ning:/etc/systemd/system/docker.service.d# vim http-proxy.conf      # 编辑docker配置文件
    [Service]
    Environment="HTTP_PROXY=http://10.100.166.50:7890"      # IP为物理主机IP，端口是代理VPN的端口
    Environment="HTTPS_PROXY=http://10.100.166.50:7890"
    Environment="NO_PROXY=localhost,127.0.0.1"

root@ning:/etc/systemd/system/docker.service.d# systemctl daemon-reload 
root@ning:/etc/systemd/system/docker.service.d# systemctl restart docker          # 重启docker服务

root@ning:/etc/systemd/system/docker.service.d# docker pull nginx          # 尝试使用docker爬取nginx服务（正常显示下载即为成功）
```

3，在VS Code中连接虚拟机，方便后续修改相关文件的程序代码

1. 在VS Code扩展中搜索并下载：`Remote - SSH` 和 `Docker` 插件；

2. 点击**远程资源管理器**，在SSH中点击**新建远程**，并在弹出的窗口输入ssh连接指令

   ```
   ssh root@192.168.64.133 -A
   ```

   ![](E:\渗透\docker环境搭建\4.png)

3. 选择相应的ssh配置文件

   ![](E:\渗透\docker环境搭建\5.png)

4. 输入root用户的密码，即可连接成功

   ![](E:\渗透\docker环境搭建\6.png)

特殊情况：重置docker-compose：

```
root@ning:~# find / -name docker-compose
find: ‘/run/user/1000/gvfs’: 权限不够
find: ‘/run/user/1000/doc’: 权限不够
/usr/bin/docker-compose
/usr/share/doc/docker-compose
/usr/share/bash-completion/completions/docker-compose
root@ning:~# cd /usr/bin
root@ning:/usr/bin# rm -rf docker-compose
root@ning:/usr/bin# cd /usr/share/doc
root@ning:/usr/share/doc# rm -rf docker-compose
root@ning:/usr/share/doc# cd /usr/share/bash-completion/completions/
root@ning:/usr/share/bash-completion/completions# rm -rf docker-compose
root@ning:/usr/share/bash-completion/completions# cd
root@ning:~# wget https://github.com/docker/compose/releases/download/v2.38.2/docker-compose-linux-x86_64      # 这条命令可能会执行不成功，需要多试几次
--2025-07-09 18:08:01--  https://github.com/docker/compose/releases/download/v2.38.2/docker-compose-linux-x86_64

root@ning:~# mv docker-compose-linux-x86_64 docker-compose    # 重命名
root@ning:~# mv docker-compose /usr/bin
root@ning:~# cd /usr/bin
root@ning:/usr/bin# chmod +x docker-compose      # 给docker文件添加执行权限
```



### 三，拉取镜像，部署漏洞环境

1，安装代理工具，并配置文件

```
root@ning:~# apt-get install proxychains -y

root@ning:~# vim /etc/proxychains.conf
socks4  10.100.166.50 7890         # 在文件最后一行添加

root@ning:~# proxychains curl http://www.google.com       # 尝试访问外网服务器（能成功访问则代理工具正常）
```

![](E:\渗透\docker环境搭建\7.png)

```
###### 全局代理
在 Shell 中设置全局代理环境变量（在拉取时关闭Windows防火墙，我配置了防火墙规则没有用，具体原因不清楚）

vim ~/.bashrc    在kali中配置~/.zshrc的内容

#添加以下内容
export HTTP_PROXY="http://192.168.64.1:7890"
export HTTPS_PROXY="http://192.168.64.1:7890"
export NO_PROXY="localhost,127.0.0.1"

source ~/.bashrc #重启环境

env | grep -i proxy  #查看当前环境变量的代理配置
curl -I https://www.google.com  #查看全局代理是否正常工作
```

2，拉取镜像

![](E:\渗透\docker环境搭建\8.png)复制镜像文件地址。

```
root@ning:~# proxychains git clone https://github.com/vulhub/vulhub.git     # 拉取环境
```

3，查看拉取结果

```
root@ning:~# cd vulhub/        # 该文件夹下面就是拉去下来的文件，进入到仙人掌的虚拟镜像
root@ning:~/vulhub# cd cacti/
root@ning:~/vulhub/cacti# ls -al
总计 20
........
drwxr-xr-x   2 root root 4096  7月  9 13:12 CVE-2022-46169
drwxr-xr-x   2 root root 4096  7月  9 10:37 CVE-2023-39361
drwxr-xr-x   2 root root 4096  7月  9 10:37 CVE-2025-24367
root@ning:~/vulhub/cacti# cd CVE-2022-46169/
root@ning:~/vulhub/cacti/CVE-2022-46169# ls -al
总计 444
......
-rw-r--r-- 1 root root    348  7月  9 10:37 docker-compose.yml
-rw-r--r-- 1 root root    648  7月  9 10:37 entrypoint.sh
-rw-r--r-- 1 root root   1981  7月  9 10:37 README.md
-rw-r--r-- 1 root root   1746  7月  9 10:37 README.zh-cn.md
root@ning:~/vulhub/cacti/CVE-2022-46169# vim docker-compose.yml 
    version: '2'
    services:
      web:
        image: vulhub/cacti:1.2.22
        ports:
         - "8080:80"
        depends_on:
         - db
        entrypoint:
         - bash
         - /entrypoint.sh
        volumes:
         - ./entrypoint.sh:/entrypoint.sh
        command: apache2-foreground
      db:
       image: mysql:5.7
       environment:
        - MYSQL_ROOT_PASSWORD=root
        - MYSQL_DATABASE=cacti
```

4，拉取新的镜像文件，部署漏洞环境

```
root@ning:~/vulhub/cacti/CVE-2022-46169# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
root@ning:~/vulhub/cacti/CVE-2022-46169# docker images     # 查看已有的镜像文件
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE

root@ning:~/vulhub/cacti/CVE-2022-46169# docker-compose up -d    # 运行，拉取镜像下载
```

![](E:\渗透\docker环境搭建\9.png)

```
root@ning:~/vulhub/cacti/CVE-2022-46169# docker ps -a
CONTAINER ID   IMAGE                 COMMAND                   CREATED         STATUS         PORTS                                     NAMES
6028665efcca   vulhub/cacti:1.2.22   "bash /entrypoint.sh…"   4 minutes ago   Up 4 minutes   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   cve-2022-46169-web-1
2e30d034b58d   mysql:5.7             "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   3306/tcp, 33060/tcp                       cve-2022-46169-db-1
root@ning:~/vulhub/cacti/CVE-2022-46169# docker images     # 查看docker镜像，镜像的使用还需要部署
REPOSITORY     TAG       IMAGE ID       CREATED         SIZE
nginx          latest    9592f5595f2b   2 weeks ago     192MB
mysql          5.7       5107333e08a8   19 months ago   501MB
vulhub/cacti   1.2.22    c0a06715ff54   2 years ago     642MB

root@ning:~/vulhub/cacti/CVE-2022-46169# curl http://127.0.0.1:8080      # 访问本地，在浏览器中输入虚拟主机的IP后，出现如下页面，表示配置成功
```

![](E:\渗透\docker环境搭建\10.png)

6，进入指定的容器中进行配置

```
root@ning:~/vulhub/cacti/CVE-2022-46169# docker ps -a
CONTAINER ID   IMAGE                 COMMAND                   CREATED          STATUS          PORTS                                     NAMES
6028665efcca   vulhub/cacti:1.2.22   "bash /entrypoint.sh…"   12 minutes ago   Up 12 minutes   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   cve-2022-46169-web-1
2e30d034b58d   mysql:5.7             "docker-entrypoint.s…"   12 minutes ago   Up 12 minutes   3306/tcp, 33060/tcp                       cve-2022-46169-db-1
root@ning:~/vulhub/cacti/CVE-2022-46169# docker exec -it 6028665efcca /bin/bash
root@6028665efcca:/var/www/html#
root@6028665efcca:/var/www/html# cd ~
root@6028665efcca:~# cd /
root@6028665efcca:/# ls -al
-rwxr-xr-x   1 root root    0 Jul  9 11:00 .dockerenv        # 出现这个文件就表示成功进入到指定容器当中。
```

7，修改容器中的文件内容

由于docker属于最小化安装，只安装了必要的软件，大部分软件需要重新手动安装。

方法一：使用`apt-get update`更新虚拟机的源（最麻烦），然后在重新安装软件

```
root@6028665efcca:/var/www/html# apt-get update
```

方法二：将docker中的文件拷贝到本地，修改完后重新拷贝回去

```
# 以容器 6028665efcca:/var/www/html/vdef.php 为例：
root@ning:~/vulhub/cacti/CVE-2022-46169# docker cp 6028665efcca:/var/www/html/vdef.php /tmp      # 将该文件拷贝到本地的/tmp文件下
Successfully copied 30.7kB to /tmp

root@ning:~/vulhub/cacti/CVE-2022-46169# ls -al /tmp/       # 查看文件是否拷贝成功
-rw-rw-r--  1 root root 28883  8月 15  2022 vdef.php

# 在修改完指定文件中的内容后，将文件又拷贝回去
root@ning:~/vulhub/cacti/CVE-2022-46169# docker cp /tmp/vdef.php 6028665efcca:/var/www/html/vdef.php
```

方法三：在VS code中修改

在VS Code上通过访问文件列表，可以查看和编辑文件内容：

![](E:\渗透\docker环境搭建\13.png)

在VS Code中安装Docker环境：

![](E:\渗透\docker环境搭建\11.png)

选择安装的 `Containers` ：

![](E:\渗透\docker环境搭建\12.png)



### 四，Ubuntu安装nginx以及php的部署

#### 1，安装依赖包

```
apt-get install gcc
apt-get install libpcre3 libpcre3-dev
apt-get install zlib1g zlib1g-dev
apt-get install openssl
apt-get install libssl-dev
```

#### 2，安装nginx

```
cd /usr/local
root@ning:/usr/local# mkdir nginx
root@ning:/usr/local# cd nginx/

root@ning:/usr/local/nginx# wget https://nginx.org/download/nginx-1.21.6.tar.gz
root@ning:/usr/local/nginx# tar -xvf nginx-1.21.6.tar.gz
```

#### 3，编译nginx

```
root@ning:/usr/local/nginx# cd nginx-1.21.6/
root@ning:/usr/local/nginx/nginx-1.21.6# ./configure    # 执行命令

# 执行make命令
make

# 执行make install命令
root@ning:/usr/local/nginx/nginx-1.21.6# make install 
```

#### 4，启动nginx

```
cd /usr/local/nginx/sbin

# 启动nginx

./nginx
```

#### 5，访问nginx

![image-20250712091729764](E:\渗透\docker环境搭建\14.jpg)

#### 6，增加源地址

执行三条命令，添加php的源地址，更新，安装：

```
root@ning:~# apt-get install software-properties-common -y
root@ning:~# add-apt-repository -y ppa:ondrej/php
root@ning:~# apt-get update
```

#### 7，安装php

nginx使用php的话要用到php7.3-fpm，所以要安装：

```
root@ning:~# sudo apt-get install php7.3 php7.3-mysql php7.3-fpm php7.3-curl php7.3-xml php7.3-gd php7.3-mbstring php-memcached php7.3-zip
```

#### 8，配置php-fpm

更改php的www.conf配置文件：

```
root@ning:~# vim /etc/php/7.3/fpm/pool.d/www.conf 

;listen = /run/php/php7.3-fpm.sock                # nu36
listen = 127.0.0.1:9000

listen.mode = 0660          # nu51
```

#### 9，启动php-fpm

```
service php7.3-fpm start
netstat -lnt | grep 9000
```

查看9000端口

![image-20250712162406413](E:\渗透\docker环境搭建\15.png)

`如果运行 netstat -lnt | grep 9000 命令看不见结果，只需要重启虚拟机即可。`

#### 10，更改nginx配置文件

```
root@ning:/usr/local/nginx/conf# vim nginx.conf
user  www-data;           # nu3
....
location / {          # nu44
    root   html;
    index  index.html index.htm index.php;
}
.....
location ~ \.php$ {       # nu66
  root           html;
  fastcgi_pass   127.0.0.1:9000;
  fastcgi_index  index.php;
  fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
  include        fastcgi_params;
}
```

```
root@ning:/usr/local/nginx/conf# ps -ef | grep php-fpm
root        1512       1  0 16:31 ?        00:00:00 php-fpm: master process (/etc/php/7.3/fpm/php-fpm.conf)
www-data    1533    1512  0 16:31 ?        00:00:00 php-fpm: pool www
www-data    1534    1512  0 16:31 ?        00:00:00 php-fpm: pool www
root        4005    3721  0 16:39 pts/0    00:00:00 grep --color=auto php-fpm
root@ning:/usr/local/nginx/conf# cd ..
root@ning:/usr/local/nginx# cd sbin/
root@ning:/usr/local/nginx/sbin# ./nginx 
root@ning:/usr/local/nginx/sbin# ps -ef | grep nginx
root        1573       1  0 16:31 ?        00:00:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data    1576    1573  0 16:31 ?        00:00:00 nginx: worker process
www-data    1577    1573  0 16:31 ?        00:00:00 nginx: worker process
root        4012    3721  0 16:40 pts/0    00:00:00 grep --color=auto nginx
```

访问phpinfo.php文件

```
root@ning:/usr/local/nginx/html# vim phpinfo.php 
<?php phpinfo();?>
```

![image-20250714090620771](E:\渗透\docker环境搭建\16.png)

出现过的问题及解决：

问题：在编辑好nginx和php的配置文件后，能够访问nginx的web页面，但是访问不了php的web页面

解决：经过查找问题来源，是多个nginx进程（nginx和nginx-1.21.6）发生冲突，删除原本的nginx服务后，重新编译安装nginx-1.21.6.tar.gz，并重新编辑配置文件后实现访问php页面。

```
mv nginx-1.21.6.tar.gz ~        # 将nginx编译安装包移动到~目录下
cd ..
rm -rf nginx/        # 删除原本的nginx文件
cd ~
tar -zxvf nginx-1.21.6.tar.gz         # 解压nginx安装包
cd nginx-1.21.6/
./configure 
make && make install           # 重新编译安装
cd /usr/local/nginx/
cd conf/
vim nginx.conf
ps -ef | grep php-fpm
lsof -i:9000
cd ../sbin/
./nginx 
lsof -i:80
service nginx stop
./nginx 
cd ../html/
vim phpinfo.php
```





### 五，xdebug调试php

打开`小皮系统 → 启动nginx → 编辑test.php文件 → 安装php插件并点击PHP Debug扩展 → 根据PHP Debug扩展的内容进行配置`。

#### 在phpstudy中启用xdebug扩展

![img](E:\渗透\docker环境搭建\17.png)

#### 启动nginx

![image-20250714140752403](E:\渗透\docker环境搭建\18.png)

#### 编辑test.php文件

![image-20250714140829978](E:\渗透\docker环境搭建\19.png)

#### 安装相关插件

![image-20250712170141362](E:\渗透\docker环境搭建\20.png)

![image-20250714140936991](E:\渗透\docker环境搭建\21.png)

网页访问php文件并打开网页源码，将其复制到***[Xdebug installation wizard](https://xdebug.org/wizard.php)\***的网页中进行编译：

![image-20250714141234028](E:\渗透\docker环境搭建\22.png)

![image-20250714141310077](E:\渗透\docker环境搭建\23.png)

得到了主机中php的版本，以及配置Debug的方式

#### 打开php.ini，修改配置

文件路径：`D:\phpstudy_pro\Extensions\php\php7.3.4nts\php.ini`

```
[Xdebug]
zend_extension=D:/phpstudy_pro/Extensions/php/php7.3.4nts/ext/php_xdebug.dll
xdebug.collect_params=1
xdebug.collect_return=1
xdebug.auto_trace=On
xdebug.trace_output_dir=D:/phpstudy_pro/Extensions/php_log/php7.3.4nts.xdebug.trace
xdebug.profiler_enable=On
xdebug.profiler_output_dir=D:/phpstudy_pro/Extensions/php_log/php7.3.4nts.xdebug.profiler
xdebug.remote_enable=On
xdebug.remote_autostart = On
xdebug.remote_host=localhost
xdebug.remote_port=9001
xdebug.remote_handler=dbgp
// xdebug.remote_enable = 1
// xdebug.remote_autostart = 1
// xdebug.remote_port = 9001
```

端口不能用9000，不然会和Nginx的端口冲突。

#### 修改VS Code配置

文件-->首选项-->设置

![image-20250714141543569](E:\渗透\docker环境搭建\24.png)

在打开的settings.json中，添加php路径：

```
{
    "security.workspace.trust.untrustedFiles": "open",
    "php.suggest.basic": false,
    "php.validate.enable": false,
    "emmet.excludeLanguages": [
        "markdown",
        "php"
    ],
    "editor.fontSize": 14,
    "php.debug.executablePath": "D:\\phpstudy_pro\\Extensions\\php\\php7.3.4nts\\php.exe",
    "php.validate.executablePath": "D:\\phpstudy_pro\\Extensions\\php\\php7.3.4nts\\php.exe"
}
// 文件路径：D:\phpstudy_pro\Extensions\php\php7.3.4nts\php.exe
```

![image-20250714142423788](E:\渗透\docker环境搭建\25.png)

#### 配置launch.jison，注意端口号要跟php.ini中一致

```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch current script in console",
            "type": "php",
            "request": "launch",
            "program": "${file}",
            "cwd": "${fileDirname}",
            "externalConsole": false,
            "port": 9001
        },
        {
            "name": "Listen for XDebug",
            "type": "php",
            "request": "launch",
            "port": 9001
        }
    ]
}
```

![image-20250714143030668](E:\渗透\docker环境搭建\26.png)

开启调试，访问就ok了：

![image-20250714152759661](E:\渗透\docker环境搭建\27.png)

访问网页时，一直在运行（这是因为在代码中设置了断点）

![image-20250714152959305](E:\渗透\docker环境搭建\28.png)

![image-20250714153057383](E:\渗透\docker环境搭建\29.png)

![image-20250714153109337](E:\渗透\docker环境搭建\30.png)

`注意：如果在vscode上运行时，未出现上面图片上的两种状态，且访问页面能够正常输出显示，只需要在小皮面板上重启一下nginx服务即可。` 

### 六，HTTPS原理和RSA加密算法

#### 1，TLS握手过程

HTTP 由于是明文传输，所谓的明文，就是说客户端与服务端通信的信息都是肉眼可见的，随意使用一个抓包工具都可以截获通信的内容。 

所以安全上存在以下三个风险：完整性、可用性、保密性

- **窃听风险**，比如通信链路上可以获取通信内容，用户号容易没。
- **篡改风险**，比如强制植入垃圾广告，视觉污染，用户眼容易瞎。
- **冒充风险**，比如冒充淘宝网站，用户钱容易没。

HTTP**S** 在 HTTP 与 TCP 层之间加入了 TLS 协议，来解决上述的风险。

![image-20250712085630542](E:\渗透\docker环境搭建\31.png)

TLS 协议是如何解决 HTTP 的风险的呢？

- **信息加密**：HTTP 交互信息是被加密的，第三方就无法被窃取；
- **校验机制**：校验信息传输过程中是否有被第三方篡改过，如果被篡改过，则会有警告提示；
- **身份证书**：证明淘宝是真的淘宝网；

可见，有了 TLS 协议，能保证 HTTP 通信是安全的了，那么在进行 HTTP 通信前，需要先进行 TLS 握手。TLS 的握手过程，如下图：

![image-20210126102630933](E:\渗透\docker环境搭建\32.png)

上图简要概述来 TLS 的握手过程，其中每一个「框」都是一个记录（*record*），记录是 TLS 收发数据的基本单位，类似于 TCP 里的 segment。多个记录可以组合成一个 TCP 包发送，所以**通常经过「四个消息」就可以完成 TLS 握手，也就是需要 2个 RTT 的时延**，然后就可以在安全的通信环境里发送 HTTP 报文，实现 HTTPS 协议。

所以可以发现，HTTPS 是应用层协议，需要先完成 TCP 连接建立，然后走 TLS 握手过程后，才能建立通信安全的连接。

事实上，不同的密钥交换算法，TLS 的握手过程可能会有一些区别（上图的握手过程是DHE密钥协商的过程，**主要区别是DHE/ECDHE协商有server key exchange的握手过程，而RSA没有**）。

这里先简单介绍下密钥交换算法，因为考虑到性能的问题，所以双方在加密应用信息时使用的是对称加密密钥，而对称加密密钥是不能被泄漏的，为了保证对称加密密钥的安全性，所以使用非对称加密的方式来保护对称加密密钥的协商，这个工作就是密钥交换算法负责的。

接下来，我们就以最简单的 `RSA` 密钥交换算法，来看看它的 TLS 握手过程。更复杂更安全的密钥交换算法还有DHE算法，ECDHE算法。

#### ２，RSA密钥协商握手过程

![图片](E:\渗透\docker环境搭建\33.png)

对应 Wireshark 的抓包，从下图很清晰地看到该过程：

![图片](E:\渗透\docker环境搭建\34.png)

##### TLS 第一次握手

客户端首先会发一个「**Client Hello**」消息，字面意思我们也能理解到，这是跟服务器「打招呼」。

![图片](E:\渗透\docker环境搭建\35.png)

消息里面有客户端使用的 TLS 版本号、支持的密码套件列表，支持的压缩算法，以及生成的**随机数（\*Client Random\*）**，这个随机数会被服务端保留，它是生成对称加密密钥的材料之一。

##### TLS 第二次握手

当服务端收到客户端的「Client Hello」消息后，会确认 TLS 版本号是否支持，和从密码套件列表中选择一个密码套件，还有选择压缩算法（安全性原因，一般不压缩），以及生成**随机数（\*Server Random\*）**。

接着，返回「**Server Hello**」消息，消息里面有服务器确认的 TLS 版本号，也给出了随机数（Server Random），然后从客户端的密码套件列表选择了一个合适的密码套件。

![图片](E:\渗透\docker环境搭建\36.png)

可以看到，服务端选择的密码套件是 “Cipher Suite: TLS_RSA_WITH_AES_128_GCM_SHA256”。

这个密码套件看起来真让人头晕，好一大串，但是其实它是有固定格式和规范的。基本的形式是「**密钥交换算法 + 签名算法 + 对称加密算法 + 摘要算法**」， 一般 WITH 单词前面有两个单词，第一个单词是约定密钥交换的算法，第二个单词是约定证书的验证算法。比如刚才的密码套件的意思就是：

- 由于 WITH 单词只有一个 RSA，则说明握手时密钥交换算法和签名算法都是使用 RSA；
- 握手后的通信使用 AES 对称算法，密钥长度 128 位，分组模式是 GCM；
- 摘要算法 SHA256 用于消息认证和产生随机数；

就前面这两个客户端和服务端相互「打招呼」的过程，客户端和服务端就已确认了 TLS 版本和使用的密码套件，而且你可能发现客户端和服务端都会各自生成一个随机数，并且还会把随机数传递给对方。

那这个随机数有啥用呢？其实这两个随机数是后续作为生成「会话密钥」的条件，所谓的会话密钥就是数据传输时，所使用的对称加密密钥。

然后，服务端为了证明自己的身份，会发送「**Server Certificate**」给客户端，这个消息里含有数字证书。

![图片](E:\渗透\docker环境搭建\37.png)

随后，服务端发了「**Server Hello Done**」消息，目的是告诉客户端，我已经把该给你的东西都给你了，本次打招呼完毕。

![image-20210831144857433](E:\渗透\docker环境搭建\38.png)

###### 客户端验证证书

在这里刹个车，客户端拿到了服务端的数字证书后，要怎么校验该数字证书是真实有效的呢？

###### 数字证书和 CA 机构

在说校验数字证书是否可信的过程前，我们先来看看数字证书是什么，一个数字证书通常包含了：

- 公钥；
- 持有者信息；
- 证书认证机构（CA）的信息；
- CA 对这份文件的数字签名及使用的算法；
- 证书有效期；
- 还有一些其他额外信息；

那数字证书的作用，是用来认证公钥持有者的身份，以防止第三方进行冒充。说简单些，证书就是用来告诉客户端，该服务端是否是合法的，因为只有证书合法，才代表服务端身份是可信的。

我们用证书来认证公钥持有者的身份（服务端的身份），那证书又是怎么来的？又该怎么认证证书呢？

为了让服务端的公钥被大家信任，服务端的证书都是由 CA （*Certificate Authority*，证书认证机构）签名的，CA 就是网络世界里的公安局、公证中心，具有极高的可信度，所以由它来给各个公钥签名，信任的一方签发的证书，那必然证书也是被信任的。

之所以要签名，是因为签名的作用可以避免中间人在获取证书时对证书内容的篡改。

###### 数字证书签发和验证流程

如下图图所示，为数字证书签发和验证流程：

![图片](E:\渗透\docker环境搭建\39.png)

CA 签发证书的过程，如上图左边部分：

- 首先 CA 会把持有者的公钥、用途、颁发者、有效时间等信息打成一个包，然后对这些信息进行 Hash 计算，得到一个 Hash 值；
- 然后 CA 会使用自己的私钥将该 Hash 值加密，生成 Certificate Signature，也就是 CA 对证书做了签名；
- 最后将 Certificate Signature 添加在文件证书上，形成数字证书；

客户端校验服务端的数字证书的过程，如上图右边部分：

- 首先客户端会使用同样的 Hash 算法获取该证书的 Hash 值 H1；
- 通常浏览器和操作系统中集成了 CA 的公钥信息，浏览器收到证书后可以使用 CA 的公钥解密 Certificate Signature 内容，得到一个 Hash 值 H2 ；
- 最后比较 H1 和 H2，如果值相同，则为可信赖的证书，否则则认为证书不可信。

###### 证书链

但事实上，证书的验证过程中还存在一个证书信任链的问题，因为我们向 CA 申请的证书一般不是根证书签发的，而是由中间证书签发的，比如百度的证书，从下图你可以看到，证书的层级有三级：

![image-20250712084757337](E:\渗透\docker环境搭建\40.png)

对于这种三级层级关系的证书的验证过程如下：

- 客户端收到 baidu.com 的证书后，发现这个证书的签发者不是根证书，就无法根据本地已有的根证书中的公钥去验证 baidu.com 证书是否可信。于是，客户端根据 baidu.com 证书中的签发者，找到该证书的颁发机构是 “GlobalSign Organization Validation CA - SHA256 - G2”，然后向 CA 请求该中间证书。
- 请求到证书后发现 “GlobalSign Organization Validation CA - SHA256 - G2” 证书是由 “GlobalSign Root CA” 签发的，由于 “GlobalSign Root CA” 没有再上级签发机构，说明它是根证书，也就是自签证书。应用软件会检查此证书有否已预载于根证书清单上，如果有，则可以利用根证书中的公钥去验证 “GlobalSign Organization Validation CA - SHA256 - G2” 证书，如果发现验证通过，就认为该中间证书是可信的。
- “GlobalSign Organization Validation CA - SHA256 - G2” 证书被信任后，可以使用 “GlobalSign Organization Validation CA - SHA256 - G2” 证书中的公钥去验证 baidu.com 证书的可信性，如果验证通过，就可以信任 baidu.com 证书。

在这四个步骤中，最开始客户端只信任根证书 GlobalSign Root CA 证书的，然后 “GlobalSign Root CA” 证书信任 “GlobalSign Organization Validation CA - SHA256 - G2” 证书，而 “GlobalSign Organization Validation CA - SHA256 - G2” 证书又信任 baidu.com 证书，于是客户端也信任 baidu.com 证书。

总括来说，由于用户信任 GlobalSign，所以由 GlobalSign 所担保的 baidu.com 可以被信任，另外由于用户信任操作系统或浏览器的软件商，所以由软件商预载了根证书的 GlobalSign 都可被信任。

![图片](E:\渗透\docker环境搭建\41.png)

操作系统里一般都会内置一些根证书，比如我的 MAC 电脑里内置的根证书有这么多：

![图片](E:\渗透\docker环境搭建\42.png)

这样的一层层地验证就构成了一条信任链路，整个证书信任链验证流程如下图所示：

![图片](E:\渗透\docker环境搭建\43.png)

最后一个问题，为什么需要证书链这么麻烦的流程？Root CA 为什么不直接颁发证书，而是要搞那么多中间层级呢？

这是为了确保根证书的绝对安全性，将根证书隔离地越严格越好，不然根证书如果失守了，那么整个信任链都会有问题。

##### TLS 第三次握手

客户端验证完证书后，认为可信则继续往下走。接着，客户端就会生成一个新的**随机数 (pre-master)**，用服务器的 RSA 公钥加密该随机数，通过「**Change Cipher Key Exchange**」消息传给服务端。

服务端收到后，用 RSA 私钥解密，得到客户端发来的随机数 (pre-master)。

至此，**客户端和服务端双方都共享了三个随机数，分别是 Client Random、Server Random、pre-master**。

于是，双方根据已经得到的三个随机数，生成**会话密钥（Master Secret）**，它是对称密钥，用于对后续的 HTTP 请求/响应的数据加解密。

生成完会话密钥后，然后客户端发一个「**Change Cipher Spec**」，告诉服务端开始使用加密方式发送消息。

![图片](E:\渗透\docker环境搭建\44.png)

然后，客户端再发一个「**Encrypted Handshake Message（Finishd）**」消息，把之前所有发送的数据做个摘要，再用会话密钥（master secret）加密一下，让服务器做个验证，验证加密通信是否可用和之前握手信息是否有被中途篡改过。

![图片](E:\渗透\docker环境搭建\45.png)

可以发现，「Change Cipher Spec」之前传输的 TLS 握手数据都是明文，之后都是对称密钥加密的密文。

##### TLS 第四次握手

服务器也是同样的操作，发「**Change Cipher Spec**」和「**Encrypted Handshake Message**」消息，如果双方都验证加密和解密没问题，那么握手正式完成。

最后，就用「会话密钥」加解密 HTTP 请求和响应了。

#### ３，RSA算法的缺陷

**使用 RSA 密钥协商算法的最大问题是不支持前向保密**。因为客户端传递随机数（用于生成对称加密密钥的条件之一）给服务端时使用的是公钥加密的，服务端收到后，会用私钥解密得到随机数。所以一旦服务端的私钥泄漏了，过去被第三方截获的所有 TLS 通讯密文都会被破解。

为了解决这一问题，于是就有了 DH 密钥协商算法，这里简单介绍它的工作流程。

![图片](E:\渗透\docker环境搭建\46.png)

客户端和服务端各自会生成随机数，并以此作为私钥，然后根据公开的 DH 计算公示算出各自的公钥，通过 TLS 握手双方交换各自的公钥，这样双方都有自己的私钥和对方的公钥，然后双方根据各自持有的材料算出一个随机数，这个随机数的值双方都是一样的，这就可以作为后续对称加密时使用的密钥。

DH 密钥交换过程中，**即使第三方截获了 TLS 握手阶段传递的公钥，在不知道的私钥的情况下，也是无法计算出密钥的，而且每一次对称加密密钥都是实时生成的，实现前向保密**。

但因为 DH 算法的计算效率问题，后面出现了 ECDHE 密钥协商算法，我们现在大多数网站使用的正是 ECDHE 密钥协商算法。

