# Linux的Nginx 安装【转】

_2020-09-26_ _09:22_ 

转载自 [Nginx 安装配置 - 菜鸟教程](https://www.runoob.com/linux/nginx-install-setup.html)

## Nginx
Nginx("engine x")是一款是由俄罗斯的程序设计师Igor Sysoev所开发高性能的 Web和 反向代理 服务器，也是一个 IMAP/POP3/SMTP 代理服务器。

在高连接并发的情况下，Nginx是Apache服务器不错的替代品。

## Nginx 安装

一. 安装编译工具及库文件
```shell
yum -y install make zlib zlib-devel gcc-c++ libtool
```
二. 首先要安装 PCRE  
PCRE 作用是让 Nginx 支持 Rewrite 功能。

1. 下载 PCRE 安装包，下载地址： [http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz](http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz)
```shell
[root@bogon src]# cd /usr/local/src/
[root@bogon src]# wget http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz
```
2. 解压安装包:
```shell
[root@bogon src]# tar zxvf pcre-8.35.tar.gz
```
3. 进入安装包目录
```shell
[root@bogon src]# cd pcre-8.35
```
4. 编译安装 
```shell
[root@bogon pcre-8.35]# ./configure
[root@bogon pcre-8.35]# make && make install
```
5. 查看pcre版本
```shell
[root@bogon pcre-8.35]# pcre-config --version
```
三. 安装OpenSSL
1. 下载OpenSSL 安装包，下载地址：[https://www.openssl.org/source/openssl-1.1.1h.tar.gz](https://www.openssl.org/source/openssl-1.1.1h.tar.gz)
2. 解压安装包:
```shell
[root@bogon src]# tar -zxvf openssl-1.1.1h.tar.gz
```
3. 进入安装包目录
```shell
[root@bogon src]# cd openssl-1.1.1h
```
4. 编译安装 
```shell
[root@bogon openssl-1.1.1h]# ./config --prefix=/usr/local/openssl
[root@bogon openssl-1.1.1h]# make && make install
```
5. 查看OpenSSL版本
```shell
[root@bogon openssl-1.1.1h]# openssl version
```
6. 可能出现的问题
  1. 使用`openssl` 命令提示命令不存在：`openssl: command not found`

    解决方式：增加openssl的命令软连接，`ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl`
  2. 使用`openssl` 命令报错：`error while loading shared libraries: libssl.so.1.1: cannot open shared object file: No such file or directory`

    解决方式：
    ```shell
    echo "/usr/local/openssl/lib" >> /etc/ld.so.conf
    ldconfig
    ```

四. 安装 Nginx  
1. 下载 Nginx，下载地址：[http://nginx.org/download/nginx-1.6.2.tar.gz](http://nginx.org/download/nginx-1.6.2.tar.gz)
```shell
[root@bogon src]# cd /usr/local/src/
[root@bogon src]# wget http://nginx.org/download/nginx-1.6.2.tar.gz
```
 2. 解压安装包
```shell
[root@bogon src]# tar zxvf nginx-1.6.2.tar.gz
```
3. 进入安装包目录
```shell
[root@bogon src]# cd nginx-1.6.2
```
4. 编译安装
```shell
[root@bogon nginx-1.6.2]# ./configure --prefix=/usr/local/webserver/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/usr/local/src/pcre-8.35 --with-openssl=/usr/local/openssl
[root@bogon nginx-1.6.2]# make
[root@bogon nginx-1.6.2]# make install
```
5. 查看nginx版本
```shell
[root@bogon nginx-1.6.2]# /usr/local/webserver/nginx/sbin/nginx -v
```
6. 可能出现的问题
  1. make 的时候出现
  ```shell
  /bin/sh: line 2: ./config: No such file or directory
  make[1]: *** [/usr/local/openssl/.openssl/include/openssl/ssl.h] Error 127
  make[1]: Leaving directory `/usr/local/src/nginx-1.18.0'
  make: *** [build] Error 2
  ```
  路径错误导致。我们调整下nginx中openssl的配置`/usr/local/src/nginx-1.18.0/auto/lib/openssl/conf`，查找到如下配置
  ```shell
  CORE_INCS="$CORE_INCS $OPENSSL/.openssl/include"
  CORE_DEPS="$CORE_DEPS $OPENSSL/.openssl/include/openssl/ssl.h"
  CORE_LIBS="$CORE_LIBS $OPENSSL/.openssl/lib/libssl.a"
  CORE_LIBS="$CORE_LIBS $OPENSSL/.openssl/lib/libcrypto.a"
  ```
  调整为
  ```shell
  CORE_INCS="$CORE_INCS $OPENSSL/include"
  CORE_DEPS="$CORE_DEPS $OPENSSL/include/openssl/ssl.h"
  CORE_LIBS="$CORE_LIBS $OPENSSL/lib/libssl.a"
  CORE_LIBS="$CORE_LIBS $OPENSSL/lib/libcrypto.a"
  ```
  即删除`.openssl/`

到此，nginx安装完成。

## Nginx 配置
创建 Nginx 运行使用的用户 www：
```shell
[root@bogon conf]# /usr/sbin/groupadd www 
[root@bogon conf]# /usr/sbin/useradd -g www www
```
配置nginx.conf ，将/usr/local/webserver/nginx/conf/nginx.conf替换为以下内容
```shell
[root@bogon conf]#  cat /usr/local/webserver/nginx/conf/nginx.conf

user www www;
worker_processes 2; #设置值和CPU核心数一致
error_log /usr/local/webserver/nginx/logs/nginx_error.log crit; #日志位置和日志级别
pid /usr/local/webserver/nginx/nginx.pid;
#Specifies the value for maximum file descriptors that can be opened by this process.
worker_rlimit_nofile 65535;
events
{
  use epoll;
  worker_connections 65535;
}
http
{
  include mime.types;
  default_type application/octet-stream;
  log_format main  '$remote_addr - $remote_user [$time_local] "$request" '
               '$status $body_bytes_sent "$http_referer" '
               '"$http_user_agent" $http_x_forwarded_for';
  
#charset gb2312;
     
  server_names_hash_bucket_size 128;
  client_header_buffer_size 32k;
  large_client_header_buffers 4 32k;
  client_max_body_size 8m;
     
  sendfile on;
  tcp_nopush on;
  keepalive_timeout 60;
  tcp_nodelay on;
  fastcgi_connect_timeout 300;
  fastcgi_send_timeout 300;
  fastcgi_read_timeout 300;
  fastcgi_buffer_size 64k;
  fastcgi_buffers 4 64k;
  fastcgi_busy_buffers_size 128k;
  fastcgi_temp_file_write_size 128k;
  gzip on; 
  gzip_min_length 1k;
  gzip_buffers 4 16k;
  gzip_http_version 1.0;
  gzip_comp_level 2;
  gzip_types text/plain application/x-javascript text/css application/xml;
  gzip_vary on;
 
  #limit_zone crawler $binary_remote_addr 10m;
 #下面是server虚拟主机的配置
 server
  {
    listen 80;#监听端口
    server_name localhost;#域名
    index index.html index.htm index.php;
    root /usr/local/webserver/nginx/html;#站点目录
      location ~ .*\.(php|php5)?$
    {
      #fastcgi_pass unix:/tmp/php-cgi.sock;
      fastcgi_pass 127.0.0.1:9000;
      fastcgi_index index.php;
      include fastcgi.conf;
    }
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|ico)$
    {
      expires 30d;
  # access_log off;
    }
    location ~ .*\.(js|css)?$
    {
      expires 15d;
   # access_log off;
    }
    access_log off;
  }

}
```
检查配置文件nginx.conf的正确性命令：
```shell
[root@bogon conf]# /usr/local/webserver/nginx/sbin/nginx -t
```
## 启动 Nginx
Nginx 启动命令如下：
```shell
[root@bogon conf]# /usr/local/webserver/nginx/sbin/nginx
```
访问站点  
从浏览器访问我们配置的站点ip：

## Nginx 其他命令
以下包含了 Nginx 常用的几个命令：
```shell
/usr/local/webserver/nginx/sbin/nginx -s reload            # 重新载入配置文件  
/usr/local/webserver/nginx/sbin/nginx -s reopen            # 重启 Nginx  
/usr/local/webserver/nginx/sbin/nginx -s stop              # 停止 Nginx
```