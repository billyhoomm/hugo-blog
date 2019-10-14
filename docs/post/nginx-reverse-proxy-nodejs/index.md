
> 前言

空余时间学习使用vue.js+webpack制作了一个简单的nodejs网站，web服务器使用express构建，而个人的云服务器是部署的LNMP环境，已经搭建了一个wordpress网站，为了不影响wordpress的运行遂google到了使用nginx反向代理的方法，非常好用。不过也可以使用nodejs的[http-proxy](https://github.com/nodejitsu/node-http-proxy)模块，感兴趣可以查看一下，我对这个也不是很明了，不作介绍。

## 一、反向代理原理
先看图：
![](http://cdn.billyh.cn/blogUploads/1511704841000.jpeg)
对上图的理解是：

**正向代理**：在客户这一端，替客户收发请求。

**反向代理**：在服务器机房这一端，替服务器收发请求，也就是说请求和响应都先经过反向代理。具有缓存、安全、负载均衡等作用。

## 二、Nginx核心配置文件nginx.conf解释

(如无特别需要并且不懂nginx一般可使用自动配置)

**1、进程数与每个进程的最大连接数**：

- nginx进程数，建议设置为等于CPU总核心数
- 单个进程最大连接数，那么该服务器的最大连接数=连接数*进程数

```nginx
#工作进程个数，一般跟服务器cpu核数相等，或者核数的两倍
worker_processes 2;

#单个进程最大连接数
events{
    worker_connections 1024; 
}
```

**2、Nginx的基本配置**：

```nginx
#nginx基本配置
server{
    listen 8088;    #端口号
    server_name 192.168.22.227; #服务名
}
```

**3、负载均衡列表基本配置**：
- location / {}：对aspx后缀的进行负载均衡请求，假如我们要对所有的aspx后缀的文件进行负载均衡时，可以这样写：location ~ .*\.aspx$ {}
- proxy_pass：请求转向自定义的服务器列表，这里我们将请求都转向标识为http://cuitccol.com的负载均衡服务器列表；

```nginx
    #服务器集群
    upstream mycluster{
        #这里添加的是上面启动好的两台Tomcat服务器
         server 192.168.22.229:8080 weight=1;
         server 192.168.22.230:8080 weight=1;
    }

    location /{
        #将访问请求转向至服务器集群,mycluster和上面upstream mycluster 对应
        proxy_pass http://mycluster;
        # 真实的客户端IP
        proxy_set_header   X-Real-IP        $remote_addr; 
        # 请求头中Host信息
        proxy_set_header   Host             $host; 
        # 代理路由信息，此处取IP有安全隐患
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        # 真实的用户访问协议
        proxy_set_header   X-Forwarded-Proto $scheme;
    }
```

- 在负载均衡服务器列表的配置中，weight是权重，可以根据机器配置定义权重（如果某台服务器的硬件配置十分好，可以处理更多的请求，那么可以 为其设置一个比较高的weight；而有一台的服务器的硬件配置比较差，那么可以将前一台的weight配置为weight=2，后一台差的配置为 weight=1）。weigth参数表示权值，权值越高被分配到的几率越大；

**4、Nginx对于静态文件的缓存配置**

```nginx
server
    {
        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
        {
            # 过期时效
            expires      30d;
        }

        location ~ .*\.(js|css)?$
        {
            expires      12h;
        }

        location ~ /\.
        {
            deny all;
        }
    }
```

**5、虚拟主机配置**
```nginx
 include /etc/nginx/conf.d/*.conf;
```
（如果没有此配置需添加至http配置项的末尾）

## 三、反向代理配置
如图(如果没有此文件夹可新建一个conf.d文件夹)：
![](http://cdn.billyh.cn/blogUploads/1511630769000.jpeg)
在此文件夹中新建虚拟主机文件，命名为*.conf(*为自定义)，加入下列基本配置

```nginx
upstream blog {
    server 127.0.0.1:8081; # 这里的端口号写你node.js运行的端口号，也就是要代理的端口号，我的项目跑在8081端口上
    keepalive 64;
}

upstream www {
    server 127.0.0.1:8080; # 这里的端口号写你node.js运行的端口号，也就是要代理的端口号，我的项目跑在8081端口上
    keepalive 64;
}

server {
    listen 80; #这里的端口号是你要监听的端口号
    server_name blog.billyh.cn; # 这里是你的服务器名称，也就是别人访问你服务的ip地址或域名，可以写多个，用空格隔开

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-Nginx-Proxy true;
        proxy_set_header Connection "";
        proxy_pass http://blog; # 这里要和最上面upstream后的应用名一致，可以自定义
    }
}

server {
    listen 80; #这里的端口号是你要监听的端口号
    server_name www.billyh.cn; # 这里是你的服务器名称，也就是别人访问你服务的ip地址或域名，可以写多个，用空格隔开

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-Nginx-Proxy true;
        proxy_set_header Connection "";
        proxy_pass http://www; # 这里要和最上面upstream后的应用名一致，可以自定义
    }
}
```

server_name 代表需要绑定的域名

proxy_pass 设置服务器上需要代理的程序的地址（在本例中是由express监听的端口）

listen 设置 nginx 监听端口，默认80

至此就算配置成功了，只需重启nginx即可生效

## 四、nginx基本指令(基于centos6.5)
启动：
```nginx
nginx -c /etc/nginx/（配置文件路径）nginx.conf
``` 

停止：
- 查询nginx主进程号：
```
ps -ef | grep nginx
```
- 从容停止Nginx
```
kill QUIT 主进程号
```

重启：
```
kill -HUP 主进称号或进程号文件路径
```
或者
 ```
/usr/nginx/sbin/（nginx路径）nginx -s reload
```
## 五、关于反向代理的Gzip压缩配置
在使用Nginx反向代理nodejs时最好的压缩方法时在Nginx端启动Gzip，如下nginx.conf配置
```nginx
        gzip on;
        gzip_min_length  1k;
        gzip_buffers     4 16k;
        gzip_http_version 1.1;
        gzip_comp_level 4;
        gzip_types     text/plain application/javascript application/x-javascript text/javascript text/css application/xml application/xml+rss;
        gzip_vary on;
        gzip_proxied   expired no-cache no-store private auth;
        gzip_disable   "MSIE [1-6]\.";
```
**另：**

我个人网站在启动Nginx的Gzip之后，访问nodejs站的时候经常出现刷新的时候响应缓慢，于是我在nodejs上也配置了压缩，一下是以express为前提的配置：
- 首先安装compression依赖
```
npm install compression
```
- 在app.js中添加相应配置：
```js
var express = require('express');
var compression = require('compression');
var app = express();
app.use(compression());
```
在两边都开启压缩的情况下，响应变得正常了，但是却与很多官方的说法相悖，官方说法是在开发过程中只需要配置Nginx端的Gzip，nodejs端不用compression，所以我疑惑的这个问题目前还没想明白。
> PS：后续还会继续测试，如果有新的结果，会继续补上。