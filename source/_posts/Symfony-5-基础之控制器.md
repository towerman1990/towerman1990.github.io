---
title: Symfony 5 基础之控制器
date: 2020-08-29 15:23:32
tags:
 - PHP
 - Symfony
---

一个控制器本质上就是一个PHP方法：从 `Request` 对象中读取信息，创建并返回一个 `Response` 对象。响应可以是一个 HTML 页面，JSON，XML，一个文件下载，一个重定向，一个404错误或者其它的东西。控制器用来实现你的应用所需要渲染页面内容的任何逻辑。

> 如果你还没有创建你的第一个工作页面，可以先看一下我之前写的 [Symfony 5 试用](/2020/06/13/Symfony-5-试用/)，然后再回来继续！

### <span id="a-simple-controller">一个简单的控制器</span>

虽然一个控制器可以是任意可以调用的东西（函数，一个对象的方法，或者一个 `Closure`），一个控制器通常会是存在于一个控制器类中的方法：

<!-- more -->

```php
// src/Controller/LuckyController.php
namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class LuckyController
{
    /**
     * @Route("/lucky/number/{max}", name="app_lucky_number")
     */
    public function number($max)
    {
        $number = random_int(0, $max);

        return new Response(
            '<html><body>Lucky number: '.$number.'</body></html>'
        );
    }
}
```

这个控制器就是 `number()` 方法，存在于控制器类 `LuckyController` 中。

这个控制器非常简单：

+ 第2行：Symfony 利用了PHP的命名空间功能，为整个控制器类命名空间。
+ 第4行：Symfony 再次利用PHP的命名空间功能：使用 `use` 关键字引入 `Response` 类，这个类是控制器返回数据必须用到的。
+ 第7行：这个类在技术上可以命名为任何样式，但是它的后缀依照惯例必须是以 `Controller` 结尾。
+ 第12行：动作方法允许有一个 `$max` 参数要幸亏 `max` [路由通配符](/2020/07/25/Symfony-5-基础之路由/#route_parameters)。
+第16行：控制器创建并返回了一个 `Response` 对象。

#### <span id="mapping-a-url-to-a-controller">将URL映射到控制器</span>

为了 `查看` 这个控制器的结果，你需要通过路由将URL映射到该控制器上。上面是通过 `@Route("/lucky/number/{max}")` [路由注释](/2020/07/25/Symfony-5-基础之路由/#creating_routes_as_annotations)完成的。

要查看你的页面，请在浏览器中转到以下URL：[http://localhost:8000/lucky/number/100](http://localhost:8000/lucky/number/100)

有关路由的更多信息，请参见上一篇文章：[Symfony 5 基础之路由](/2020/07/25/Symfony-5-基础之路由)。

### <span id="the-base-controller-class-and-services">基本控制器类和服务</span>

为了帮助开发，Symfony附带了一个可选的基本控制器类，称为 `Symfony\Bundle\FrameworkBundle\Controller\AbstractController`。它可以被扩展以访问辅助方法。

将 `use` 语句添加到你的控制器类的顶部，然后修改 `LuckyController` 来扩展它：

```php
// src/Controller/LuckyController.php
namespace App\Controller;

+ use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

- class LuckyController
+ class LuckyController extends AbstractController
{
    // ...
}
```

仅此而已！现在，你可以访问 [$this->render()](#rendering-templates) 之类的方法，以及接下来将要学习的许多其他方法。

#### <span id="generating-urls">生成URL</span>

这个[generateUrl()](https://github.com/symfony/symfony/blob/5.1/src/Symfony/Bundle/FrameworkBundle/Controller/AbstractController.php) 方法只是一个辅助方法，它为给定的路由生成URL：

```php
$url = $this->generateUrl('app-lucky_number', ['max' => 10]);
```

#### <span id="redirecting">重定向</span>

如果你想把用户重定向到另一个页面，请使用 `redirectToRoute()` 和 `redirect()` 方法：

```php
use Symfony\Component\HttpFoundation\RedirectResponse;

// ...
public function index()
{
    // 重定向到 "homepage" 路由
    return $this->redirectToRoute('homepage');

    // redirectToRoute 是一种缩写：
    // return new RedirectResponse($this->generateUrl('homepage'));

    // 做一个永久的 - 301 重定向
    return $this->redirectToRoute('homepage', [], 301);

    // 携带参数重定向到一个路由
    return $this->redirectToRoute('app_lucky_number', ['max' => 10]);

    // 重定向到一个路由并保持原来的查询字符串参数
    return $this->redirectToRoute('blog_show', $request->query->all());

    // 重定向到外部
    return $this->redirect('http://symfony.com/doc');
}
```

> `redirect()` 方法不会以任何方式检查其目的地。如果你重定向到一个终端用户提供的URL，那你的应用程序可能会打开[未验证的重定向安全漏洞](https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html)。

#### <span id="rendering-templates">渲染模板</span>

如果要提供HTML服务，你就需要渲染一个模板。这个 `render()` 方法渲染一个模板，并且把内容放入一个 `Response` 对象中：

```php
// 渲染 templates/lucky/number.html.twig
return $this->render('lucky/number.html.twig', ['number' => $number]);
```

在 [创建和使用模板](https://symfony.com/doc/current/templates.html) 一文中详细说明了模板和Twig 。

#### <span id="fetching-services">获取服务</span>

Symfony 已经*打包*了许多有用的类和功能，叫做[服务](https://symfony.com/doc/current/service_container.html)。这些用于渲染模板，发送电子邮件，查询数据库以及你可以想到的任何其他“工作”。

如果你需要使用一个控制器中的服务，请键入带有其类（或接口）名称的参数。Symfony 将自动为你提供所需的服务：

```php
use Psr\Log\LoggerInterface;
// ...

/**
 * @Route("/lucky/number/{max}")
 */
public function number($max, LoggerInterface $logger)
{
    $logger->info('We are logging!');
    // ...
}
```

太棒了！

你还可以键入其他哪些服务？要查看它们，使用 `debug:autowiringconsole` 命令：

```bash
> php bin/console debug:autowiring
```

如果你需要控制参数的确切值，则可以按名称 [绑定](https://symfony.com/doc/current/service_container.html#services-binding) 参数：

```yaml
# config/services.yaml
services:
    # ...

    # 服务的确切配置
    App\Controller\LuckyController:
        tags: [controller.service_arguments]
        bind:
            # 对于任何 $logger 参数，传入这个特殊服务
            $logger: '@monolog.logger.doctrine'
            # 对于任何 $projectDir 参数，传入这个参数值
            $projectDir: '%kernel.project_dir%'
```

```xml
<!-- config/services.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        https://symfony.com/schema/dic/services/services-1.0.xsd">

    <services>
        <!-- ... -->

        <!-- Explicitly configure the service -->
        <service id="App\Controller\LuckyController">
            <tag name="controller.service_arguments"/>
            <bind key="$logger"
                type="service"
                id="monolog.logger.doctrine"
            />
            <bind key="$projectDir">%kernel.project_dir%</bind>
        </service>
    </services>
</container>
```

```php
// config/services.php
use App\Controller\LuckyController;
use Symfony\Component\DependencyInjection\Reference;

$container->register(LuckyController::class)
    ->addTag('controller.service_arguments')
    ->setBindings([
        '$logger' => new Reference('monolog.logger.doctrine'),
        '$projectDir' => '%kernel.project_dir%'
    ])
;
```

像所有的服务一样，你也可以在控制器中使用常规 [构造函数](https://symfony.com/doc/current/service_container.html#services-constructor-injection) 注入。

有关服务的更多信息，请参见 [服务容器](https://symfony.com/doc/current/service_container.html) 文章。

### <span id="generating-controllers">生成控制器</span>

为了节省时间，你可以安装 [Symfony Maker](https://symfony.com/doc/current/bundles/SymfonyMakerBundle/index.html) 并告诉 Symfony 生成新的控制器类：

```bash
> php bin/console make:controller BrandNewController

created: src/Controller/BrandNewController.php
created: templates/brandnew/index.html.twig
```

如果你要从Doctrine 实体生成整个CRUD ，请使用：

```bash
> php bin/console make:crud Product

created: src/Controller/ProductController.php
created: src/Form/ProductType.php
created: templates/product/_delete_form.html.twig
created: templates/product/_form.html.twig
created: templates/product/edit.html.twig
created: templates/product/index.html.twig
created: templates/product/new.html.twig
created: templates/product/show.html.twig
```

> 1.2版的新功能：`make:crud` 命令在MakerBundle 1.2中引入。

### <span id="managing-errors-and-404-pages">管理错误和404页面</span>

如果找不到任何内容，则应返回404响应。为此，需要抛出一种特殊类型的异常：

```php
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

// ...
public function index()
{
    // 从数据库中检索对象
    $product = ...;
    if (!$product) {
        throw $this->createNotFoundException('The product does not exist');

        // 上面语句的只是一个简写：
        // throw new NotFoundHttpException('The product does not exist');
    }

    return $this->render(...);
}
```

[createNotFoundException()](https://github.com/symfony/symfony/blob/5.1/src/Symfony/Bundle/FrameworkBundle/Controller/AbstractController.php) 方法只是创建特殊`Symfony\Component\HttpKernel\Exception\NotFoundHttpException` 对象的快捷方式 ，该对象最终会在Symfony中触发404 HTTP响应。

如果你抛出的异常继承自或就是一个 `Symfony\Component\HttpKernel\Exception\HttpException` 对象，那么Symfony将使用适当的HTTP状态代码。否则，响应将带有500 HTTP状态代码：

```php
// 这个异常最终会生成一个 500 状态错误
throw new \Exception('Something went wrong!');
```

在每种情况下，都会向最终用户显示一个错误页面，并向开发人员显示一个完整的调试错误页面（即，当你处于“调试”模式时-请参阅 [配置环境](https://symfony.com/doc/current/configuration.html#page-creation-environments)）。

要自定义显示给用户的错误页面，请参阅 [如何自定义错误页面](https://symfony.com/doc/current/controller/error_pages.html) 文章。

### <span id="the-request-object-as-a-controller-argument">请求对象作为控制器参数</span>

如果你需要读取查询参数，获取请求头或访问上传的文件该怎么办？这些信息存储在 Symfony 的 `Request` 对象中。要想在你控制器中访问它，请将其添加为参数并**使用Request类作为它的类型提示**：

```php
use Symfony\Component\HttpFoundation\Request;

public function index(Request $request, $firstName, $lastName)
{
    $page = $request->query->get('page', 1);

    // ...
}
```

[继续阅读](https://symfony.com/doc/current/controller.html#request-object-info) 有关使用Request对象的更多信息。

### <span id="managing-the-session">管理会话</span>

Symfony 提供了一个会话服务，你可以用它在两次请求之间存储有关用户的信息。会话默认情况下处于启用状态，但只有在你对其进行读写时才会启动。

会话存储和其他配置可以在 `config/packages/framework.yaml` 文件中 [framework.session 配置](https://symfony.com/doc/current/reference/configuration/framework.html#config-framework-session) 下面。

要获取会话，请添加一个参数，并使用  `Symfony\Component\HttpFoundation\Session\SessionInterface` 作为类型提示：

```php
use Symfony\Component\HttpFoundation\Session\SessionInterface;

public function index(SessionInterface $session)
{
    // 存储属性以在随后的用户请求期间重用
    $session->set('foo', 'bar');

    // 获取另一个请求中另一个控制器设置的属性
    $foobar = $session->get('foobar');

    // 如果属性不存在，则使用一个默认值
    $filters = $session->get('filters', []);
}
```

在该用户会话的其余时间内，存储的属性将保留在会话中。

想要获取更多信息，请参见 [会话](https://symfony.com/doc/current/session.html)。

#### <span id="flash-messages">Flash消息</span>

你还可以在用户的​​会话中存储特殊消息，称为“闪烁”消息。按照设计，即时消息应仅使用一次：一旦你检索它们，它们就会自动从会话中消失。此功能使“闪烁”消息特别适合存储用户通知。

例如，假设你正在处理 [表单](https://symfony.com/doc/current/forms.html) 提交：

```php
use Symfony\Component\HttpFoundation\Request;

public function update(Request $request)
{
    // ...

    if ($form->isSubmitted() && $form->isValid()) {
        // 做某种处理

        $this->addFlash(
            'notice',
            'Your changes were saved!'
        );
        // $this->addFlash() 等同于 $request->getSession()->getFlashBag()->add()

        return $this->redirectToRoute(...);
    }

    return $this->render(...);
}
```

处理完请求后，控制器在会话中设置一条Flash消息，然后进行重定向。消息的键（`notice` 在此示例中）可以是任何东西：你将使用这个键来检索消息。

在下一页的模板中（甚至更好的是，在你的基本布局模板中），使用 [Twig全局应用程序变量](https://symfony.com/doc/current/templates.html#twig-app-variable) 提供的 `flashes()` 从会话中读取所有Flash消息：

```twig
{# templates/base.html.twig #}

{# 仅读取和显示一种flash消息类型 #}
{% for message in app.flashes('notice') %}
    <div class="flash-notice">
        {{ message }}
    </div>
{% endfor %}

{# 阅读并显示几种类型的flash消息 #}
{% for label, messages in app.flashes(['success', 'warning']) %}
    {% for message in messages %}
        <div class="flash-{{ label }}">
            {{ message }}
        </div>
    {% endfor %}
{% endfor %}

{# 阅读并显示所有的flash消息 #}
{% for label, messages in app.flashes %}
    {% for message in messages %}
        <div class="flash-{{ label }}">
            {{ message }}
        </div>
    {% endfor %}
{% endfor %}
```

通常使用 `notice`，`warning` 和 `error` 作为不同类型的flash信息的键，但你可以使用符合你的需求的任意键。

> 你可以使用 `peek()` 方法来检索消息，同时将其保存在包中。

### <span id="the-request-and-response-object">Request和Response对象

如 [前面](the_request_object_as_a_controller_argument) 所述，Symfony将把Request对象传递给任何带有Request类类型提示的控制器参数：：

```php
use Symfony\Component\HttpFoundation\Request;

public function index(Request $request)
{
    $request->isXmlHttpRequest(); // 是否是一个 Ajax 请求？

    $request->getPreferredLanguage(['en', 'fr']);

    // 分别获取 GET and POST 变量
    $request->query->get('page');
    $request->request->get('page');

    // 获取 SERVER 变量
    $request->server->get('HTTP_HOST');

    // 获取一个用 foo 标识的上传文件对象
    $request->files->get('foo');

    // 获取一个 COOKIE 值
    $request->cookies->get('PHPSESSID');

    // 获取一个规范化的小写键 HTTP 请求头
    $request->headers->get('host');
    $request->headers->get('content-type');
}
```

`Request` 类具有几个公共属性和方法，这些属性和方法返回有关该请求所需的任何信息。

像 `Request` 一样，`Response` 对象有一个公共头属性。该对象属于 `Symfony\Component\HttpFoundation\ResponseHeaderBag` 类，并提供用于获取和设置响应头的方法。头名称已标准化。作为一个结果，该名称 `Content-Type` 等同于名称 `content-type` 或 `content_type`。

在Symfony中，需要一个控制器来返回一个 `Response` 对象：

```php
use Symfony\Component\HttpFoundation\Response;

// 创建一个简单的携带 200 状态码（默认）的响应
$response = new Response('Hello '.$name, Response::HTTP_OK);

// 创建一个携带 200 状态码的 CSS-response
$response = new Response('<style> ... </style>');
$response->headers->set('Content-Type', 'text/css');
```

为方便起见，包含了不同的响应对象以解决不同的响应类型。其中一些会在下面提到。要了解更多有关 `Request` 和 `Response`（以及不同的Response类）的信息 ，请参见 [HttpFoundation组件文档](https://symfony.com/doc/current/components/http_foundation.html#component-http-foundation-request)。

#### <span id="accessing-configuration-values">访问配置值</span>

要从控制器获取任何 [配置参数](https://symfony.com/doc/current/configuration.html#configuration-parameters) 的值，请使用 `getParameter()` 辅助方法：

```php
// ...
public function index()
{
    $contentsDir = $this->getParameter('kernel.project_dir').'/contents';
    // ...
}
```

#### <span id="returning-json-response">返回JSON响应</span>

要从控制器返回 JSON 数据，请使用 `json()` 辅助方法。这将返回一个 `JsonResponse` 对象，该对象会自动对数据进行编码：

```php
// ...
public function index()
{
    // 返回 '{"username":"jane.doe"}' 并且设置属性 Content-Type 头
    return $this->json(['username' => 'jane.doe']);

    // 这个缩写定义了三个选项参数
    // return $this->json($data, $status = 200, $headers = [], $context = []);
}
```

如果在你的应用程序中启用了 [序列化程序服务](https://symfony.com/doc/current/serializer.html)，它将用于将数据序列化为JSON。否则，将使用 [json_encode](https://www.php.net/manual/en/function.json-encode.php) 实现该功能。

#### <span id="streaming-file-responses">流文件响应

你可以使用 `file()` 辅助方法从控制器内部提供一个文件：

```php
public function download()
{
    // 发送文件内容并强制浏览器下载它
    return $this->file('/path/to/some_file.pdf');
}
```

`file()` 辅助方法提供了一些参数来配置它的行为：

```php
use Symfony\Component\HttpFoundation\File\File;
use Symfony\Component\HttpFoundation\ResponseHeaderBag;

public function download()
{
    // 从 filesystem 中加载文件
    $file = new File('/path/to/some_file.pdf');

    return $this->file($file);

    // 重命名下载文件
    return $this->file($file, 'custom_name.pdf');

    // 在浏览器中显示文件内容而不是下载它
    return $this->file('invoice_3241.pdf', 'my_invoice.pdf', ResponseHeaderBag::DISPOSITION_INLINE);
}
```

### <span id="final-thoughts">最后的思考</span>

在Symfony中，控制器通常是一个类方法，用于接受请求并返回Response对象。当使用URL映射时，一个控制器将变得可访问并且可以查看其响应。

为了促进控制器的开发，Symfony提供了一个 `AbstractController`。它可用于扩展控制器类，从而允许访问一些常用的实用程序，例如 `render()` 和 `redirectToRoute()`。`AbstractController` 还提供了 `createNotFoundException()` 用于返回未找到响应的网页工具。

在其他文章中，你将学习如何在控制器内部使用特定的服务，这些服务将帮助你持久化数据库中的对象并从中获取对象，处理表单提交，处理缓存等等。

### <span id="keep-going">继续！</span>

接下来，了解有关 [使用Twig渲染模板](https://symfony.com/doc/current/templates.html) 的所有信息。

### <span id="learn-more-about-controllers">学习更多有关控制器的知识</span>

+ [扩展动作参数解析](https://symfony.com/doc/current/controller/argument_value_resolver.html)
+ [如何自定义错误页面](https://symfony.com/doc/current/controller/error_pages.html)
+ [如何将请求转发到另一个控制器](https://symfony.com/doc/current/controller/forwarding.html)
+ [如何将控制器定义为服务](https://symfony.com/doc/current/controller/service.html)
+ [如何在Symfony控制器中创建SOAP Web服务](https://symfony.com/doc/current/controller/soap_web_service.html)
+ [如何上传文件](https://symfony.com/doc/current/controller/upload_file.html)