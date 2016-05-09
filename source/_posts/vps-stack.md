title: 把你的 VPS 用起来
date: 2014-05-17 10:21
categories: Tech Logs
tags:
- vps
- vpn
- DigitalOcean
- Linode
---


# Todolist

- 配置 epel yum 源
- 提醒在用 rpm 安装包前，首先修改 /etc/my.cnf 文件
- 配置 wordpress 固定连接
- 怎样启动 nginx 与 php-fpm

> 这里是我购买 VPS，然后在上面搭建这个网站的笔记，供参考。

## VPS 的选择

请看这里 [[http://hiaero.net/faster-vps/|速度更快的海外 VPS]]

===== 搭建一个 vpn，翻阅长城 =====

待完成

===== 安装 LEMP =====

> 为什么是 LEMP?
> ''**因为 Nginx 是 EngineX**''

==== 安装 MySQL ====

如果买的是内存为 512M 的实例，需要做一个 SWAP 文件并挂载，否则 MySQL 无法安装成功。 \\ <code>
dd if=/dev/zero of=/swapfile bs=1M count=1024
mkswap /swapfile
swapon /swapfile
echo 'swapon /swapfile' >> /etc/rc.d/rc.local
</code>

添加用户，并安装 MySQL ，我使用的使 percona 的 rpm 包。
<code>useradd -s /sbin/nologin mysql</code>

MySQl 安装好后，删除其中的匿名用户
<code>
mysql> delete from mysql.user where user='';
mysql> flush privileges;
</code>

==== 源码安装 PHP ====

安装依赖包

<code>
yum install -y gcc gcc-c++ autoconf cmake libjpeg-devel libpng-devel freetype-devel libxml2-devel zlib-devel glibc-devel glib2-devel bzip2-devel curl-devel openssl-devel pcre-devel libmcrypt-devel
</code>

下载安装 php

<code>
tar zxf php-5.5.13.tar.gz
cd php-5.5.13
./configure --prefix=/usr/local/php --disable-fileinfo --enable-fpm --enable-ftp --with-gd --with-mysql --with-curl --with-mcrypt --with-zlib --enable-zip --with-jpeg-dir --with-png-dir --with-openssl --enable-mbstring --with-pdo-mysql
make
make install
ln -s /usr/local/php/bin/php /usr/bin/php
</code>

复制 php 配置文件

<code>
cp -f php.ini-production /usr/local/php/lib/php.ini
mv /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
</code>

添加 php 的 bin 目录到 PATH

<code>
PATH=$PATH:$HOME/bin:/usr/local/php/bin/
</code>

==== 源码安装 Memcached ====

安装 Memcached

<code>
useradd -M -s /sbin/nologin memcached
yum -y install libevent-devel

tar zxf memcached-1.4.20.tar.gz
cd memcached-1.4.20
./configure && make && make install

ln -s /usr/local/bin/memcached /usr/bin/memcached
mkdir /var/run/memcached
cp scripts/memcached.sysv /etc/init.d/memcached
chkconfig memcached on
</code>

添加 Memcached 配置文件，创建 ''/etc/sysconfig/memcached'' 文件并加入以下内容

<code>
PORT="11211"
USER="memcached"
MAXCONN="256"
CACHESIZE="32"
OPTIONS="-l 127.0.0.1"
</code>

启动 Memcached

<code>service memcached start</code>

安装 PHP 的 Memcached 扩展

<code>
tar zxf libmemcached-1.0.18.tar.gz
cd libmemcached-1.0.18
./configure && make && make install

tar zxf memcached-2.2.0.tgz
cd memcached-2.2.0
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config --disable-memcached-sasl
make && make install
</code>

==== 安装 Nginx ====

<code>
tar zxf nginx-1.6.0.tar.gz
cd nginx-1.6.0
./configure --with-http_stub_status_module --with-http_ssl_module --prefix=/opt/nginx
make
make install
</code>

修改一些 nginx 常用设置：

<code>
error_log  logs/error.log;
pid        logs/nginx.pid;
</code>

==== 两个方便的函数 ====

在 ~/.bash_profile 中添加两个方便的函数，用来重载 Nginx 和 php-fpm

<code>
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
</code>

==== 配置 Nginx 支持 Wordpress 固定链接 ====

待完成

==== 静态文件 CDN 加速等 ====

使用 upyuan 参考：
http://www.84tt.com/web/2013/04/718.html
http://www.meshuo.com/archives/2280.html
