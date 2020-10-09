## nginx 
### 1. 简介
一个高性能，轻量级的web服务器，可做反向代理，负载均衡。

注：
- Web服务器如nginx，apache
- 应用服务器如tomcat，resin

### 2. 正向代理与反向代理
![image.png](https://upload-images.jianshu.io/upload_images/6123292-0945a48c19dae03c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**总结**

- 正向是代理客户端，比如用户用vpn访问服务器，服务器不用知道客户端是如何请求服务器的。
- 反向代理是代理服务端，将请求转发给某一台服务器。
### 3. 同apache，resin对比
##### 1. 同apache
- nginx是非阻塞性异步，多个连接对应一个进程；apache是阻塞性异步，一个连接对应一个进程。
- nginx体积小，CPU内存占用低
- nginx性能好，抗并发
- nginx相对apache的漏洞会多一些，apache更为成熟
- apache的rewrite和处理动态资源更有优势

https://cloud.tencent.com/developer/article/1501191
##### 2. 同resin
- resin本身包含了一个支持http/1.1的web服务器，因此可以处理静态和动态的内容；nginx主要用做反向代理服务器以及处理负载均衡
- resin商用，小额费用；nginx开源免费

https://www.caucho.com/resin-4.0/admin/http-server.xtp

### 4. nginx安装及配置
##### 1. 安装
```
wget http://nginx.org/download/nginx-1.15.11.tar.gz  #下载nginx
tar -zxvf nginx-1.15.11,tar,gz    #解压
./configure --prefix=[path]       #配置nginx存储路径
make && make install              #编译安装
```
##### 2. nginx配置
```
# 配置文件 nginx/conf/nginx.conf

...              #全局块 配置影响nginx全局的指令，如worker process树等

events {         #events块 配置网络连接相关
   ...
}

http      #http块 可嵌套多个server，配置代理、缓存、日志等
{
    ...   #http全局块
    server        #server块 配置虚拟主机参数
    { 
        ...       #server全局块
        location [PATTERN]   #location块 配置请求的路由，及各个页面的处理情况
        {
            ...
        }
        location [PATTERN] 
        {
            ...
        }
    }
    server
    {
      ...
    }
    ...     #http全局块
}
```
##### 3. nginx 基本操作
```
nginx -s start    #直接nginx也可以
nginx -s reload
nginx -s stop     # 快速停止，不管有没有正在处理的连接请求
nginx -s quit     # 完整有序的停止
```
### 5. 举例
```
# nginx作为web服务器
server {
    listen 8080;
    root /data/up1;
    location / {
    }
}
# nginx作为反向代理
server {
    location / {
        proxy_pass http://localhost:8080;
    }
    location /images/ {
        root /data;
    }
}
```