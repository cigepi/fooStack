title: 个人站搭建手册
date: 2015-06-28 18:00
categories: Tech Logs
tags:
- DigitalOcean
- vpn
- vps
---

这里是我购买 VPS，然后在上面搭建这个网站的笔记，供参考。

## VPS 的选择

请看这里 [速度更快的海外 VPS](http://hiaero.net/faster-vps/)

## 搭建一个 VPN，翻阅长城

参照了 DigitalOcean 的[教程](https://www.digitalocean.com/community/tutorials/how-to-setup-a-multi-protocol-vpn-server-using-softether)，使用了 [SoftEther](http://www.softether.org/) 来创建了 VPN 服务。

另外，根据 SoftEther 的官方 [L2TP/IPsec VPN Server 教程](http://www.softether.org/4-docs/2-howto/9.L2TPIPsec_Setup_Guide_for_SoftEther_VPN_Server/1.Setup_L2TP%2F%2F%2F%2FIPsec_VPN_Server_on_SoftEther_VPN_Server)，要使用 SoftEther 的 L2TP/IPsec VPN 服务，需要在 iptables 中开启 `UDP 500` 与 `4500` 端口。除此之外，为了使 L2TP 能正常连接，还需要开启 `UDP 1701` 端口。



## 安装 LEMP

> 为什么是 LEMP?
> ''**因为 Nginx 是 EngineX**''

### 安装 MySQL

如果买的是内存为 512M 的实例，需要做一个 SWAP 文件并挂载，否则 MySQL 无法安装成功。

```console
dd if=/dev/zero of=/swapfile bs=1M count=1024
mkswap /swapfile
swapon /swapfile
echo 'swapon /swapfile' >> /etc/rc.d/rc.local
```

添加用户，并安装 MySQL ，我使用的使 percona 的 rpm 包。

```console
useradd -s /sbin/nologin mysql
```

MySQl 安装好后，删除其中的匿名用户

```console
mysql> delete from mysql.user where user='';
mysql> flush privileges;
```

### 源码安装 PHP

安装依赖包

```console
yum install -y gcc gcc-c++ autoconf cmake libjpeg-devel libpng-devel freetype-devel libxml2-devel zlib-devel glibc-devel glib2-devel bzip2-devel curl-devel openssl-devel pcre-devel libmcrypt-devel
```

下载安装 php

```console
tar zxf php-5.5.13.tar.gz
cd php-5.5.13
./configure --prefix=/usr/local/php --disable-fileinfo --enable-fpm --enable-ftp --with-gd --with-mysql --with-curl --with-mcrypt --with-zlib --enable-zip --with-jpeg-dir --with-png-dir --with-openssl --enable-mbstring --with-pdo-mysql
make
make install
ln -s /usr/local/php/bin/php /usr/bin/php
```

复制 php 配置文件

```console
cp -f php.ini-production /usr/local/php/lib/php.ini
mv /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
```

添加 php 的 bin 目录到 PATH

```console
PATH=$PATH:$HOME/bin:/usr/local/php/bin/
```

### 源码安装 Memcached

安装 Memcached

```console
useradd -M -s /sbin/nologin memcached
yum -y install libevent-devel

tar zxf memcached-1.4.20.tar.gz
cd memcached-1.4.20
./configure && make && make install

ln -s /usr/local/bin/memcached /usr/bin/memcached
mkdir /var/run/memcached
cp scripts/memcached.sysv /etc/init.d/memcached
chkconfig memcached on
```

添加 Memcached 配置文件，创建 ''/etc/sysconfig/memcached'' 文件并加入以下内容

```console
PORT="11211"
USER="memcached"
MAXCONN="256"
CACHESIZE="32"
OPTIONS="-l 127.0.0.1"
```

启动 Memcached

```console
service memcached start
```

安装 PHP 的 Memcached 扩展

```console
tar zxf libmemcached-1.0.18.tar.gz
cd libmemcached-1.0.18
./configure && make && make install

tar zxf memcached-2.2.0.tgz
cd memcached-2.2.0
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config --disable-memcached-sasl
make && make install
```

### 安装 Nginx

```console
tar zxf nginx-1.6.0.tar.gz
cd nginx-1.6.0
./configure --with-http_stub_status_module --with-http_ssl_module --prefix=/opt/nginx
make
make install
```

修改一些 nginx 常用设置：

```console
error_log  logs/error.log;
pid        logs/nginx.pid;
```

### 两个方便的函数

在 ~/.bash_profile 中添加两个方便的函数，用来重载 Nginx 和 php-fpm

```console
export PATH

# Reload Nginx
function rex()
{
    kill -HUP `cat /opt/nginx/logs/nginx.pid`
}

# Reload php-fpm
function rep()
{
    killall php-fpm
    /usr/local/php/sbin/php-fpm
}
```

### 配置 Nginx 支持 Wordpress 固定链接

待完成

### 静态文件 CDN 加速等

使用 upyuan 参考：

http://www.84tt.com/web/2013/04/718.html
http://www.meshuo.com/archives/2280.html
