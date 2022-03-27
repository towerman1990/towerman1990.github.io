---
title: PHP 8 安装amqp扩展
date: 2021-02-19 13:52:12
tags:
---

最近公司新项目开始使用PHP8，之前使用的扩展也需要升级，更换成支持PHP8的版本。在使用RabbitMQ时需要使用到php-amqp扩展，我安装的时候发现网上的一些安装教程都比较旧了，顶多支持到php7对应的版本。下面记录一下php8相关扩展的安装过程。

### 安装rabbitmq-c

安装PHP-AMQP扩展前首先要安装 rabbitmq-c，rabbitmq-c 是一个支持 RabbitMQ v2.0+ 版本的由C语言开发的 AMQP 客户端库。

按照官方说法安装rabbitmq-c需要使用 v2.6及以上版本的`CMake`进行构建，我们先安装一下它：

``` bash
sudo dnf install cmake
```

<!-- more -->

- 注：本人开发环境用的是CentOS8，所以默认使用的软件包管理器是dnf，也可以换成yum。

在[发布地址](https://github.com/alanxz/rabbitmq-c/releases/tag/v0.10.0)获取最新版本的源代码，下载后解压并进入项目目录：

``` bash
wget -c https://github.com/alanxz/rabbitmq-c/archive/v0.10.0.tar.gz
tar -zxvf v0.10.0.tar.gz && cd rabbitmq-c-0.10.0
```

创建一个构建目录并进入它：

``` bash
sudo mkdir build && cd build
```

使用`CMake`进行构建，在构建的时候可以设置想要安装的目录：

``` bash
sudo cmake -DCMAKE_INSTALL_PREFIX=/usr/local/rabbitmq-c-0.10.0/ ..
```

按照官网的说明，如果已经有运行中的 `RabbitMQ`，现在就可以测试刚刚构建的 `rabbitmq-c` 是否可用。
在一个客户端中打开构建目录（即上文中新建的build目录）并执行命令：

``` bash
./examples/amqp_listen localhost 5672 amq.direct test
```
- 注：localhost 为 `RabbitMQ` 的地址。

在一个客户端中同样打开构建目录并执行命令：

``` bash
./examples/amqp_sendstring localhost 5672 amq.direct test "hello world"
```

可以看到下面的输出，就表明构建成功：

```bash
Delivery 1, exchange amq.direct routingkey test
Content-type: text/plain
----
00000000: 68 65 6C 6C 6F 20 77 6F : 72 6C 64                 hello world
0000000B:
```

执行安装操作：

``` bash
sudo cmake --build .  --target install
```

### 安装 php-amqp

当我打开php-amqp的发布页的时候，我惊奇的发现没有支持PHP8的release，最新版本还是20年4月发布的...
去Issues区看了一下，发现第一个[问题](https://github.com/php-amqp/php-amqp/issues/386)就是询问支持PHP8的release。根据下面的讨论来看，支持PHP8的代码已经合并进项目了，作者准备发布一个beta版的release，然而两个月已经过去了还是什么都没有...
不过既然支持PHP8的代码已经合并了，我们可以自己进行安装工作。
首先克隆项目到本地：

``` bash
git clone https://github.com/php-amqp/php-amqp.git
```

进入项目目录执行配置操作：

``` bash
cd php-amqp
phpize
./configure --with-librabbitmq-dir=/usr/local/rabbitmq-c-0.10.0/ --with-php-config=/usr/bin/php-config
```

- 注：phpize的版本应为20200930，如果低于此版本或没有phpize请先安装新版本的php-devel。

最后执行安装操作：

``` bash
make && make install
```

此时出现了错误：

``` bash
/bin/ld: cannot find -lrabbitmq
```

因为我们之前安装rabbitmq-c时把它安装到了一个单独的目录，在lib目录下是找不到。所以我们需要做一个软连接：

``` bash
sudo ln -s /usr/local/rabbitmq-c-0.10.0/lib64/librabbitmq.so /usr/lib/librabbitmq.so
```

然后执行安装操作就成功了，但是使用`php -m`命令查看扩展的时候，发现找不到刚刚安装的扩展，因为php的配置文件中没有引入安装的扩展。
在php的配置文件中增加扩展的引入：

``` bash
extension=amqp.so
```

- 注：php在cli和php-fpm两种环境下可能使用了不同的配置文件，所以最好通过`phpinfo()`方法确认配置文件的位置，这个问题曾经给我带过了极大的困扰 =_="。

同时确定php的扩展安装目录，将文件移动过去：

``` bash
sudo cp /usr/lib64/php/modules/redis.so /usr/local/php8/lib/php/extensions/no-debug-non-zts-20200930/
```

最后重启php-fpm进程，访问`phpinfo()`方法，第一个扩展就是amqp：

``` bash
sudo service php-fpm restart
```

{% asset_img 01.amqp.png amqp %}