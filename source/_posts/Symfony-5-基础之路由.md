---
title: Symfony 5 基础之路由
date: 2020-07-25 09:53:41
tags:
 - PHP
 - Symfony
---

在任何一个web框架中，路由都是其最基础也是最核心的功能。Symfony 的路由功能十分强大且灵活，我们可以用它非常方便的创建一个美观的路由。今天我就基于 Symfony 的官方文档，完整地讲讲 Symfony 路由功能的使用。

### <span id="what-is-a-route">什么是路由</span>

当我们的应用程序接收到一个请求后，就会执行一个控制器方法来创建响应信息。路由功能定义了通过哪一个具体的控制器方法，来响应不同URL的请求。形象的讲，路由就是一个工头，负责把不同的任务，分配给不同岗位的工人完成。配置路由其实就是在配置工头的分配规则。当然，Symfony 允许我们在路由功能基础之上，做一些额外的事情，比如安全校验、路由美化等。

<!-- more -->

### <span id="creating-routes">创建路由</span>

Symfony 的路由功能十分灵活，我们可以使用传的方式，在不同类型的文件中配置路由，如YAML, XML, PHP。也可以使用 annotations（注解路由）来定义一个路由。

#### <span id="creating-routes-as-annotations">Annotations 方式创建路由</span>

在使用 annotations 前，需要在命令行安装 annotations 依赖：

`composer require annotations`

在安装完依赖后，执行的命令会创建一个新的配置文件：

```yaml
# config/routes/annotations.yaml
controllers:
    resource: '../../src/Controller/'
    type: annotation

kernel:
    resource: ../../src/Kernel.php
    type: annotation
```

这个配置文件定义了 `src/Controller` 目录下的控制器，都可以使用 annotations 方式配置路由。

在 `src/Controller` 目录中创建一个新的PHP文件，并写入下面这些内容：

```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class RouterFuncController extends AbstractController
{
    # "/router"代表路由的URL routers_test代表路由名称 test()为路由所指向的控制器方法
    /**
     * @Route("/router", name="routers_test")
     */
    public function test(): Response
    {
        return new Response(
            '<html><body>This is router test.</body></html>'
        );
    }
}
```

在浏览器中打开 `http://symfony_dev.vm/router` 这个地址，就可以看到在Response中所写的内容了。

这个配置定义了一个名为 `routers_test` 的路由，当用户请求 `/router` URL时就会匹配到它。当匹配发生时，应用会调用 `RouterFuncController` 类中的 `test()` 方法。

> 这个路由不仅仅会匹配 `http://symfony_dev.vm/router`，还会匹配 `http://symfony_dev.vm/router?foo=bar` 之类的URL。

路由名 `routers_test` 现在还没有用到，但是当使用 [生成 URL](#generating-urls) 功能时，它是必须的。一定要注意，每个路由的名称都必须是“唯一”的。

#### <span id="creating-routes-in-files">以文件（YAML,XML,PHP）的方式创建路由</span>

使用配置文件配置路由不需要安装任何依赖，直接配置对应格式的文件即可：

```yaml
# config/routes.yaml
# routers_test为路由名称
routers_test:
    # /router为URL路径
    path: /router
    # controller值的格式为：控制器类名::方法名
    controller: App\Controller\RouterFuncController::test
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">
    <route id="routers_test" path="/router"
           controller="App\Controller\RouterFuncController::test"/>
</routes>
```

```php
// config/routes.php
use App\Controller\RouterFuncController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('routers_test', '/router')
        ->controller([RouterFuncController::class, 'test'])
    ;
};
```

可以看到，相对于用配置文件的方式配置路由，annotations 方式，明显更简洁一些。之前本人用过的一些框架都是在配置文件中配置路由，后来看到 Symfony 这种在注解中写路由的方式还感觉挺奇怪的。但是随着时间的推移，写的路由越来越多，就会发现 annotations 这种方式的优势：路由规则与控制器方法是一体的，所以不用专门写很多的配置文件。当需要对路由或方法的功能做修改，或是排查出现的 bug 时，不用在各种文件中跳来跳去，非常的方便。而且 annotations 还支持一些特有的功能，这些功能在某些场景下非常实用，后面会提到。

如果在使用 Symfony 时需要配置路由，个人推荐使用 annotations 这种方式。假如出于习惯，你仍然希望使用配置文件的方式配置路由，推荐优先使用 Symfony 推荐的配置文件格式：YAML。

之后的路由代码示例，默认都会包含 annotations、YAML、XML、PHP这四种方式（某些与定义路由无直接关系的配置将会不包含 annotations 方式），代码展示顺序也是依次的。

#### <span id="matching-http-methods">匹配HTTP方法</span>

Symfony 默认的路由会匹配所有类型的方法，可以增加一个 `methods` 属性选项，对请求的方法做限制：

```php
// src/Controller/BlogApiController.php
namespace App\Controller;

// ...

class BlogApiController extends AbstractController
{
    /**
     * @Route("/api/posts/{id}", methods={"GET","HEAD"})
     */
    public function show(int $id)
    {
        // ... 以json格式返回一篇文章的详情
    }

    /**
     * @Route("/api/posts/{id}", methods={"PUT"})
     */
    public function edit(int $id)
    {
        // ... 编辑一篇文章的数据
    }
}
```

```yaml
# config/routes.yaml
api_post_show:
    path:       /api/posts/{id}
    controller: App\Controller\BlogApiController::show
    methods:    GET|HEAD

api_post_edit:
    path:       /api/posts/{id}
    controller: App\Controller\BlogApiController::edit
    methods:    PUT
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="api_post_show" path="/api/posts/{id}"
        controller="App\Controller\BlogApiController::show"
        methods="GET|HEAD"/>

    <route id="api_post_edit" path="/api/posts/{id}"
        controller="App\Controller\BlogApiController::edit"
        methods="PUT"/>
</routes>
```

```php
// config/routes.php
use App\Controller\BlogApiController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('api_post_show', '/api/posts/{id}')
        ->controller([BlogApiController::class, 'show'])
        ->methods(['GET', 'HEAD'])
    ;
    $routes->add('api_post_edit', '/api/posts/{id}')
        ->controller([BlogApiController::class, 'edit'])
        ->methods(['PUT'])
    ;
};
```

#### <span id="matching-expressions">匹配表达式</span>

也可以使用 `condition` 选项来实现一些任意匹配逻辑，比如匹配请求头中的 `User-Agent`：

```php
// src/Controller/DefaultController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class DefaultController extends AbstractController
{
    /**
     * @Route(
     *     "/contact",
     *     name="contact",
     *     condition="context.getMethod() in ['GET', 'HEAD'] and request.headers.get('User-Agent') matches '/firefox/i'"
     * )
     *
     * 表达式中也可以包含配置参数:
     * condition: "request.headers.get('User-Agent') 匹配 '%app.allowed_browsers%'"
     */
    public function contact()
    {
        // ...
    }
}
```

```yaml
# config/routes.yaml
contact:
    path:       /contact
    controller: 'App\Controller\DefaultController::contact'
    condition:  "context.getMethod() in ['GET', 'HEAD'] and request.headers.get('User-Agent') matches '/firefox/i'"
    # 表达式中也可以包含配置参数:
    # condition: "request.headers.get('User-Agent') 匹配 '%app.allowed_browsers%'"
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="contact" path="/contact" controller="App\Controller\DefaultController::contact">
        <condition>context.getMethod() in ['GET', 'HEAD'] and request.headers.get('User-Agent') matches '/firefox/i'</condition>
        <!-- 表达式中也可以包含配置参数: -->
        <!-- <condition>request.headers.get('User-Agent') 匹配 '%app.allowed_browsers%'</condition> -->
    </route>
</routes>
```

```php
// config/routes.php
use App\Controller\DefaultController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('contact', '/contact')
        ->controller([DefaultController::class, 'contact'])
        ->condition('context.getMethod() in ["GET", "HEAD"] and request.headers.get("User-Agent") matches "/firefox/i"')
        // 表达式中也可以包含配置参数:
        // 'request.headers.get("User-Agent") 匹配 "%app.allowed_browsers%"'
    ;
};
```

+ `context`
    是 `Symfony\Component\Routing\RequestContext`的一个对象，它囊括了与路由匹配相关的大多数重要信息。
+ `request`
    [Symfony Request](https://symfony.com/doc/current/components/http_foundation.html#component-http-foundation-request) 对象代表当前请求。

在应用真正运行的时候，这些表达式将会被编译成原生的PHP代码。所以不用担心使用 `condition` 后产生额外的时间开销。

> 在使用创建URL功能的时候，`condition` 是无效的。

#### <span id="debugging-routes">调试路由</span>

随着我们对应用开发维护工作的进行，应用中的路由也会越来越多。Symfony 提供了一些命令，以供调试路由使用。首先，`debug:router` 命令可以列出应用内的所有路由，顺序是 Symfony 评估它们的顺序：

```bash
> php bin/console debug:router

----------------  -------  -------  -----  --------------------------------------------
Name              Method   Scheme   Host   Path
----------------  -------  -------  -----  --------------------------------------------
homepage          ANY      ANY      ANY    /
contact           GET      ANY      ANY    /contact
contact_process   POST     ANY      ANY    /contact
article_show      ANY      ANY      ANY    /articles/{_locale}/{year}/{title}.{_format}
blog              ANY      ANY      ANY    /blog/{page}
blog_show         ANY      ANY      ANY    /blog/{slug}
----------------  -------  -------  -----  --------------------------------------------
```

传入路由的名称或是部分名称，可以打印出该路由的详情：

```bash
> php bin/console debug:router app_lucky_number

+-------------+---------------------------------------------------------+
| Property    | Value                                                   |
+-------------+---------------------------------------------------------+
| Route Name  | app_lucky_number                                        |
| Path        | /lucky/number/{max}                                     |
| ...         | ...                                                     |
| Options     | compiler_class: Symfony\Component\Routing\RouteCompiler |
|             | utf8: true                                              |
+-------------+---------------------------------------------------------+
```

还有一个命令是 `router:match`，它能够显示匹配所给URL的路由。对于找出那些请求URL不能够匹配到我们所期望的控制器方法的路由，是非常有用的。

```bash
> php bin/console router:match /lucky/number/8

  [OK] Route "app_lucky_number" matches
```

### <span id="route-parameters">路由参数</span>

前面举的例子，使用的都是不可变的 `静态路由`。换句话说，就是定义了什么URL规则，就只能匹配什么样的URL规则。看起来好像没什么问题，但是在实际的开发过程中，我们经常需要某些路由中能够包含一些变量。比如说，就像我现在用的Hexo 搭建的博客，每篇文章的标题都是被包含在请求的URL中的。我们不可能为每一篇文章创建一个路由，这时候就需要创建一个动态路由（即路由中包含参数）来访问每一篇文章。

在Symfony 的路由中，变量要由一对大括号包裹起来，并且要有一个唯一名称。举个例子，一个显示博客文章的路由，就可以定义成 `/blog/{slug}`：

```php
// src/Controller/BlogController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends AbstractController
{
    // ...

    /**
     * @Route("/blog/{slug}", name="blog_show")
     */
    public function show(string $slug)
    {
        // $slug 能够匹配URL中动态的部分
        // 比如请求 /blog/yay-routing, 此时 $slug='yay-routing'

        // ...
    }
}
```

```yaml
# config/routes.yaml
blog_show:
    path:       /blog/{slug}
    controller: App\Controller\BlogController::show
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="blog_show" path="/blog/{slug}"
           controller="App\Controller\BlogController::show"/>
</routes>
```

```php
// config/routes.php
use App\Controller\BlogController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('blog_show', '/blog/{slug}')
        ->controller([BlogController::class, 'show'])
    ;
};

```

变量部分（`{slug} 在这个例子中`）的名称被用来在路由存储和传递给控制器的地方，创建一个 PHP 变量。如果一个用户访问 `/blog/my-first-post` 这个 URL，Symfony 将会在 `BlogController` 类中执行 `show()` 方法，并且传递一个 `$slug = 'my-first-post` 参数到 `show()` 方法中。

路由可以定义任意数量的参数，但是它们中的每一个都只能在各自的路由中使用一次（例如 `/blog/posts-about-{category}/page/{pageNumber}`）。

#### <span id="parameters-validation">参数校验</span>

想象一下你的应用程序有一个 `blog_show` 路由（URL：`/blog/{slug}`）和一个 `blog_list` 路由（URL：`/blog/{page}`）。鉴于路由的参数可以接受任何值，就没有办法区分这两个路由。

如何用户访问了 `/blog/my-first-post`，两个路由都将会匹配并且 Symfony 会使用先被定义的那个路由。要解决这个问题，使用 `requirements` 选项给 `{page}` 参数添加一些校验规则：

```php
// src/Controller/BlogController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends AbstractController
{
    /**
     * @Route("/blog/{page}", name="blog_list", requirements={"page"="\d+"})
     */
    public function list(int $page)
    {
        // ...
    }

    /**
     * @Route("/blog/{slug}", name="blog_show")
     */
    public function show($slug)
    {
        // ...
    }
}
```

```yaml
# config/routes.yaml
blog_list:
    path:       /blog/{page}
    controller: App\Controller\BlogController::list
    requirements:
        page: '\d+'

blog_show:
    path:       /blog/{slug}
    controller: App\Controller\BlogController::show
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="blog_list" path="/blog/{page}" controller="App\Controller\BlogController::list">
        <requirement key="page">\d+</requirement>
    </route>

    <route id="blog_show" path="/blog/{slug}"
           controller="App\Controller\BlogController::show"/>
</routes>
```

```php
// config/routes.php
use App\Controller\BlogController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('blog_list', '/blog/{page}')
        ->controller([BlogController::class, 'list'])
        ->requirements(['page' => '\d+'])
    ;

    $routes->add('blog_show', '/blog/{slug}')
        ->controller([BlogController::class, 'show'])
    ;
    // ...
};
```

`requirements` 选项的定义使用了[PHP正则表达式](https://www.php.net/manual/zh/book.pcre.php)这种方式，路由的参数匹配对于匹配整个路由是必须的。在上面的例子中， `\d+` 在正则中表示匹配任意长度的*数字*。

| URL | Route | Parameters |
| --- | --- | --- |
| /blog/2 | blog_list | $page = 2 |
| /blog/my-first-post | blog_show | $slug = my-first-post |

> 路由的 `requirements` 选项中，也可以包含[容器参数](https://symfony.com/doc/current/configuration.html#configuration-parameters)。“容器参数”都是定义在配置文件中，在此期间可以一次性定义一些复杂的正则表达式内容，然后我们就可以在不同的路由中复用它。

> 参数也支持 [PCRE Unicode字符属性](https://www.php.net/manual/zh/regexp.reference.unicode.php)，能够匹配通用字符类型的转义序列。比如，`\p{Lu}` 可以匹配任意语言中所有的大写字符，`\p{Greek}` 可以匹配任意的希腊字符。

> 在路由参数中使用常规参数时，可以设置 `utf-8` 选项为 `true`， 然后就可以使用 `.` 字符匹配任意的 UTF-8 字符，而不只是单字节的字符。

如果你愿意，也可以使用 `{参数名<条件>}` 这种语法，将各个参数变为内联样式。这个功能可以让配置更简洁，不过当条件比较复杂的时候，会降低路由的可读性。

```php
// src/Controller/BlogController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends AbstractController
{
    /**
     * @Route("/blog/{page<\d+>}", name="blog_list")
     */
    public function list(int $page)
    {
        // ...
    }
}
```

```yaml
# config/routes.yaml
blog_list:
    path:       /blog/{page<\d+>}
    controller: App\Controller\BlogController::list
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="blog_list" path="/blog/{page<\d+>}"
           controller="App\Controller\BlogController::list"/>

    <!-- ... -->
</routes>
```

```php
// config/routes.php
use App\Controller\BlogController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('blog_list', '/blog/{page<\d+>}')
        ->controller([BlogController::class, 'list'])
    ;
    // ...
};
```

#### <span id="optional-parameters">可选参数</span>

在前面的例子里，路由 `blog_list` 的匹配条件是 `/blog/{page}`。如果用户访问 `/blog/1`，路由可以匹配。但是如果它们访问了 `/blog`，路由就不会匹配到。因为一旦给路由添加了一个参数，这个参数就必须要有一个值。

我们可以通过给 `{page}` 参数添加一个默认值的方式，来让用户访问 `/blog` 时匹配到 `blog_list` 路由。当使用 annotations 时，默认值可以定义到控制器方法的参数中。在其他的配置格式中，需要将默认参数配置到 `dafault` 选项中。

```php
// src/Controller/BlogController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends AbstractController
{
    /**
     * @Route("/blog/{page}", name="blog_list", requirements={"page"="\d+"})
     */
    public function list(int $page = 1)
    {
        // ...
    }
}
```

```yaml
# config/routes.yaml
blog_list:
    path:       /blog/{page}
    controller: App\Controller\BlogController::list
    defaults:
        page: 1
    requirements:
        page: '\d+'

blog_show:
    # ...
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="blog_list" path="/blog/{page}" controller="App\Controller\BlogController::list">
        <default key="page">1</default>

        <requirement key="page">\d+</requirement>
    </route>

    <!-- ... -->
</routes>
```

```php
// config/routes.php
use App\Controller\BlogController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('blog_list', '/blog/{page}')
        ->controller([BlogController::class, 'list'])
        ->defaults(['page' => 1])
        ->requirements(['page' => '\d+'])
    ;
};
```

现在当用户访问 `/blog`时，`blog_list` 路由也能够匹配到，并且此时 `$page` 为默认值1。

> 我们也可以使用多个可选参数，例如 `/blog/{slug}/{page}`，但是可选参数之后的所有内容都必须是可选的。例如，`/{page}/blog` 是一个合法路径，但是 `page` 参数将会是必须的（`/blog` 就不会匹配这个路由）。

如果要在生成的 URL 中始终包含一些默认值（例如，强制生成 `/blog/1` 而不是上一示例中的 `/blog`），需要在参数名称钱添加 `!` 字符：`/blog/{!page}`

在条件中，还可以使用语法 `{参数名<条件>}` 让每个参数中使用内联默认值。这个功能与内联条件是兼容的，所以可以在一个参数中内联这两个功能：

```php
// src/Controller/BlogController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends AbstractController
{
    /**
     * @Route("/blog/{page<\d+>?1}", name="blog_list")
     */
    public function list(int $page)
    {
        // ...
    }
}
```

```yaml
# config/routes.yaml
blog_list:
    path:       /blog/{page<\d+>?1}
    controller: App\Controller\BlogController::list
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="blog_list" path="/blog/{page<\d+>?1}"
           controller="App\Controller\BlogController::list"/>

    <!-- ... -->
</routes>
```

```php
// config/routes.php
use App\Controller\BlogController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('blog_list', '/blog/{page<\d+>?1}')
        ->controller([BlogController::class, 'list'])
    ;
};
```

> 如果想用 `null` 作为默认值，就在 `?` 字符后什么都不加（例如 `/blog/{page?}`）。

#### <span id="priority-parameter">优先级参数</span>

> 5.1 版本新特性：`priority` 参数在 Symfony 5.1 中首次被引入。

当我们定义一个可以匹配许多路由的贪婪模式后，很可能在同类路由集合的一开始就匹配到，然后阻止了在此之后定义的任何同类路由被匹配。增加一个 `priority` 选项参数可以让我们选择路由的顺序，不过这个功能只在使用 annotations 方式时可用。

```php
// src/Controller/BlogController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends AbstractController
{
    /**
     * 这个路由使用了贪婪模式，并且被首先定义了。
     *
     * @Route("/blog/{slug}", name="blog_show")
     */
    public function show(string $slug)
    {
        // ...
    }

    /**
     * 如果不定义一个比0更高的优先级，这个路由就不能被访问到。
     *
     * @Route("/blog/list", name="blog_list", priority=2)
     */
    public function list()
    {
        // ...
    }
}
```

配置优先级参数需要使用一个整型值。更高优先级的路由比优先级低的路由匹配序列更高。如果优先级参数没有定义，那么默认值为0。

#### <span id="parameter-conversion">参数转换</span>

我们经常会遇到这样一种场景：将一个普通路由中的参数转换成另一种类型的值（例如，将一个标识用户的ID转换成用户对象）。Symfony 提供了这种叫做“参数转换”的功能，并且这个功能仍然只在使用 annotations
时可用。

将前一个路由配置中控制器方法的参数变一下，将 `string $slug` 变为 `BlogPost $post`：

```php
// src/Controller/BlogController.php
namespace App\Controller;

use App\Entity\BlogPost;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends AbstractController
{
    // ...

    /**
     * @Route("/blog/{slug}", name="blog_show")
     */
    public function show(BlogPost $post)
    {
        // $post is the object whose slug matches the routing parameter

        // ...
    }
}
```

如果控制器参数中包含了对象类型（此例中的 `BlogPost`）的类型约束，“参数转换”功能就会利用请求参数（此例中的 `slug`），发出一个数据库请求来查询这个参数代表的对象。如果找不到对象，Symfony 会自动创建一个 404 响应。怎么样，是不是感到非常的方便与智能。

想要知道更多的转换功能以及如何配置它们，可以阅读完整的[参数转换文档](https://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/annotations/converters.html)。

#### <span id="special-parameters">特殊参数</span>

除了我们自己定义的参数，路由中还包含一些 Symfony 创建的特殊参数：

+ `_controller`
    这个参数决定了当一个路由被匹配到时执行哪个控制器和方法。

+ `_format`
    匹配的值用于设置 `Request` 对象的“请求格式”。用来做设置响应的 `Content-Type`（例如，`json` 格式转换为 `application/json` 类型的`Content-Type`）之类的事情。

+ `_fragment`
    用于设置片段标识符，这是 URL 的最后一部分，是以 `#` 字符开头并且用于标识文档的一部分（其实就是html的锚点）。

+ `_locale`
    用于设置请求的[locale](https://symfony.com/doc/current/translation/locale.html#translation-locale-url)信息。

我们可以在一个路由导入的地方引入这些属性（除了 `_fragment`）。Symfony 定义了一些使用同样名称的特殊属性（没有前导下划线），以便你更方便的定义它们：

```php
// src/Controller/ArticleController.php
namespace App\Controller;

// ...
class ArticleController extends AbstractController
{
    /**
     * @Route(
     *     "/articles/{_locale}/search.{_format}",
     *     locale="en",
     *     format="html",
     *     requirements={
     *         "_locale": "en|fr",
     *         "_format": "html|xml",
     *     }
     * )
     */
    public function search()
    {
    }
}
```

```yaml
# config/routes.yaml
article_search:
  path:        /articles/{_locale}/search.{_format}
  controller:  App\Controller\ArticleController::search
  locale:      en
  format:      html
  requirements:
      _locale: en|fr
      _format: html|xml
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="article_search"
        path="/articles/{_locale}/search.{_format}"
        controller="App\Controller\ArticleController::search"
        locale="en"
        format="html">

        <requirement key="_locale">en|fr</requirement>
        <requirement key="_format">html|rss</requirement>

    </route>
</routes>
```

```php
// config/routes.php
namespace Symfony\Component\Routing\Loader\Configurator;

use App\Controller\ArticleController;

return function (RoutingConfigurator $routes) {
    $routes->add('article_show', '/articles/{_locale}/search.{_format}')
        ->controller([ArticleController::class, 'search'])
        ->locale('en')
        ->format('html')
        ->requirements([
            '_locale' => 'en|fr',
            '_format' => 'html|rss',
        ])
    ;
};
```

#### <span id="extra-parameters">额外参数</span>

我们可以在路由的 `defaults` 选项中定义一些不在路由配置中的参数。这可以用来给路由所在的控制器传递额外的参数。

```php
// src/Controller/BlogController.php
namespace App\Controller;

use Symfony\Component\Routing\Annotation\Route;

class BlogController
{
    /**
     * @Route("/blog/{page}", name="blog_index", defaults={"page": 1, "title": "Hello world!"})
     */
    public function index(int $page, string $title)
    {
        // ...
    }
}
```

```yaml
# config/routes.yaml
blog_index:
    path:       /blog/{page}
    controller: App\Controller\BlogController::index
    defaults:
        page: 1
        title: "Hello world!"
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="blog_index" path="/blog/{page}" controller="App\Controller\BlogController::index">
        <default key="page">1</default>
        <default key="title">Hello world!</default>
    </route>
</routes>
```

```php
// config/routes.php
use App\Controller\BlogController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('blog_index', '/blog/{page}')
        ->controller([BlogController::class, 'index'])
        ->defaults([
            'page'  => 1,
            'title' => 'Hello world!',
        ])
    ;
};
```

#### <span id="slash-characters-in-route-parameters">路由参数中的斜线字符</span>

路由参数中可以包含除 `/` 斜线字符外的所有字符，因为斜线字符一般用来将 URL 分割成不同的部分。例如，如果 `/share/{token}` 路由中的 `token` 值包含一个 `/` 字符，这个路由就不会匹配。

一个可行的解决方案就是修改参数条件，来接受更多可能的字符。

```php
// src/Controller/DefaultController.php
namespace App\Controller;

use Symfony\Component\Routing\Annotation\Route;

class DefaultController
{
    /**
     * @Route("/share/{token}", name="share", requirements={"token"=".+"})
     */
    public function share($token)
    {
        // ...
    }
}
```

```yaml
# config/routes.yaml
share:
    path:       /share/{token}
    controller: App\Controller\DefaultController::share
    requirements:
        token: .+
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="share" path="/share/{token}" controller="App\Controller\DefaultController::share">
        <requirement key="token">.+</requirement>
    </route>
</routes>
```

```php
// config/routes.php
use App\Controller\DefaultController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('share', '/share/{token}')
        ->controller([DefaultController::class, 'share'])
        ->requirements([
            'token' => '.+',
        ])
    ;
};
```

> 如果路由定义了多个参数，并且将这些“宽容”正则表达式应用于所有参数，你很可能得到不期待的结果。例如，如果你定义的路由是 `/share/{path}/{token}` 并且 `path` 和 `token` 都接受 `/`。`token` 将会只能获取到最后的路径，并且其余的部分会被第一个参数（`path`）所匹配。

> 如果路由中包含了特殊的 `{_format}` 参数，你就不应该在允许斜线的参数中使用 `.+` 条件。例如，如果匹配模式为 `/share/{token}.{_format}` 并且 `{token}` 允许任意字符， `/share/foo/bar.json` 这个URL将会把 `foo/bar.json`作为 `token`，而 `format` 将会是空的。要解决这个问题，可以通过将条件从 `.+` 替换为 `[^.]+` 来允许除 `.` 以外的字符。

### <span id="route-groups-and-prefixes">路由组和前缀</span>

我们通常会让一组路由共享一些选项（例如所有的关于文章的路由都应该以 `/blog` 开头），这就是为什么 Symfony 会有一个共享路由配置的功能。

在使用 annotations 方式定义路由时，把通用配置放到控制器类的 `@Route` 注释中。其他的路由格式，在输入路由的选项定义通用配置。

```php
// src/Controller/BlogController.php
namespace App\Controller;

use Symfony\Component\Routing\Annotation\Route;

/**
 * @Route("/blog", requirements={"_locale": "en|es|fr"}, name="blog_")
 */
class BlogController
{
    /**
     * @Route("/{_locale}", name="index")
     */
    public function index()
    {
        // ...
    }

    /**
     * @Route("/{_locale}/posts/{slug}", name="show")
     */
    public function show(Post $post)
    {
        // ...
    }
}
```

```yaml
# config/routes/annotations.yaml
controllers:
    resource: '../../src/Controller/'
    type: annotation
    # 这个配置会将 '/blog' 添加到到所有输入路由 URL 的开头
    prefix: '/blog'
    # 这个配置会将 'blog_' 添加到所有输入路由名称的开头
    name_prefix: 'blog_'
    # 这些条件会添加到所有的输入路由
    requirements:
        _locale: 'en|es|fr'
    # 一个空 URL 的输入路由会变成 "/blog/"
    # 注释这个选项将会使用取消尾随斜线的 "/blog" 代替它
    # trailing_slash_on_root: false
    # 在使用 annotations 的时候 需要排除的文件或子目录
    # exclude: '../../src/Controller/{DebugEmailController}.php'
```

```xml
<!-- config/routes/annotations.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <!--
        'prefix' 将会被添加到所有输入路由 URL 的开头
        'name-prefix' 将会被添加到所有输入路由名称的开头
        'exclude' 定义了当加载 annotations 时排除的文件和子目录
    -->
    <import resource="../../src/Controller/"
        type="annotation"
        prefix="/blog"
        name-prefix="blog_"
        exclude="../../src/Controller/{DebugEmailController}.php">
        <!-- 这些条件会被添加到所有的输入路由中 -->
        <requirement key="_locale">en|es|fr</requirement>
    </import>

    <!-- 一个空 URL 的输入路由会是 "/blog/"
         注释这个选项将会用 URL "/blog" 代替 -->
    <import resource="../../src/Controller/" type="annotation"
            prefix="/blog"
            trailing-slash-on-root="false">
            <!-- ... -->
    </import>
</routes>
```

```php
// config/routes/annotations.php
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    // 使用 import() 方法的第五个参数 来定义在使用 annotations 时需要
    // 排除的文件和子目录
    $routes->import('../../src/Controller/', 'annotation')
        // this is added to the beginning of all imported route URLs
        ->prefix('/blog')
        // 一个空 URL 的输入路由会是 "/blog/"
        // 传递 FALSE 作为第二个参数将会用 URL "/blog" 替代它
        // ->prefix('/blog', false)
        // 这个前缀将会添加到所有输入路由的开头
        ->namePrefix('blog_')
        // 这些条件将会添加到所有的输入路由中
        ->requirements(['_locale' => 'en|es|fr'])
    ;
};
```

在这个例子当中，`index()` 方法的路由名称将会被定义为 `blog_index`，并且它的 URL 将会是 `/blog`。`show()` 方法的路由名称会被定义为 `blog_show`，并且它的 URL 会是 `/blog/{_locale}/posts/{slug}`。这两个路由都会包含定义在类注释中匹配正则表达式的 `_locale` 参数。

*Symfony 从[不同的来源](https://symfony.com/doc/current/routing/custom_route_loader.htmlppp)加载路由，你甚至可以自己创建一个路由的加载器。*

### <span id="getting-the-route-name-and-parameters">获取路由名称和参数</span>

Symfony 创建的 `Request` 对象会在 “请求属性” 中储存所有的路由配置（例如名称和参数）。你可以在一个控制器中，通过 `Request` 对象获取到这些信息：

```php
// src/Controller/BlogController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends AbstractController
{
    /**
     * @Route("/blog", name="blog_list")
     */
    public function list(Request $request)
    {
        // ...

        $routeName = $request->attributes->get('_route');
        $routeParameters = $request->attributes->get('_route_params');

        // 使用下面的方法就可以获取到所有的可用属性 (不仅仅是路由信息):
        $allAttributes = $request->attributes->all();
    }
}
```

你可以通过注入 `request_stack` 服务，从而[在一个服务中获取 `Request` 对象](https://symfony.com/doc/current/service_container/request.html)，然后就能在服务中获取到这些信息。在模板中，使用[Twig 全局应用变量](https://symfony.com/doc/current/templates.html#twig-app-variable)来获取请求和它的属性：

```twig
{% set route_name = app.request.attributes.get('_route') %}
{% set route_parameters = app.request.attributes.get('_route_params') %}

{# use this to get all the available attributes (not only routing ones) #}
{% set all_attributes = app.request.attributes.all %}
```

### <span id="special-routes">特殊路由</span>

Symfony 定义了一些特殊控制器，用来渲染模板和重定向到路由配置的其他路由，所以你不必创建一个控制器方法。

#### <span id="rendering-a-template-directly-from-a-route">直接从一个路由渲染模板</span>

在有关 Symfony 模板的主文章中，阅读关于[从一个路由渲染模板](https://symfony.com/doc/current/templates.html#templates-render-from-route)的章节。

#### <span id="redirecting-to-urls-and-routes-directly-from-a-route">直接从一个路由重定向到 URL 和路由</span>

使用 `RedirectController` 来重定向到其他的路由和 URL：

```yaml
# config/routes.yaml
doc_shortcut:
    path: /doc
    controller: Symfony\Bundle\FrameworkBundle\Controller\RedirectController
    defaults:
        route: 'doc_page'
        # 你可以选择定义一些参数传递到路由中
        page: 'index'
        version: 'current'
        # 默认情况下，重定向是暂时的 (code 302) 但是你也可以让它们永久化 (code 301)
        permanent: true
        # 添加这个配置，可以让重定向时，保持原查询字符串参数
        keepQueryParams: true
        # 添加这个配置，来让当重定向时，保持请求的 HTTTP 方法。重定向的状态将会发生改变
        # * 对于临时重定向，将会用 307 状态码代替 302
        # * 对于永久重定向，将会用 308 状态码代替 301
        keepRequestMethod: true

legacy_doc:
    path: /legacy/doc
    controller: Symfony\Bundle\FrameworkBundle\Controller\RedirectController
    defaults:
        # 这个值可以是一个绝对路径或是绝对 URL
        path: 'https://legacy.example.com/doc'
        permanent: true
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="doc_shortcut" path="/doc"
           controller="Symfony\Bundle\FrameworkBundle\Controller\RedirectController">
        <default key="route">doc_page</default>
        <!-- 你可以选择定义一些参数传递到路由中 -->
        <default key="page">index</default>
        <default key="version">current</default>
        <!-- 默认情况下，重定向是暂时的 (code 302) 但是你也可以让它们永久化 (code 301)-->
        <default key="permanent">true</default>
        <!-- 添加这个配置，可以让重定向时，保持原查询字符串参数 -->
        <default key="keepQueryParams">true</default>
        <!-- 添加这个配置，来让当重定向时，保持请求的 HTTTP 方法。重定向的状态将会发生改变：
             * 对于临时重定向，将会用 307 状态码代替 302
             * 对于永久重定向，将会用 308 状态码代替 301 -->
        <default key="keepRequestMethod">true</default>
    </route>

    <route id="legacy_doc" path="/legacy/doc"
           controller="Symfony\Bundle\FrameworkBundle\Controller\RedirectController">
        <!-- 这个值可以是一个绝对路径或是绝对 URL -->
        <default key="path">https://legacy.example.com/doc</default>
        <!-- 默认情况下，重定向是暂时的 (code 302) 但是你也可以让它们永久化 (code 301)-->
        <default key="permanent">true</default>
    </route>
</routes>
```

```php
// config/routes.php
use App\Controller\DefaultController;
use Symfony\Bundle\FrameworkBundle\Controller\RedirectController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('doc_shortcut', '/doc')
        ->controller(RedirectController::class)
         ->defaults([
            'route' => 'doc_page',
            // 你可以选择定义一些参数传递到路由中
            'page' => 'index',
            'version' => 'current',
            // 默认情况下，重定向是暂时的 (code 302) 但是你也可以让它们永久化 (code 301)
            'permanent' => true,
            // 添加这个配置，可以让重定向时，保持原查询字符串参数
            'keepQueryParams' => true,
            // 添加这个配置，来让当重定向时，保持请求的 HTTTP 方法。重定向的状态将会发生改变：
            // * 对于临时重定向，将会用 307 状态码代替 302
            // * 对于永久重定向，将会用 308 状态码代替 301
            'keepRequestMethod' => true,
        ])
    ;

    $routes->add('legacy_doc', '/legacy/doc')
        ->controller(RedirectController::class)
         ->defaults([
            // 这个值可以是一个绝对路径或是绝对 URL
            'path' => 'https://legacy.example.com/doc',
            // 默认情况下，重定向是暂时的 (code 302) 但是你也可以让它们永久化 (code 301)
            'permanent' => true,
        ])
    ;
};
```

> Symfony 也提供了一些公共工具来实现[控制器内的重定向](https://symfony.com/doc/current/controller.html#controller-redirect)

#### <span id="redirecting-urls-with-trailing-slashes">使用拖尾斜杠重定向</span>

从历史上看，URL 遵循 UNIX 约定，即为目录添加拖尾斜杠（例如 `https://example.com/foo/`），并且删除它们以引入文件（`https://example.com/foo`）。尽管为这两个 URL 提供不同的内容是可以的，但是现在我们一般将它们视作同样的 URL 并在它们之间做重定向。

Symfony 遵循了这个逻辑，在具有拖尾斜杠的 URL 之间重定向（但只限 GET 和 HEAD 请求）：

| Route URL | 如果请求的 URL 是 /foo | 如果请求的 URL 是 /foo/ |
| --- | --- | --- |
| /foo | 匹配 (200 状态响应) | 做一个 301 重定向到 /foo |
| /foo/ | 做一个 301 重定向到 /foo/ | 匹配 (200 状态响应) |

### <span id="sub-domain-routing">子域名路由</span>

路由可以配置一个 `host` 选项来要求访问请求的 HTTP 主机匹配一些特定的值。在接下来的例子里，两个域名匹配同样的路径（`/`），但只有它们中的一个会为特殊的主机域名做出响应：

```php
// src/Controller/MainController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class MainController extends AbstractController
{
    /**
     * @Route("/", name="mobile_homepage", host="m.example.com")
     */
    public function mobileHomepage()
    {
        // ...
    }

    /**
     * @Route("/", name="homepage")
     */
    public function homepage()
    {
        // ...
    }
}
```

```yaml
# config/routes.yaml
mobile_homepage:
    path:       /
    host:       m.example.com
    controller: App\Controller\MainController::mobileHomepage

homepage:
    path:       /
    controller: App\Controller\MainController::homepage
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="mobile_homepage"
        path="/"
        host="m.example.com"
        controller="App\Controller\MainController::mobileHomepage"/>

    <route id="homepage" path="/" controller="App\Controller\MainController::homepage"/>
</routes>
```

```php
// config/routes.php
use App\Controller\MainController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('mobile_homepage', '/')
        ->controller([MainController::class, 'mobileHomepage'])
        ->host('m.example.com')
    ;
    $routes->add('homepage', '/')
        ->controller([MainController::class, 'homepage'])
    ;
};
```

`host` 选项的值可以包含参数（对于多租户应用程序很有用）并且这些参数可以使用 `requirements` 做校验：

```php
// src/Controller/MainController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class MainController extends AbstractController
{
    /**
     * @Route(
     *     "/",
     *     name="mobile_homepage",
     *     host="{subdomain}.example.com",
     *     defaults={"subdomain"="m"},
     *     requirements={"subdomain"="m|mobile"}
     * )
     */
    public function mobileHomepage()
    {
        // ...
    }

    /**
     * @Route("/", name="homepage")
     */
    public function homepage()
    {
        // ...
    }
}
```

```yaml
# config/routes.yaml
mobile_homepage:
    path:       /
    host:       "{subdomain}.example.com"
    controller: App\Controller\MainController::mobileHomepage
    defaults:
        subdomain: m
    requirements:
        subdomain: m|mobile

homepage:
    path:       /
    controller: App\Controller\MainController::homepage
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="mobile_homepage"
        path="/"
        host="{subdomain}.example.com"
        controller="App\Controller\MainController::mobileHomepage">
        <default key="subdomain">m</default>
        <requirement key="subdomain">m|mobile</requirement>
    </route>

    <route id="homepage" path="/" controller="App\Controller\MainController::homepage"/>
</routes>
```

```php
// config/routes.php
use App\Controller\MainController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('mobile_homepage', '/')
        ->controller([MainController::class, 'mobileHomepage'])
        ->host('{subdomain}.example.com')
        ->defaults([
            'subdomain' => 'm',
        ])
        ->requirements([
            'subdomain' => 'm|mobile',
        ])
    ;
    $routes->add('homepage', '/')
        ->controller([MainController::class, 'homepage'])
    ;
};
```

在上面这些例子中，`subdomain` 参数定义了一个默认值，因为如果不这样做，你每次用这些路由生成 URL 都需要包含一个域名。

> 你也可以在引入路由组的时设置 `host` 选项，从而使所有的路由都要求提供限定的主机名。

> 当使用子域名路由时，如果是在进行[功能测试](https://symfony.com/doc/current/testing.html)，你就必须设置主机的 HTTP 头，否则路由是不会被匹配到的。
> ```php
$crawler = $client->request(
    'GET',
    '/',
    [],
    [],
    ['HTTP_HOST' => 'm.example.com']
    // or get the value from some container parameter:
    // ['HTTP_HOST' => 'm.' . $client->getContainer()->getParameter('domain')]
);
```

### <span id="localized_routes">国际化路由（i18n）</span>

如果我们的应用程序中使用了多语言，每一个路由都为每个[翻译区域](https://symfony.com/doc/current/translation/locale.html)定义一个不同的 URL。这个功能避免了重复的路由，也减少了潜在的错误：

```php
// src/Controller/CompanyController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class CompanyController extends AbstractController
{
    /**
     * @Route({
     *     "en": "/about-us",
     *     "nl": "/over-ons"
     * }, name="about_us")
     */
    public function about()
    {
        // ...
    }
}
```

```yaml
# config/routes.yaml
about_us:
    path:
        en: /about-us
        nl: /over-ons
    controller: App\Controller\CompanyController::about
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="about_us" controller="App\Controller\CompanyController::about">
        <path locale="en">/about-us</path>
        <path locale="nl">/over-ons</path>
    </route>
</routes>
```

```php
// config/routes.php
use App\Controller\CompanyController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('about_us', [
        'en' => '/about-us',
        'nl' => '/over-ons',
    ])
        ->controller([CompanyController::class, 'about'])
    ;
};
```

当本地化路由被匹配到时，Symfony 会在请求进入期间自动使用同样的区域设置。

> 当应用程序使用完整的 “语言 + 领地” 区域设置（例如 `fr_FR`, `fr_BE`），如果所有相关区域设置的 URL 都是相同的，路由将会只使用语言部分（例如 `fr`）来阻止相同的 URL 出现重复。

国际化应用程序的一个常见要求，是使用区域设置为所有的路由添加前缀。这可以通过为每个区域设置定义不同的前缀（如果你愿意的话，可以为默认区域设置设置一个空前缀）：

```yaml
# config/routes/annotations.yaml
controllers:
    resource: '../../src/Controller/'
    type: annotation
    prefix:
        en: '' # don't prefix URLs for English, the default locale
        nl: '/nl'
```

```xml
<!-- config/routes/annotations.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <import resource="../../src/Controller/" type="annotation">
        <!-- don't prefix URLs for English, the default locale -->
        <prefix locale="en"></prefix>
        <prefix locale="nl">/nl</prefix>
    </import>
</routes>
```

```php
// config/routes/annotations.php
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->import('../../src/Controller/', 'annotation')
        ->prefix([
            // don't prefix URLs for English, the default locale
            'en' => '',
            'nl' => '/nl'
        ])
    ;
};
```

### <span id="stateless-routes">无状态路由</span>

> 5.1 版本新特性：`stateless` 选项在 Symfony 5.1 中首次被引入。

某些情况下，当一个 HTTP 响应需要被缓存时，确保这件事发生是非常重要的。然而在请求过程中，无论 session 是何时开启的，Symfony 都会将响应转换为一个私有的不可缓存的响应。

获取更多细节，详见[HTTP Cache](https://symfony.com/doc/current/http_cache.html)。

路由可以配置一个 `stateless` 布尔值选项， 来声明路由匹配到的一个请求中 session 是不可用的。

```php
// src/Controller/MainController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class MainController extends AbstractController
{
    /**
     * @Route("/", name="homepage", stateless=true)
     */
    public function homepage()
    {
        // ...
    }
}
```

```yaml
# config/routes.yaml
homepage:
    controller: App\Controller\MainController::homepage
    path: /
    stateless: true
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">
    <route id="homepage" controller="App\Controller\MainController::homepage" path="/" stateless="true"/>
</routes>
```

```php
// config/routes.php
use App\Controller\MainController;
use Symfony\Bundle\FrameworkBundle\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('homepage', '/')
        ->controller([MainController::class, 'homepage'])
        ->stateless()
    ;
};
```

现在，如果 session 被使用了，应用程序会基于你的 `kernel.debug` 参数配置：\* `enable`：将会抛出一个 `Symfony\Component\HttpKernel\Exception\UnexpectedSessionUsageException` 异常，\* `disabled` 将会记录一个警告

它将会帮助你更好的理解并更容易修复应用程序中不期待的行为。

### <span id="generating-urls">生成 URL</span>

路由系统是双向的：1）它们将 URL 与控制器关联（如前几节所述）;2）它们为给定路由生成 URL。通过从路由生成 URL，可以允许你不用在 HTML 模板中手动写入 `<a href="...">`这样的代码。此外，如果某些路由的 URL 发生更改，你只需更新路由配置，所有链接都将更新。

为了生成一个路由，你需要指定路由的名称（例如 `blog_show`）以及被路由定义的参数值（例如 `slug = my-blog-post`）。

因此，每个路由都需要有一个内部名称，并且该名称在应用程序中必须是唯一的。如果你没有使用 `name` 选项显式地设置路由名称，Symfony 将基于控制器和动作生成自动的名称。

#### <span id="generating-urls-in-controllers">在控制器中生成 URL</span>

如果你的控制器继承自 [AbstractController](https://symfony.com/doc/current/controller.html#the-base-controller-class-services)，使用 `generateUrl()` 辅助方法：

```php
// src/Controller/BlogController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Component\Routing\Generator\UrlGeneratorInterface;

class BlogController extends AbstractController
{
    /**
     * @Route("/blog", name="blog_list")
     */
    public function list()
    {
        // ...

        // 生成一个不携带路由参数的 URL
        $signUpPage = $this->generateUrl('sign_up');

        // 生成一个携带路由参数的 URL
        $userProfilePage = $this->generateUrl('user_profile', [
            'username' => $user->getUsername(),
        ]);

        // 默认生成的URL都是 "绝对路径" 。 传递第三个可选
        // 参数来生成不同的 URL (例如一个 "absolute URL")
        $signUpPage = $this->generateUrl('sign_up', [], UrlGeneratorInterface::ABSOLUTE_URL);

        // 当一个路由是本地化的时， Symfony 默认使用当前请求的的区域设置
        // 如果你想显式的设置区域，需要传一个不同的 '_locale' 值
        $signUpPageInDutch = $this->generateUrl('sign_up', ['_locale' => 'nl']);
    }
}
```

> 如果你向 `generateUrl()` 方法传递了一些不属于路由定义的参数，这些参数会在生成的 URL 中作为查询字符串存在：
> ```php
$this->generateUrl('blog', ['page' => 2, 'category' => 'Symfony']);
// 'blog' 路由只定义了 'page' 参数； 生成的 URL 为：
// /blog/2?category=Symfony
```

如果你的控制器没有继承自 `AbstractController`，你需要在[控制器中获取服务](https://symfony.com/doc/current/controller.html#controller-accessing-services)，并按照下一节的说明进行操作。

#### <span id="generating-urls-in-sevices">在服务中生成 URL</span>

在你自己的服务中注入 `router` Symfony 服务就可以使用它的 `generate()` 方法。当使用[服务重写](https://symfony.com/doc/current/service_container/autowiring.html)时，你只需要在服务的构造方法中添加一个参数，并且使用 `Symfony\Component\Routing\Generator\UrlGeneratorInterface` 作为参数的类型约束：

```php
// src/Service/SomeService.php
namespace App\Service;

use Symfony\Component\Routing\Generator\UrlGeneratorInterface;

class SomeService
{
    private $router;

    public function __construct(UrlGeneratorInterface $router)
    {
        $this->router = $router;
    }

    public function someMethod()
    {
        // ...

        // 生成一个不携带路由参数的 URL
        $signUpPage = $this->router->generate('sign_up');

        // 生成一个携带路由参数的 URL
        $userProfilePage = $this->router->generate('user_profile', [
            'username' => $user->getUsername(),
        ]);

        // 默认生成的URL都是 "绝对路径" 。 传递第三个可选
        // 参数来生成不同的 URL (例如一个 "absolute URL")
        $signUpPage = $this->router->generate('sign_up', [], UrlGeneratorInterface::ABSOLUTE_URL);

        // 当一个路由是本地化的时， Symfony 默认使用当前请求的的区域设置
        // 如果你想显式的设置区域，需要传一个不同的 '_locale' 值
        $signUpPageInDutch = $this->router->generate('sign_up', ['_locale' => 'nl']);
    }
}
```

#### <span id="generating-urls-in-templates">在模板中生成 URL</span>

在 Symfony 模板的主文章中，阅读关于[在页面间创建链接](https://symfony.com/doc/current/templates.html#templates-link-to-pages)章节。

#### <span id="generating-urls-in-templates">在 JavaScript 中生成 URL</span>

如果你的 JavaScript 代码包含在一个 Twig 模板中，你可以使用 `path()` 和 `url()` 这两个 Twig 函数来生成 URL 已经将它们存储到 JavaScript 变量中。`escape()` 函数需要避免任何*非JavaScript安全*的值。

```html
<script>
    const route = "{{ path('blog_show', {slug: 'my-blog-post'})|escape('js') }}";
</script>
```

如果你想动态的生成 URL 或是你正在使用纯 JavaScript 代码，这个方案是无效的。在这种情况下，你可以考虑使用[FOSJsRoutingBundle](https://github.com/FriendsOfSymfony/FOSJsRoutingBundle)。

#### <span id="generating-urls-in-commands">在命令行中生成 URL</span>

在命令行中生成 URL 的工作与[在服务中生成 URL](#generating-urls-in-sevices)的方式类似。唯一的不同在于命令行中不能执行 HTTP 的上下文。因此，如果你想生成绝对 URL，你需要使用 `http://localhost/` 作为主机名来代替真实主机名。

解决方案就是配置 `default_uri` 选项，来定义被命令行生成 URL 时使用的 “request context”：

```yaml
# config/packages/routing.yaml
framework:
    router:
        # ...
        default_uri: 'https://example.org/my/path/'
```

```xml
<!-- config/packages/routing.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:framework="http://symfony.com/schema/dic/symfony"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        https://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/symfony
        https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

    <framework:config>
        <framework:router default-uri="https://example.org/my/path/">
            <!-- ... -->
        </framework:router>
    </framework:config>
</container>
```

```php
// config/packages/routing.php
$container->loadFromExtension('framework', [
    'router' => [
        // ...
        'default_uri' => "https://example.org/my/path/",
    ],
]);
```

> 5.1 版本新特性：`default_uri` 选项在 Symfony 5.1 中首次被引入。

现在你可以在你的命令行中生成 URL，以获得你所期待的结果。

```php
// src/Command/SomeCommand.php
namespace App\Command;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Routing\Generator\UrlGeneratorInterface;
use Symfony\Component\Routing\RouterInterface;
// ...

class SomeCommand extends Command
{
    private $router;

    public function __construct(RouterInterface $router)
    {
        parent::__construct();

        $this->router = $router;
    }

    protected function execute(InputInterface $input, OutputInterface $output)
    {
        // 生成一个不携带路由参数的 URL
        $signUpPage = $this->router->generate('sign_up');

        // 生成一个携带路由参数的 URL
        $userProfilePage = $this->router->generate('user_profile', [
            'username' => $user->getUsername(),
        ]);

        // 默认生成的URL都是 "绝对路径" 。 传递第三个可选
        // 参数来生成不同的 URL (例如一个 "absolute URL")
        $signUpPage = $this->router->generate('sign_up', [], UrlGeneratorInterface::ABSOLUTE_URL);

        // 当一个路由是本地化的时， Symfony 默认使用当前请求的的区域设置
        // 如果你想显式的设置区域，需要传一个不同的 '_locale' 值
        $signUpPageInDutch = $this->router->generate('sign_up', ['_locale' => 'nl']);

        // ...
    }
}
```

> 默认情况下，网络资产生成的 URL 使用同样的 `default_uri` 值，不过你也可以使用 `asset.request_context.base_path` 和 `asset.request_context.secure` 这两个容器参数来改变它。

#### <span id="checking-if-a-route-exists">检查一个路由是否存在</span>

在高度动态化的应用程序中，在使用一个路由生成 URL 前，检查它是否存在是很必要的。在这种情况下，不要使用[getRouteCollection()](https://github.com/symfony/symfony/blob/5.1/src/Symfony/Component/Routing/Router.php)方法，因为它将再次创建路由缓存并拖慢应用程序。

取而代之，你可以尝试生成 URL 并在路由不存在的时候捕捉抛出的 `Symfony\Component\Routing\Exception\RouteNotFoundException`。

```php
use Symfony\Component\Routing\Exception\RouteNotFoundException;

// ...

try {
    $url = $this->router->generate($routeName, $routeParameters);
} catch (RouteNotFoundException $e) {
    // 路由未被定义...
}
```

#### <span id="forcing-https-on-generated-urls">在生成的 URL 中强制使用 HTTPS</span>

默认情况下，生成的 URL 使用当前请求同样的 HTTP 协议。在控制台的命令行中，没有 HTTP 请求，URL 就默认使用 `http`。你可以改变每一个命令（通过路由的 `getContext()` 方法），或者使用全局配置参数：

```yaml
# config/services.yaml
parameters:
    router.request_context.scheme: 'https'
    asset.request_context.secure: true
```

```xml
<!-- config/services.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        https://symfony.com/schema/dic/services/services-1.0.xsd">

    <parameters>
        <parameter key="router.request_context.scheme">https</parameter>
        <parameter key="asset.request_context.secure">true</parameter>
    </parameters>

</container>
```

```php
// config/services.php
$container->setParameter('router.request_context.scheme', 'https');
$container->setParameter('asset.request_context.secure', true);
```

控制台命令行之外的地方，使用 `schemes` 选项来显式地定义每个路由的协议：

```php
// src/Controller/SecurityController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class SecurityController extends AbstractController
{
    /**
     * @Route("/login", name="login", schemes={"https"})
     */
    public function login()
    {
        // ...
    }
}
```

```yaml
# config/routes.yaml
login:
    path:       /login
    controller: App\Controller\SecurityController::login
    schemes:    [https]
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>

<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing https://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="login" path="/login" schemes="https"
           controller="App\Controller\SecurityController::login"/>
</routes>
```

```php
// config/routes.php
use App\Controller\SecurityController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('login', '/login')
        ->controller([SecurityController::class, 'login'])
        ->schemes(['https'])
    ;
};
```

`login` 路由生成的 URL 会一直使用 HTTPS。这意味着当使用 `path()` 这个 Twig 模板函数生成 URL，如果原始请求的 HTTP 协议同路由使用的协议不同，你可能会获取到一个绝对路径的 URL，而不是一个相对路径的 URL：

```twig
{# 如果当前协议是 HTTPS, 生成了一个相对 URL: /login #}
{{ path('login') }}

{# 如果当前协议是 HTTP, 生成了一个绝对 URL 就会改变
   协议：https://example.com/login #}
{{ path('login') }}
```

协议条件对于访问的请求是强制性的。如果你试图使用 HTTP 的协议访问 `/login` URL，你将会被自动地重定向到同样的 URL，但是使用了 HTTPS 协议。

如果你想强制一组路由使用 HTTPS，你可以定义在引入路由的时候的定义默认的协议。下面的例子就展示了如何让所有使用 annotations 方式定义的路由，强制使用 HTTPS：

```yaml
# config/routes/annotations.yaml
controllers:
    resource: '../../src/Controller/'
    type: annotation
    defaults:
        schemes: [https]
```

```xml
<!-- config/routes/annotations.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <import resource="../../src/Controller/" type="annotation">
        <default key="schemes">HTTPS</default>
    </import>
</routes>
```

```php
// config/routes/annotations.php
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->import('../../src/Controller/', 'annotation')
        ->schemes(['https'])
    ;
};
```

> 安全组件提供了另一种通过 `requires_channel` 设置来[强制使用 HTTP 或 HTTPS](https://symfony.com/doc/current/security/force_https.html) 的方案。

### <span id="troubleshooting">故障排除</span>

这是一些你在使用路由功能的工作过程中可能会遇到的常见错误：

> Controller “App\Controller\BlogController::show()” requires that you provide a value for the “$slug” argument.

这经常发生在当你的控制器方法中含有一个参数时（例如 `$slug`）：

```php
public function show($slug)
{
    // ...
}
```

但是你的路由路径中*没有*包含一个 `{slug}` 参数（例如它可能是 `/blog/show`）。在你的路由路径上添加一个 `{slug}`：`/blog/show/{slug}` 或者给参数一个默认值（例如 `$slug = null`）：

> Some mandatory parameters are missing (“slug”) to generate a URL for route “blog_show”.

这句话的意思是你正尝试给 `blog_show` 路由生成一个 URL，但是你*没有*传一个 `slug` 值（这个值是必须的，因为它在路由路径上有一个 `{slug}` 参数）。要解决这个问题，在生成路由的时候传递一个 `slug` 值：

```php
$this->generateUrl('blog_show', ['slug' => 'slug-value']);

// 或者，在 Twig 中
// {{ path('blog_show', {slug: 'slug-value'}) }}
```

### <span id="learn-more-about-routing">学习更多有关路由的知识</span>

+ [如何创建一个定制的路由加载器](https://symfony.com/doc/current/routing/custom_route_loader.html)
+ [如何从数据库中查找路由：Symfony CMF 动态路由](https://symfony.com/doc/current/routing/routing_from_database.html)