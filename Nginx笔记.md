# Nginx笔记

## 信号量

kill -INT 进程号

#### 信号量相关参数

- TERM,INT                   Quick shutdown
- QUIT  							优雅的关闭进程，即等请求结束后关闭
- HUP 							修改配置文件后，热加载配置文件
- USR1  							重读日志,在日志按月/日分割时用
- USR2							平滑升级
- WINCH 						优雅关闭旧的进程，(配合USR2来进行升级)

## 虚拟机配置

```properties
#全局变量

//有一个工作的子进程,可以自行修改。一般设置为CPU数×核心数
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

#一般配置nginx连接的特性
events {
	//一个worker能同时允许多少连接
    worker_connections  1024;　#这是指定一个子进程最大允许1024个连接
}

#配置http服务器的主要段
http {
    include       mime.types;
    default_type  application/octet-stream;


    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        #server样例
        server{
        	listen 81;
        	server_name test.com;
        	
        	location / {
        		#此处test.com为路径名称test.com是相对路径，也可以写绝对路径
        		root test.com;
        		index index.html;
        	}
        }
    }
}
```

## 日志管理

​	nginx允许针对不同的server做不同的log

​	nginx的server段中有如下信息：

```properties
#access_log  logs/host.access.log  main;
```

这说明该虚拟主机(server),它的访问日志文件在　llogs/host.access.log　使用的格式是"main"格式。

除了main格式，还可以自定义其他格式

nginx日志的main格式(nginx默认日志输出格式):

```properties
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
```

​						$remote_addr                              		远程地址

​						$remote_user [$time_local]          远程用户访问时间

​						$request												 请求头

​						$status													 请求状态

​						$body_bytes_sent							发送了多少个字节的请求

​						$http_referer										上一个请求来自哪里

​						$http_user_agent								用户代理,请求浏览器

​						$http_x_forwarded_for					该请求由哪个ip转发过来的

设置虚拟主机log存放位置和输出格式:

1. ​	将全局变量中的,以下注解放开

   ```
   #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
   #                  '$status $body_bytes_sent "$http_referer" '
   #                  '"$http_user_agent" "$http_x_forwarded_for"';
   ```

2. 在server加入以下配置,例如:

   ```
   server{
              listen 80;
              server_name test.com;
   
              location / {
                   root test.com;
                   index index.html;
             }
           access_log  logs/test.com.access.log  main;
   }
   ```

   ### 定时任务完成日志切割

   切割日志脚本

   ​	输出昨天日期

   ```shell
   #!/bin/bash
   echo `date -d yesterday +%Y%m%d`
   #或者echo $(date -d yesterday +%Y%m%d)
   ```

   ​	日志切割文件脚本

   ​	runlog.sh

   ```shell
   #!/bin/bash
   LOGPATH=/usr/local/nginx/logs/test.com.access.log
   BASEPATH=/root/logs
   back=$BASEPATH/$(date -d yesterday +%Y%m%d%H%M).test.com.access.log
   #echo $back
   mv $LOGPATH $back
   touch $LOGPATH
   kill -USER1 `cat /usr/local/nginx/logs/nginx.pid`
   ```

   配置linux定时任务

   crontab -e

   ```shell
   */1 * * * * sh /usr/local/nginx/sbin/runlog.sh
   ```

   ### 	

# Location详解

## 精准匹配

#### location　语法

​		location有"定位"的意思,根据uri来进行不同的定位

​		在虚拟主机的配置中，是必不可少的,location可以把网站的不同部分,定位到不同的处理方式上。

​		比如,碰到.php,如何调用PHP解释器?这时就需要location

​		location的语法

​		

```
location [=|~\~*|^~] patt{

}
```

中括号可以不写任何参数，此时称为一般匹配

也可以写参数

因此,大类型可以分为3种:

- location = patt{}     [精准匹配]
- location patt{}        [一般匹配]
- location ~ patt{}     [正则匹配]

lication匹配如何发生作用？

首先看有没有精准匹配，如果有，则停止匹配过程

```
location = patt{
	config A
}
如果有 $uri== patt,匹配成功,使用config A
```

#### Location之正则匹配

样例配置

图片所在目录:  /usr/local/nginx/image/image/dev/sdb

```
location ~ image{
     root /usr/local/nginx/image;
     index index.html;
 }
```

## rewrite重写

#### rewrite规则常用命令：

if(条件) { } 设定条件，再进行重写

set #设置变量

return #返回状态码

break #跳出rewrite

rewrite  #重写

#### if　语法格式

```
if 空格 (条件){
	重写格式
}
```

条件写法:

1. ​	"＝"来判断相等，用于字符串比较
2. ​	"~"　用正则来匹配（此处的正则区分大小写）~*　不区分大小写
3. ​	"-f -d -e" 来判断是否为文件，为目录，是否存在

例：

等于号用法（当地址172.20.10.13访问时，返回状态码403）

```properties
location / {
            if ($remote_addr = 172.20.10.13) {
                return 403;
            }
            root   html;
            index  index.html index.htm;
        }
```

#### 正则表达式用法

```por
        location / {
            if ($http_user_agent ~* chrome) {
                rewrite ^.*$ /chrome.html;
            }
            root   html;
            index  index.html index.htm;
        }
```

当使用上边的方式配置的时候，使用chrom浏览器访问，会匹配到/chrom.html页面，请求会被重定向到chrome.html页面。此时请求的浏览器还是chrome，所以该段代码还会被匹配到，这样就会造成无限死循环，所以此时使用chrom.html会报错500,应该添加break，跳出循环

应该使用该配置:

```properties
        location / {
            if ($http_user_agent ~* chrome) {
                rewrite ^.*$ /chrome.html;
                break;
            }
            root   html;
            index  index.html index.htm;
        }
```





配置文件

```properties

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65; 

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

