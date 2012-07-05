视图
====

看来你在阅读完《快速入门》的第一章之后，决定再多花10分钟继续判断Symfony2是否适合你的需要。有眼光！本篇介绍的是Symfony2默认使用的模板引擎——“\ `Twig`_\ ”。Twig是一款功能强大、高性能、并且安全的PHP模板引擎。用Twig书写的模板可读性更好，代码更简洁，自然也更受页面制作人员的欢迎。

.. note::

    你也可以在Symfony2里使用纯\ :doc:`PHP </cookbook/templating/PHP>`\ 的模板。

了解Twig
--------

.. tip::

    如果你想学习Twig，我们强烈建议你阅读《\ `Twig官方文档`_\ 》，本文只是一些重要概念的介绍。

Twig模板可以用来生成任意类型的文本内容，如：（HTML，XML，CSV，LaTex，等等），Twig里定义了两种分隔符：

* ``{{ ... }}``\：输出变量或者表达式结果；

* ``{% ... %}``\：界定条件语句，如\ ``for``\ 循环和\ ``if``\ 语句。

下面是一个简单的例子，用到了两个变量：\ ``page_title``\ 和\ ``navigation``\ 。

.. code-block:: html+jinja

    <!DOCTYPE html>
    <html>
        <head>
            <title>网页标题</title>
        </head>
        <body>
            <h1>{{ page_title }}</h1>

            <ul id="navigation">
                {% for item in navigation %}
                    <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
                {% endfor %}
            </ul>
        </body>
    </html>


.. tip::

   Twig模板里的注释可以写在\ ``{# ... #}``\ 内。

在Symfony2里，可以通过调用Controller类的\ ``render``\ 方法，传入模板文件名和模板里用到的变量来输出最终的HTML，如：

.. code-block:: php

    $this->render('AcmeDemoBundle:Demo:hello.html.twig', array(
        'name' => $name,
    ));

赋值给模板的变量可以是字符串、数组甚至对象。Twig会自动地识别变量类型，你可以通过“\ ``.``\ ”符号来访问复合变量里的属性值，如：

.. code-block:: jinja

    {# array('name' => 'Fabien') #}
    {{ name }}

    {# array('user' => array('name' => 'Fabien')) #}
    {{ user.name }}

    {# force array lookup #}
    {{ user['name'] }}

    {# array('user' => new User('Fabien')) #}
    {{ user.name }}
    {{ user.getName }}

    {# force method name lookup #}
    {{ user.name() }}
    {{ user.getName() }}

    {# pass arguments to a method #}
    {{ user.date('Y-m-d') }}

.. note::

    需要注意的是，花括号仅仅用于需要输出的场合。

嵌套模板
--------

有经验的开发者知道，模板是可以复用的，比如页首、页尾等等。Symfony2框架支持模板的嵌套，这和PHP类的继承十分类似。有了这个机制，你就可以创建包含公共元素的“布局”模板，并定义由子模板继承的“区块”。

下面的\ ``hello.html.twig``\ 模板就通过\ ``extends``\ 语法继承了\ ``layout.html.twig``\ 模板：

.. code-block:: html+jinja

    {# src/Acme/DemoBundle/Resources/views/Demo/hello.html.twig #}
    {% extends "AcmeDemoBundle::layout.html.twig" %}

    {% block title "Hello " ~ name %}

    {% block content %}
        <h1>Hello {{ name }}!</h1>
    {% endblock %}

``AcmeDemoBundle::layout.html.twig``\ 好像在哪里见到过，是么？这是一个标准的模板代称。其中，\ ``::``\ 表示模板文件并没有指定具体的Controller，而是位于\ ``/app/Resources/views/``\ 目录。

让我们先看一个简化版的\ ``layout.html.twig``\ ：

.. code-block:: jinja

    {# src/Acme/DemoBundle/Resources/views/layout.html.twig #}
    <div class="symfony-content">
        {% block content %}
        {% endblock %}
    </div>

``{% block %}``\ 标签定义了可以载入子模板的区域。

在例子里，\ ``hello.html.twig``\ 模板重载了\ ``content``\ 区块，意味着文本：“Hello Fabien”将被输出到\ ``div.symfony-content``\ 内。

使用标签、过滤和函数
--------------------

Twig最突出的特性之一就是可以通过标签、过滤和函数来进行功能扩展。Symfony2内置了很多相关的插件，可以极大简化模板编写的工作。

模板的包含
~~~~~~~~~~

在不同模板之间共享一段相同代码的最佳方式，就是创建一个可以被其他模板包含的共用模板。

比如，创建一个\ ``embedded.html.twig``\ 模板：

.. code-block:: jinja

    {# src/Acme/DemoBundle/Resources/views/Demo/embedded.html.twig #}
    Hello {{ name }}

然后修改\ ``index.html.twig``\ 来包含这个模板：

.. code-block:: jinja

    {# src/Acme/DemoBundle/Resources/views/Demo/hello.html.twig #}
    {% extends "AcmeDemoBundle::layout.html.twig" %}

    {# override the body block from embedded.html.twig #}
    {% block content %}
        {% include "AcmeDemoBundle:Demo:embedded.html.twig" %}
    {% endblock %}

包含控制器的输出
~~~~~~~~~~~~~~~~

那么如何在模板里嵌入另外的控制器（controller）的输出？在开发Ajax应用，或者被包含的模板引用了主模板里并不存在的变量时，这个特性就会变得十分有用。

假设你已经创建了一个\ ``fancy``\ 动作方法（action），打算将其输出包含在\ ``index``\ 模板里，可以通过使用\ ``render``\ 标签来实现：

.. code-block:: jinja

    {# src/Acme/DemoBundle/Resources/views/Demo/index.html.twig #}
    {% render "AcmeDemoBundle:Demo:fancy" with { 'name': name, 'color': 'green' } %}

``AcmeDemoBundle:Demo:fancy``\ 字符串指代的是\ ``Demo``\ 控制器的\ ``fancy``\ 动作方法。\ ``name``\ 和\ ``color``\ 此时就代替了请求参数，用来执行对\ ``fancyAction``\ 的调用。

.. code-block:: php

    // src/Acme/DemoBundle/Controller/DemoController.php
    
    class DemoController extends Controller
    {
        public function fancyAction($name, $color)
        {
            // create some object, based on the $color variable
            $object = ...;

            return $this->render('AcmeDemoBundle:Demo:fancy.html.twig', array(
                'name' => $name,
                'object' => $object
            ));
        }
        
        // ...
    }

创建链接
~~~~~~~~

对于Web应用，总是会用到链接。把URL静态的写在模板里十分不灵活，因为任何的变化都可能需要你费时去查找大量的文件；而\ ``path``\ 函数可以根据路由配置的调整，始终输出有效的URL：

.. code-block:: html+jinja

    <a href="{{ path('_demo_hello', { 'name': 'Thomas' }) }}">Greet Thomas!</a>

``path``\ 函数以路由名称和一组取值作为参数，取值对应的是路由规则里的占位符（可变量）：

.. code-block:: php

    // src/Acme/DemoBundle/Controller/DemoController.php
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

    /**
     * @Route("/hello/{name}", name="_demo_hello")
     * @Template()
     */
    public function helloAction($name)
    {
        return array('name' => $name);
    }

.. tip::

    ``url``\ 函数可以用来生成\ *绝对* 路径的URL：\ ``{{ url('_demo_hello', { 'name': 'Thomas' }) }}``\ 。

引用外部文件：图片，JavaScript和样式表
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

互联网怎么能少得了图片，JavaScripts和样式表？Symfony2提供了\ ``asset``\ 函数来简化对这些资源的引用：

.. code-block:: jinja

    <link href="{{ asset('css/blog.css') }}" rel="stylesheet" type="text/css" />

    <img src="{{ asset('images/logo.png') }}" />

``asset``\ 函数的主要目标，是为了使你的应用拥有更好的可移植性。有了这个函数，你可以任意地改变资源文件相对于Web根目录的位置，而不用去修改模板。

转义变量
--------

Twig默认对所有输出进行转义，你可以阅读《\ `Twig官方文档`_\ 》来了解转义，以及Twig提供了那些与转义有关的功能。

结论
----

Twig很简单，但功能很强大。通过综合运用布局、区块、模板和对动作方法（controller action）的包含，前端开发会变得更轻松。当然，如果你不想使用Twig，你也可以使用纯PHP的模板。

至此，虽然你只花了20分钟来了解Symfony2，但你已经可以用它做到一些让人印象深刻的事了。使Symfony2的这些特性成为现实的，是一整套复杂的架构……

我不应该用“复杂的架构”来吓唬你。按部就班地，你可以在下一篇中先了解\ :doc:`控制器<the_controller>`\ 。

.. _Twig:         http://twig.sensiolabs.org/
.. _Twig官方文档: http://twig.sensiolabs.org/documentation
