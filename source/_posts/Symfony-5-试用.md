---
title: Symfony 5 试用
date: 2020-06-13 17:54:14
tags:
 - PHP
 - Symfony
 - Nginx
---

在公司的PHP项目中，一直都在使用Symfony这个框架。这个框架的优点有很多，比如它的功能强大而全面，以至于众多PHP项目都在使用Symfony提供的组件；长期而稳定的技术支持，整个项目从2005年到今天已经走过了十几个年头，最终演化到了当前的5.1版本；此外还有丰富全面的文档、强大的社区支持等等。而它最大缺点恐怕就是因为过于强大的功能设计，导致学习的曲线有些陡峭。

正好前几天Symfony更新了5.1的release版本[<sup>1</sup>](#refer-anchor-1)，修复了大量的bug，感觉已经比较成熟了。所以今天就带大家一块来试用一下这个新的版本。

<!-- more -->

安装之前需要确认部署的环境是否符合Symfony的要求，比如Symfony5就要求PHP的版本要大于7.2.5，并且开启了Ctype, iconv, JSON, PCRE, Session, SimpleXML, Tokenizer这些扩展。

Symfony的安装方式主要有两种，一种是使用官方提供的CLI工具[<sup>1</sup>](#refer-anchor-2)，另一种是大家熟悉的Composer。现在几乎所有的现代PHP项目都在使用Composer，来进行第三方的依赖管理，所以这次试用我们就使用Composer来完成。

1. 安装Composer

  Composer是PHP用来管理依赖（dependency）关系的工具。换句话说，就是我们可以利用这个工具，直接在我们的项目中引入并使用别人写好的功能模块。

  首先我们要下载composer.phar文件（windows可以去官网下载安装包）：

    `# curl -sS https://getcomposer.org/installer | php`

  为了全局调用Composer 我们把下载好的文件移动一下位置：

    `# mv composer.phar /usr/local/bin/composer`

  最后查看一下Composer的版本，判断是否安装成功：

    `# composer -v`

    {% asset_img 01.composer_version.png composer_version %}

  默认安装依赖会使用国外的链接，所以很多时候安装过程会很慢，以至于安装失败。我们可以配置成使用国内的镜像，比如淘宝：

    `# composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/`

2. 安装Symfony

  首先我们建一个项目所在的文件夹：

  `# mkdir /var/www && cd /var/www`

  然后需要我们选择安装的类别，一种是安装一个传统的web项目，另一种是现在流行的微服务。两者在安装的组件上有一些小的差异，比如web项目会安装twig模板组件，用来做web页面的渲染。这次我们选择传统的web项目：

  `# composer create-project symfony/website-skeleton my_project_name`

  或者你也可以选择做一个微服务：

  `# composer create-project symfony/skeleton my_project_name`

3. 配置服务器

  配置就以Nginx为例，创建一个新的Nginx配置文件：

  `# vim /etc/nginx/conf.d/symfony_dev.conf`

  写入如下配置内容：

    ```bash
    server {
        server_name symfony_dev.vm;
        root /var/www/symfony_dev/public;

        location / {
            # try to serve file directly, fallback to index.php
            try_files $uri /index.php$is_args$args;
        }

        # optionally disable falling back to PHP script for the asset directories;
        # nginx will return a 404 error when files are not found instead of passing the
        # request to Symfony (improves performance but Symfony's 404 page is not displayed)
        # location /bundles {
        #     try_files $uri =404;
        # }

        location ~ ^/index\.php(/|$) {
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_split_path_info ^(.+\.php)(/.*)$;
            include fastcgi_params;

            # optionally set the value of the environment variables used in the application
            # fastcgi_param APP_ENV prod;
            # fastcgi_param APP_SECRET <app-secret-id>;
            # fastcgi_param DATABASE_URL "mysql://db_user:db_pass@host:3306/db_name";

            # When you are using symlinks to link the document root to the
            # current version of your application, you should pass the real
            # application path instead of the path to the symlink to PHP
            # FPM.
            # Otherwise, PHP's OPcache may not properly detect changes to
            # your PHP files (see https://github.com/zendtech/ZendOptimizerPlus/issues/126
            # for more information).
            fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
            fastcgi_param DOCUMENT_ROOT $realpath_root;
            # Prevents URIs that include the front controller. This will 404:
            # http://domain.tld/index.php/some-path
            # Remove the internal directive to allow URIs like this
            internal;
        }

        # return 404 for all other php files not matching the front controller
        # this prevents access to other php files you don't want to be accessible.
        location ~ \.php$ {
            return 404;
        }

        error_log /var/log/nginx/symfony_dev_error.log;
        access_log /var/log/nginx/symfony_dev_access.log;
    }
    ```

4. 访问Symfony

  最后打开浏览器，输入我们配置好的域名访问，就可以看到Symfony的欢迎页面：

  {% asset_img 02.symfony_welcome.png symfony_welcome %}

  在src/Controller目录中，我们新建一个控制器文件：

    ```php
    <?php
    // src/Controller/LuckyController.php
    namespace App\Controller;

    use Symfony\Component\HttpFoundation\Response;

    class DefaultController
    {
        public function hello(): Response
        {
            return new Response(
                '<html><body>Hello World!</body></html>'
            );
        }
    }
    ```
  在config目录中的路由配置文件中增加一个路由：

    ```yaml
    # config/routes.yaml

    app_hello:
        path: /default/hello
        controller: App\Controller\DafualtController::hello
    ```
  最后在浏览器中访问配置好的地址，就可以看到我们写的“Hello World!”：

  {% asset_img 03.hello.png hello %}

<div id="refer-anchor-1"></div>

- [1] [Symfony Blog](https://symfony.com/blog/symfony-5-1-1-released)

<div id="refer-anchor-2"></div>

- [2] [Symfony CLI](https://symfony.com/download)
