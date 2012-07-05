架构
====

本篇将说明Symfony2的架构。

理解文件目录结构
----------------

基于Symfony2的应用（\ :term:`application`\ ），代码、配置、资源文件等的目录结构可以很灵活。\ *标准发行版*\ 里采用的是官方推荐的典型方案：

* ``app/``:    应用的配置文件；
* ``src/``:    项目的PHP代码；
* ``vendor/``: 第三方代码；
* ``web/``:    站点根目录。

``web/``\ 目录
~~~~~~~~~~~~~~

站点根目录里放的是图片、样式表和Javscript这一类可以公开访问的静态文件。此外，前端控制器（\ :term:`front controller`\ ）也位于这个目录里：

.. code-block:: php

    // web/app.php
    require_once __DIR__.'/../app/bootstrap.php.cache';
    require_once __DIR__.'/../app/AppKernel.php';

    use Symfony\Component\HttpFoundation\Request;

    $kernel = new AppKernel('prod', false);
    $kernel->loadClassCache();
    $kernel->handle(Request::createFromGlobals())->send();

可以看到，框架的核心代码首先引用了\ ``bootstrap.php.cache``\ 文件，对框架的各组件进行预加载，并注册autoloader。

与其他的入口文件一样，\ ``app.php``\ 使用一个Kernel Class（\ ``AppKernel``\ ）来初始化整个程序。

.. _the-app-dir:

``app/``\ 目录
~~~~~~~~~~~~~~

``AppKernel``\ 类是项目配置的起始文件，位于\ ``app/``\ 目录。

这个类必须实现两个方法：

* ``registerBundles()`` 返回项目用到的代码包；

* ``registerContainerConfiguration()`` 加载项目的配置信息。

PHP的自动加载可以通过\ ``app/autoload.php``\ 来配置：

.. code-block:: php

    // app/autoload.php
    use Symfony\Component\ClassLoader\UniversalClassLoader;

    $loader = new UniversalClassLoader();
    $loader->registerNamespaces(array(
        'Symfony'          => array(__DIR__.'/../vendor/symfony/src', __DIR__.'/../vendor/bundles'),
        'Sensio'           => __DIR__.'/../vendor/bundles',
        'JMS'              => __DIR__.'/../vendor/bundles',
        'Doctrine\\Common' => __DIR__.'/../vendor/doctrine-common/lib',
        'Doctrine\\DBAL'   => __DIR__.'/../vendor/doctrine-dbal/lib',
        'Doctrine'         => __DIR__.'/../vendor/doctrine/lib',
        'Monolog'          => __DIR__.'/../vendor/monolog/src',
        'Assetic'          => __DIR__.'/../vendor/assetic/src',
        'Metadata'         => __DIR__.'/../vendor/metadata/src',
    ));
    $loader->registerPrefixes(array(
        'Twig_Extensions_' => __DIR__.'/../vendor/twig-extensions/lib',
        'Twig_'            => __DIR__.'/../vendor/twig/lib',
    ));

    // ...

    $loader->registerNamespaceFallbacks(array(
        __DIR__.'/../src',
    ));
    $loader->register();

如果PHP文件名和类名符合\ `PHP5.3命名空间规范`_\ （standards），或者符合\ `PEAR类命名范例`_\ （convention），就可以由\ :class:`Symfony\\Component\\ClassLoader\\UniversalClassLoader`\ 实现自动加载。例子里的第三方依赖都位于\ ``vendor/``\ 目录directory, 但这只是一个推荐的做法，实际上，如果配置正确，你可以把文件放在任何的位置。

.. note::

    如果你想了解更多关于Symfony2自动加载的细节，可以阅读《\ ":doc:`/components/class_loader`"\ 》。

理解代码包体系
--------------

接下来介绍的是Symfony2里最有特色、也最强大的设计：代码包（\ :term:`bundle`\ ）。

代码包有点类似其他软件系统里的“插件”。那为什么要称之为\ *代码包*\ ，而不是\ *插件*\ 呢？这是因为，Symfony2里\ *所有的*\ 组成部分，不管是你自己写的代码，还是核心的框架功能都是以代码包的形式存在的。代码包在Symfony2里是“有本地户口的”（意译）。你可以非常方便的引入第三方的代码，或者向其他开发者或者开源社区发布你自己的代码包。你可以指定要启用哪些功能，并进行必要的优化。Symfony2将你的代码和框架代码视作同等重要。

注册代码包
~~~~~~~~~~

每个应用都是由在\ ``AppKernel``\ 类的\ ``registerBundles()``\ 方法里声明的代码包组成的。每一个代码包都包含了一个描述性的\ ``Bundle``\ 类：

.. code-block:: php

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
            new Symfony\Bundle\SecurityBundle\SecurityBundle(),
            new Symfony\Bundle\TwigBundle\TwigBundle(),
            new Symfony\Bundle\MonologBundle\MonologBundle(),
            new Symfony\Bundle\SwiftmailerBundle\SwiftmailerBundle(),
            new Symfony\Bundle\DoctrineBundle\DoctrineBundle(),
            new Symfony\Bundle\AsseticBundle\AsseticBundle(),
            new Sensio\Bundle\FrameworkExtraBundle\SensioFrameworkExtraBundle(),
            new JMS\SecurityExtraBundle\JMSSecurityExtraBundle(),
        );

        if (in_array($this->getEnvironment(), array('dev', 'test'))) {
            $bundles[] = new Acme\DemoBundle\AcmeDemoBundle();
            $bundles[] = new Symfony\Bundle\WebProfilerBundle\WebProfilerBundle();
            $bundles[] = new Sensio\Bundle\DistributionBundle\SensioDistributionBundle();
            $bundles[] = new Sensio\Bundle\GeneratorBundle\SensioGeneratorBundle();
        }

        return $bundles;
    }

除了我们已经多次提到的\ ``AcmeDemoBundle``\ ，可以看到框架还启用了很多其他的代码包，如\ ``FrameworkBundle``\ ，\ ``DoctrineBundle``\ ，\ ``SwiftmailerBundle``\ ，\ ``AsseticBundle``\ 等等，他们都是框架的核心组成部分。

配置代码包
~~~~~~~~~~

代码包可以通过YAML，XML，或PHP代码格式的配置文件进行配置，下面是框架的默认配置：

.. code-block:: yaml

    # app/config/config.yml
    imports:
        - { resource: parameters.ini }
        - { resource: security.yml }

    framework:
        secret:          "%secret%"
        charset:         UTF-8
        router:          { resource: "%kernel.root_dir%/config/routing.yml" }
        form:            true
        csrf_protection: true
        validation:      { enable_annotations: true }
        templating:      { engines: ['twig'] } #assets_version: SomeVersionScheme
        session:
            default_locale: "%locale%"
            auto_start:     true

    # Twig Configuration
    twig:
        debug:            "%kernel.debug%"
        strict_variables: "%kernel.debug%"

    # Assetic Configuration
    assetic:
        debug:          "%kernel.debug%"
        use_controller: false
        filters:
            cssrewrite: ~
            # closure:
            #     jar: "%kernel.root_dir%/java/compiler.jar"
            # yui_css:
            #     jar: "%kernel.root_dir%/java/yuicompressor-2.4.2.jar"

    # Doctrine Configuration
    doctrine:
        dbal:
            driver:   "%database_driver%"
            host:     "%database_host%"
            dbname:   "%database_name%"
            user:     "%database_user%"
            password: "%database_password%"
            charset:  UTF8

        orm:
            auto_generate_proxy_classes: "%kernel.debug%"
            auto_mapping: true

    # Swiftmailer Configuration
    swiftmailer:
        transport: "%mailer_transport%"
        host:      "%mailer_host%"
        username:  "%mailer_user%"
        password:  "%mailer_password%"

    jms_security_extra:
        secure_controllers:  true
        secure_all_services: false

每一个类似\ ``framework``\ 的键值对应的都是某一个具体代码包的配置。比如，\ ``framework``\ 配置的是\ ``FrameworkBundle``\ ，而\ ``swiftmailer``\ 配置的是\ ``SwiftmailerBundle``\ 。

每一个运行环境（\ :term:`environment`\ ）的默认配置都可以被覆盖。比如，\ ``dev``\ 环境所加载的\ ``config_dev.yml``\ 文件，即是对主配置（\ ``config.yml``\ ）的一个扩展，其启用了一些调试的工具：

.. code-block:: yaml

    # app/config/config_dev.yml
    imports:
        - { resource: config.yml }

    framework:
        router:   { resource: "%kernel.root_dir%/config/routing_dev.yml" }
        profiler: { only_exceptions: false }

    web_profiler:
        toolbar: true
        intercept_redirects: false

    monolog:
        handlers:
            main:
                type:  stream
                path:  "%kernel.logs_dir%/%kernel.environment%.log"
                level: debug
            firephp:
                type:  firephp
                level: info

    assetic:
        use_controller: true

扩展代码包的功能
~~~~~~~~~~~~~~~~

除了可以对代码进行组织、管理相关的配置，代码包还可以被扩展，从而具备你所需要的功能。代码包的继承特性允许你对任何代码包进行重新定义，如控制器或者模板。这正是逻辑代称适用的场合（如\ ``@AcmeDemoBundle/Controller/SecuredController.php``\ ），因为代称可以避免将代码的路径信息写死。

文件名的代称
............

当你需要引用代码包里的某一个文件，你可以使用类似这样的格式：\ ``@代码包名称/目录1/目录2/文件名``\ ，Symfony2将把\ ``@代码包``\ 解析成实际的路径。举例来说，由于Symfony2框架知道\ ``AcmeDemoBundle``\ 所在的位置，\ ``@AcmeDemoBundle/Controller/DemoController.php``\ 将会被正确地解析成\ ``src/Acme/DemoBundle/Controller/DemoController.php``\ 。

控制器的代称
............

控制器代称的格式是：``代码包名称:控制器:动作方法``\ 。例如，\ ``AcmeDemoBundle:Welcome:index``\ 指向的是\ ``Acme\DemoBundle\Controller\WelcomeController``\ 类的\ ``indexAction`` 方法。

模板的代称
..........

对于模板的代称，如\ ``AcmeDemoBundle:Welcome:index.html.twig``\ 将被转化为实际的文件路径：\ ``src/Acme/DemoBundle/Resources/views/Welcome/index.html.twig``\ 。模板还可以有别的更有意思的实现方式，比如，可以保存在数据库里，而不是存成文件。

扩展代码包
..........

如果你的代码遵循了以上的格式，那么你就可以使用代码包的继承（\ :doc:`bundle inheritance</cookbook/bundles/inheritance>`\ ）了，对任何一个文件、控制器或者模板进行重载。例如，你可以创建一个名为\ ``AcmeNewBundle``\ 的代码包，并指明其所继承对象是\ ``AcmeDemoBundle``\ 。当Symfony加载\ ``AcmeDemoBundle:Welcome:index``\ 控制器时，框架首先将在\ ``AcmeNewBundle``\ 所在路径里寻找\ ``WelcomeController``\ 控制器类。所以，在Symfony2里，代码包里几乎任何部分都可以被继承、重载。

通过使用Symfony2，在你的多个项目之间重用代码变得很容易了，对吧？

.. _using-vendors:

引入第三方代码
--------------

大多数情况，你的项目都会需要用到一些第三方的代码。这些代码应该被保存在\ ``vendor/``\ 目录里。在这个目录里你可以找到诸如：Symfony2、SwiftMailer、Doctrine ORM、Twig模板引擎等等组件。

理解缓存和日志
--------------

Symfony2有可能是速度最快的全功能框架之一。但如果框架需要处理大量的YAML和XML文件，又是怎么保证运行速度的？这就归功于缓存系统了。由于应用程序的配置文件相对固定，所以Symfony2只在第一次处理请求时对各类配置文件进行转换，并在\ ``app/cache/``\ 目录里保存为纯PHP代码的格式。在开发环境下使用时，Symfony2会有判断地自动更新配置文件的缓存。在生产环境里，配置的更新则需要你手工来进行。

开发Web应用可能遇到各种各样的问题，\ ``app/logs/``\ 目录里的日志文件将会记录框架对请求进行处理的细节，为你解决问题提供依据。

使用命令行工具
--------------

你可以通过命令行工具(\ ``app/console``\ ）来管理你的应用。这样就避免了在一些重复的、麻烦的工作上浪费时间。

你可以直接运行这个命令来查看说明：

.. code-block:: bash

    php app/console

或者通过\ ``--help``\ 选项来查看具体命令的帮助信息：

.. code-block:: bash

    php app/console router:debug --help

结论
----

Symfony2的设计目标之一就是为你提供“自由度”，诸如改变文件路径、管理配置、对复杂架构的日常维护等等。所以，当你认为有必要，你完全可以对Symfony2框架进行改造。

本篇是快速入门的最后一篇，你还可以在不同的专题教程里了解如何成为一个Symfony2的专家。访问地址是：《\ :doc:`/book/index`\ 》。

.. _PHP5.3命名空间规范: http://symfony.com/PSR0
.. _PEAR类命名范例: http://pear.php.net/
