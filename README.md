#### Linux绿色版Nginx

##### 背景

Nginx想必大家都比较熟悉了，这里就不做过多的诉述它的强大和用处了。上次在开发时，产品提出了一个需求：在不同网段的实现请求的转发和响应，最终讨论使用 *nginx+autossh* 做成一个小安装包 + 代码的形式实现这一功能（这里不具体展开这个需求，主要描述一下怎么制作一个绿色版、解压即用的nginx）；由于我在上一家公司用到的nginx，就是解压即用的；所以准备直接拿一个zip压缩包，来进行配置，但是问了一下这边同事，发现这边的nginx是安装版本的，即是下载安装包，然后安装进 */usr/local/nginx* ；但是大伙都知道，很多项目现场部署的服务器都是没有外网的，并且可能存在相关Linux依赖都不齐全 甚至 root 用户都没权限的情况，所以让现场直接去源码编译安装有时是很不现实的，所以，本文就是为了制作一个绿色版、任意用户、解压即用的nginx。

##### 准备

-   Linux（虚拟机/服务器）

-   Linux目录结构

        |-------nginx (要制作的nginx的空目录)

        |-------nginx-src（存在nginx源码、依赖等）

            |----nginx-1.16.1

            |----openssl-1.0.2s

            |----pcre-8.43

            |----zlib-1.2.11

-   在*nginx-src*目录下载nginx源码及相关依赖（Linux上没wget，可以本地直接访问网址下载后上传）

    ```
    wget https://www.openssl.org/source/openssl-1.0.2s.tar.gz
    wget https://ftp.exim.org/pub/pcre/pcre-8.43.tar.gz
    wget https://zlib.net/zlib-1.2.11.tar.gz
    wget http://nginx.org/download/nginx-1.16.1.tar.gz
    ​
    解压缩
    tar -zxvf nginx-1.16.1.tar.gz 
    tar -zxvf openssl-1.0.2s.tar.gz 
    tar -zxvf pcre-8.43.tar.gz
    tar -zxvf zlib-1.2.11.tar.gz 
    ```

    **准备完成后文件目录如下**：*nginx*为空，*nginx-src*为nginx源码和依赖

    ```
    [root@localhost nginx-green]# pwd
    /root/nginx-green
    [root@localhost nginx-green]# ll
    total 0
    drwxr-xr-x 6 root root  54 Jan 17 16:35 nginx  
    drwxr-xr-x 6 root root 191 Jan 17 16:35 nginx-src
    [root@localhost nginx-green]# cd nginx-src/
    [root@localhost nginx-src]# ll
    total 8892
    drwxr-xr-x  9 root root     186 Jan 17 16:35 nginx-1.16.1
    -rw-r--r--  1 root root 1032630 Jan 17 16:35 nginx-1.16.1.tar.gz
    drwxr-xr-x 21 root root    4096 Jan 17 16:35 openssl-1.0.2s
    -rw-r--r--  1 root root 5349149 Jan 17 16:35 openssl-1.0.2s.tar.gz
    drwxr-xr-x  9 root root    8192 Jan 17 16:35 pcre-8.43
    -rw-r--r--  1 root root 2085854 Jan 17 16:35 pcre-8.43 .tar.gz
    drwxr-xr-x 14 root root    4096 Jan 17 16:35 zlib-1.2.11
    -rw-r--r--  1 root root  607698 Jan 17 16:35 zlib-1.2.11.tar.gz
    ```

##### 配置及安装

###### 配置

```
./configure \
   --with-openssl=../openssl-1.0.2s \
   --with-pcre=../pcre-8.43 \
   --with-zlib=../zlib-1.2.11 \
   --with-pcre-jit --user=root \
   --prefix=../../nginx \
   --with-http_ssl_module \
   --with-http_v2_module
  
  
# 出现以下提示表示配置完成
# 如果有ERROR需要手动解决 ： 
# 参考链接：https://www.klavor.com/dev/20190724-586.html
 Configuration summary
      + using PCRE library: ../pcre-8.43
      + using OpenSSL library: ../openssl-1.0.2s
      + using zlib library: ../zlib-1.2.11
​
      nginx path prefix: "../../nginx"
      nginx binary file: "../../nginx/sbin/nginx"
      nginx modules path: "../../nginx/modules"
      nginx configuration prefix: "../../nginx/conf"
      nginx configuration file: "../../nginx/conf/nginx.conf"
      nginx pid file: "../../nginx/logs/nginx.pid"
      nginx error log file: "../../nginx/logs/error.log"
      nginx http access log file: "../../nginx/logs/access.log"
      nginx http client request body temporary files: "client_body_temp"
      nginx http proxy temporary files: "proxy_temp"
      nginx http fastcgi temporary files: "fastcgi_temp"
      nginx http uwsgi temporary files: "uwsgi_temp"
      nginx http scgi temporary files: "scgi_temp"
```

**注意：** 前3项为安装时指定依赖的位置，这里直接使用我们自己下载的依赖，因为系统的依赖可能会因为版本不符或者不存在会导致配置失败；其中最重要的配置是 ***--prefix=../../nginx***，这个配置指定了我们待会编译安装的nginx的保存目录，并且， ***../../nginx***这个路径在编译安装完成后，在后续启动的会引用这个配置（这里有投机取巧嫌疑，如果不是 ***../../nginx***的话，会在nginx启动时读取不到正确的目录，需要手动指定绝对路径，所以这里强烈!!!建议设置成 ***../../nginx***）。

###### 安装

```
make 
make install
```

##### 打包

至此绿色版本nginx已经制作完成，直接把nginx文件夹打包迁移置其他环境解压即用了。

```
[root@localhost nginx-green]# pwd
/root/nginx-green
[root@localhost nginx-green]# ll
total 0
drwxr-xr-x 6 root root  54 Jan 17 16:35 nginx
drwxr-xr-x 6 root root 191 Jan 17 16:35 nginx-src
[root@localhost nginx-green]# zip -r nginx.zip nginx
.
.
.
[root@localhost nginx-green]# ll
total 4328
drwxr-xr-x 6 root root      54 Jan 17 16:35 nginx
drwxr-xr-x 6 root root     191 Jan 17 16:35 nginx-src
-rw-r--r-- 1 root root 4431284 Jan 17 17:01 nginx.zip
​
```

##### 下载链接

<https://github.com/wuzefang/nginx-green>
