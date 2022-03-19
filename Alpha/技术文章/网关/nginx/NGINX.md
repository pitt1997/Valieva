## Nginx基本概念

nginx官网：https://www.runoob.com/linux/nginx-install-setup.html

视频教学：https://www.bilibili.com/video/BV1zJ411w7SV?p=5&spm_id_from=pageDriver

## Nginx安装

### 一、安装编译工具及库文件

```
yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
```

### 二、安装 PCRE

PCRE 作用是让 Nginx 支持 Rewrite 功能。

1、下载 PCRE 安装包，下载地址： http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz

```
[root@localhost src]# cd /usr/local/src/
[root@localhost src]# wget http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz
```

**-bash: wget: command not found**  **需要wget软件包**

```
yum -y install wget
```

2、解压安装包

```
[root@localhost src]# tar zxvf pcre-8.35.tar.gz
```

3、进入安装包目录

```
[root@bogon src]# cd pcre-8.35
```

4、编译安装

```
[root@localhost pcre-8.35]# ./configure
[root@localhost pcre-8.35]# make && make install
```

5、查看pcre版本

```
[root@localhost pcre-8.35]# pcre-config --version
```

### 三、安装 Nginx

1、下载 Nginx，下载地址：https://nginx.org/en/download.html

```
[root@localhost src]# cd /usr/local/src/
[root@localhost src]# wget http://nginx.org/download/nginx-1.6.2.tar.gz
```

2、解压安装包

```
[root@localhost src]# tar zxvf nginx-1.6.2.tar.gz
```

3、进入安装包目录

```
[root@localhost src]# cd nginx-1.6.2
```

4、编译安装

```
[root@localhost nginx-1.6.2]# ./configure --prefix=/usr/local/webserver/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/usr/local/src/pcre-8.35
[root@localhost nginx-1.6.2]# make
[root@localhost nginx-1.6.2]# make install
```

5、查看nginx版本

```
[root@localhost nginx-1.6.2]# /usr/local/webserver/nginx/sbin/nginx -v
```

到此，nginx安装完成。

## Nginx 配置

创建 Nginx 运行使用的用户 www：

```
[root@localhost nginx-1.6.2]# /usr/sbin/groupadd www 
[root@localhost nginx-1.6.2]# /usr/sbin/useradd -g www www
```

配置nginx.conf ，将/usr/local/webserver/nginx/conf/nginx.conf替换为以下内容

```
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

> 检查配置文件nginx.conf的正确性命令

```
[root@localhost nginx-1.6.2]# /usr/local/webserver/nginx/sbin/nginx -t
```

> 配置成功

```
nginx: the configuration file /usr/local/webserver/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/webserver/nginx/conf/nginx.conf test is successful
```

### 全局块

设置一些影响nginx服务器整体运行的配置指令，主要包括配置运行nginx服务器的用户（组）、允许生成的work process数，进程PID存放路径、日志存放路径和类型以及配置文件的引入等。

### events块

events块涉及的指令主要影响nginx服务器与用户的网络连接，常用的设置包括是否开启对多work process下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个word process可以同时支持的最大连接数等。

```
events
{
  use epoll;
  worker_connections 65535;
}
```

### http块

http全局块配置的指令包括文件引入、MIME-TYPE定义、日志自定义、连接超时时间、单链接请求数上限等。

### server块

这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器的硬件成本。

每个http块可以包括多个server块，而每个server块就相当于一个虚拟主机。每个server块也分为全局server块，以及可以同时包含多个location块。

#### 全局server块

最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或IP配置。

#### location块

一个server块可以配置多个location块

这块的主要作用是基于nginx服务器接收到的请求字符串（例如server_name/uri-string），对虚拟主机名称（也可以是IP别名）之外的字符串进行匹配，对特定的请求进行处理。地址定向，数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。

##### location正则匹配

1、=：用于不含正则表达式的rui前，要求请求的字符串与uri严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求。

2、~：用于表示uri包含正则表达式，并且区分大小写

3、~*：用于表示uri包含正则表达式，并且不区分大小写

4、^~：用于不含正则表达式的uri前，要求nginx服务器找到标识uri和请求字符串匹配度最高的location后，立即使用此location处理请求，而不再使用location块中的正则uri和请求字符串做匹配。

注意：如果uri包含正则表达式，则必须要有~或者~*标识

## Nginx 命令

进入nginx目录 /usr/local/webserver/nginx/sbin

1、查看nginx版本号

```
[root@localhost sbin]# ./nginx -v
```

2、启动nginx

```
[root@localhost sbin]# ./nginx
```

检查是否启动成功

```
[root@localhost nginx-1.6.2]# ps -ef|grep nginx
root      24392      1  0 09:07 ?        00:00:00 nginx: master process /usr/local/webserver/nginx/sbin/nginx
www       24393  24392  0 09:07 ?        00:00:00 nginx: worker process
www       24394  24392  0 09:07 ?        00:00:00 nginx: worker process
root      24396  18999  0 09:08 pts/2    00:00:00 grep --color=auto nginx
```

3、关闭nginx

```
[root@localhost sbin]# ./nginx -s stop
```

4、重新加载nginx （nginx.conf配置文件修改需要重新加载）

```
[root@localhost sbin]# ./nginx -s reload
```

5、重启 Nginx

```
/usr/local/webserver/nginx/sbin/nginx -s reopen
```

## Nginx 配置好后不能访问

### iptables防火墙命令

配置好nginx之后，通过外部不能访问，防火墙可以ping通虚拟机，虚拟机也可以ping通防火墙。接着检查了服务器端的80端口是否可以访问的到：telnet 192.168.131.130 80， 结果访问不到，原来果真防火墙的问题。 

> 放开80端口

```
[root@localhost html]# /sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT
[root@localhost html]# /etc/init.d/iptables save
[root@localhost html]# /etc/init.d/iptables restart
```

> 查看CentOS防火墙信息

    /etc/init.d/iptables status  
>  关闭CentOS防火墙服务

```
/etc/init.d/iptables stop  
```

### firewall-cmd防火墙命令(centos 7)

> 查看开放的端口号

```
firewall-cmd --list-all
```

> 设置开放的端口号

```
firewall-cmd --add-service=http --permanent
sudo firewall-cmd --add-port=80/tcp --permanent
```

> 重启防火墙

```
firewall-cmd --reload
```

> 参考文章

```
https://blog.csdn.net/haitun312366/article/details/8511475
```

## Nginx配置实例

### 反向代理

#### 什么是代理

代理其实就是一个中介，A和B本来可以直连，中间插入一个C，C就是中介。刚开始的时候，代理多数是帮助内网client访问外网server用的，后来出现了反向代理，"反向"这个词在这儿的意思其实是指方向相反，即代理将来自外网客户端的请求转发到内网服务器，从外到内。

#### 正向代理

正向代理类似一个跳板机，代理访问外部资源，比如我们国内访问谷歌，直接访问访问不到，我们可以通过一个正向代理服务器，请求发到代理服，代理服务器能够访问谷歌，这样由代理去谷歌取到返回数据，再返回给我们，这样我们就能访问谷歌。

![1623165823846](C:\Users\17996\AppData\Roaming\Typora\typora-user-images\1623165823846.png)

**正向代理的用途：**

（1）访问原来无法访问的资源，如google

（2） 可以做缓存，加速访问资源

（3）对客户端访问授权，上网进行认证

（4）代理可以记录用户访问记录（上网行为管理），对外隐藏用户信息

#### 反向代理

反向代理（Reverse Proxy）实际运行方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。

![1623165907703](C:\Users\17996\AppData\Roaming\Typora\typora-user-images\1623165907703.png)

**反向代理的作用：**

（1）保证内网的安全，阻止web攻击，大型网站，通常将反向代理作为公网访问地址，Web服务器是内网

（2）负载均衡，通过反向代理服务器来优化网站的负载

**正向代理即是客户端代理, 代理客户端, 服务端不知道实际发起请求的客户端；反向代理即是服务端代理, 代理服务端, 客户端不知道实际提供服务的服务端**



#### 实现效果

打开浏览器，地址栏输入www.123.com，跳转至linux系统tomcat主页面

#### 具体实现

1、在linux安装tomcat或者使用已经安装好的tomcat

2、配置域名host映射

```
C:\Windows\System32\drivers\etc\hosts
```

3、/usr/local/webserver/nginx/conf/nginx.conf

```
server{
	listen 80;
	server_name 192.168.1.100;
    location / {
        root html;
        proxy_pass http://127.0.0.1:8080;
        index index.html index.htm;
    }
}
```

#### 实现效果

使用nginx反向代理，根据访问路径跳转到不同端口的服务中，nginx监听端口为9001;访问http://127.0.0.1:9001/edu/ 直接跳转至127.0.0.1:8080；访问http://127.0.0.1:9001/vod/ 直接跳转至127.0.0.1:8082 

#### 具体实现

nginx配置文件配置/usr/local/webserver/nginx/conf/nginx.conf   新增一个server

```
server{
	listen 9001;
	server_name 192.168.1.100;
    location ~ /edu/ {  #正则写法
        proxy_pass http://127.0.0.1:8080;
    }
    location ~ /vod/ {
        proxy_pass http://127.0.0.1:8082;
    }
}
```

### 负载均衡

#### 实现效果

浏览器地址栏输入地址 http://192.168.1.100/edu/a.txt，负载均衡效果，平均到8080和8082端口中

#### 具体实现

负载均很配置 在http块中新加

```
http{
......
	upstream myserver{
		server 192.168.1.100:8080 weight=1;
		server 192.168.1.100:8082 weight=1;
	}

}

server
  {
    listen 80;#监听端口
    #server_name localhost;#域名
    server_name 192.168.1.100;

   #location / {
    #    root html;
    #    proxy_pass http://127.0.0.1:8080;
    #    index index.html index.htm;
    #}

    location / {
        root html;
        proxy_pass http://myserver;
        index index.html index.htm;
    }
}

```

#### 分配策略

##### 1、轮询（默认）

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。

##### 2、weight

weight代表权重，权重默认为1，权重越高被分配的客户端越多；指定轮询几率。weight和访问比率成正比，用于后端服务器性能不均的情况。

```
upstream myserver {
		ip_hash;
		server 192.168.1.100:8080 weight=5;
		server 192.168.1.100:8082 weight=10;
}
```



##### 3、ip_hash

每个请求按访问ip的hash结果分配，这样每个访客固定一个后端服务器，可以解决session的问题。第一次访问某台机器之后后面都是访问这台机器

```
upstream myserver {
		ip_hash;
		server 192.168.1.100:8080;
		server 192.168.1.101:8080;
}
```



##### 4、fair（第三方）

按后端服务器的响应时间来分配请求，响应时间短的优先分配。

```
upstream myserver {
		fair;
		server 192.168.1.100:8080;
		server 192.168.1.100:8081;
}
```

访问8080和8081谁响应时间短那么返回最快响应。

### 动静分离

动态请求和静态资源没有一起部署

###  高可用集群

环境准备

> keepalived安装

```
yum install keepalived -y
```

> 配置文件keepalived.conf

```
/etc/keepalived/keepalived.conf
```



## Nginx配置实例 



## Nginx原理





## 相关资料

美团点评高性能四层负载均衡
