---
title: JW-Nginx
date: 2019.04.08 17:17:22
tags: JW
categories: JW
---


Nginx (engine x) 是一个高性能支持高并发的HTTP和反向代理服务。
应用场景：
1、http服务器。可以独立提供http服务。可以做网页静态服务器。
2、虚拟主机。可实现一台服务器虚拟出多个网站。
3、反向代理，负载均衡。多台服务器集群使用nginx做反向代理，是多台服务器负载均衡，不会因为某台服务器负载过高宕机而某台服务器闲置的情况。


**配置虚拟主机**
nginx配置文件位于nginx目录下的conf文件夹下
![Nginx](/images/Nginx/Nginx.jpg)

server{}块每 部分就代表每一个web站点
根据端口区分：固定域名为loaclhost
![Nginx1](/images/Nginx/Nginx1.jpg)
根据域名区分：默认端口80，修改hosts文件同一IP对应不同域名
![Nginx2](/images/Nginx/Nginx2.jpg)
![Nginx3](/images/Nginx/Nginx3.jpg)


**反向代理**（相对于服务端，接收请求分配到响应服务器）
![Nginx4](/images/Nginx/Nginx4.jpg)

正向代理（相对于客户端）
![Nginx5](/images/Nginx/Nginx5.jpg)

```
    upstream sina{
       server 192.168.25.158:8080  #同一服务器不同端口 tomcat上部署
    }
    server {
        listen       80;
        server_name  www.sina.com.cn;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            proxy_pass   http://sina
            index  index.html index.htm;
        }
    }

    upstream sohu{
        server 192.168.25.158:8081  #同一服务器不同端口 xtomcat上部署
     }
    server {
        listen       80;
        server_name  www.sohu.com;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            proxy_pass   http://sohu
            index  index.html index.htm;
        }
    }
```

**负载均衡**

```
    upstream sohu{
        server 192.168.25.158:8081           #tomcat上部署
        server 192.168.25.158:8082 weight=2  #如果一个服务由多条服务器提供，实现负载均衡。weight越高，权重分配的请求越多
     }
    server {
        listen       80;
        server_name  www.sohu.com;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            proxy_pass   http://sohu
            index  index.html index.htm;
        }
    }
```

**Nginx实现高可用**
![Nginx6](/images/Nginx/Nginx6.jpg)

Keepalived+Nginx实现主备
![Nginx7](/images/Nginx/Nginx7.jpg)

VIP虚拟IP动态地指定主备服务器
![Nginx8](/images/Nginx/Nginx8.jpg)

其他更高并发、高可用服务 ：LVS、F5
