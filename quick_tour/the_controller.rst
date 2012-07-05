控制器
======

阅读到这里，说明你已经对Symfony2有好感了。接下来，进一步了解Symfony2的控制器（controller）。

返回格式
--------

现在的Web应用已经不仅限于输出HTML页面了：还有用于RSS订阅和Web服务的XML，Ajax请求的JSON，等等。支持这些格式的输出，在Symfony2里很容易。你只需要在路由配置里为\ ``_format``\ 设置默认值，如\ ``xml``\ ：

.. code-block:: php

    // src/Acme/DemoBundle/Controller/DemoController.php
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

    /**
     * @Route("/hello/{name}", defaults={"_format"="xml"}, name="_demo_hello")
     * @Template()
     */
    public function helloAction($name)
    {
        return array('name' => $name);
    }

通过指定返回格式（即\ ``_format``\ ），Symfony2可以自动选择正确的模板，按照上面的例子，\ ``hello.xml.twig``\ 将被加载：

.. code-block:: xml+php

    <!-- src/Acme/DemoBundle/Resources/views/Demo/hello.xml.twig -->
    <hello>
        <name>{{ name }}</name>
    </hello>

就这么简单。对于标准的格式（即HTTP、XML、JSON等等），Symfony2会自动为其输出相应的\ ``Content-Type``\ 头信息（浏览器依赖这个信息来识别文本内容的格式）。如果你想要在一个动作方法（action）里支持不同的格式，可以将\ ``_format``\ 作为占位符来添加：

.. code-block:: php

    // src/Acme/DemoBundle/Controller/DemoController.php
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

    /**
     * @Route("/hello/{name}.{_format}", defaults={"_format"="html"}, requirements={"_format"="html|xml|json"}, name="_demo_hello")
     * @Template()
     */
    public function helloAction($name)
    {
        return array('name' => $name);
    }

``/demo/hello/Fabien.xml``\ 或\ ``/demo/hello/Fabien.json``\ 这样的URL可以使控制器分别返回不同的格式。

``requirements``\ 属性定义的是占位符对应变量必须符合的正则表达式。在例子里，如果你请求的是\ ``/demo/hello/Fabien.js``\ ，那么Symfony2框架将会报404错误，因为“js”并不符合\ ``_format``\ 的规则。

跳转与重定向
------------

如果你要做一个HTTP跳转，可以使用 ``redirect()`` 方法：

.. code-block:: php

    return $this->redirect($this->generateUrl('_demo_hello', array('name' => 'Lucas')));

``generateUrl()``\ 与Twig模板里的\ ``path()``\ 函数是同一个。基于路由的名称以及一组路由规则里的参数值，该函数可以输出完整、美观的URL。

你还可以通过调用\ ``forward()``\ 方法来做重定向，Symfony框架将创建一个“子请求”，并获得这个子请求所返回的 ``Response`` 对象：

.. code-block:: php

    $response = $this->forward('AcmeDemoBundle:Hello:fancy', array('name' => $name, 'color' => 'green'));

    // do something with the response or return it directly

获取与请求相关的信息
--------------------

除了路由占位符对应的变量，控制器还可以访问\ ``Reqeust``\ 对象：

.. code-block:: php

    $request = $this->getRequest();

    $request->isXmlHttpRequest(); // 是否是Ajax请求？

    $request->getPreferredLanguage(array('en', 'fr'));

    $request->query->get('page'); // 获取一个 $_GET 参数

    $request->request->get('page'); // 获取一个 $_POST 参数

在Twig模板里，你也可以通过\ ``app.request``\ 变量来访问\ ``Request``\ 对象：

.. code-block:: html+jinja

    {{ app.request.query.get('page') }}

    {{ app.request.parameter('page') }}

在Session里保存变量
-------------------

虽然HTTP协议是无状态的，Symfony2依然提供了一个用来代表“客户端”（可以是一个使用浏览器的访问者，也可以是搜索引擎爬虫程序或Web服务）的Session对象。在请求之间，Symfony2通过在cookie里设置标识来维持一些与访问相关的变量值。

在控制器里，可以很简单地读/写Session变量：

.. code-block:: php

    $session = $this->getRequest()->getSession();

    // 在Session里保存一个变量，可以在其他请求里读取
    $session->set('foo', 'bar');

    // 在另一个请求里
    $foo = $session->get('foo');

    // 设置用户的本地化选项
    $session->setLocale('fr');

你也可以设置一个只在下一次请求里可见的“消息”：

.. code-block:: php

    // 为下一个请求设置一个消息（控制器里）
    $session->setFlash('notice', 'Congratulations, your action succeeded!');

    // 在模板里显示这个消息
    {{ app.session.flash('notice') }}

这个功能可以用来设置需要在下一个页面显示的信息（比如提示操作成功）。

安全机制
--------

Symfony2标准版包含了能满足常规需要的安全设置：

.. code-block:: yaml

    # app/config/security.yml
    security:
        encoders:
            Symfony\Component\Security\Core\User\User: plaintext

        role_hierarchy:
            ROLE_ADMIN:       ROLE_USER
            ROLE_SUPER_ADMIN: [ROLE_USER, ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH]

        providers:
            in_memory:
                users:
                    user:  { password: userpass, roles: [ 'ROLE_USER' ] }
                    admin: { password: adminpass, roles: [ 'ROLE_ADMIN' ] }

        firewalls:
            dev:
                pattern:  ^/(_(profiler|wdt)|css|images|js)/
                security: false

            login:
                pattern:  ^/demo/secured/login$
                security: false

            secured_area:
                pattern:    ^/demo/secured/
                form_login:
                    check_path: /demo/secured/login_check
                    login_path: /demo/secured/login
                logout:
                    path:   /demo/secured/logout
                    target: /demo/


这个配置使得用户需要先登录，才能访问以\ ``/demo/secured/``\ 开头的URL。配置里还定义了两个用户：\ ``user``\ 和\ ``admin``\ 。\ ``admin``\ 用户有一个\ ``ROLE_ADMIN``\ 的身份，这个身份包含了\ ``ROLE_USER``\ （即角色层次结构/\ ``role_hierarchy``\ ）。

.. tip::

    为了方便阅读，例子里的密码都是明文的，但你在实际代码里应该运用hash算法来增强安全性。

由于有“防火墙”（\ ``firewall``\ ）的保护，访问\ ``http://localhost/Symfony/web/app_dev.php/demo/secured/hello``\ 会自跳转到登录页面。

你还可以通过\ ``@Secure``\ 注解为控制器的某个动作增加用户必须具有指定角色的限制：

.. code-block:: php

    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;
    use JMS\SecurityExtraBundle\Annotation\Secure;

    /**
     * @Route("/hello/admin/{name}", name="_demo_secured_hello_admin")
     * @Secure(roles="ROLE_ADMIN")
     * @Template()
     */
    public function helloAdminAction($name)
    {
        return array('name' => $name);
    }

如是，以\ ``user``\ （不具备\ ``ROLE_ADMIN``\ 角色）访问被保护的“hello”页面，然后点击“Hello resource secured”链接，这时的Symfony2框架将返回一个HTTP状态码403，即不允许当前用户访问。

.. note::

    Symfony2的安全组件可扩展性很好，也包含了很多不同的用户整合方式（如Doctrine ORM）和验证方式（如HTTP Basic、HTTP Digest或X509证书等等）。你可以阅读《\ :doc:`/book/security`\ 》来了解如何配置和使用这些功能

缓存
----

当你的网站流量越来越高，就应该避免反复地生成相同的内容。Symfony2可以使用HTTP缓存头信息来管理页面缓存。最简单的用法是使用\ ``@Cache()``\ 注解：

.. code-block:: php

    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Cache;

    /**
     * @Route("/hello/{name}", name="_demo_hello")
     * @Template()
     * @Cache(maxage="86400")
     */
    public function helloAction($name)
    {
        return array('name' => $name);
    }

例子里，\ ``helloAction``\ 方法返回的HTML页面将提示浏览器可以将其缓存一天。你也可以使用验证，或者配合使用验证和过期时间来进行更细粒度的控制。

Symfony2以内置代理的方式来实现HTTP缓存的管理，所以你可以非常容易地替换成更有效的Varnish或者Squid，使你的网站获得更高的性能。

.. note::

    如果页面不能被整体缓存怎么办（即各部分内容的刷新时间不一致）？Symfony2通过支持Edge Side Includes（ESI）来解决这个问题，你可以阅读《\ :doc:`/book/http_cache`\ 》来了解细节。

总结
----

到目前为止，我们了解的都是核心的框架代码包（framework bundle），正因为代码包的机制，你可以自由地扩展甚至替换代码，获得你想要的功能：《\ :doc:`the_architecture`\ 》