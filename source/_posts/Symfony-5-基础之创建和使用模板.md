---
title: Symfony 5 基础之创建和使用模板
date: 2020-09-05 09:44:37
tags:
 - PHP
 - Symfony
---

无论你是需要从一个控制器渲染 HTML 还是生成一封 email 的内容，模板都是从你的应用中组织并渲染 HTML 的最佳方式。在 Symfony 中模板的创建工作是由 Twig 完成：一个灵活、快速并且安全的模板引擎。

### <span id="twig-templating-language">Twig 模板语言</span>

[Twig](https://twig.symfony.com/) 模板语言允许你撰写简明、高可读性的模板，这种模板对于 web 设计者更友好，在某种程度上，比 PHP 模板更加强大。看一下后面这个 Twig 模板实例。即使你第一次看到 Twig，你也很可能明白它的大部分含义：

```twig
<!DOCTYPE html>
<html>
    <head>
        <title>Welcome to Symfony!</title>
    </head>
    <body>
        <h1>{{ page_title }}</h1>

        {% if user.isLoggedIn %}
            Hello {{ user.name }}!
        {% endif %}

        {# ... #}
    </body>
</html>
```

<!-- more -->

Twig 语法基于这三个结构：

+ <code>{% raw %}{{ ... }}{% endraw %}</code>，用来显示变量的内容或是执行一个表达式的结果；

+ <code>{% raw %}{% ... %}{% endraw %}</code>，用来运行一些逻辑，比如一个条件语句或是循环语句；

+ <code>{% raw %}{# ... #}{% endraw %}</code>，用来添加注释到模板中（不像 HTML 注释，这些注释不会被包含进渲染的页面中）。

你不能在一个 Twig 模板中运行 PHP 代码，但是 Twig 提供了一些公共方法来用来在模板中运行一些逻辑。比如，**过滤器**可以在渲染前修改内容，像 `upper` 过滤器将内容转为大写字母：

```twig
{{ title|upper }}
```

Twig 内置了一系列高灵活性的 [标签](https://twig.symfony.com/doc/2.x/tags/index.html)，[过滤器](https://twig.symfony.com/doc/2.x/filters/index.html) 和 [方法](https://twig.symfony.com/doc/2.x/functions/index.html)。在 Symfony 应用中你可以使用这些 [由Symfony 定义的 Twig 过滤器和方法](https://symfony.com/doc/current/reference/twig_reference.html)，并且你可以[创建你自己的 Twig 过滤器和方法](https://symfony.com/doc/current/templating/twig_extension.html)。

Twig 在 `生产` [环境](https://symfony.com/doc/current/configuration.html#configuration-environments)（因为模板被编译成 PHP 代码并且会被自动缓存） 中是非常快速的，而使用 `开发` 环境（因为当你改变模板内容的时候，它们会被自动编译）更加方便一些。

#### <span id="twig-configuration">Twig 配置</span>

Twig 有一些配置选项来定义一些选项，比如用来显示数字和日期的格式化规则，模板的缓存等等。阅读 [Twig 配置参考](https://symfony.com/doc/current/reference/configuration/twig.html) 来了解更多内容。

### <span id="creating-templates">创建模板</span>

在解释如何创建和渲染一个模板的细节之前，快速观察一下后面示例的完整处理过程。首先，你需要在 `template` 目录创建一个新文件用来存储模板内容：

```twig
{# templates/user/notifications.html.twig #}
<h1>Hello {{ user_first_name }}!</h1>
<p>You have {{ notifications|length }} new notifications.</p>
```

然后创建一个 [控制器](/2020/08/29/Symfony-5-基础之控制器/) 用来渲染模板并传入需要的变量：

```php
// src/Controller/UserController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

class UserController extends AbstractController
{
    // ...

    public function notifications()
    {
        // 获取用户信息和显示通知消息
        $userFirstName = '...';
        $userNotifications = ['...', '...'];

        // 模板路径来自 `templates/` 文件夹
        return $this->render('user/notifications.html.twig', [
            // 这个数组定义了传入模板的变量，
            // 其中键是变量名而值是变量的值
            // (Twig 推荐使用蛇形命名法的变量名：如 'foo_bar' 而不是 'fooBar')
            'user_first_name' => $userFirstName,
            'notifications' => $userNotifications,
        ]);
    }
}
```

#### <span id="template-naming">模板命名</span>

Symfony 推荐一下命名规则：

+ 使用 [蛇形命名法](https://en.wikipedia.org/wiki/Snake_case) 命名文件名和路径名（例如 `blog_posts.twig`，`admin/default_theme/blog/index.twig` 等）；
+ 为文件名定义两个扩展名（例如 `index.html.twig` 或 `blog_posts.xml.twig`）其中首个扩展名（`html`，`xml`等）会成为模板最后生成文件的格式。

尽管模板通常生成 HTML 内容，他们也可以生成任何其他基于文本的格式。这就是为什么双扩展约定简化了为多种格式创建和渲染模板的方式。

#### <span id="template-location">模板位置</span>

模板一般默认保存在 `tempalte/` 目录。当一个服务或是控制器渲染 `product/index.html.twig` 模板，它们通常引用 `<your-project>/templates/product/index.html.twig` 文件。

默认模板路径可以使用 [twig.default_path](https://symfony.com/doc/current/reference/configuration/twig.html#config-twig-default-path) 选项定义，并且你可以在这篇文章的[后面部分](#template-namespaces)学习添加更多的模板路径。

#### <span id="template-variables">模板变量</span>

模板的常见需求是打印从控制器或服务传递的模板中存储的值。变量通常存储对象和数组，而不是字符串、数字和布尔值。这就是为什么Twig提供对复杂PHP变量的快速访问。请考虑以下模板：

```twig
<p>{{ user.name }} added this comment on {{ comment.publishedAt|date }}</p>
```

`user.name` 表示法表示要你想要显示存储在一个变量（`user`）中的信息（`name`）。`user` 是一个数组还是一个对象？ `name` 是一个属性还是一个方法？在 Twig 中，这并不重要。

当使用 `foo.bar` 标记时，Twig 试图用一下顺序获取变量中的值：

1. `$foo['bar']` （数组和元素）;
2. `$foo->bar` （对象和公共属性）;
3. `$foo->bar()` （对象和公共方法）;
4. `$foo->getBar()` （对象和 *getter* 方法）;
5. `$foo->isBar()` （对象和 *isser* 方法）;
6. `$foo->hasBar()` （对象和 *hasser* 方法）;
7. 如果以上都不符合，使用 `null`.

这允许改进你的应用程序代码，而无需更改模板代码（你可以从应用程序概念验证的数组变量开始，然后移动到具有方法的对象等）。

#### <span id="linking-to-pages">链接到页面</span>

不需要手写 URL 链接，使用 `path()` 函数可以生成基于 [路由配置](https://symfony.com/doc/current/routing.html#routing-creating-routes) 的 URL。

之后，如果你想修改部分页面的 URL，你所需要做的只是修改路由配置：模板会自动生成新的 URL。

请考虑以下路由配置：

```php
// src/Controller/BlogController.php
namespace App\Controller;

// ...
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends AbstractController
{
    /**
     * @Route("/", name="blog_index")
     */
    public function index()
    {
        // ...
    }

    /**
     * @Route("/article/{slug}", name="blog_post")
     */
    public function show(string $slug)
    {
        // ...
    }
}
```

```yaml
# config/routes.yaml
blog_index:
    path:       /
    controller: App\Controller\BlogController::index

blog_post:
    path:       /article/{slug}
    controller: App\Controller\BlogController::show
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        https://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="blog_index"
        path="/"
        controller="App\Controller\BlogController::index"/>

    <route id="blog_post"
        path="/article/{slug}"
        controller="App\Controller\BlogController::show"/>
</routes>
```

```php
// config/routes.php
use App\Controller\BlogController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('blog_index', '/')
        ->controller([BlogController::class, 'index'])
    ;

    $routes->add('blog_post', '/articles/{slug}')
        ->controller([BlogController::class, 'show'])
    ;
};
```

使用 `path()` 这个 Twig 函数链接这些页面，同时传递路由名作为首个参数并且路由参数作为选项的第二个参数：

```twig
<a href="{{ path('blog_index') }}">Homepage</a>

{# ... #}

{% for post in blog_posts %}
    <h1>
        <a href="{{ path('blog_post', {slug: post.slug}) }}">{{ post.title }}</a>
    </h1>

    <p>{{ post.excerpt }}</p>
{% endfor %}
```

`path()` 函数生成相关的 URL。如果你需要生成绝对 URL（例如为 Email 或是 RSS 订阅渲染模板），使用 `url()` 函数，它将使用同 `path()` 一样的参数（例如 <code>{% raw %}<a href="{{ url('blog_index') }}"> ... </a>{% endraw %}</code>）。

#### <span id="linking-to-cSS-avaScript-and-image-assets">链接到 CSS，JavaScript 和图片资产</span>

如果一个模板需要连接到一个静态资产（例如一个图片），Symfony 提供了一个 `asset()` Twig 函数用来帮助生成 URL。首先，安装 `asset` 包：

```bash
> composer require symfony/asset
```

你现在就可以在模板中使用 `asset()` 函数了：

```twig
{# the image lives at "public/images/logo.png" #}
<img src="{{ asset('images/logo.png') }}" alt="Symfony!"/>

{# the CSS file lives at "public/css/blog.css" #}
<link href="{{ asset('css/blog.css') }}" rel="stylesheet"/>

{# the JS file lives at "public/bundles/acme/js/loader.js" #}
<script src="{{ asset('bundles/acme/js/loader.js') }}"></script>
```

`asset()` 函数的主要目的是是你的应用程序更加轻便。如果你的应用程序处于主机的根目录（例如 `https://example.com`），那么渲染路径就会是 `/images/logo.png`。但是如果你的应用程序处于一个子目录（比如 `https://example.com/my_app`），每个资产的路径将会携带子目录渲染（例如 `/my_app/images/logo.png`）。`asset()` 函数通过确定应用程序的使用方式，并相应地生成正确的路径来处理此问题。

> `asset()` 函数支持各种缓存破坏技术，可以基于 [版本](https://symfony.com/doc/current/reference/configuration/framework.html#reference-framework-assets-version)，[版本格式](https://symfony.com/doc/current/reference/configuration/framework.html#reference-assets-version-format)，[json_manifest_path](https://symfony.com/doc/current/reference/configuration/framework.html#reference-assets-json-manifest-path) 配置选项。

> 如果你需要以现代方式对 JavaScript 和 CSS 资产进行打包、版本控制和压缩，请阅读 [Symfony 的 Webpack Encore](https://symfony.com/doc/current/frontend.html)，

如果你需要 URL 的绝对路径，像下面一样使用 `absolute_url()` Twig 函数：

```twig
<img src="{{ absolute_url(asset('images/logo.png')) }}" alt="Symfony!"/>

<link rel="shortcut icon" href="{{ absolute_url('favicon.png') }}">
```

#### <span id="the-app-global-variable">应用全局变量</span>

Symfony 会创建一个上下文对象，这个对象会以命名为 `app` 的变量的方式自动注入每一个 Twig 模板。它提供了访问一些应用程序信息的方法：

```twig
<p>Username: {{ app.user.username ?? 'Anonymous user' }}</p>
{% if app.debug %}
    <p>Request method: {{ app.request.method }}</p>
    <p>Application Environment: {{ app.environment }}</p>
{% endif %}
```

`app` 变量（它是 `Symfony\Bridge\Twig\AppVariable` 的一个实例）可以让你访问这些变量：

`app.session`
  [当前用户对象](https://symfony.com/doc/current/security.html#create-user-class) 或者如果用户未被授权就为 `null`

`app.request`
  `Symfony\Component\HttpFoundation\Request` 的对象，存储了当前的 [请求数据](https://symfony.com/doc/current/components/http_foundation.html#accessing-request-data)（根据你的应用程序，这可以是一个[子请求](https://symfony.com/doc/current/components/http_kernel.html#http-kernel-sub-requests) 或是一个常规请求）。

`app.session`
  `Symfony\Component\HttpFoundation\Session\Session` 的对象，可以代表当前[用户的会话](https://symfony.com/doc/current/session.html) ，而会话不存在时为 `null`。

`app.flashes`
  一个存储会话中所有 [闪现消息](https://symfony.com/doc/current/controller.html#flash-messages) 的数组。你也可以获取指定类型的消息（例如 `app.flashes('notice')`）。

`app.environment`
  当前[配置环境](https://symfony.com/doc/current/configuration.html#configuration-environments)（`dev`，`prod` 等）的名称。

`app.debug`
  如果在[调试模式](https://symfony.com/doc/current/configuration/front_controllers_and_kernel.html#debug-mode) 中值为 True，其他情况下为 False。

`app.token`
  一个 `Symfony\Component\Security\Core\Authentication\Token\TokenInterface` 对象，代表当前安全令牌。

除了 Symfony 注入的全局应用变量外，您还可以 [自动将变量注入到所有 Twig 模板](https://symfony.com/doc/current/templating/global_variables.html)。

### <span id="rendering-templates">渲染模板</span>

#### <span id="rendering-a-template-in-controllers">在控制器中渲染模板</span>

如果你的控制器继承自 [AbstractController](https://symfony.com/doc/current/controller.html#the-base-controller-class-services)，使用 `render()` 辅助方法：

```php
// src/Controller/ProductController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;

class ProductController extends AbstractController
{
    public function index()
    {
        // ...

        // `render()` 方法返回一个 `Response` 对象 这个对象携带了
        // 被模板创建的内容
        return $this->render('product/index.html.twig', [
            'category' => '...',
            'promotions' => ['...', '...'],
        ]);

        // `renderView()` 方法只返回由模板创建的内容，
        // 所以随后你可以在一个 `Response` 对象中使用这些内容
        $contents = $this->renderView('product/index.html.twig', [
            'category' => '...',
            'promotions' => ['...', '...'],
        ]);

        return new Response($contents);
    }
}
```

如果你的控制器不是继承自 `AbstractController`，你需要 [在控制器中调用服务](/2020/08/29/Symfony-5-基础之控制器/#fetching-services) 并且使用 `twig` 服务的 `render()` 方法。

#### <span id="rendering-a-template-in-services">在服务中渲染模板</span>

注入 `twig` Symfony 服务到你自己的服务中，并且使用它的 `render()` 方法。当时使用 [服务自动写入](https://symfony.com/doc/current/service_container/autowiring.html) 时，你只需要在服务的构造函数中添加一个参数，并使用 `Twig\Environment` 类作为类型约束1参数：

```php
// src/Service/SomeService.php
namespace App\Service;

use Twig\Environment;

class SomeService
{
    private $twig;

    public function __construct(Environment $twig)
    {
        $this->twig = $twig;
    }

    public function someMethod()
    {
        // ...

        $htmlContents = $this->twig->render('product/index.html.twig', [
            'category' => '...',
            'promotions' => ['...', '...'],
        ]);
    }
}
```

#### <span id="rendering-a-template-in-emails">在服务中渲染模板</span>

阅读关于 [邮件与 Twig 集成](https://symfony.com/doc/current/mailer.html#mailer-twig) 的文档。

#### <span id="rendering-a-template-directly-from-a-route">在服务中渲染模板</span>

尽管模板通常在服务器或是服务中进行渲染，你也可以不需要任何变量，直接从路由定义中渲染静态页面。使用特殊的由 Symfony 提供的 `Symfony\Bundle\FrameworkBundle\Controller\TemplateController`。

```yaml
# config/routes.yaml
acme_privacy:
    path:          /privacy
    controller:    Symfony\Bundle\FrameworkBundle\Controller\TemplateController
    defaults:
        # 渲染模板的路径
        template:  'static/privacy.html.twig'

        # Symfony 定义的设置页面缓存的特殊选项
        maxAge:    86400
        sharedAge: 86400

        # 缓存是否应仅适用于客户端缓存
        private: true

        # 你可以选择定义传递给模板的一些参数
        context:
            site_name: 'ACME'
            theme: 'dark'
```

```xml
<!-- config/routes.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing https://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="acme_privacy"
        path="/privacy"
        controller="Symfony\Bundle\FrameworkBundle\Controller\TemplateController">
        <!-- 渲染模板的路径 -->
        <default key="template">static/privacy.html.twig</default>

        <!-- Symfony 定义的设置页面缓存的特殊选项 -->
        <default key="maxAge">86400</default>
        <default key="sharedAge">86400</default>

        <!-- 缓存是否应仅适用于客户端缓存 -->
        <default key="private">true</default>

        <!-- 你可以选择定义传递给模板的一些参数 -->
        <default key="context">
            <default key="site_name">ACME</default>
            <default key="theme">dark</default>
        </default>
    </route>
</routes>
```

```php
// config/routes.php
use Symfony\Bundle\FrameworkBundle\Controller\TemplateController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('acme_privacy', '/privacy')
        ->controller(TemplateController::class)
        ->defaults([
            // 渲染模板的路径
            'template'  => 'static/privacy.html.twig',

            // Symfony 定义的设置页面缓存的特殊选项
            'maxAge'    => 86400,
            'sharedAge' => 86400,

            // 缓存是否应仅适用于客户端缓存
            'private' => true,

            // 你可以选择定义传递给模板的一些参数
            'context' => [
                'site_name' => 'ACME',
                'theme' => 'dark',
            ]
        ])
    ;
};
```

> 5.1 版本新特性：`context` 选项在 Symfony 5.1 中首次被引入

#### <span id="checking-if-a-template-exists">检查一个模板是否存在</span>

模板在应用程序使用一个 [Twig 模板加载器](https://twig.symfony.com/doc/2.x/api.html#loaders) 加载，它也提供了一个方法来检测模板的存在状态。首先，获取加载器：

```php
// 在一个继承自 AbstractController 的控制器中
$loader = $this->get('twig')->getLoader();

// 在一个自动写入功能的服务中
use Twig\Environment;

public function __construct(Environment $twig)
{
    $loader = $twig->getLoader();
}
```

然后，传入 Twig 模板的路径到加载器的 `exist()` 方法中：

```php
if ($loader->exists('theme/layout_responsive.html.twig')) {
    // 模板存在，做些什么
    // ...
}
```

### <span id="debugging-templates">调试模板</span>

Symfony 提供了一些辅助方法来帮助你调试模板中的问题。

#### <span id="linting-twig-templates">检查模板语法</span>

`lint:twig` 命令检查你的 Twig 模板中是否存在任何的语法错误。在发布你的应用程序到生产环境之前运行它是非常有用的（例如在你的持续集成服务器）。

```bash
# 检查所有的应用程序模板
> php bin/console lint:twig

# 你也可以检查目录和单个模板
> php bin/console lint:twig templates/email/
> php bin/console lint:twig templates/article/recent_list.html.twig

# 你还可以显示模板中使用的已弃用功能
> php bin/console lint:twig --show-deprecations templates/email/
```

#### <span id="inspecting-twig-information">检查模板信息</span>

`debug:twig` 命令列出有关 Twig 的所有可用信息（函数，过滤器，全局变量等）。检查你的 [自定义 Twig 扩展](https://symfony.com/doc/current/templating/twig-extension.html) 是否正常工作，以及检查 [安装包](https://symfony.com/doc/current/setup.html#symfony-flex) 时添加的 Twig 功能非常有用：

```bash
# 列出一般信息
> php bin/console debug:twig

# 任何关键字的过滤输出
> php bin/console debug:twig --filter=date

# 传递一个模板路径来显示将要被加载的物理文件
> php bin/console debug:twig @Twig/Exception/error.html.twig
 ```

#### <span id="the-dump-twig-utilities">模板输出公共方法</span>

Symfony 提供了一个 [dump() 函数](https://symfony.com/doc/current/components/var_dumper.html#components-var-dumper-dump) 作为 PHP 的 `var_dump()` 函数的更高级的替代品。这个函数在检查任何变量内容时非常有用，并且你也可以在 Twig 模板中使用它。

首先，确保 VarDumper 组件已经安装到了应用程序中：

```bash
> composer require symfony/var-dumper
```

然后，根据你的需求选择使用 <code>{% raw %}{% dump %}{% endraw %}</code> 或是 <code>{% raw %}{{ dump() }}{% endraw %}</code> 函数：

```twig
{# templates/article/recent_list.html.twig #}
{# 这个变量的内容将会发送给 Web Debug Toolbar
   而不是在页面内容中输出它们 #}
{% dump articles %}

{% for article in articles %}
    {# 这个变量的内容将会输出到页面内容中
       并且它们是页面可见的 #}
    {{ dump(article) }}

    <a href="/article/{{ article.slug }}">
        {{ article.title }}
    </a>
{% endfor %}
```

为了防止泄漏敏感信息， `dump()` 函数（标签）只在 `dev` 和 `test` [配置环境](https://symfony.com/doc/current/configuration.html#configuration-environments) 中可用。如果你想在生产环境中使用它，你将会见到一个 PHP 错误。

### <span id="reusing-template-contents">复用模板内容</span>

#### <span id="including-templates">引入模板</span>

如果确认一些 Twig 代码在一些模板中是重复的，你可以把它提取到一个单一的 “模板片” 中并且在另外的模板中引入它。假设以下显示用户信息代码将在几个位置重复：

```twig
{# templates/blog/index.html.twig #}

{# ... #}
<div class="user-profile">
    <img src="{{ user.profileImageUrl }}" alt="{{ user.fullName }}"/>
    <p>{{ user.fullName }} - {{ user.email }}</p>
</div>
```

首先，创建一个名为 `blog/_user_profile.html.twig` 的新 Twig 模板（`_` 前缀是可选的，但它是用于更好地区分完整模板和模板片段的约定）。

然后，从原始的 `blog/index.html.twig` 模板中删除该内容，并添加以下内容以包含模板片段：

```twig
{# templates/blog/index.html.twig #}

{# ... #}
{{ include('blog/_user_profile.html.twig') }}
```

`include()` Twig 函数将要包含的模板的路径作为参数。包含的模板可以访问包含它的所有模板变量（使用 [with_context](https://twig.symfony.com/doc/2.x/functions/include.html) 选项来控制此变量）。

你还可以将变量传递给包含的模板。这对于重命名变量很有用。假设你的模板将用户信息存储到名为 `blog_post.author` 的变量中，而不是模板片段期望的 `user` 变量中。使用以下方法*重命名*变量：

```twig
{# templates/blog/index.html.twig #}

{# ... #}
{{ include('blog/_user_profile.html.twig', {user: blog_post.author}) }}
```

#### <span id="embedding-controllers">嵌入控制器</span>

[包含模板片段](#including-templates) 可用于在多个页面上重用相同的内容。但是，在某些情况下，这种技术不是最佳解决方案。

假设模板片段显示三篇最新的博客文章。为此，它需要进行数据库查询来获取这些文章。使用 `include()` 函数时，你需要在包含片段的每个页面中执行相同的数据库查询。这不是很方便。

更好的选择是将**执行某些控制器的结果嵌入到** `render()` 和 `controller()` Twig 函数中。

首先，创建呈现一定数量的最近文章的控制器：

```php
// src/Controller/BlogController.php
namespace App\Controller;

// ...

class BlogController extends AbstractController
{
    public function recentArticles($max = 3)
    {
        // 获取最近的文章 (例如 进行数据库查询)
        $articles = ['...', '...', '...'];

        return $this->render('blog/_recent_articles.html.twig', [
            'articles' => $articles
        ]);
    }
}
```

然后，创建 `blog/_recent_articles.html.twig` 模板片段（模板名称中的 `_` 前缀是可选的，但它是用于更好地区分完整模板和模板片段的约定）：

```twig
{# templates/blog/_recent_articles.html.twig #}
{% for article in articles %}
    <a href="{{ path('blog_show', {slug: article.slug}) }}">
        {{ article.title }}
    </a>
{% endfor %}
```

现在，你可以通过任何模板调用此控制器以嵌入其结果：

```twig
{# templates/base.html.twig #}

{# ... #}
<div id="sidebar">
    {# 如果控制器与一个路由相关, 使用 path() 或者 url() 函数 #}
    {{ render(path('latest_articles', {max: 3})) }}
    {{ render(url('latest_articles', {max: 3})) }}

    {# 如果您不想使用公共 URL 暴露控制器，
       使用 controller() 函数定义需要执行的控制器 #}
    {{ render(controller(
        'App\\Controller\\BlogController::recentArticles', {max: 3}
    )) }}
</div>
```

使用 `controller()` 函数时，不会使用常规 Symfony 路由访问控制器，而是通过专用于为这些模板片段服务的特殊 URL 访问控制器。在 `fragments` 选项中配置该特殊 URL：

```yaml
# config/packages/framework.yaml
framework:
    # ...
    fragments: { path: /_fragment }
```

```xml
<!-- config/packages/framework.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:framework="http://symfony.com/schema/dic/symfony"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        https://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/symfony https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

    <!-- ... -->
    <framework:config>
        <framework:fragment path="/_fragment"/>
    </framework:config>
</container>
```

```php
// config/packages/framework.php
$container->loadFromExtension('framework', [
    // ...
    'fragments' => ['path' => '/_fragment'],
]);
```

> 嵌入控制器需要向这些控制器发出请求，并因此渲染一些模板。如果嵌入大量控制器，这会对应用程序性能产生重大影响。如果可能，请 [缓存模板片段](https://symfony.com/doc/current/http_cache/esi.html)。

> 模板还可以使用 `hinclude.js` JavaScript 库 [异步嵌入内容](https://symfony.com/doc/current/templating/hinclude.html) 。

#### <span id="template-inheritance-and-layouts">模板继承和布局</span>

随着应用程序的增长，你将在页面之间找到越来越多的重复元素，如页眉、页脚、边栏等。包括模板和嵌入控制器可以提供帮助，但当页面共享公共结构时，最好使用**继承**。

[Twig 模板继承](https://twig.symfony.com/doc/2.x/tags/extends.html) 的概念与 PHP 类继承类似。定义其他模板可以扩展的父模板，子模板可以覆盖父模板的某些部分。

Symfony 建议为中型和复杂应用程序提供以下三级模板继承：

+ `templates/base.html.twig`， 定义所有应用程序的通用元素，例如 `<head>`，`<header>`，`<footer>` 等；

+ `templates/layout.html.twig`，继承自 `base.html.twig` 并且定义了所有或大部分页面里的内容结构，例如两列内容 + 侧边栏布局。应用程序的某些部分可以定义自己的布局（例如 `templates/blog/layout.html.twig`）。

+ `templates/*.html.twig`， 从 `layout.html.twig` 模板或任何其他部分布局扩展的应用程序页面。

在实践中，`base.html.twig` 模板通常会想下面这样：

```twig
{# templates/base.html.twig #}
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}My Application{% endblock %}</title>
        {% block stylesheets %}
            <link rel="stylesheet" type="text/css" href="/css/base.css"/>
        {% endblock %}
    </head>
    <body>
        {% block body %}
            <div id="sidebar">
                {% block sidebar %}
                    <ul>
                        <li><a href="{{ path('homepage') }}">Home</a></li>
                        <li><a href="{{ path('blog_index') }}">Blog</a></li>
                    </ul>
                {% endblock %}
            </div>

            <div id="content">
                {% block content %}{% endblock %}
            </div>
        {% endblock %}
    </body>
</html>
```

[Twig 块标记](https://twig.symfony.com/doc/2.x/tags/block.html) 定义了可以在子模板中覆盖的页面部分。它们可以为空，如 `content` 块或定义默认内容（如 `title` 块），当子模板不覆盖它们时，将显示这些内容。

`blog/layout.html.twig` 模板可以像这样：

```twig
{# templates/blog/layout.html.twig #}
{% extends 'base.html.twig' %}

{% block content %}
    <h1>Blog</h1>

    {% block page_contents %}{% endblock %}
{% endblock %}
```

模板继承自 `base.html.twig`，仅定义 `content` 块的内容。父模板块的其余部分将显示其默认内容。但是，它们可以被第三级继承模板（如 `blog/index.html.twig`）覆盖，该模板显示博客索引：

```twig
{# templates/blog/index.html.twig #}
{% extends 'blog/layout.html.twig' %}

{% block title %}Blog Index{% endblock %}

{% block page_contents %}
    {% for article in articles %}
        <h2>{{ article.title }}</h2>
        <p>{{ article.body }}</p>
    {% endfor %}
{% endblock %}
```

这个模板从第二级模板（`blog/layout.html.twig`）扩展，但覆盖不同父模板的块：`page_contents` 来自 `blog/layout.html.twig` 并且 `title` 来自 `base.html.twig`

当你渲染 `blog/index.html.twig` 模板时，Symfony 使用三个不同的模板来创建最终内容。此继承机制可提高工作效率，因为每个模板仅包含其唯一的内容，并且将重复的内容和 HTML 结构留给某些父模板。

> 在使用 `extend` 时，禁止子模板在块外部定义模板部分。以下代码引发一个 `语法错误`：
```twig
{# app/Resources/views/blog/index.html.twig #}
{% extends 'base.html.twig' %}

{# the line below is not captured by a "block" tag #}
<div class="alert">Some Alert</div>

{# the following is valid #}
{% block content %}My cool blog posts{% endblock %}
```

阅读 [Twig 模板继承](https://twig.symfony.com/doc/2.x/tags/extends.html) 文档，详细了解如何在重写模板和其他高级功能时重用父块内容。

### <span id="output-escaping">输出转义</span>

假设你的模板包含显示用户名的 `Hello {{ name }}` 代码。如果恶意用户将 `<script>alert('hello!')</script>` 为用户名称，并且输出该值不变，则应用程序将显示 JavaScript 弹出窗口。

这被称为 [跨站点脚本](https://en.wikipedia.org/wiki/Cross-site_scripting)（XSS） 攻击。虽然前面的示例似乎无害，但攻击者可以编写更高级的 JavaScript 代码来执行恶意操作。

要防止这类攻击，使用“*输出转义*”转换具有特殊含义的字符（例如，将 `<` 替换为 `&lt;` HTML 实体）。默认情况下，Symfony 应用程序是安全的，因为它们通过 [Twig 自动逃逸选项](https://symfony.com/doc/current/reference/configuration/twig.html#config-twig-autoescape) 执行自动输出转义：

```twig
<p>Hello {{ name }}</p>
{# 如果 'name' 是 '<script>alert('hello!')</script>', Twig 将会这样输出：
   '<p>Hello &lt;script&gt;alert(&#39;hello!&#39;)&lt;/script&gt;</p>' #}
```

如果要呈现受信任的变量并包含 HTML 内容，请使用 [Twig 原始过滤器](https://twig.symfony.com/doc/2.x/filters/raw.html) 禁用该变量的输出转义：

```twig
<h1>{{ product.title|raw }}</h1>
{# 如果 'product.title' 是 'Lorem <strong>Ipsum</strong>'， Twig 将会精确输出
   而不是 'Lorem &lt;strong&gt;Ipsum&lt;/strong&gt;' #}
```

阅读 [Twig 输出转义文档](https://twig.symfony.com/doc/2.x/api.html#escaper-extension)，详细了解如何禁用块甚至整个模板的输出转义。

### <span id="template-namespaces">模板命名空间</span>

尽管大多数应用程序将其模板存储在默认的 `templates/` 目录中，但你可能需要将部分模板或所有模板存储在不同的目录中。使用 `twig.paths` 选项配置这些额外的目录。每个路径都定义为一个 `key: value`，其中 `key` 是模板目录，`value` 是 Twig 命名空间，下面将对此进行解释：

```yaml
# config/packages/twig.yaml
twig:
    # ...
    paths:
        # 目录是相对于项目根目录 （但是你
        # 也可以使用绝对目录）
        'email/default/templates': ~
        'backend/templates': ~
```

```xml
<!-- config/packages/twig.xml -->
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:twig="http://symfony.com/schema/dic/twig"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        https://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/twig https://symfony.com/schema/dic/twig/twig-1.0.xsd">

    <twig:config>
        <!-- ... -->
        <!-- 目录是相对于项目根目录 （但是你
             也可以使用绝对目录） -->
        <twig:path>email/default/templates</twig:path>
        <twig:path>backend/templates</twig:path>
    </twig:config>
</container>
```

```php
// config/packages/twig.php
$container->loadFromExtension('twig', [
    // ...
    'paths' => [
        // 目录是相对于项目根目录 （但是你
        // 也可以使用绝对目录） -->
        'email/default/templates' => null,
        'backend/templates' => null,
    ],
]);
```

渲染模板时，Symfony 首先在不定义命名空间的 `twig.paths` 目录中搜索模板，然后返回默认模板目录（通常为 `templates/`）。

使用上述的配置，如果应用程序呈现例如 `layout.html.twig` 模板，Symfony 将首先查找 `email/default/templates/layout.html.twig` 和 `backend/templates/layout.html.twig`。如果存在这些模板中的任何一个，Symfony 将使用它，而不是使用 `templates/layout.html.twig`，这可能是你想要使用的模板。

Twig 使用**命名空间**解决了这个问题，命名空间将多个模板分组到与其实际位置无关的逻辑名称下。更新以前的配置以为每个模板目录定义命名空间：

```yaml
# config/packages/twig.yaml
twig:
    # ...
    paths:
        'email/default/templates': 'email'
        'backend/templates': 'admin'
```

```xml
<!-- config/packages/twig.xml -->
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:twig="http://symfony.com/schema/dic/twig"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        https://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/twig https://symfony.com/schema/dic/twig/twig-1.0.xsd">

    <twig:config>
        <!-- ... -->
        <twig:path namespace="email">email/default/templates</twig:path>
        <twig:path namespace="admin">backend/templates</twig:path>
    </twig:config>
</container>
```

```
// config/packages/twig.php
$container->loadFromExtension('twig', [
    // ...
    'paths' => [
        'email/default/templates' => 'email',
        'backend/templates' => 'admin',
    ],
]);
```

现在，如果渲染 `layout.html.twig` 模板，Symfony 将渲染 `templates/layout.html.twig` 文件。使用特殊语法 `@` + 命名空间来引用其他命名空间模板（例如 `@email/layout.html.twig` 和 `@admin/layout.html.twig`）。

> 单个 Twig 命名空间可以与多个模板目录关联。在这种情况下，添加路径的顺序很重要，因为 Twig 将从第一个定义的路径开始查找模板。

#### <span id="bundle-templates">bundle模板</span>

如果你在应用程序中 [安装 packages/bundles](https://symfony.com/doc/current/setup.html#symfony-flex)，它们可能包含自己的 Twig 模板（在每个 bundle 的资源/视图/目录中）。为了避免弄乱自己的模板，Symfony 在 bundle 名称后创建的自动命名空间下添加 bundle 模板。

例如，名为 `AcmeFobundle` 的 bundle 的模板可在 `AcmeFoo` 命名空间下使用。如果此 bundle 包含模板 `<your-project>/vendor/acmefoo-bundle/Resources/views/user/profile.html.twig`，你可以用 `@AcmeFoo/user/profile.html.twig` 引用它。

> 你还可以 [覆盖 bundle 模板](https://symfony.com/doc/current/bundles/override.html#override-templates) ，以防要更改原始 bundle 模板的某些部分。

### <span id="learn-more">了解更多</span>

+ [如何在模板中使用 PHP 代替 Twig](https://symfony.com/doc/current/templating/PHP.html)
+ [如何在所有的模板中自动注入变量](https://symfony.com/doc/current/templating/global_variables.html)
+ [如何使用 hinclude.js 嵌入异步内容](https://symfony.com/doc/current/templating/hinclude.html)
+ [如何写入一个定制 Twig 扩展](https://symfony.com/doc/current/templating/twig_extension.html)