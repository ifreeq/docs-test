---
description: '昨天在配置google cdn的时候, 缺乏对Nginx的基本了解'
---

# Nginx

## 1. 基本介绍和配置文件语法

### 1, 介绍

Ngin基于时间的异步IO的并发模型

### 2, 安装

ubuntu 安装nginx:

sudo apt-get install nginx

编译安装: ./configure make sudo make install

其中关于configure是可以按照自己的需求来配置安装的参数的。比如:

```text
./configure \
--user=nginx                          \
--group=nginx                         \
--prefix=/etc/nginx                   \
--sbin-path=/usr/sbin/nginx           \
--conf-path=/etc/nginx/nginx.conf     \
--pid-path=/var/run/nginx.pid         \
--lock-path=/var/run/nginx.lock       \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module        \
--with-http_stub_status_module        \
--with-http_ssl_module                \
--with-pcre                           \
--with-file-aio                       \
--with-http_realip_module             \
--without-http_scgi_module            \
--without-http_uwsgi_module           \
--without-http_fastcgi_module
```

### 3, 命令行语法\(Command line Syntax\)

Start, restart, stop

```text
sudo /etc/init.d/nginx restart # or start, stop
```

Or do like this

```text
sudo service ngingx restart # or start, stop
```

Reload

```text
sudo nginx -s reload
```

### 4, 配置文件语法\(Configuration file syntax\)

nginx是模块化的系统，整个系统是分成一个个模块的。每个模块负责不同的功能。比如http\_gzip\_static\_module就是负责压缩的，http\_ssl\_module就是负责加密的，如果不用某个模块的话，也可以去掉，可以让整个nginx变得小巧，更适合自己。在上面的configure指令中带了很多参数，就是在这里编译之前可以加入某些模块或去掉某些模块的。

要用的模块已经被编译进nginx了，成为nginx的一部分了，那要怎么用这些模块呢？那就得通过配置文件，这跟传统的linux服务差不多，都是通过配置文件来改变功能。nginx的模块是通过一个叫指令\(directive\)的东西来用的。整个配置文件都是由指令来控制的。nginx也有自己内置的指令，比如events, http, server, 和 location等，下面会提到的。

如果是ubuntu系统，安装后，配置文件存放在`/etc/nginx/nginx.conf`

```text
user www-data;
worker_processes 1;
pid /run/nginx.pid;

events {
  worker_connections 768;
}

http {

  sendfile on;
  tcp_nopush on;
  tcp_nodelay on;
  keepalive_timeout 65;
  types_hash_max_size 2048;

  include /etc/nginx/mime.types;
  default_type application/octet-stream;

  access_log /var/log/nginx/access.log;
  error_log /var/log/nginx/error.log;

  gzip on;
  gzip_disable "msie6";

  include /etc/nginx/conf.d/*.conf;
  include /etc/nginx/sites-enabled/*;
}

```

在这个示例文件, 先不管上面3行, 就是由两个block\(块\)组成的.

```text
events{
}

http{
}
```

块和块之前还可以嵌套

```text
http{
server{
}
}
```

这个是主配置文件。有时候仅仅一个配置文件是不够的，由其是当配置文件很大时，总不能全部逻辑塞一个文件里，所以配置文件也是需要来管理的。看最后两行`include`，也就是说会自动包含目录`/etc/nginx/conf.d/`的以conf结尾的文件，还有目录`/etc/nginx/sites-enabled/`下的所有文件。

在`/etc/nginx/conf.d/`下有个配置文件叫rails.conf，它的内容大体是这样的。

```text
upstream rails365 {
    # Path to Unicorn SOCK file, as defined previously
    server unix:///home/yinsigan/rails365/shared/tmp/sockets/unicorn.sock fail_timeout=0;
}
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;
    server_name www.rails365.net;
    root         /home/yinsigan/rails365/current/public;
    keepalive_timeout 70;

  ...

    # redirect server error pages to the static page /50x.html
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }

  ...
}

server {
  listen 80;
  server_name rails365.net;

  return 301 $scheme://www.rails365.net$request_uri;
}

```

整个配置文件的结构大体是这样子的

```text
# 这里是一些配置
...
http {
  # 这里是一些配置
  ...
  # 这部分可能存在于/etc/nginx/conf.d/目录下
  upstream {

  }
  server {
    listen 8080;
    root /data/up1;

    location / {
    }
  }
  server {
    listen 80;
    root /data/up2;

    location / {
    }
  }
  这里是一些配置
  ...
}

mail {
}

```

指令和指令之间是有层级和继承关系的。比如http内的指令会影响到server的。

http那部分除非必要，我们不动它，假如你现在要部署一个web服务，那就在/etc/nginx/conf.d/目录下新增一个文件就好了。

http和events还有mail是同级的。http就是跟web有关的。

server，顾名思义就是一个服务，比如你现在有一个域名，要部署一个网站，那就得创建一个server块。

就假设你有一个域名叫xxx.xxx.xxx，要在浏览器输入这个域名时就能访问，那可能得这样。

```text
server {
  listen 80;
  root /home/hijason/xxx;
  server_name xxx.xxx.xxx;
  location / {

  }
}

```

具体的意思是这样的。listen是监听的端口。如果没有特殊指定，一般网站用的都是80端口。

root是网站的源代码静态文件的根目录。一般来说会在root指定的目录下放置网站最新访问的那个html文件，比如index.html等。

server\_name指定的是域名。

 有了这些，在浏览器下输入`http://xxx.xxx.xxx`就能访问到目录`/home/hijason/xxx`下的index.html文件的内容。但是有时候我们得访问`http://xxx.xxx.xxx/articles`呢？那得用location。像这样:

```text
server {
  ...
  server_name xxx.xxx.xxx;
  location /articles {

  }
}

```

除了`http://xxx.xxx.xxx/articles`，还有`http://xxx.xxx.xxx/groups`，`/menus/1`等很多，是不是每个都要创建一个location啊，肯定不可能。我们有动态的方法来处理的。

下面来看一个例子:

```text
server {
    listen      80;
    server_name example.org www.example.org;
    root        /data/www;

    location / {
        index   index.html index.php;
    }

    location ~* \.(gif|jpg|png)$ {
        expires 30d;
    }

    location ~ \.php$ {
        fastcgi_pass  localhost:9000;
        fastcgi_param SCRIPT_FILENAME
                      $document_root$fastcgi_script_name;
        include       fastcgi_params;
    }
}

```

 当用户访问`http://example.org`时就会去读取`/var/root/index.html`，如果找不到就会读取index.php，就会转发到`fastcgi_pass`里面的逻辑。当用户访问`http://example.org/about.html`就会去读取`/var/root/about.html`文件，同样道理，当用户访问`http://example.org/about.gif`就会读取`/var/root/about.gif`文件，并会在30天后过期，也就是缓存30天。

## 2, 反向代理

**1. 什么叫反向代理服务器?**

要说反向代理服务器，先来说一般的代理服务器。代理就是受委托去做一些事。假如用户A委托B去做一些事，做完之后B告诉A结果。在代理服务器中也是一样的道理，用户A通过代理服务器B访问网站C\(`www.example.com`\)，请求先到代理服务器B，B再转发请求到网站C，代理服务器B是真正访问网站C的，访问之后再把网站C的应答结果发给用户A。这样给用户A的感觉是C直接提供服务的一样，因为看不到B的整个处理过程。代理服务器是一个中间者，是充当转发请求的角色。这种代理也叫`正向代理`。

使用正向代理是要在客户端进行设置，比如浏览器设置代理服务器的域名或IP，还有端口等。

正向代理的作用有很多，例如，能访问本无法访问的，加速，cache，隐藏访问者的行踪等，具体的不再详述了。

`反向代理`\(reverse proxy\)正好与正向代理相反，对于客户端而言代理服务器就像是原始服务器，并且客户端不需要进行任何特别的设置。假如用户A访问网站B，这个时候网站B充当了web服务器，也充当了反向代理服务器，它充当的代理服务器的角色是这样，假如用户A要得到网站C的内容，而用户A又不能直接访问到\(例如网络原因\)，而服务器B可以访问到网站C，那服务器可以得到网站C的内容再存起来发给用户A，这整个过程用户A是直接和代理服务器B交互的，用户A不知道网站C的存在，这个web服务器B就是一台反向代理服务器，这个网站C就是上游服务器\(upstream servers\)。

反向代理的作用是，隐藏和保护原始服务器，就像刚才的例子，用户A根本不知道服务器C的存在，但服务器C确实提供了服务。还有，就是负载均衡。当反向代理服务器不止一个的时候，就可以做成一个集群，当用户A访问网站B时，用户A又需要网站C的内容，而网站C有好多服务器，这些服务器就形成了集群，而网站B在请求网站C，就可以有多种方式\(轮循，hash等\)，把请求均匀地分配给集群中的服务器，这个就是负载均衡。

![](https://box.kancloud.cn/b9b93fbf080b4b68a9d876f5f0eafb8d_750x498.jpg)

**2. 示例**

我们先来看最一个最简单的例子。

**2.1 最简单的反向代理**

nginx的反向代理是依赖于[ngx\_http\_proxy\_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html)这个module来实现的。

反向代理服务器能代理的请求的协议包括http\(s\)，FastCGI，SCGI，uwsgi，memcached等。我们这里主要集中在http\(s\)协议。

我有一个网站，用的是https协议来访问的。用这个协议访问的网站在chrome等浏览器的地址栏是可以看到一个绿色的代表安全的标志的。你请求的所有资源都要是https的，它才会出现。假如你请求了一张外部的图片，而这张图片是以http协议请求的，那这个时候那个安全的标志就不存在的。

所以我要把这个https协议的图片请求反向代理到http协议的真实图片上。https协议的这张图片是不存在，而它有一个地址实际指向的内容是http协议中的图片。

```text
# https
server {
  server_name www.example.com;
  listen       443;
  location /newchart/hollow/small/nsh000001.gif {
    proxy_pass http://image.sinajs.cn/newchart/hollow/small/nsh000001.gif;
  }

  location /newchart/hollow/small/nsz399001.gif {
    proxy_pass http://image.sinajs.cn/newchart/hollow/small/nsz399001.gif;
  }

```

 假如我的网站是`www.example.com`这样就能使用`https://www.example.com/newchart/hollow/small/nsh000001.gif`，它指向是`http://image.sinajs.cn/newchart/hollow/small/nsh000001.gif`

## 3, gzip 压缩提升网站性能

**1. 介绍**

网站开发到一定程度，可能css文件或js文件会越来越大，因为有可能加载了很多的插件。这个时候如果能把这些文件压缩一下就好了。

nginx就支持这种功能，它可以把静态文件压缩好之后再传给浏览器。浏览器也要支持这种功能，只要浏览器的请求头带上`Accept-Encoding: gzip`就可以了。假如有一个文件叫application.css，那nginx就会使用gzip模块把这个文件压缩，然后传给浏览器，浏览器再解压缩成原来的css文件，就能读取了。

所有的这一切都需要nginx已经有编译过`ngx_http_gzip_module`这个模块。这个模块能对需要的静态文件压缩大小，比如图片，css，javascript，html等。压缩是需要消耗CPU，但能提高传缩的速度，因为传缩量少了许多，从而节省带宽。

**2. 使用**

使用之前先来查看一下是否编译了`ngx_http_gzip_module`这个模块。

```text
sudo nginx -V
```

配置nginx的gzip

```text
http {
        gzip on;
        gzip_disable "msie6";

        gzip_vary on;
        gzip_proxied any;
        gzip_comp_level 6;
        gzip_buffers 16 8k;
        gzip_http_version 1.1;
        gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
        server {
                location ~ ^/assets/ {
                   gzip_static on;
                   expires max;
                   add_header Cache-Control public;
                }        
        }
}

```

上面最重要的是http中`gzip on;`还有`gzip_types`这两行，是一定要写的。其他的`gzip_vary`等都是一些配置，可以不写。

然后在需要压缩的静态资源那里加上下面三行:

```text
gzip_static on;
expires max;
add_header Cache-Control public;
```



