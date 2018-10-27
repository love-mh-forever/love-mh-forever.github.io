---
layout: post
title: nginx学习汇总
category: base-command
tags: [base-commond]
---
## 配置网络监听

- 配置监听IP地址

```
listen address[:port] [default_server] [setfib=number] [backlog=number] 
        [rcvbuf=size] [sendbuf=size] [deferred] [accept_filter=filter] [bind] [ssl];
```

- 配置监听端口

```
listen port [default_server] [setfib=number] [backlog=number] [rcvbuf=size] 
        [sendbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on | off] [ssl];
```

## 基于名称的虚机主机设置

- 可以配置多个name

`server_name name name1 name2 ....;`

- 可以使用通配符* 只能位于三段字符串组成的首尾或者两段字符串的尾部

`server_name *.alleyz.com www.alleyz.*`

- 使用正则表达式，~作为正则开始的标记，并且正则支持捕获

`server_name ~([a-zA-Z\d]{1,4})\.alleyz.com$;`

``` xml
此时，如果通过xisuo.alleyz.com访问的话，可使用$1捕获xisuo; 
  一个名称若被多次匹配的访问优先级： 
  - 匹配方式不同时 
  1. 精确匹配 
  2. 通配符在开始 
  3. 通配符在结尾 
  4. 正则表达式匹配 
  相同匹配方式时，首次处理优先（顺序）
```

## 配置location块

`location [ = | ~ | ~* | ^~] uri {...}`

- =用于普通uri之前，表示严格匹配
- ~ uri包含正则表达式，并且区分大小写
- ~* 表示包含正则表达式，并且不区分大小写
- ^~ 如果找到与uri匹配度最高的location，立即处理请求。 会对uri进行反编码

## 配置请求的根目录 `http server location`

`服务器收到请求后查找资源的根目录路径，可以使用nginx预设的大多变量，唯$document_root $realpath_root不能使用；通常在location块中使用。`

`root path;`

## 更改location的URI

`除了使用root指定根目录，还可以使用alias指令改变location接收到的请求路径`

```
alias path;

location ~ ^/data/(.+\.(htm|html))${
    alias html/data/other/$1;
}
```

## 设置网站的默认首页

可以针对不同的访问设置不同的首页
`index index.html index.htm;`

## 基于IP配置访问权限 http server location

```
allow address | CIDR | all;
deny address | CIDR | all;

location / {
    root html;
    index index.html;
    deny 10.8.177.26;
}
```

## 测试配置

``` xml
user  alleyz;
worker_processes  2;

pid        logs/nginx_alleyz.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main1  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main1;

    sendfile        on;

    keepalive_timeout  65;

    server {
        listen       3022;
        server_name  10.8.177.32;

        deny 10.8.177.26;
        location / {
            auth_basic "it`s auth test msg!";
            auth_basic_user_file passwd;
            root   html/22;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }



    server {
        listen       3022;
        server_name  10.8.177.21;


        location / {
            root   html;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
}
```

### 图片服务器的搭建

```xml
server {
        listen       8088;
        server_name  localhost;
location ~ .*\.(gif|jpg|jpeg|png)$ {  
            expires 24h;  
            root /home/images/;#指定图片存放路径  
            access_log /home/nginx/logs/images.log;#图片 日志路径  
            proxy_store on;  
            proxy_store_access user:rw group:rw all:rw;  
            proxy_temp_path         /home/images/;#代理临时路径
            proxy_redirect          off;  

            proxy_set_header        Host 127.0.0.1;  
            proxy_set_header        X-Real-IP $remote_addr;  
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;  
            client_max_body_size    10m;  
            client_body_buffer_size 1280k;  
            proxy_connect_timeout   900;  
            proxy_send_timeout      900;  
            proxy_read_timeout      900;  
            proxy_buffer_size       40k;  
            proxy_buffers           40 320k;  
            proxy_busy_buffers_size 640k;  
            proxy_temp_file_write_size 640k;  
            if ( !-e $request_filename)  
            {  
                 proxy_pass  http://127.0.0.1:8088;#代理访问地址  
            }  
        }

        location / {
            root   html;
            index  index.html index.htm;
        }
}

```