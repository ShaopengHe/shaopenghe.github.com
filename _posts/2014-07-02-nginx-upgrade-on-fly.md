---
title: Nginx 平滑升级
layout: post
tags:
    - nginx
---

Nginx作为Web服务器的后起之秀，因其显著的性能，越来越受到企业的青睐，众多大型网站都从Apache迁移到Nginx，淘宝更是在其基础上发起了[Tengine](http://tengine.taobao.org/)开源项目，可见Nginx在Web服务器中地位之重。

Web服务器作为用户请求后端服务器的入口，不单只负责建立连接，处理请求，还承担着“守门卫”的安全职责。从安全角度来说，Nginx本身的安全性十分重要，因此必须时刻关注Nginx的安全升级补丁。

本篇文章主要介绍“如何在不影响用户访问的情况下平滑升级Nginx”。

----------

下载nginx最新稳定版， 这里以0.7升级到1.4.1为例： [[下载地址]](http://nginx.org/download/)

```bash
$ wget "http://nginx.org/download/nginx-1.4.1.tar.gz"
$ tar zxvf nginx-1.4.1.tar.gz   #解压
```

查看当前已安装的nginx版本的编译参数（configure arguments）：

```bash
$ nginx -V
nginx version: nginx/0.7.67
TLS SNI support enabled
configure arguments: --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log 
--http-client-body-temp-path=/var/lib/nginx/body 
--http-fastcgi-temp-path=/var/lib/nginx/fastcgi 
--http-log-path=/var/log/nginx/access.log 
--http-proxy-temp-path=/var/lib/nginx/proxy 
--lock-path=/var/lock/nginx.lock 
--pid-path=/var/run/nginx.pid 
--with-debug --with-http_dav_module --with-http_flv_module 
--with-http_geoip_module --with-http_gzip_static_module 
--with-http_realip_module --with-http_stub_status_module 
--with-http_ssl_module --with-http_sub_module 
--with-ipv6 --with-mail --with-mail_ssl_module 
--add-module=/build/buildd/nginx-0.7.67/modules/nginx-upstream-fair
```

根据旧版本的编译参数重新编译新版本：

```bash
$ cd nginx-1.4.1

$ ./configure --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log 
--http-client-body-temp-path=/var/lib/nginx/body 
--http-fastcgi-temp-path=/var/lib/nginx/fastcgi 
--http-log-path=/var/log/nginx/access.log 
--http-proxy-temp-path=/var/lib/nginx/proxy
--lock-path=/var/lock/nginx.lock
--pid-path=/var/run/nginx.pid
--with-debug --with-http_dav_module --with-http_flv_module 
--with-http_geoip_module --with-http_gzip_static_module 
--with-http_realip_module --with-http_stub_status_module 
--with-http_ssl_module --with-http_sub_module 
--with-ipv6 --with-mail --with-mail_ssl_module

$ make      #编译，编译后这里不要执行make install，后面手动安装
```

P.S: 编译过程中可能会出现缺少xxx libary的情况， 可参考[这遍文章](http://blog.linuxphp.org/archives/1430/)来安装缺少的lib。

编译完成后，替换 nginx 二进制文件。

```bash
$ sudo mv /usr/sbin/nginx  /usr/sbin/nginx.old  #备份旧文件， 具体路径可能有所不同， 可以通过命令which nginx查看）
$ sudo cp objs/nginx /usr/sbin/nginx    #替换为新编译的nginx
$ sudo /usr/sbin/nginx -t               #测试是否正常
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: [emerg] mkdir() "/usr/local/nginx/uwsgi_temp" failed (2: No such file or directory)
nginx: configuration file /etc/nginx/nginx.conf test failed
```
提示/usr/local/nginx目录不存在，创建该目录：

```
$ sudo mkdir /usr/local/nginx
```

再次测试，一切正常：

```bash
$ sudo /usr/sbin/nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```



启动新进程，关闭旧进程：

```
$ sudo kill -USR2 `cat /var/run/nginx.pid` #pid的文件路径可以参考第一步 nginx -V 的pid-path
```

执行完上一步后，nginx会把 *nginx.pid* 备份成 *nginx.pid.oldbin*，并启动新的nginx进程接收用户请求。

平滑关闭旧进程：

```
$ sudo kill -QUIT `cat /var/run/nginx.pid.oldbin`  
```

**升级完成：**

```
$ nginx -V
nginx version: nginx/1.4.1
built by gcc 4.4.7 (Ubuntu/Linaro 4.4.7-1ubuntu2) 
TLS SNI support enabled
configure arguments: --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-log-path=/var/log/nginx/access.log --http-proxy-temp-path=/var/lib/nginx/proxy --lock-path=/var/lock/nginx.lock --pid-path=/var/run/nginx.pid --with-debug --with-http_dav_module --with-http_flv_module --with-http_gzip_static_module --with-http_realip_module --with-http_stub_status_module --with-http_ssl_module --with-http_sub_module --with-ipv6 --with-mail --with-mail_ssl_module
```

建议在整个升级过程中，运行一个脚本不断访问升级的网站，检测是否会出现服务中断的现象。

```
#!/bin/bash

while [ 0 ]
do
    curl -I 'http://localhost:80/'
    sleep 1
done
```


附其他参考资料：
>http://blog.sina.com.cn/s/blog_537977e50100i1hz.html
>
>http://weizhifeng.net/nginx-signal-processing-and-upgrade.html
>
>http://www.vpsee.com/2011/03/install-nginx-with-geoip-module-for-country-targeting/
>
>http://blog.linuxphp.org/archives/1430/

