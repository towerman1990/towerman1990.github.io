---
title: Centos 7 LNMP环境搭建
date: 2020-02-13 23:08:08
tags:
 - Linux
 - Nginx
 - MySQL
 - PHP
---


  在上一篇文章中，简单演示了如何在VMware上安装使用CentOS系统。而在真实的生产环境中，拥有一台安装了Linux操作系统的机器只是一个开始。以前最流行的web架构是LAMP，许多现在大名鼎鼎的互联网公司都是从它开始的。随着时间的推移，轻巧且高性能的Nginx逐渐取代Apache地位，成为服务器部署的首选，LAMP也就演变成了LNMP。下面我们将在安装好的CentOS系统上搭建一套完整的LNMP环境。

<!-- more -->

- 环境准备：

  Centos 7 操作系统

    版本 CentOS Linux release 7.7.1908 (Core)

    首先更新系统：

    `# yum update`

一. Nginx安装

  1. 安装nginx源：

    `# yum localinstall http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm`

  2. 执行安装操作：

    `# yum install nginx`

  3. 启动nginx：

    `# service nginx start`

    此时可以通过curl命令直接访问本机地址，返回welcome页面：

    `# curl localhost`

    {% asset_img 01.curl_localhost.png curl_localhost %}

    如果希望通过其他机器访问，可以通过命令关闭防火墙：

    `systemctl stop firewalld.service`

    还可以禁止防火墙开机启动：

    `systemctl disable firewalld.service`

    主机通过ip访问结果：

    {% asset_img 02.access.png access %}



二. MySQL安装

  1. 下载yum资源包（[地址](https://dev.mysql.com/downloads/repo/yum/)）：

    {% asset_img 03.myql_yum.png myql_yum %}

    *注意：现在下载mysql的yum安装包必须先登录，然后手动下载，所以以前像安装nginx一样，直接通过localinstall参数安装mysql源的方式都失效了*

  2. mysql源：

    `# rpm -ivh mysql57-community-release-el7-7.noarch.rpm`

  3. 安装mysql:

    `# yum install mysql-community-server`

    *如果出现提示：`-bash: wget: command not found`  首先执行：`# yum install wget` 进行wget的安装*

  4. mysql的开发包:

    `# yum install mysql-community-devel`

  5. 启动mysql:

    `# service mysqld start`

  6. 查看mysql启动状态:

    `# service mysqld status`

    {% asset_img 04.mysql_status.png mysql_status %}

  7. 获取mysql默认生成的密码：

    `# grep 'temporary password' /var/log/mysqld.log`

    {% asset_img 05.default_pass.png default_pass%}

  8. 使用获取的默认密码登录：

    `# mysql -uroot -p`

  9. 修改密码：

    `mysql>  ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';`

    *注意：由于mysql5.6以后的版本新增了密码强度验证插件，所以强度不够会提示密码非法。如果希望使用易记的弱密码可以查询validate_password的配置参数。*

  10. 修改完密码后，保存退出。再次使用新密码登录，以确认新密码无误：

    `mysql> exit`

    `# mysql -uroot -p`

三. PHP安装

  1. 下载php源码包：

    `# wget https://www.php.net/distributions/php-7.2.27.tar.gz`

  2. 解压php源码包：

    `# tar -zxvf php-7.2.27.tar.gz`

  3. 进入解压好的源码包目录：

     `# cd php-7.2.27`

  4. 安装php依赖：

    `# yum install gcc libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel`

  5. 编译配置:

    ```bash
    ./configure \
    --prefix=/usr/local/php \
    --with-config-file-path=/etc \
    --enable-fpm \
    --with-fpm-user=nginx  \
    --with-fpm-group=nginx \
    --enable-inline-optimization \
    --disable-debug \
    --disable-rpath \
    --enable-shared  \
    --enable-soap \
    --with-libxml-dir \
    --with-xmlrpc \
    --with-openssl \
    --with-mhash \
    --with-pcre-regex \
    --with-sqlite3 \
    --with-zlib \
    --enable-bcmath \
    --with-iconv \
    --with-bz2 \
    --enable-calendar \
    --with-curl \
    --with-cdb \
    --enable-dom \
    --enable-exif \
    --enable-fileinfo \
    --enable-filter \
    --with-pcre-dir \
    --enable-ftp \
    --with-gd \
    --with-openssl-dir \
    --with-jpeg-dir \
    --with-png-dir \
    --with-zlib-dir  \
    --with-freetype-dir \
    --enable-gd-jis-conv \
    --with-gettext \
    --with-gmp \
    --with-mhash \
    --enable-json \
    --enable-mbstring \
    --enable-mbregex \
    --enable-mbregex-backtrack \
    --with-libmbfl \
    --with-onig \
    --enable-pdo \
    --with-mysqli=mysqlnd \
    --with-pdo-mysql=mysqlnd \
    --with-zlib-dir \
    --with-pdo-sqlite \
    --with-readline \
    --enable-session \
    --enable-shmop \
    --enable-simplexml \
    --enable-sockets  \
    --enable-sysvmsg \
    --enable-sysvsem \
    --enable-sysvshm \
    --enable-wddx \
    --with-libxml-dir \
    --with-xsl \
    --enable-zip \
    --enable-mysqlnd-compression-support \
    --with-pear \
    --enable-opcache
    ```

  6. 编译并安装：

    `# make && make install`

  7. 打开环境变量文件：

    `# vim /etc/profile`

  8. 在句尾加入一下两条命令：

    `PATH=$PATH:/usr/local/php/bin`
    `export PATH`

  9. 此时php命令已经被加入到环境变量中，使之立即生效：

    `# source /etc/profile`

  10. 查看环境变量与php版本:

    `# echo $PATH`
    `# php -v`

    结果如下：

    {% asset_img 06.profile.png profile %}

  11. 配置php-fpm：

    `# cp php.ini-production /etc/php.ini`
    `# cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf`
    `# cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf`
    `# cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm`
    `# chmod +x /etc/init.d/php-fpm`

  12. 启动php-fpm：

    `# /etc/init.d/php-fpm start`

四. Nginx绑定PHP：

  1. 创建nginx配置文件：

    `# vim /etc/nginx/conf.d/lnmp.vm.conf`

  2. 写入配置内容：

    ```bash
    server {
      listen 80;
      server_name  lnmp.vm;
      root /var/www/html/lnmp.vm; # 项目存放路径
      location / {
          index  index.php index.html index.htm;
          # 如果请求既不是一个文件，也不是一个目录，则执行一下重写规则
          if (!-e $request_filename)
          {
              # 地址作为将参数rewrite到index.php上。
              rewrite ^/(.*)$ /index.php/$1;
              # 若是子目录则使用下面这句，将subdir改成目录名称即可。
              # rewrite ^/subdir/(.*)$ /subdir/index.php/$1;
          }
      }
      # proxy the php scripts to php-fpm
      location ~ \.php {
              include fastcgi_params;
              # pathinfo支持start
              # 定义变量 $path_info ，用于存放pathinfo信息
              set $path_info "";
              # 定义变量 $real_script_name，用于存放真实地址
              set $real_script_name $fastcgi_script_name;
              # 如果地址与引号内的正则表达式匹配
              if ($fastcgi_script_name ~ "^(.+?\.php)(/.+)$") {
                      # 将文件地址赋值给变量 $real_script_name
                      set $real_script_name $1;
                      # 将文件地址后的参数赋值给变量 $path_info
                      set $path_info $2;
              }
              #配置fastcgi的一些参数
              fastcgi_param SCRIPT_FILENAME $document_root$real_script_name;
              fastcgi_param SCRIPT_NAME $real_script_name;
              fastcgi_param PATH_INFO $path_info;
              ###pathinfo支持end
          fastcgi_intercept_errors on;
          fastcgi_pass   127.0.0.1:9000;
      }

      location ^~ /data/runtime {
        return 404;
      }

      location ^~ /application {
        return 404;
      }

      location ^~ /simplewind {
        return 404;
      }
    }
    ```

  3. 重启nginx：

    `# service nginx reload`

  4. 创建项目的默认访问文件：

    `# vim /var/www/html/lnmp.com/index.php`

  5. 写入phpinfo函数：

  `<?php
   phpinfo();`

  6. 找到主机的host文件，并加入一条域名映射：

    win10的host文件地址为：`C:\Windows\System32\drivers\etc\hosts\host`
    加入一条映射关系：`192.168.220.233  lnmp.vm`

  7. 在主机访问配置过的域名，结果如下：

    {% asset_img 07.php_info.png php_info %}
