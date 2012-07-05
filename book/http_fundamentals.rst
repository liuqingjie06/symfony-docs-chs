.. index::
   single: Symfony2 Fundamentals

Symfony2与HTTP原理
==================

恭喜！通过学习Symfony2，你将能成长为一个\ *高效*\ 、\ *全面*\ 和\ *受欢迎*\ 的Web开发者（当然，要受到用人单位或同行的欢迎，还是得靠你自己）。Symfony2的存在是为了要解决最根本的问题：即提供一个开发工具，使开发者能以自己的方式更快速地开发出更为健壮的应用程序。Symfony2是许多技术实践的优点之集大成，通过本教程，你将要学习的工具和理论，代表了很多人多年来的努力成果。换句话说，你不只是在学习Symfony2，你还将学习Web基础原理、最佳开发实践以及如何使用许多新的、优秀的PHP开发库。所以，请做好准备！

本章将从解释Web开发过程中最常接触的基础概念开始：HTTP协议。无论技术背景或首选编程语言是什么，本章的内容对于所有人来说都是\ *必须要了解*\ 的。

HTTP协议很简单
--------------

HTTP（超文本传输协议）是用来在两个机器之间实现通信的文本语言。没错，定义就这么简单！举例来说，当你要去\ `xkcd`_\ 网站查看最新的漫画时，下列会话（近似地）将在你的浏览器和服务器之间发生：

.. image:: /images/http-xkcd.png
   :align: center

虽然实际使用的语言将更加正规，但它依然是很简单的。HTTP定义了这种简单文本语言的语义和语法。而无论你从事何种的Web开发，你的服务器\ *总是*\ 要理解基于文本的请求，并返回基于文本的响应。

Symfony2正是以处理HTTP请求为基础的。无论你是否意识到了，你确实每天都在使用HTTP协议。随着对Symfony2学习的深入，你将学会如何掌握它。

.. index::
   single: HTTP; Request-response paradigm

步骤1：客户端发出请求
~~~~~~~~~~~~~~~~~~~~~

Web上的每个会话都是从一个\ *请求*\ 开始的，这个请求是由客户端（如：网页浏览器、iPhone应用程序等）创建的一种特殊格式的文本消息，该格式符合HTTP协议规范。客户端将该请求发送到服务端，然后等待服务端响应。

下图体现的是，浏览器与xkcd服务器之间交互的第一阶段（请求）：

.. image:: /images/http-xkcd-request.png
   :align: center

按照HTTP协议，HTTP请求的实际内容会类似于：

.. code-block:: text

    GET / HTTP/1.1
    Host: xkcd.com
    Accept: text/html
    User-Agent: Mozilla/5.0 (Macintosh)

这个简单的消息\ *准确*\ 地描述了客户端所请求的到底是哪个资源。HTTP请求的第一行是最重要的，它包含了两项信息：URI和HTTP方法。

URI（如：\ ``/``\ 、\ ``/contact``\ 等）表明了客户端所请求资源的唯一地址或位置。HTTP方法（如：\ ``GET``\ ）则是向服务器说明你想对资源\ *做什么*\ ，HTTP方法是请求的“\ *动词*\ ”，用以定义你对资源的操作：

+----------+------------------------+
| *GET*    | 从服务器上获取资源     |
+----------+------------------------+
| *POST*   | 在服务器上创建一个资源 |
+----------+------------------------+
| *PUT*    | 更新服务器上的资源     |
+----------+------------------------+
| *DELETE* | 从服务器上删除资源     |
+----------+------------------------+

按照这个规则，你可以发送一条要求删除指定博文的请求，如：

.. code-block:: text

    DELETE /blog/15 HTTP/1.1

.. note::

    实际上，HTTP协议里一共定义了9种HTTP方法，但它们中的大部分并没有得到广泛的使用和支持。实际上，许多现代的浏览器并不支持\ ``PUT``\ 和\ ``DELETE``\ 方法（译者：随着RESTful的Web Service的应用，这个情况正在发生改变。）

第一行(URI和HTTP方法)，与HTTP请求里的其他文本行一起被称为HTTP请求头，头信息还包括：被请求的主机（\ ``Host``\ ）、客户端接受的响应格式（\ `ACCEPT`\ ）、客户端用来发送请求的应用程序（\ ``User-Agent``\ ）等。还有许多其它的HTTP请求头存在，你可以在维基百科的\ `HTTP头字段列表`_\ 中找到它们。

步骤2：服务端返回响应
~~~~~~~~~~~~~~~~~~~~~

服务端得到请求，它就明确地知道客户端需要哪个资源（通过URI）以及希望对该资源进行什么操作（通过HTTP方法）。例如，对于一个GET请求，服务端将准备资源，并在HTTP响应中将其返回给客户端。如，xkcd服务端返回的响应：

.. image:: /images/http-xkcd.png
   :align: center

按照HTTP协议的格式，被返回给客户端的响应如下所示：

.. code-block:: text

    HTTP/1.1 200 OK
    Date: Sat, 02 Apr 2011 21:05:05 GMT
    Server: lighttpd/1.4.19
    Content-Type: text/html

    <html>
      <!-- HTML for the xkcd comic -->
    </html>

HTTP响应包含了客户端所请求的资源（在这个例子里是HTML），以及与响应相关的信息。与请求头类似，第一行也最重要，它给出的是HTTP状态码（在上面的例子里是200）。状态码报告了响应的状态，如，请求是否成功？是否存在错误？不同的状态码表示着成功、错误或者通知客户端需要做其它一些事（如重定向到另一页）。完整的列表可以在维基百科的\ `HTTP状态代码列表`_\ 中找到。

所有这些响应信息，组成了响应头。其中一个重要的HTTP响应头消息被称为\ ``Content-Type``\ 。服务器上的每个资源都可以以不同的格式返回给客户端，如HTML、XML或JSON等；通过在\ ``Content-Type``\ 里设置如\ ``text/html``\ 这样的互联网媒体类型码，可以告知客户端，服务器给出的响应格式是什么。常见的媒体类型可以在维基百科的\ `互联网媒体类型列表`_\ 里找到。

还存在很多其他的HTTP响应头，其中有些可以起到很重要的作用。比如，某些响应头可以用来维护HTTP缓存。

请求、响应和Web开发
~~~~~~~~~~~~~~~~~~~

“请求 <-> 响应”这个会话模式是驱动Web上所有通信的基础。虽然它作用如此重要，但又如此简单、清晰。

最重要的事实是，不管你使用哪种开发语言，构建哪种应用（Web、手机、JSON应用程序接口等），遵循哪种开发理论，应用程序的最终目标\ **总是**\ 一致的：理解每个请求，创建并返回相应的响应。

Symfony2就是来完成这一“使命”的：

.. tip::

    要更了解HTTP协议规范，可以参考\ `HTTP 1.1 RFC`_\ 或者\ `HTTP Bis`_\ （用更直白明了方式的来说明HTTP协议规范）。另外，有一款叫做\ `Live HTTP Headers`_\ 的Firefox浏览器扩展可以用来查看上网过程中请求、响应头的内容（译者：当然，你也可以用Firebug）。

.. index::
   single: Symfony2 Fundamentals; Requests and responses

PHP是如何处理请求和返回响应的
-----------------------------

那么，怎么用PHP来获知“请求”，并创建“响应”呢？PHP对实际的操作进行了封装，你要做的，相对还算简单：

.. code-block:: php

    <?php
    $uri = $_SERVER['REQUEST_URI'];
    $foo = $_GET['foo'];

    header('Content-type: text/html');
    echo 'The URI requested is: '.$uri;
    echo 'The value of the "foo" parameter is: '.$foo;

虽然看上去有点奇怪，但这一小段代码可以从HTTP请求头里提取信息，并依据这些信息创建了响应内容。你并不需要自己写代码来解析HTML消息格式，因为PHP为你准备了一系列的全局变量，如：\ ``$_SERVER``\ 和\ ``$_GET``\ ，它们包含了关于请求的所有信息。你还可以通过调用\ ``header()``\ 函数来指定响应头信息；或者用\ ``echo``\ 一类的语句直接输出响应里的正文内容。PHP会负责把你指定的所有信息组成一个有效的HTTP响应，返回给客户端（注意，中文会经过编码传输）。

.. code-block:: text

    HTTP/1.1 200 OK
    Date: Sat, 03 Apr 2011 02:14:33 GMT
    Server: Apache/2.2.17 (Unix)
    Content-Type: text/html

    The URI requested is: /testing?foo=symfony
    The value of the "foo" parameter is: symfony

Symfony2中的请求和响应
----------------------

通过使用Symfony2提供的两个类，你可以用更简单的方式来处理请求和响应：\ :class:`Symfony\\Component\\HttpFoundation\\Request`\ 类将HTTP请求封装成了一个对象，有了它，你获取请求相关的信息就变得易如反掌了：

.. code-block:: php

    use Symfony\Component\HttpFoundation\Request;

    $request = Request::createFromGlobals();

    // 不包含参数的URI地址（如 /about）
    $request->getPathInfo();

    // 获得GET或POST请求传入的参数
    $request->query->get('foo');
    $request->request->get('bar', 'default value if bar does not exist');

    // 获得$_SERVER里的值
    $request->server->get('HTTP_HOST');

    // 获得一个文件对象
    $request->files->get('foo');

    // 获得Cooki的值
    $request->cookies->get('PHPSESSID');

    // 获得头信息，对应名称全为小写，中划线转换成下划线
    $request->headers->get('host');
    $request->headers->get('content_type');

    $request->getMethod();          // GET, POST, PUT, DELETE, HEAD
    $request->getLanguages();       // 客户端接受的语言

\ ``Request``\ 类还能帮你做其他的事情。比如，\ ``isSecure()``\ 方法可以通过检查\ *三个*\ 不同的值，来确定用户是否以安全的SSL连接方式访问的（即\ ``https``\ ）。

.. sidebar:: ParameterBags 和 Request 类的成员

    如上，\ ``$_GET`` 和 ``$_POST`` 里的值可以通过公有成员\ ``query``\ 和\ ``request``\ 来访问。每一个都是 :class:`Symfony\\Component\\HttpFoundation\\ParameterBag` 类的对象，拥有以下方法：
    :method:`Symfony\\Component\\HttpFoundation\\ParameterBag::get`,
    :method:`Symfony\\Component\\HttpFoundation\\ParameterBag::has`,
    :method:`Symfony\\Component\\HttpFoundation\\ParameterBag::all` 等等。
    事实上，上面例子里提到的公有成员都是ParameterBag的实例。
    
    .. _book-fundamentals-attributes:
    
    Request类还有一个公有的 ``attributes`` 成员，里面包含了PHP框架的一些运行数据。Symfony2框架在这个变量里保存了当前请求所匹配的URL路由，比如\ ``_controller``\ ，\ ``id``\ （如果你使用了 ``{id}`` 通配符），甚至当前路由的名称（\ ``_route``\ ）。\ ``attributes``\ 成员实际就是用来保存和提供与当前请求相关的运行环境信息。
    
Symfony2还提供了一个\ ``Response``\ 类，是对“响应”的简单封装。你可以用面向对象的方式来构建和向客户端返回其所需的响应：

.. code-block:: php

    use Symfony\Component\HttpFoundation\Response;
    $response = new Response();

    $response->setContent('<html><body><h1>Hello world!</h1></body></html>');
    $response->setStatusCode(200);
    $response->headers->set('Content-Type', 'text/html');

    // 输出内容
    $response->send();

就算Symfony2没提供其它工具，你也已经有了可以轻松获取请求相关信息的工具包，以及用来创建响应的面向对象的接口。即使你将要掌握更多的Symfony2的功能，请牢记，你所写应用程序的目标始终是\ *处理请求，并根据你应用程序的逻辑创建相应的响应*\ 。

.. tip::

    \ ``Request``\ 和\ ``Response``\ 类都出自Symfony2中名为\ ``HttpFoundation``\ 的组件。这个组件可以独立使用，还提供了处理会话和文件上传等等其他的功能。

从请求到响应（之间发生了什么？）
--------------------------------

同HTTP协议一样，\ ``Request``\ 和\ ``Response``\ 对象也很简单。应用程序里实现起来最复杂的部分是在两者之间写些什么。换句话说，真正的体力活儿来自于处理请求并创建响应。

你的应用程序可能会做很多事情，诸如发送电子邮件、处理提交表单、向数据库写入数据、渲染HTML页面和确保内容的安全等等，但你如何来管理这一切，同时还保持代码的组织性和可维护性呢？

Symfony2就是用来解决上述问题的，以节约你的精力。

前端控制器
~~~~~~~~~~

传统方式里，网站里的每一“页”都有对应的PHP文件，如：

.. code-block:: text

    index.php
    contact.php
    blog.php

这种方式存在几个问题：不灵活的URL（如果你需要把\ ``blog.php``\ 文件改名为\ ``news.php``\ ，还得保证这里或那里的链接仍然可以正常访问呢？）；另外，在每个文件里都\ *必须*\ 书写代码来包含一些核心文件以确保安全策略、数据库连接和网站外观等等的一致性。

更好的办法是，使用\ :term:`front controller`:\ （前端控制器），以一个PHP文件作为所有请求的入口，例如：

+------------------------+--------------------+
| ``/index.php``         | 运行 ``index.php`` |
+------------------------+--------------------+
| ``/index.php/contact`` | 运行 ``index.php`` |
+------------------------+--------------------+
| ``/index.php/blog``    | 运行 ``index.php`` |
+------------------------+--------------------+

.. tip::

    通过配置使用Apache的\ ``mod_rewrite``\ （其他Web服务器软件里一般都有类似的组件），URL可以被改写成更简洁美观的形式，如：\ ``/``\ ，\ ``/contact``\ 和\ ``/blog``\ 。

如此，所有的请求都能得到统一的处理。相比于用不同的URL来执行不同的PHP文件，前端控制器在每次请求\ *都会*\ 被执行。调用应用程序里各处逻辑的分发动作，也将由框架来处理。这就解决了前面提到的两个问题。几乎所有“现代”的Web应用程序都是这么做的，比如WordPress。

保持代码的组织性
~~~~~~~~~~~~~~~~

前端控制器又是如何知道哪个页面要被渲染，如何正确渲染呢？这需要判断传入的URI，针对性地调用不同的代码。这也不是容易的差事：

.. code-block:: php

    // index.php

    $request = Request::createFromGlobals();
    $path = $request->getPathInfo(); // 获取传入的URI

    if (in_array($path, array('', '/')) {
        $response = new Response('欢迎来首页');
    } elseif ($path == '/contact') {
        $response = new Response('联系我们');
    } else {
        $response = new Response('页面没有找到', 404);
    }
    $response->send();

幸运的是，这\ *正是*\ Symfony2被设计来解决的问题之一。

Symfony2执行流程
~~~~~~~~~~~~~~~~

让Symfony2来处理请求，开发工作就会变得简单很多。Symfony2在每次处理，都会遵循下面的模式：

.. _request-flow-figure:

.. figure:: /images/request-flow.png
   :align: center
   :alt: Symfony2 request flow

   传入的请求经路由，会由具体的控制器函数进行处理，并返回\ ``Response``\ 对象。

你站点里的每一个“页面”将通过URL路由配置文件来指定对特定PHP功能函数的调用。这些函数被称作\ :term:`controller`\ （控制器），他们将从请求中获取信息（也依赖Symfony2框架里提供的其他工具）来创建并返回一个\ ``Response``\ 。因此，\ *你*\ 所写的的逻辑代码都将位于controller方法内。

就这么简单！回顾一下：

* 每个请求都将执行一个controller文件；

* 基于请求和路由配置，URL路由系统将决定哪个PHP函数将被执行；

* 正确的PHP函数被执行，你的业务逻辑代码将创建并返回相应的\ ``Response``\ 对象。

Symfony2处理请求的实例
~~~~~~~~~~~~~~~~~~~~~~

先不考虑太多的细节，举一个请求处理实例。假设你要在Symfony2应用里增加一个\ ``/contact``\ 页面。首先，修改路由配置文件：

.. code-block:: yaml

    contact:
        pattern:  /contact
        defaults: { _controller: AcmeDemoBundle:Main:contact }

.. note::

   这个例子使用 :doc:`YAML</components/yaml>` 来定义路由的配置，你也可以用XML或PHP来写。

当有人访问\ ``/contact``\ 页面，在前面配置的路由将匹配，指定的控制器将执行。你在\ :doc:`routing chapter</book/routing>`\ 里将会了解到，\ ``AcmeDemoBundle:Main:contact``\ 是简写法，指向的是\ ``MainController``\ 类里的\ ``contactAction``\ 方法函数。

.. code-block:: php

    class MainController
    {
        public function contactAction()
        {
            return new Response('<h1>联系我们！</h1>');
        }
    }

这个控制器非常简单，仅仅创建了一个内容为HTML“<h1>联系我们！</h1>”的\ ``Response``\ 。参考\ :doc:`controller chapter</book/controller>`\ 你可以了解到控制器如何渲染模板，从而使你的“表现层”代码可以被写在单独的模板文件里。控制器不需要考虑一些复杂的工作，如：读写数据库，处理由用户提交的数据，发送电子邮件等。

Symfony2让你可以写应用，而不是写工具
------------------------------------

现在你知道任何应用的目的都是处理传入的请求，创建相应的响应。当应用程序的规模逐渐增长，要保持代码的结构和易维护性就变得越来越困难。毕竟，有很多事情是你不得不反复做的：写数据库，渲染和重用模板，处理表单提交，发送电子邮件，验证用户的输入和保证安全性。

好消息是，这些事情都不是发射神舟飞船，并不特殊。Symfony2提供了你构建应用所需的几乎全部工具，所以你可以专心于创造应用，而不是“重新发明轮子”。Symfony2还有一点值得表扬，就是你可以选择是使用整个框架，还是只使用它部分的功能。

.. index::
   single: Symfony2 Components

独立的工具：Symfony2的\ *组件*\
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

那Symfony2到底\ *是什么*\ ？首先，Symfony2是一个由20多个独立的开发库组成的工具集，你可以在任何PHP项目里使用这些代码。这些开发库，被称作\ *Symfony2组件*\ ，功能涵盖了绝大部分的开发需求。举一些例子：

* `HttpFoundation`_ - 包含\ ``Request``\ 和 ``Response``\ 相关的类，以及处理会话和文件上传的类；

* `Routing`_ - 强大、快速的URL路由系统，你可以指定对于特定的URI（如：\ ``/contact``\ ），请求该如何处理（如：执行\ ``contactAction()``\ 方法）；

* `Form`_ - 一个灵活的、全功能的创建表单和处理表单提交的框架；

* `Validator`_ - 创建数据规则，并可以验证数据（不仅是用户提交的数据）是否符合所创规则的系统；

* `ClassLoader`_ - 类的自动加载器，无需在使用PHP类时写\ ``require``\ 来包含对应的源文件；

* `Templating`_ - 一个渲染模板、处理模板继承关系（即模板嵌套）和执行其他通用模板任务的工具包；

* `Security`_ - 一个功能强大的，能处理应用程序内所有安全任务的库；

* `Translation`_ - 一个用来翻译应用程序内字符串的框架。

每一个组件都是独立的，可用于任何PHP项目中，而不管你是否使用了Symfony2框架。它们的每一个既可以在需要时使用，也可以在必要时被替换。

完整的解决方案：Symfony2\ *框架*
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

那么，什么\ *是*\ Symfony2\ *框架*\ 呢？\ *Symfony2框架*\ 是个PHP库，它实现两个功能：

#. 提供经选择的组件（如：Symfony2组件）和第三方库（如：用Swiftmailer发送电子邮件）；

#. 提供合理的配置以及将这一切都粘合起来的“胶水”库。

这个框架的目标是整合许多独立的工具，以期给开发人员一致的体验。甚至就连Symfony2框架本身也是一个Bundle（类似插件），在必要时也可以被重新配置甚至替换掉。

Symfony2为快速开发应用程序提供了强大的工具集，普通用户可以通过Symfony2的发行版（缺省提供了合理的项目架构）迅速上手，对于更高级的用户而言，只有想不到，没有做不到（The sky is the limit.）。

.. _`xkcd`: http://xkcd.com/
.. _`HTTP 1.1 RFC`: http://www.w3.org/Protocols/rfc2616/rfc2616.html
.. _`HTTP Bis`: http://datatracker.ietf.org/wg/httpbis/
.. _`Live HTTP Headers`: https://addons.mozilla.org/en-US/firefox/addon/live-http-headers/
.. _`HTTP状态代码列表`: http://en.wikipedia.org/wiki/List_of_HTTP_status_codes
.. _`HTTP头字段列表`: http://en.wikipedia.org/wiki/List_of_HTTP_header_fields
.. _`互联网媒体类型列表`: http://en.wikipedia.org/wiki/Internet_media_type#List_of_common_media_types
.. _`HttpFoundation`: https://github.com/symfony/HttpFoundation
.. _`Routing`: https://github.com/symfony/Routing
.. _`Form`: https://github.com/symfony/Form
.. _`Validator`: https://github.com/symfony/Validator
.. _`ClassLoader`: https://github.com/symfony/ClassLoader
.. _`Templating`: https://github.com/symfony/Templating
.. _`Security`: https://github.com/symfony/Security
.. _`Translation`: https://github.com/symfony/Translation
