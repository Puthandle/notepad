# Nginx使用总结，体系化带你全面认识 Nginx ！

<img src="./pic/nginx.png" alt="nginx logo" style="zoom:100%;" />

## 概述：
> Nginx 是开源、高性能、高可靠的 Web 和反向代理服务器。

## 使用场景：

> 1. 静态资源托管，通过本地文件系统提供服务；
> 2. 反向代理服务，延伸出包括缓存、负载均衡等；

## 如何使用：

### 安装

> 本文是基于 Linux centOS 7.x 的操作系统上安装 Nginx ，至于在其它操作系统上进行安装可以网上自行搜索。

```shell
yum info nginx
yum install nginx
yum install nginx-all-modules
```

### 启动与停止

```shell
systemctl start nginx ## 启动nginx
systemctl stop nginx  ## 停止nginx
systemctl status nginx  ## 查看nginx状态
systemctl enable nginx ## 开机自启动
systemctl disable nginx ## 关闭开机自启动
```

### 核心配置：

```nginx
# main段配置信息
user  nginx;                        		# 运行用户，默认即是nginx，可以不进行设置
worker_processes  auto;             		# Nginx 进程数，一般设置为和 CPU 核数一样
error_log  /var/log/nginx/error.log warn;    # Nginx 的错误日志存放目录
pid        /var/run/nginx.pid;      		# Nginx 服务启动时的 pid 存放位置
# events段配置信息
events {
    use epoll;     				# 使用epoll的I/O模型(如果你不知道Nginx该使用哪种轮询方法，会自动选择一个最适合你操作系统的)
    worker_connections 1024;   	 # 每个进程允许最大并发数
}
# http段配置信息
# 配置使用最频繁的部分，代理、缓存、日志定义等绝大多数功能和第三方模块的配置都在这里设置
http { 
    # 设置日志模式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;   # Nginx访问日志存放位置

    sendfile            on;   # 开启高效传输模式
    tcp_nopush          on;   # 减少网络报文段的数量
    tcp_nodelay         on;
    keepalive_timeout   65;   # 保持连接的时间，也叫超时时间，单位秒
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;      # 文件扩展名与类型映射表
    default_type        application/octet-stream;   # 默认文件类型
    include /etc/nginx/conf.d/*.conf;   # 加载子配置项

    # server段配置信息
    server {
    	listen       80;       	   # 配置监听的端口
    	server_name  localhost;    # 配置的域名
      
    	# location段配置信息
    	location / {
    		root   /usr/share/nginx/html;  # 网站根目录
    		index  index.html index.htm;   # 默认首页文件
    		deny 172.168.22.11;   # 禁止访问的ip地址，可以为all
    		allow 172.168.33.44；# 允许访问的ip地址，可以为all
    	}
    	error_page 500 502 503 504 /50x.html;  # 默认50x对应的访问页面
    	error_page 400 404 error.html;   	   # 同上
    }
}
```

#### 参考文章 https://juejin.cn/post/6942607113118023710

## 配置详解：

###  文件主要结构
```nginx
user  nginx;                        		# 运行用户，默认即是nginx，可以不进行设置
worker_processes  auto;             		# Nginx 进程数，一般设置为和 CPU 核数一样
error_log  /var/log/nginx/error.log warn;   # Nginx 的错误日志存放目录
pid        /var/run/nginx.pid;      		# Nginx 服务启动时的 pid 存放位置
# events段配置信息
events {
    worker_connections 1024;   	 # 每个进程允许最大并发数
}
# http段配置信息
http { 
	...
    # server段配置信息
    server {
		...
    }
}
# stream 模块配置信息
# stream模块默认没有编译到nginx， 编译nginx时候 ./configure –with-stream 即可
stream {
	...
    # server段配置信息
    server {
    	...
    }
}
```
### Http配置结构
```nginx
# http段配置信息
http { 
    # 设置日志模式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;   # Nginx访问日志存放位置

    sendfile            on;   # 开启高效传输模式
    tcp_nopush          on;   # 减少网络报文段的数量
    tcp_nodelay         on;
    keepalive_timeout   65;   # 保持连接的时间，也叫超时时间，单位秒
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;      # 文件扩展名与类型映射表
    default_type        application/octet-stream;   # 默认文件类型
    include /etc/nginx/conf.d/*.conf;   # 加载子配置项
    # server段配置信息
    server {
    	...
    }
}


```

#### Server配置结构

    server {
        listen       80;
        server_name  rabbit.development.sxtkj.site;
        index index.html index.htm;
        location / {
            proxy_pass http://127.0.0.1:15672;
        }
        error_page 404 /404.html;
        location = /404.html {
        }
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }

