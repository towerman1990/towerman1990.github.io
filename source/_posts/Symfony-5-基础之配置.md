---
title: Symfony 5 基础之配置
date: 2020-09-13 15:57:09
tags:
 - PHP
 - Symfony
---

### <span id="configuration-files">配置文件</span>

Symfony 应用程序的配置文件存储在 `conifg/` 目录，这个目录有默认的结构：

```bash
your-project/
├─ config/
│  ├─ packages/
│  ├─ bundles.php
│  ├─ routes.yaml
│  └─ services.yaml
├─ ...
```

`routes.yaml` 文件定义了 [路由配置](/2020/07/25/Symfony-5-基础之路由/)；`services.yaml` 文件配置了 [服务容器](https://symfony.com/doc/current/service_container.html) 中的服务；`bundles.php` 文件使你的应用程序中的软件包可用/不可用。

你将主要在 `config/packages/` 目录中工作。这个目录存储了应用程序中安装的每个软件包的配置。软件包（在Symfony中也称为“捆绑包”，在其他项目中也称为“插件/模块”）为你的项目添加了现成的功能。

<!-- more -->

使用 [Symfony Flex](https://symfony.com/doc/current/setup.html#symfony-flex)（默认在Symfony应用程序中启用）时，软件包会在安装过程中自动更新 `bundles.php` 文件并在 `config/ packages/` 中自动创建新文件。例如，这是 “API平台” 包创建的默认文件：

```yaml
# config/packages/api_platform.yaml
api_platform:
    mapping:
        paths: ['%kernel.project_dir%/src/Entity']
```

将配置拆分成许多小文件对于某些Symfony新手来说是令人生畏的。但是你会很快习惯它们，并且在安装软件包后几乎不需要更改这些文件。

> 要了解所有可用的配置选项，请查看 [Symfony配置参考](https://symfony.com/doc/current/reference/index.html) 或运行 `config：dump-reference` 命令。

#### <span id="configuration-formates">配置格式</span>

不像其他的框架，Symfony不会让你使用特定格式来配置应用程序。 Symfony 允许你在 YAML，XML 和 PHP 之间进行选择，在整个 Symfony 文档中，所有配置示例都将以这三种格式显示。

格式之间没有任何实际区别。实际上，Symfony 在运行应用程序之前将它们全部转换并缓存到 PHP 中，因此它们之间甚至没有任何性能差异。

安装软件包时，默认情况下会使用 YAML，因为它简洁易读。这些是每种格式的主要优点和缺点：

+ **YAML**，简单，干净，可读，但并非所有IDE都支持自动完成和验证。[了解YAML语法](https://symfony.com/doc/current/components/yaml/yaml_format.html)；
+ **XML**，在大多数的 IDE 中都能自动完成/验证，并由 PHP 进行本机解析，但有时它会生成过于冗长的配置。[学习XML语法](https://en.wikipedia.org/wiki/XML)；
+ **PHP**，强大的功能，它允许你创建动态配置，但是生成的配置比其他格式的配置可读性差。

#### <span id="importing-configuration-files">引入配置文件</span>

Symfony 使用 [Config组件](https://symfony.com/doc/current/components/config.html) 加载配置文件，该组件提供高级功能，例如导入其他配置文件，即使它们使用不同的格式也是如此：

```yaml
# config/services.yaml
imports:
    - { resource: 'legacy_config.php' }
    # 如果所加载的文件不存在，ignore_errors 会静默丢弃错误
    - { resource: 'my_config_file.xml', ignore_errors: true }
    # 还支持glob表达式来加载多个文件
    - { resource: '/etc/myapp/*.yaml' }

# ...
```

```xml
<!-- config/services.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        https://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/symfony
        https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

    <imports>
        <import resource="legacy_config.php"/>
        <!-- 如果所加载的文件不存在，ignore_errors 会静默丢弃错误 -->
        <import resource="my_config_file.yaml" ignore-errors="true"/>
        <!-- 还支持glob表达式来加载多个文件 -->
        <import resource="/etc/myapp/*.yaml"/>
    </imports>

    <!-- ... -->
</container>
```

```php
// config/services.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container) {
    $container->import('legacy_config.php');
    // 如果所加载的文件不存在，ignore_errors (第三个参数) 会静默丢弃错误
    $container->import('my_config_file.xml', null, true);
    // 还支持 glob 表达式来加载多个文件
    $container->import('/etc/myapp/*.yaml');
};

// ...
```

### <span id="configuration-parameters">配置参数</span>

有时在几个配置文件中使用相同的配置值。无需重复配置，你可以将其定义为“参数”，就像可重用的配置值一样。按照惯例，参数是在 `config/services.yaml` 文件中的 `parameter` 键下定义的：

```yaml
# config/services.yaml
parameters:
    # 参数名称是任意字符串（建议使用“app.”前缀，
    # 以更好地将您的参数与 Symfony 参数区分开）。
    app.admin_email: 'something@example.com'

    # 布尔值参数
    app.enable_v2_protocol: true

    # 数组/集合参数
    app.supported_locales: ['en', 'es', 'fr']

    # 二进制内容参数（使用 base64_encode() 编码内容）
    app.some_parameter: !!binary VGhpcyBpcyBhIEJlbGwgY2hhciAH

    # PHP 常量作为参数
    app.some_constant: !php/const GLOBAL_CONSTANT
    app.another_constant: !php/const App\Entity\BlogPost::MAX_ITEMS

# ...
```

```xml
<!-- config/services.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:framework="http://symfony.com/schema/dic/symfony"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        https://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/symfony
        https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

    <parameters>
        <!-- 参数名称是任意字符串（建议使用“app.”前缀，
             以更好地将您的参数与 Symfony 参数区分开）。 -->
        <parameter key="app.admin_email">something@example.com</parameter>

        <!-- 布尔值参数 -->
        <parameter key="app.enable_v2_protocol">true</parameter>
        <!-- if you prefer to store the boolean value as a string in the parameter -->
        <parameter key="app.enable_v2_protocol" type="string">true</parameter>

        <!-- 数组/集合参数 -->
        <parameter key="app.supported_locales" type="collection">
            <parameter>en</parameter>
            <parameter>es</parameter>
            <parameter>fr</parameter>
        </parameter>

        <!-- 二进制内容参数（使用 base64_encode() 编码内容） -->
        <parameter key="app.some_parameter" type="binary">VGhpcyBpcyBhIEJlbGwgY2hhciAH</parameter>

        <!-- PHP 常量作为参数 -->
        <parameter key="app.some_constant" type="constant">GLOBAL_CONSTANT</parameter>
        <parameter key="app.another_constant" type="constant">App\Entity\BlogPost::MAX_ITEMS</parameter>
    </parameters>

    <!-- ... -->
</container>
```

```php
// config/services.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

use App\Entity\BlogPost;

return static function (ContainerConfigurator $container) {
    $container->parameters()
        // 参数名称是任意字符串（建议使用“app.”前缀，
        // 以更好地将您的参数与 Symfony 参数区分开）。
        ->set('app.admin_email', 'something@example.com')

        // 布尔值参数
        ->set('app.enable_v2_protocol', true)

        // 数组/集合参数
        ->set('app.supported_locales', ['en', 'es', 'fr'])

        // 二进制内容参数（使用 PHP 转义串行）
        ->set('app.some_parameter', 'This is a Bell char: \x07')

        // PHP 常量作为参数
        ->set('app.some_constant', GLOBAL_CONSTANT)
        ->set('app.another_constant', BlogPost::MAX_ITEMS);
};

// ...
```

> 使用XML配置时，不会修剪标记之间的值。这意味着以下参数的值将为 '\n    something@example.com\n'：
> ```xml
<parameter key="app.admin_email">
    something@example.com
</parameter>
```

一旦定义完成，您可以使用特殊语法从任何其他配置文件中引用此参数值：将参数名称包装在两个 `%` 中（例如，`%app.admin_email%`）：

```yaml
# config/packages/some_package.yaml
some_package:
    # 任何被两个 % 包围的字符串都将替换为该参数值
    email_address: '%app.admin_email%'

    # ...
```

```xml
<!-- config/packages/some_package.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:framework="http://symfony.com/schema/dic/symfony"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        https://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/symfony
        https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

    <!-- 任何被两个 % 包围的字符串都将替换为该参数值 -->
    <some-package:config email-address="%app.admin_email%">
        <!-- ... -->
    </some-package:config>
</container>
```

```php
// config/packages/some_package.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container) {
    $container->extension('some_package', [
        // 任何被两个 % 包围的字符串都将替换为该参数值
        'email_address' => '%app.admin_email%',

        // ...
    ]);
};
```

> 如果某些参数值包含％字符，则需要通过添加另一个 `%` 来对其进行转义，因此 Symfony 不会将其视为对参数名称的引用：
> ```yaml
# config/services.yaml
parameters:
    # 解析为 'https://symfony.com/?foo=%s&amp;bar=%d'
    url_pattern: 'https://symfony.com/?foo=%%s&amp;bar=%%d'
```
> ```xml
<!-- config/services.xml -->
<parameters>
    <parameter key="url_pattern">http://symfony.com/?foo=%%s&amp;bar=%%d</parameter>
</parameters>
```
> ```php
// config/services.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container) {
    $container->parameters()
        ->set('url_pattern', 'http://symfony.com/?foo=%%s&amp;bar=%%d');
};
```

> 由于解析参数的方式，您不能使用它们动态地在导入中构建路径。
这表示以下内容无法正常运作：
> ```yaml
# config/services.yaml
imports:
    - { resource: '%kernel.project_dir%/somefile.yaml' }
```
> ```xml
<!-- config/services.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        https://symfony.com/schema/dic/services/services-1.0.xsd">

    <imports>
        <import resource="%kernel.project_dir%/somefile.yaml"/>
    </imports>
</container>
```
> ```
// config/services.php
$loader->import('%kernel.project_dir%/somefile.yaml');
```

配置参数在 Symfony 应用程序中非常常见。有些包甚至定义了自己的参数（例如，在安装翻译包时，会将新的语言环境参数添加到 `config/services.yaml`文件中）。

> 在本文的后面，您可以阅读如何 [在控制器和服务中获取配置参数](#accessing-configuration-parameters)。

### <span id="configuration-environments">配置环境</span>

你只有一个应用程序，但是无论你是否意识到它，都需要它在不同时间表现不同：

+ 在 **开发环境** 过程中，您想记录所有内容并公开漂亮的调试工具；
+ 部署到 **生产环境** 后，您希望对同一应用程序进行优化以提高速度，并且仅记录日志错误。

Symfony使用 `config/packages/` 中存储的文件来配置 [应用程序服务](https://symfony.com/doc/current/service_container.html)。换句话说，你可以通过更改加载的配置文件来更改应用程序的行为。这就是 Symfony 的 **配置环境** 的想法。

典型的 Symfony 应用程序始于三种环境：`dev`（用于本地开发），`prod`（用于生产服务器）和 `test`（用于自动测试）。在运行应用程序时，Symfony 会按以下顺序加载配置文件（最后一个文件可以覆盖先前文件中设置的值）：

1. `config/packages/*.yaml` （`*.xml` 和 `*.php` 文件也一样）；
2. `config/packages/<environment-name>/*.yaml` （`*.xml` 和 `*.php` 文件也一样）；
3. `config/packages/services.yaml` （`services.xml` 和 `services.php` 文件也一样）；

以默认情况下安装的框架包为例：

+ 首先，`config/packages/framework.yaml` 会在所有环境中加载，并使用一些选项配置框架。
+ 在 **生产** 环境中，将没有任何其他设置，因为没有 `config/packages/prod/framework.yaml` 文件。
+ 在 **开发** 环境中，也没有文件（`config/packages/dev/framework.yaml` 不存在）。
+ 在 **测试** 环境中，将加载 `config/packages/test/framework.yaml` 文件，以覆盖先前在 `config/packages/framework.yaml` 中配置的某些设置。

实际上，每种环境仅与其他环境有所不同。这意味着所有环境都共享大量的通用配置，这些通用配置直接放在 `config/packages/` 目录中的文件中。

> 请参阅 [Kernel类](https://symfony.com/doc/current/configuration/front_controllers_and_kernel.html) 的 `configureContainer()` 方法以了解有关配置文件的加载顺序的所有信息。

#### <span id="selecting-the-active-environment">选择活动环境</span>

Symfony 应用程序具有一个名为 `.env` 的文件，该文件位于项目根目录中。此文件用于定义环境变量的值，本文稍后将对此进行详细解释。

打开 `.env` 文件（或者更好的是，创建文件为 `.env.local` 文件），并编辑 `APP_ENV` 变量的值以更改应用程序运行的环境。例如，若要在生产环境中运行应用程序：

```bash
# .env (或 .env.local)
APP_ENV=prod
```

此值用于 Web 和控制台命令。但是，您可以通过在运行命令之前设置"APP_ENV来重写命令：

```bash
# 使用定义在 .env 文件中的环境变量
> php bin/console command_name

# 忽略 .env 文件并在生成环境中执行此命令
> APP_ENV=prod php bin/console command_name
```

#### <span id="creating-a-new-environment">创建一个新的环境</span>

Symfony 提供的默认三个环境对于大多数项目来说已经足够了，但您也可以定义自己的环境。例如，这是如何定义一个暂存环境，客户端可以在该环境中测试项目，然后再进入生产环境：

1. 创建与环境同名的配置目录（本例中为 `config/packages/staging/`）;
2. 在 `config/packages/staging/` 中添加所需的配置文件/以定义新环境的行为。Symfony 首先加载 `config/packages/*.yaml` 文件，因此你只需将差异配置为这些文件;
3. 使用 `APP_ENV` 环境变量选择 `staging` 环境，如上一节所述。

> 环境彼此相似是很常见的，因此可以在 `config/packages/<environment-name>/` 目录之间使用 [符号链接](https://en.wikipedia.org/wiki/Symbolic_link) 来重用相同的配置。

### <span id="configuration-based-on-environment-variables">基于环境变量的配置</span>

使用 [环境变量](https://en.wikipedia.org/wiki/Environment_variable) 是配置取决于应用程序运行地点的选项的常见做法（例如，数据库凭据在生产中通常与本地计算机不同）。如果值是敏感的，您甚至可以 [将它们加密为加密串](https://symfony.com/doc/current/configuration/secrets.html)。

你可以使用特殊语法 `%env(ENV_VAR_NAME)%` 引用环境变量。这些选项的值在运行时解析（每个请求只解析一次，不会影响性能）。

此示例演示如何使用环境变量配置数据库连接：

```yaml
# config/packages/doctrine.yaml
doctrine:
    dbal:
        # 根据惯例， 环境变量名称总是大写
        url: '%env(resolve:DATABASE_URL)%'
    # ...
```

```xml
<!-- config/packages/doctrine.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:doctrine="http://symfony.com/schema/dic/doctrine"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        https://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/doctrine
        https://symfony.com/schema/dic/doctrine/doctrine-1.0.xsd">

    <doctrine:config>
        <!-- 根据惯例， 环境变量名称总是大写 -->
        <doctrine:dbal url="%env(resolve:DATABASE_URL)%"/>
    </doctrine:config>

</container>
```

```php
// config/packages/doctrine.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

return static function (ContainerConfigurator $container) {
    $container->extension('doctrine', [
        'dbal' => [
            // 根据惯例， 环境变量名称总是大写
            'url' => '%env(resolve:DATABASE_URL)%',
        ]
    ]);
};
```

> 环境变量的值只能是字符串，但 Symfony 包括一些 [环境变量处理器](https://symfony.com/doc/current/configuration/env_var_processors.html) 来转换其内容（例如，将字符串值转换为整数）。

要定义环境变量的值，有几个选项：

+ [将值加到 .env 文件](#configuring-environment-variables-in-env-files);
+ [将值加密为加密串]();
+ 将值设置为 shell 或 Web 服务器中的实际环境变量。

> 一些主机 - 如 SymfonyCloud - 提供简单的 [实用程序](https://symfony.com/doc/master/cloud/cookbooks/env.html) 来管理生产中的环境变量。

>请注意，转储 `_SERVER` 和 `$_ENV` 变量的内容或输出 `phpinfo()` 内容将显示环境变量的值，公开敏感信息，如数据库凭据。
>
> 环境变量的值也会在 [Symfony 探查器](https://symfony.com/doc/current/profiler.html) 的 Web 界面中公开。实际上，这应该不是什么问题，因为绝不能在生产环境中启用 Web 探查器。

#### <span id="configuring-environment-variables-in-env-files">在 .env 文件中配置环境变量</span>

Symfony 没有在 shell 或 Web 服务器中定义环境变量，而是提供了一种在位于项目根目录的 `.env`（带前导点）文件中定义它们的便捷方法。

`.env` 文件在每个请求上读取并解析，其环境变量将添加到 `$_ENV` 和 `_ENV _SERVER` PHP 变量中。任何现有的环境变量永远不会被 `.env` 中定义的值覆盖，因此你可以同时使用这两个值。

例如，若要定义前文所说的 `DATABASE_URL` 环境变量，可以添加：

```bash
# .env
DATABASE_URL="mysql://db_user:db_password@127.0.0.1:3306/db_name"
```

此文件应提交到存储库，并且（由于这一事实）应仅包含适合本地发展的“默认”值。此文件不应包含生产值.

除了你自己的环境变量之外，此 `.env` 文件还包含由应用程序中安装的第三方包定义的环境变量（在安装包时，[Symfony Flex](https://symfony.com/doc/current/setup.html#symfony-flex) 会自动添加它们）。

#### <span id="env-file-syntax">.env 文件语法</span>

添加注释，用 `#`：

```bash
# 数据库凭证
DB_USER=root
DB_PASS=pass # 这里是密码
```

通过将变量前缀为 `$`：

```bash
DB_USER=root
DB_PASS=${DB_USER}pass # include the user as a password prefix
```

> 当某些环境变量依赖于其他环境变量的值时，顺序很重要。在上面的示例中，DB_PASS 在操作后 DB_USER。此外，如果定义多个 .env 文件并 DB_PASS，则其值将取决于 DB_USER 文件中定义的 DB_USER 值，而不是在此文件中定义的值。

定义一个默认值，以防未设置环境变量：

```bash
DB_USER=
DB_PASS=${DB_USER:-root}pass # 在 DB_PASS=rootpass 中的结果
```

通过 `$()` 嵌入命令（Windows 上不支持）：

```bash
START_TIME=$(date)
```

> 使用 $() 可能不会生效，这依赖于你使用的 shell。

> 由于 .env 文件是常规 shell 脚本，你可以在你自己的 shell 脚本中生成该文件：
> ```bash
> source .env
```

#### <span id="overriding-environment-values-via-envlocal">通过 .env.local 覆盖环境值</span>

如果需要重写环境值（例如，本地计算机上的不同值），可以在 .env.local 文件中这样做：

```bash
# .env.local
DATABASE_URL="mysql://root:@127.0.0.1:3306/my_database_name"
```

这个文件应被 git 忽略，不应提交到存储库。其他几个 .env 文件可用于在正确的情况下设置环境变量：

+ `.env`：定义应用程序所需的环境变量的默认值;
+ `.env.local`：覆盖所有环境的默认值，但仅在包含该文件的计算机上。此文件不应提交到存储库，并且在测试环境中被忽略（因为测试应该为每个人产生相同的结果）;
+ `.env.<environment>` （例如 .env.test）：仅覆盖一个环境，但所有计算机（提交这些文件）;
+ `.env.<environment>.local` （例如 .env.test.local）：定义特定于计算机的环境变量仅针对一个环境进行覆盖。它类似于 `.env.local`，但重写仅适用于一个环境。

*真正*的环境变量总是赢得由任何 .env 文件创建的环境变量。

`.env` 和 `.env.<environment>` 文件应提交到存储库，因为它们对于所有开发人员和计算机都是一样的。但是，以 `.local` （`.env.local` 和 `.env.<environment>.local`）结尾的 env 文件，**不应提交**，因为只有你使用它们。事实上，Symfony 附带的 `.gitignore` 文件会阻止它们被提交。

> 2018 年 11 月之前创建的应用程序的系统略有不同，涉及 `.env.dist` 文件。有关升级的信息，请参阅：[2018 年 11 月对 .env 的更改和如何更新](https://symfony.com/doc/current/configuration/dot-env-changes.html)。

#### <span id="configuring-environment-variables-in-production">在生产环境中配置环境变量</span>

在生产中，也会在每个请求上解析和加载 `.env` 文件。因此，定义环境变量的最简单方法是将 `.env.local` 文件部署到具有生产值的生产服务器。

为了提高性能，您可以选择运行转储 env 命令（在 [Symfony Flex](https://symfony.com/doc/current/setup.html#symfony-flex) 1.2 或更晚一些中提供）：

```bash
# 解析所有的 .env 文件并导出他们的最终值到 .env.local.php
>  composer dump-env prod
```

运行此命令后，Symfony 将加载 `.env.local.php` 文件来获取环境变量，并且不会花时间分析 `.env` 文件。

> 更新部署工具/工作流以在每次部署后运行 `dump-env` 命令以提高应用程序性能。

#### <span id="encrypting-environment-variables">加密环境变量（机密）</span>

如果变量的值是敏感的（例如 API 密钥或数据库密码），则可以使用 [机密管理系统](https://symfony.com/doc/current/configuration/secrets.html) 对值进行加密，而不是定义真实环境变量或将变量添加到 `.env` 文件中。

#### <span id="listing-environment-variables">列出环境变量</span>

无论你如何设置环境变量，都可以通过运行下面命令来查看包含其值的完整列表：

```bash
 php bin/console debug:container --env-vars

---------------- ----------------- ---------------------------------------------
 Name             Default value     Real value
---------------- ----------------- ---------------------------------------------
 APP_SECRET       n/a               "471a62e2d601a8952deb186e44186cb3"
 FOO              "[1, "2.5", 3]"   n/a
 BAR              null              n/a
---------------- ----------------- ---------------------------------------------

# 你也可以通过名称过滤环境变量列表：
> php bin/console debug:container --env-vars foo

# 运行这个命令来运行一个特殊环境变量所有的细节：
> php bin/console debug:container --env-var=FOO
 ```

### <span id="accessing-configuration-parameters">访问配置参数</span>

控制器和服务可以访问所有配置参数。这包括 [你自己定义的参数](#configuration-parameters) 和包/捆绑包创建的参数。运行以下命令以查看应用程序中存在的所有参数：

```bash
php bin/console debug:container --parameters
```

在从 [AbstractController](https://symfony.com/doc/current/controller.html#the-base-controller-class-services) 扩展的控制器中，使用 `getParameter()` 辅助函数：

```bash
// src/Controller/UserController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

class UserController extends AbstractController
{
    // ...

    public function index()
    {
        $projectDir = $this->getParameter('kernel.project_dir');
        $adminEmail = $this->getParameter('app.admin_email');

        // ...
    }
}
```

在服务和控制器中，不是从抽象控制器扩展，将参数注入为其构造函数的参数。你必须显式注入它们，因为服务 [自动写入](https://symfony.com/doc/current/service_container/autowiring.html) 对参数不起作用：

```yaml
# config/services.yaml
parameters:
    app.contents_dir: '...'

services:
    App\Service\MessageGenerator:
        arguments:
            $contentsDir: '%app.contents_dir%'
```

```xml
<!-- config/services.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        https://symfony.com/schema/dic/services/services-1.0.xsd">

    <parameters>
        <parameter key="app.contents_dir">...</parameter>
    </parameters>

    <services>
        <service id="App\Service\MessageGenerator">
            <argument key="$contentsDir">%app.contents_dir%</argument>
        </service>
    </services>
</container>
```

```php
// config/services.php
namespace Symfony\Component\DependencyInjection\Loader\Configurator;

use App\Service\MessageGenerator;

return static function (ContainerConfigurator $container) {
    $container->parameters()
        ->set('app.contents_dir', '...');

    $container->services()
        ->get(MessageGenerator::class)
            ->arg('$contentsDir', '%app.contents_dir%');
};
```

如果一遍又一遍地注入相同的参数，请 `services._defaults.bind` 选项。每当服务构造函数或控制器操作定义具有该确切名称的参数时，都会自动注入该选项中定义的参数。例如，每当服务/控制器定义 [kernel.project_dir 参数](https://symfony.com/doc/current/reference/configuration/kernel.html#configuration-kernel-project-directory) 时，请注入 `$projectDir` 参数的值，请使用以下方法：

```bash
# config/services.yaml
services:
    _defaults:
        bind:
            # 将此值传递给$projectDir文件（包括控制器参数）
            # 创建的任何服务的任何参数
            $projectDir: '%kernel.project_dir%'

    # ...
```

> 阅读有关 [按名称和/或类型绑定参数](https://symfony.com/doc/current/service_container.html#services-binding) 的文章，了解有关此强大功能的详细信息。

最后，如果某些服务需要访问大量参数，而不是单独注入每个参数，你可以通过使用 `Symfony\Component\DependencyInjection\ParameterBag\ContainerBagInterfac` 参数约定其任何构造函数参数，一次注入所有应用程序参数：

```php
// src/Service/MessageGenerator.php
namespace App\Service;

// ...

use Symfony\Component\DependencyInjection\ParameterBag\ContainerBagInterface;

class MessageGenerator
{
    private $params;

    public function __construct(ContainerBagInterface $params)
    {
        $this->params = $params;
    }

    public function someMethod()
    {
        // 从存储所有参数的 $this 参数获取任何容器参数，这些参数存储所有参数
        $sender = $this->params->get('mailer_sender');
        // ...
    }
}
```

### <span id="keep-going">继续前进！</span>

祝贺！你已经解决了 Symfony 的基础知识。接下来，通过按照指南单独了解 Symfony 的每个部分。通过考核：

+ [表单](https://symfony.com/doc/current/forms.html)
+ [数据库和 Doctrine ORM](https://symfony.com/doc/current/doctrine.html)
+ [服务容器](https://symfony.com/doc/current/service_container.html)
+ [安全](https://symfony.com/doc/current/security.html)
+ [使用 Mailer 发送邮件](https://symfony.com/doc/current/mailer.html)
+ [日志](https://symfony.com/doc/current/logging.html)

以及与配置相关的所有其他主题：

+ [2018 年 11 月 更改至.env 及如何更新](https://symfony.com/doc/current/configuration/dot-env-changes.html)
+ [环境变量处理器](https://symfony.com/doc/current/configuration/env_var_processors.html)
+ [了解前控制器、内核和环境如何协同工作](https://symfony.com/doc/current/configuration/front_controllers_and_kernel.html)
+ [使用 MicroKernelTrait 构建您自己的框架](https://symfony.com/doc/current/configuration/micro_kernel_trait.html)
+ [如何创建具有多个内核的符号应用程序](https://symfony.com/doc/current/configuration/multiple_kernels.html)
+ [如何覆盖 Symfony 的默认目录结构](https://symfony.com/doc/current/configuration/override_dir_structure.html)
+ [如何保持敏感信息的秘密](https://symfony.com/doc/current/configuration/secrets.html)
+ [在依赖注入类中使用参数](https://symfony.com/doc/current/configuration/using_parameters_in_dic.html)
