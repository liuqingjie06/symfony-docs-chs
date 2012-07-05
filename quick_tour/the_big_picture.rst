概述
====

快速了解Symfony2：本文将向你介绍Symfony2最重要的几个概念，并用一个简单的例子向你演示如何用Symfony2进行开发。

如果你使用过其他的Web开发框架，你对Symfony2应该已经有几分熟悉了。从没用过框架？那我们就一起了解这个开发Web应用的新方式吧。

.. tip::

    想知道为什么你应该使用开发框架？在什么条件下使用？你可以参考《\ `5分钟了解Symfony`_\ 》（英文）。

下载Symfony2
------------

首先，确认你安装了Web服务器（如Apache、Nginx）和5.3.2以上版本的PHP。

然后，下载“\ `Symfony2标准版`_\ ”。这个“标准版”，是Symfony发行版（\ :term:`distribution`\ ）之一，标准版按照常见的需求进行配置，包含了简单的示例代码。

把下载下来的文件解压到你Web站点的根目录，你应该可以看到\ ``Symfony/``\ 目录里的文件结构如下所示：

.. code-block:: text

    www/ <- 你的Web服务器根目录
        Symfony/ <- 解压后的文件
            app/
                cache/
                config/
                logs/
                Resources/
            bin/
            src/
                Acme/
                    DemoBundle/
                        Controller/
                        Resources/
                        ...
            vendor/
                symfony/
                doctrine/
                ...
            web/
                app.php
                ...

.. note::

    如果你下载的是不包括第三方代码包的发行版（\ *without vendors*\ ），可以通过执行下面的命令来手动安装：

    .. code-block:: bash

        php bin/vendors install

检查配置
--------

Symfony2自带了一个用来测试服务器配置是否符合运行要求的脚本，你可以通过访问下面的URL来查看（以下示例都假设你的Web服务器是通过localhost来访问，如果你配置了其他的域名，请注意替换）：

.. code-block:: text

    http://localhost/Symfony/web/config.php

如果有不满足Symfony2运行条件的问题，运行环境测试脚本会向你提供修正的建议。一切就绪后，点击“*Bypass configuration and go to the Welcome page*”来访问“真正的”Symfony2页面。

.. code-block:: text

    http://localhost/Symfony/web/app_dev.php/

Symfony2这时应该会显示一个欢迎页面，辛苦了：）

.. image:: /images/quick_tour/welcome.jpg
   :align: center

了解基本概念
------------

使用开发框架的主要目的之一就是“\ `任务的抽象和分工`_\ ”（英文）。这么做可以使你的代码组织结构更好，在应用程序的开发、维护过程中，可以避免数据库操作、HTML和业务逻辑等等不同方面的代码混在一起。要通过使用Symfony2来达成这个目标，你首先需要了解几个概念和术语。

.. tip::

    想要了解为什么使用框架可以避免代码的混淆？请阅读“\ :doc:`/book/from_flat_php_to_symfony2`\ ”

标准发行版包含了一些示例代码，你可以阅读它们来了解Symfony2框架里一些主要的概念。试试访问下面的URL，让Symfony2和你（把\ *Fabien*\ 替换成你自己的名字）打个招呼：

.. code-block:: text

    http://localhost/Symfony/web/app_dev.php/demo/hello/Fabien

.. image:: /images/quick_tour/hello_fabien.png
   :align: center

Symfony2框架到底怎么实现这个页面的？我们先看看这个URL：

* ``app_dev.php``\ ：这是一个前端控制器（\ :term:`front controller`\ ）。它是应用程序的一个入口，用来响应用户各种各样的请求；

* ``/demo/hello/Fabien``\ ：这是一个虚拟的访问路径（\ *virtual path*\ ，因为实际的文件并不存在），指向用户想要访问的资源。

作为一个程序员，你需要做的是把用户的\ *请求*\ （\ ``/demo/hello/Fabien``\ ）指向与其相关的资源（\ *resource*\ ），在上面的例子里，所请求的资源是显示“\ ``Hello Fabien!``\ ”的HTML页面。

URL路由
~~~~~~~

Symfony2可以预先配置一系列规则，基于这些规则将用户的请求交给与之对应的代码来处理。默认情况下，这些规则（称之为“路由”）定义在\ ``app/config/routing.yml``\ 文件里。如果你运行的是开发\ :ref:`环境<quick-tour-big-picture-environments>`\ （\ ``dev``\ ），即访问的是app\_\ *dev*\ .php入口文件，那么\ ``app/config/routing_dev.yml``\ 配置文件也会被加载。在标准版里，指向这些“demo”页面的路由规则如下：

.. code-block:: yaml

    # app/config/routing_dev.yml
    _welcome:
        pattern:  /
        defaults: { _controller: AcmeDemoBundle:Welcome:index }

    _demo:
        resource: "@AcmeDemoBundle/Controller/DemoController.php"
        type:     annotation
        prefix:   /demo

    # ...

“#”所在的行是注释，之后的最前面三行，定义了哪些代码与用户请求“\ ``/``\ ”（也就是你之前看到的欢迎页面）相对应。当请求发生时，\ ``AcmeDemoBundle:Welcome:index``\ 控制器将会执行。随后的章节里，你将了解更多细节。

.. tip::

    Symfony2标准版使用\ `YAML`_\ 作为配置文件的格式，但Symfony2还支持XML、PHP和注解。这些不同的格式互相之间是兼容的，可以混合起来使用。同时，你的程序的性能与配置文件的格式无关，因为所有的配置都会被缓存起来。

控制器
~~~~~~

“控制器”这个有点诈唬的名字实际上是指用来处理\ *请求*\ ，并返回相应\ *响应*\ 的PHP函数或类方法。Symfony2并不直接操作PHP全局变量和公共函数——如\ ``$_GET``\ 或\ ``header()``\ ——来处理HTTP请求相关的信息，而是与对象交互：即\ :class:`Symfony\\Component\\HttpFoundation\\Request`\ 和\ :class:`Symfony\\Component\\HttpFoundation\\Response`\ 。下面举一个最简单的控制器的例子：创建一个响应，并返回给用户：

.. code-block:: php

    use Symfony\Component\HttpFoundation\Response;

    $name = $request->query->get('name');

    return new Response('Hello '.$name, 200, array('Content-Type' => 'text/plain'));

.. note::

    Symfony2完全遵守HTTP协议，而HTTP协议定义了Web上几乎所有的通信。你可以阅读\ ":doc:`/book/http_fundamentals`"\ 来了解协议的细节，以及Symfony2提供了哪些开发的便利。

Symfony2通过路由配置里的\ ``_controller``\ 来选择控制器，在上面的例子里这个值是\ ``AcmeDemoBundle:Welcome:index``\ 。它是一个\ *代称*\ ，指的是\ ``Acme\DemoBundle\Controller\WelcomeController``\ 类的\ ``indexAction``\ 方法：

.. code-block:: php

    // src/Acme/DemoBundle/Controller/WelcomeController.php
    namespace Acme\DemoBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class WelcomeController extends Controller
    {
        public function indexAction()
        {
            return $this->render('AcmeDemoBundle:Welcome:index.html.twig');
        }
    }

.. tip::

    你也可以使用完整的类名和方法名，如\ ``Acme\DemoBundle\Controller\WelcomeController::indexAction``\ 来指定\ ``_controller``\ 。但如果你的代码符合框架的惯例，那么代称格式更简短，而且可以提供一些灵活性。

``WelcomeController``\ 类扩展了框架内置的\ ``Controller``\ 类，继承了一些十分有用的方法，如：\ :method:`Symfony\\Bundle\\FrameworkBundle\\Controller\\Controller::render`\ 可以调用并输出模板（\ ``AcmeDemoBundle:Welcome:index.html.twig``\ ）。这个方法的返回值是一个包含了渲染后的HTML内容的“响应”对象。如果有需要的话，这个响应可以在返回给用户之前被修改：

.. code-block:: php

    public function indexAction()
    {
        $response = $this->render('AcmeDemoBundle:Welcome:index.txt.twig');
        $response->headers->set('Content-Type', 'text/plain');

        return $response;
    }

不管你如何写代码，你所编写的控制器的最终目的，都是要返回一个\ ``Response``\ 对象给发出请求的用户。\ ``Response``\ 对象可以是HTML代码，也可以是一个客户端的跳转命令，甚至可以是一张头信息里\ ``Content-Type``\ 的值为\ ``image/jpg``\ 的jpg图片。

.. tip::

    并不是必须要扩展\ ``Controller``\ 基类。实际上，控制器可以是一个PHP函数或者一个PHP闭包。教程里的“\ :doc:`控制器</book/controller>`\ "一章，会对控制器进行更全面的说明。

模板的名称\ ``AcmeDemoBundle:Welcome:index.html.twig``\ 也是一个“\ *代称*\ ”。其指向的是\ ``AcmeDemoBundle``\ （位于\ ``src/Acme/DemoBundle``\ ）里的\ ``Resources/views/Welcome/index.html.twig``\ 文件。

现在，让我们再看路由配置，找到\ ``_demo``\ 键：

.. code-block:: yaml

    # app/config/routing_dev.yml
    _demo:
        resource: "@AcmeDemoBundle/Controller/DemoController.php"
        type:     annotation
        prefix:   /demo

Symfony2可以以YAML、XML、PHP甚至PHP注解的格式载入和输出路由的配置信息。例子里的resource是一个\ *代称*\ ：\ ``@AcmeDemoBundle/Controller/DemoController.php``\ ，表示的是\ ``src/Acme/DemoBundle/Controller/DemoController.php``\ 。在控制器PHP文件里，具体的路由配置以注解的方式标注在相应的Action方法上：

.. code-block:: php

    // src/Acme/DemoBundle/Controller/DemoController.php
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

    class DemoController extends Controller
    {
        /**
         * @Route("/hello/{name}", name="_demo_hello")
         * @Template()
         */
        public function helloAction($name)
        {
            return array('name' => $name);
        }

        // ...
    }

``@Route()``\ 注解定义的是符合\ ``/hello/{name}``\ 规则的请求将执行\ ``helloAction``\ 方法。用括号括起来的字符串，如\ ``{name}``\ 称作“占位符”。占位符的值可以通过Action方法的参数来获取。

.. note::

    就算PHP不支持注解，你也可以在Symfony2项目里大量使用，因为这可以使配置信息挨着与之对应的代码，便于管理。

如果你仔细阅读控制器的代码，可以发现，与前例不同，返回值并不是包含经渲染的HTML的\ ``Response``\ 对象，而是直接返回了一个数组。\ ``@Template``\ 注解可以让Symfony框架按照惯例来输出，模板的名称参照控制器的名称。所以，在这个例子里，实际渲染的是\ ``AcmeDemoBundle:Demo:hello.html.twig``\ 文件（位于\ ``src/Acme/DemoBundle/Resources/views/Demo/hello.html.twig``\ ）。

.. tip::

    ``@Route()``\ 和\ ``@Template()`` 注解的功能远比这个简单例子里的要强大，要了解更多内容可以参考：“\ `控制器的注解`_\ ”。

模板
~~~~

例子里控制器渲染的是\ ``src/Acme/DemoBundle/Resources/views/Demo/hello.html.twig``\ 模板，代称是\ ``AcmeDemoBundle:Demo:hello.html.twig``\ ：

.. code-block:: jinja

    {# src/Acme/DemoBundle/Resources/views/Demo/hello.html.twig #}
    {% extends "AcmeDemoBundle::layout.html.twig" %}

    {% block title "Hello " ~ name %}

    {% block content %}
        <h1>Hello {{ name }}!</h1>
    {% endblock %}

Symfony2默认使用\ `Twig`_\ 作为模板引擎，但你也可以选择使用纯PHP模板。之后的章节会介绍Symfony2里模板的工作原理。

代码包
~~~~~~

你可能已经注意到，\ :term:`bundle`\ 这个词的使用频率很高。你在Symfony2框架里编写的代码，都需要按照代码包的形式来组织。根据Symfony2里的定义，一个代码包是指用来实现一个独立功能的符合一定目录结构的一系列文件（包括PHP文件、样式表文件，JavaScript文件，图片，等等）。代码包使得代码的复用变得更容易。前面例子里提到的代码都是来自\ ```AcmeDemoBundle``\ 。

.. _quick-tour-big-picture-environments:

设置运行环境
------------

至此，你应该对Symfony2的工作原理有了一定的了解了。Symfony2输出的页面底部有一个小横条，被称作“Web调试工具条”，它对开发人员很有用处。

.. image:: /images/quick_tour/web_debug_toolbar.png
   :align: center

但你现在看到的只是冰山一角，点击那个看起来有点奇怪的16进制数字串，你可以访问到另一个十分有用的Symfony2调试工具：分析器（profiler）。

.. image:: /images/quick_tour/profiler.png
   :align: center

当然，你不会希望你的代码在发布以后，这些组件仍然能被用户看到。所以，你可以看到在\ ``web/``\ 文件夹里，还有另一个针对生产环境进行了参数优化的入口文件：\ ``app.php``\ 。

.. code-block:: text

    http://localhost/Symfony/web/app.php/demo/hello/Fabien

如果你的Web服务器支持URL重写，（如Apache的\ ``mod_rewrite``\ ），你可以将\ ``app.php``\ 从URL里去掉：

.. code-block:: text

    http://localhost/Symfony/web/demo/hello/Fabien

你还可以将Web服务器上站点的根目录指向\ ``web/``，从而获得更好的安全性，URL也更美观。

.. code-block:: text

    http://localhost/demo/hello/Fabien

.. note::

    上面的三个URL都是\ **示例**\ ，其路由规则都定义在“\ *app/config/routing_dev.yml*\ ”文件里。而标准版的默认配置里，demo代码（\ *AcmeDemoBundle*\ ）并没有在生产环境中启用，所以你会遇到404错误。

为了使你的应用程序的有更快的响应，Symfony2在\ ``app/cache/``\ 文件夹里管理着大量的缓存。与生产环境（\ ``app.php``\ ）不同，在开发环境中，你对代码作出任何的修改，都会重建所有的缓存。这个设置是为了方便调试，毕竟每改一遍代码就要手动清除缓存会令人非常恼火，对吧？

不同的\ :term:`运行环境<environment>`\ 只是配置的不同。实际上，配置也可以继承。

.. code-block:: yaml

    # app/config/config_dev.yml
    imports:
        - { resource: config.yml }

    web_profiler:
        toolbar: true
        intercept_redirects: false

开发环境（\ ``dev``\ ）加载\ ``config_dev.yml``\ 文件，包含了全局的\ ``config.yml``\ ，并做了一些针对性的修改：如在这个例子里的，启用了Web调试工具条。

总结
----

恭喜你！你已经对Symfony2有一个初步的印象了。还不赖，是么？依然有很多的信息需要你去了解，但你应该已经知道Symfony2能让你更轻松地创建更好的Web应用。如果你打算一鼓作气，请继续阅读下一篇吧：“\ :doc:`视图<the_view>`\ ”。

.. _Symfony2标准版:     http://symfony.com/download
.. _5分钟了解Symfony:   http://symfony.com/symfony-in-five-minutes
.. _任务的抽象和分工:   http://en.wikipedia.org/wiki/Separation_of_concerns
.. _YAML:               http://www.yaml.org/
.. _控制器的注解:       http://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/index.html#annotations-for-controllers
.. _Twig:               http://twig.sensiolabs.org/
