## Nginx是什么？

Nginx 是**高性能的 HTTP** 和**反向代理**的服务器，处理高并发能力是十分强大的，能经受高负 载的考验,有报告表明能支持高达 50,000 个并发连接数。

### 正向代理

需要在客户端配置代理服务器进行指定网站访问

![image-20230801104434990](Nginx/image-20230801104434990.png)

### 反向代理

暴露的是代理服务器地址，隐藏了真实服务器 IP 地址。

![image-20230801104523081](Nginx/image-20230801104523081.png)

**正向代理就是客户端自己设置的代理**，明确知道请求的目标服务器地址是什么，**反向代理就是服务端设置的代理**，客户端不知道目标服务器的地址，只能拿到代理服务器的地址

### 负载均衡

增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的 情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们所说的负 载均衡

![image-20230801104600218](Nginx/image-20230801104600218.png)

### 动静分离

![image-20230801104616882](Nginx/image-20230801104616882.png)



## 安装

### **安装要求的环境**

**安装gcc环境**

```shell
yum install gcc-c++
```

**第三方的开发包**

```shell
yum install -y pcre pcre-devel
```

PCRE(Perl Compatible Regular Expressions)是一个Perl库，包括 perl 兼容的正则表达式库。nginx的http模块使用pcre来解析正则表达式，所以需要在linux上安装pcre库；pcre-devel是使用pcre开发的一个二次开发库。nginx也需要此库

**zlib**

```shell
yum install -y zlib zlib-devel
```

zlib库提供了很多种压缩和解压缩的方式，nginx使用zlib对http包的内容进行gzip，所以需要在linux上安装zlib库

**openssl**

```shell
yum install -y openssl openssl-devel
```

nginx不仅支持http协议，还支持https（即在ssl协议上传输http），所以需要在linux安装openssl库

### 安装Nginx

下载nginx源码包，**把nginx源码包上传到linux系统上**

```
https://nginx.org/download/nginx-1.24.0.tar.gz
```

解压到/usr/local下面

```shell
tar -xvf nginx-1.24.0.tar.gz -C /usr/local
```

使用cofigure命令创建一个makeFile文件，**执行下面的命令的时候，一定要进入到nginx-1.24.0目录里面去**

```shell
./configure \
--prefix=/usr/local/nginx \   #表示软件安装到/usr/local/nginx下面
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-file-aio \
--with-http_realip_module
```

启动nginx之前，上边将临时文件目录指定为/var/temp/nginx，需要在/var下创建temp及nginx目

```shell
mkdir /var/temp/nginx -p
```

进入nginx-1.24.0里面执行make命令进行编译,执行make install 命令进行安装

```shell
make && make install
```

进入安装位置/usr/local/nginx查看目录结构

```shell
cd /usr/local/nginx && ll
```

进入` /usr/local/nginx/sbin`目录，执行命令./nginx，**启动nginx**

查看nginx是否启动` ps -aux | grep nginx`



## 访问Nginx服务

在 windows 系统中访问 linux 中 nginx，默认不能访问的，因为防火墙问题 

1. 关闭防火墙 
2. 开放访问的端口号，80 端口

```shell
#查看开放的端口号
firewall-cmd --list-all
#设置开放的端口号
firewall-cmd --add-service=http –permanent
firewall-cmd --add-port=80/tcp --permanent
#重启防火墙
firewall-cmd –reload
```

之后在浏览器输入`http://IP地址`



## Nginx 的常用的命令

进入 nginx 目录中

```shell
cd /usr/local/nginx/sbin
```

```shell
# 1、查看 nginx 版本号
./nginx -v
# 2、启动 nginx
./nginx
# 3、停止 nginx
./nginx -s stop
# 4、重新加载 nginx
./nginx -s reload
```



## Nginx 的配置文件

nginx 配置文件位置`/usr/local/nginx/conf/nginx.conf`

![img](https://pic2.zhimg.com/80/v2-7cc8fd22835e0afd81d876c1dfd43bfd_720w.webp)

配置文件中的内容，包含三部分内容

1. 全局块：配置服务器整体运行的配置指令,比如 `worker_processes 1;`处理并发数的配置 
2. events 块：影响 Nginx 服务器与用户的网络连接，比如 `worker_connections 1024; `支持的最大连接数为 1024 
3. http 块
   1. http 全局块
   2. server 块



## 实例

### 准备工作

先在linux上安装`nodejs`，然后安装`http-server`，之后在`/home/用户名/`下创建一个目录用来开启http-server服务

```shell
#安装nodejs,下载相应版本的.gz源码，并放到linux上
tar -xvf node-v18.10.0-linux-arm64.tar.gz -C /usr/local
#安装 http-server
pnpm install http-server -g
#创建服务目录
cd /home/xxx && mkdir test-server
```

然后在`test-server`下创建一个`index.html`

```html
<!DOCTYPE html>
<html>
  <head></head>
  <body>
    <h1>hello nginx 80</h1>
  </body>
</html>
```

启动服务

```shell
cd /home/xxx/test-server && http-server -p 8080
```

对外开放端口 8080

```shell
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd –reload
```

查看已经开放的端口号

```shell
firewall-cmd --list-all
```



### 反向代理

**例子1:**

打开浏览器，在浏览器地址栏输入地址 www.123.com，跳转到 liunx 系统`http-server`服务主页面中

在当前电脑中配置 hosts（linux的ip为 192.168.162.128）

```
192.168.162.128 www.123.com
```

设置nginx的代理

```config
server {
  listen 			80;
  server_name 192.168.162.128;
	
	# 
  location / { 
    proxy_pass http://127.0.0.1:8080
  }
}
```

重启nginx服务

```shell
./nginx -s reload
```

**例子2:**

使用 nginx 反向代理，根据访问的路径跳转到不同端口的服务中 nginx 监听端口为 9001

访问 http://192.168.162.128:9001/edu/ 直接跳转到 127.0.0.1:8080 

访问 http://192.168.162.128:9001/vod/ 直接跳转到 127.0.0.1:8081

## 参考

[nginx安装及其配置详细教程](https://zhuanlan.zhihu.com/p/83890573)

[最全Nginx 配置文件详解及安装](https://zhuanlan.zhihu.com/p/92995126)