.. index::
   single: Tests

测试
====

程序员每写一行代码，都有可能制造了新的bug。所以，为了使你的程序更高效和可靠，你应该对你的代码进行功能和单元层面的测试。

PHPUnit测试框架
---------------

Symfony2整合了PHPUnit，但本章并不会介绍PHPUnit的细节，因为PHPUnit项目本身就有高质量的 `documentation`_ 供你参考。

.. note::

    Symfony2支持3.5.11以上版本的PHPUnit。

Symfony框架下，每个测试用例（不管是单元测试还是功能测试），都以PHP类的形式存在于 `Test/` 目录里。如果你写的测试代码遵循了这个规则，就可以通过下面的命令来测试你的Symfony应用程序：

.. code-block:: bash

    # 指定配置文件所在位置
    $ phpunit -c app/

``-c`` 选项指示PHPUnit在 ``app/`` 目录里查找配置文件。如果你想要了解PHPUnit还有哪些运行选项，请阅读： ``app/phpunit.xml.dist`` 。

.. tip::

    如果指定了 ``--coverage-html`` 选项，PHPUnit会生成测试的覆盖率。

.. index::
   single: Tests; Unit Tests

单元测试
--------

单元测试是针对某个PHP类编写的测试。如果你想要测试你应用程序的整体运行效果，你需要编写 `功能测试`_ 。

在Symfony2应用程序里编写单元测试，和写一个普通的PHP类没有区别。假设你在 ``Utility`` 目录里有一个简单的类 ``Calculator`` ：

.. code-block:: php

    // src/Acme/DemoBundle/Utility/Calculator.php
    namespace Acme\DemoBundle\Utility;
    
    class Calculator
    {
        public function add($a, $b)
        {
            return $a + $b;
        }
    }

要测试这个类，在 ``Tests/Utility`` 目录下创建包含下面代码的 ``CalculatorTest`` 文件：

.. code-block:: php

    // src/Acme/DemoBundle/Tests/Utility/CalculatorTest.php
    namespace Acme\DemoBundle\Tests\Utility;

    use Acme\DemoBundle\Utility\Calculator;

    class CalculatorTest extends \PHPUnit_Framework_TestCase
    {
        public function testAdd()
        {
            $calc = new Calculator();
            $result = $calc->add(30, 12);

            // 检查加法运算的正确性
            $this->assertEquals(42, $result);
        }
    }

.. note::

    ``Tests/`` 目录下的子目录结构，应该与被测试的代码包有对应关系。所以，如果你要测试的类位于 ``Utility/`` 目录，你的单元测试代码就应该保存在 ``Tests/Utility/`` 目录里。

和你实际的应用程序一样，类的自动加载通过 ``bootstrap.php.cache`` 文件来实现（由 ``phpunit.xml.dist`` 文件里的配置启用）。

你可以只对某个PHP文件或者目录进行测试：

.. code-block:: bash

    # 执行Utility目录下的测试
    $ phpunit -c app src/Acme/DemoBundle/Tests/Utility/

    # 测试Calculator类
    $ phpunit -c app src/Acme/DemoBundle/Tests/Utility/CalculatorTest.php

    # 测试整个Bundle
    $ phpunit -c app src/Acme/DemoBundle/

.. index::
   single: Tests; Functional Tests

功能测试
--------

功能测试会检查应用程序执行的各个环节（从URL路由到视图）。功能测试的细节逻辑和单元测试并没有区别，但会执行特定的Web测试流程：

* 模拟HTTP请求
* 检查返回结果
* 模拟链接的访问，以及表单的提交
* 检查返回结果
* 重复以上步骤

编写第一个功能测试
~~~~~~~~~~~~~~~~~~

功能测试文件应保存在 ``Tests/Controller`` 目录。如果你想要测试由 ``DemoController`` 生成的页面，你首先需要创建一个继承了 ``WebTestCase`` 的 ``DemoControllerTest.php`` 文件。

Symfony2标准版里 ``DemoController`` 有一个简单的功能测试文件（ `DemoControllerTest`_ ），你可以参考：

.. code-block:: php

    // src/Acme/DemoBundle/Tests/Controller/DemoControllerTest.php
    namespace Acme\DemoBundle\Tests\Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class DemoControllerTest extends WebTestCase
    {
        public function testIndex()
        {
            $client = static::createClient();

            $crawler = $client->request('GET', '/demo/hello/Fabien');

            $this->assertGreaterThan(0, $crawler->filter('html:contains("Hello Fabien")')->count());
        }
    }

.. tip::

    要执行单元测试， ``WebTestCase`` 类会加载应用程序的框架核心文件，创建运行实例。大多数情形下，这些操作都是自动完成的，如果你的核心文件不是位于默认的目录，你需要修改 ``phpunit.xml.dist`` 文件里的 ``KERNEL_DIR`` 变量：

.. code-block:: xml

    <phpunit>
        <!-- ... -->
        <php>
            <server name="KERNEL_DIR" value="/path/to/your/app/" />
        </php>
        <!-- ... -->
    </phpunit>

``createClient()`` 方法返回的是一个HTTP客户端对象，可以模拟用户浏览器的行为：

.. code-block:: php

    $crawler = $client->request('GET', '/demo/hello/Fabien');

``request()`` 方法（参考： :ref:`book-testing-request-method-sidebar` ）能返回一个 :class:`Symfony\\Component\\DomCrawler\\Crawler` 对象，这个对象包含了页面响应里一些可以交互的元素，如链接和表单等。

.. tip::

    Crawler类只支持XML或HTML格式的内容，要获得响应结果的源代码，可以调用 ``$client->getResponse()->getContent()`` 。

你可以通过XPath表达式或者CSS选择器来选中Crawler对象里包含的链接，然后由Client来发起一个访问。下面的代码就选中了包含 ``Greet`` 文本的第二个链接，并模拟了用户的“点击”：

.. code-block:: php

    $link = $crawler->filter('a:contains("Greet")')->eq(1)->link();

    $crawler = $client->click($link);

表单提交的测试方法也很简单：先选中一个表单按钮，按需要对表单元素进行赋值，然后执行表单的提交：

.. code-block:: php

    $form = $crawler->selectButton('submit')->form();

    // 设置表单值
    $form['name'] = '张三';
    $form['form_name[subject]'] = '你好！';

    // 提交表单
    $crawler = $client->submit($form);

.. tip::

    还可以测试文件上传，以及其他输入形式的表单项（如下拉菜单的 ``select()`` ，以及选择框的 ``tick()`` ）。阅读 `表单`_ 一节以了解更多细节。

你可以用很简单的代码来遍历Web应用，并通过断言语句（assertion）来判断应用是否按照设计的意图运行。比如，使用Crawler来检查页面的DOM：

.. code-block:: php

    // 检查页面中是否存在h1标签
    $this->assertGreaterThan(0, $crawler->filter('h1')->count());

如果你想检查返回结果里是否包含了指定的文字内容，或者返回结果不是XML/HTML格式，你也可以直接检查响应的源代码：

.. code-block:: php

    $this->assertRegExp('/Hello Fabien/', $client->getResponse()->getContent());

.. _book-testing-request-method-sidebar:

.. sidebar:: ``request()`` 的更多用法：

    ``request()`` 方法的参数列表：

        request(
            $method,
            $uri, 
            array $parameters = array(), 
            array $files = array(), 
            array $server = array(), 
            $content = null, 
            $changeHistory = true
        )

    你可以通过 ``server`` 数组来设置 `$_SERVER`_ 里的全局变量。比如，如果你要指定 `Content-Type` 和 `Referer` HTTP头信息，你可以写：

        $client->request(
            'GET',
            '/demo/hello/Fabien',
            array(),
            array(),
            array(
                'CONTENT_TYPE' => 'application/json',
                'HTTP_REFERER' => '/foo/bar',
            )
        );



.. index::
   single: Tests; Assertions

.. sidebar:: 常用的断言语句

    为了方便你上手，列举一些在测试用例里常用的断言语句：

    .. code-block:: php

        // 检查页面里是否存在包含名为“subtitle”的CSS类的h2标签
        $this->assertGreaterThan(0, $crawler->filter('h2.subtitle')->count());

        // 检查页面上是否正好有4个h2标签
        $this->assertCount(4, $crawler->filter('h2'));

        // 检查“Content-Type”头信息的值是“application/json”
        $this->assertTrue($client->getResponse()->headers->contains('Content-Type', 'application/json'));

        // 检查响应的内容，符合一个正则表达式
        $this->assertRegExp('/foo/', $client->getResponse()->getContent());

        // 检查响应的状态码是否是“成功”（20X）
        $this->assertTrue($client->getResponse()->isSuccessful());
        // 检查响应的状态码是否是404
        $this->assertTrue($client->getResponse()->isNotFound());
        // 检查响应的状态码是否是200
        $this->assertEquals(200, $client->getResponse()->getStatusCode());

        // 检查响应结果是一个指向 /demo/contact 的URL跳转
        $this->assertTrue($client->getResponse()->isRedirect('/demo/contact'));
        // 或者仅仅检查是否是跳转
        $this->assertTrue($client->getResponse()->isRedirect());

.. index::
   single: Tests; Client

HTTP测试
--------

Symfony2测试框架可以模拟浏览器发出的HTTP请求：

.. code-block:: php

    $crawler = $client->request('GET', '/hello/Fabien');

例子里， ``request()`` 方法接收了两个参数，HTTP方法和URL，并返回一个 ``Crawler`` 实例。

Crawler可以用来查找DOM元素。接下来，就可以模拟“点击”某个链接，或者“提交”表单了：

.. code-block:: php

    $link = $crawler->selectLink('Go elsewhere...')->link();
    $crawler = $client->click($link);

    $form = $crawler->selectButton('validate')->form();
    $crawler = $client->submit($form, array('name' => 'Fabien'));

``click()`` 和 ``submit()`` 方法都返回 ``Crawler`` 对象。你应该已经可以理解，使用由测试框架提供的方法来遍历你的Web应用程序，可以简化你的测试工作（如果你想要做自动测试的话），如检测HTTP方法和文件的上传。

.. tip::

    ``Link`` 和 ``Form`` 对象在 :ref:`Crawler<book-testing-crawler>` 一节里有更详细的介绍。

``request`` 方法还可以用来模拟表单提交，或者处理其他更复杂的情形：

.. code-block:: php

    // 提交一个表单（但使用Crawler的方式更简单）
    $client->request('POST', '/submit', array('name' => 'Fabien'));

    // 通过表单上传文件
    use Symfony\Component\HttpFoundation\File\UploadedFile;

    $photo = new UploadedFile(
        '/path/to/photo.jpg',
        'photo.jpg',
        'image/jpeg',
        123
    );
    // 或者
    $photo = array(
        'tmp_name' => '/path/to/photo.jpg',
        'name' => 'photo.jpg',
        'type' => 'image/jpeg',
        'size' => 123,
        'error' => UPLOAD_ERR_OK
    );
    $client->request(
        'POST',
        '/submit',
        array('name' => 'Fabien'),
        array('photo' => $photo)
    );

    // 测试DELETE请求，提交指定的HTTP头信息
    $client->request(
        'DELETE',
        '/post/12',
        array(),
        array(),
        array('PHP_AUTH_USER' => 'username', 'PHP_AUTH_PW' => 'pa$$word')
    );

你还可以让每个测试请求都在独立的PHP进程里运行，从而避免多个同时运行多个Client可能带来的干扰：

.. code-block:: php

    $client->insulate();

测试访问
~~~~~~~~

“Client”对象还支持用户的很多浏览器操作：

.. code-block:: php

    $client->back();
    $client->forward();
    $client->reload();

    // 清除cookie，重新测试
    $client->restart();

访问内部变量
~~~~~~~~~~~~

使用Client测试你的应用程序时，你可能会需要访问Client内部的变量：

.. code-block:: php

    $history   = $client->getHistory();
    $cookieJar = $client->getCookieJar();

或者获取与最近一个请求相关的对象：

.. code-block:: php

    $request  = $client->getRequest();
    $response = $client->getResponse();
    $crawler  = $client->getCrawler();

如果测试请求不是封闭的，你还可以访问 ``Container`` 和 ``Kernel`` 对象：

.. code-block:: php

    $container = $client->getContainer();
    $kernel    = $client->getKernel();

访问Container
~~~~~~~~~~~~~

功能测试应该只针对响应进行测试。但有些情况下，你编写的断言语句可能需要访问一些内部的对象，你可以按照下面的代码来获取依赖注入的容器对象：

.. code-block:: php

    $container = $client->getContainer();

需要注意的是，如果你的请求是封闭的，或者你的操作是HTTP协议层面的，你将不能访问容器对象。要知道哪些服务在你当前的应用中可见，你可以使用 ``container:debug`` 命令行脚本。

.. tip::

    请确认你要查看的信息是否在profiler里已经存在。

访问Profiler的信息
~~~~~~~~~~~~~~~~~~

处理每一个请求的时候，Symfony的Profiler都会收集很多关于处理细节的信息。比如，你可以使用profiler来检查在处理某个请求时，数据库查询的次数是否低于某一个限定值。

获得最近一次请求的Profiler的代码如下：

.. code-block:: php

    $profile = $client->getProfile();

更多关于如何在测试中使用profiler的内容，请参考： :doc:`/cookbook/testing/profiling` 。

重定向
~~~~~~

当一个请求的响应是一个URL重定向，测试Client并不会自动跟进这个跳转。你可以先检查响应，然后调用 ``followRedirect()`` 强制Client执行重定向：

.. code-block:: php

    $crawler = $client->followRedirect();
    
如果你希望Client能够自动的跟进所有的跳转，你可以调用 ``followRedirects()`` 方法：

    $client->followRedirects();

.. index::
   single: Tests; Crawler

.. _book-testing-crawler:

Crawler
-------

Client对象的request方法会返回Crawler对象，这个对象使你可以遍历响应所对应的HTML文档，选择HTML标签节点，定位链接和表单。

遍历查找
~~~~~~~~

和jQuery类似，Crawler提供了一组方法，使你可以遍历HTML/XML文档的DOM树来进行查找。下面的代码例子，先找出了所有的 ``input[type=submit]`` 元素，然后选中其中的最后一个，最后选中其第一个父节点：

.. code-block:: php

    $newCrawler = $crawler->filter('input[type=submit]')
        ->last()
        ->parents()
        ->first()
    ;

还有很多其他的方法：

+------------------------+----------------------------------------------------+
| 方法名                 | 描述                                               |
+========================+====================================================+
| ``filter('h1.title')`` | 符合CSS选择器的节点                                |
+------------------------+----------------------------------------------------+
| ``filterXpath('h1')``  | 符合XPath规则的节点                                |
+------------------------+----------------------------------------------------+
| ``eq(1)``              | 指定坐标的节点                                     |
+------------------------+----------------------------------------------------+
| ``first()``            | 一组节点中的第一个                                 |
+------------------------+----------------------------------------------------+
| ``last()``             | 最末一个节点                                       |
+------------------------+----------------------------------------------------+
| ``siblings()``         | 兄弟（同级）节点                                   |
+------------------------+----------------------------------------------------+
| ``nextAll()``          | 向后遍历所有的兄弟节点                             |
+------------------------+----------------------------------------------------+
| ``previousAll()``      | 向前遍历所有的兄弟节点                             |
+------------------------+----------------------------------------------------+
| ``parents()``          | 获得所有的父节点                                   |
+------------------------+----------------------------------------------------+
| ``children()``         | 获得所有的子节点                                   |
+------------------------+----------------------------------------------------+
| ``reduce($lambda)``    | 返回符合过滤函数的节点                             |
+------------------------+----------------------------------------------------+

由于以上方法返回的都是 ``Crawler`` 实例，所以你可以链式调用来对功能进行组合：

.. code-block:: php

    $crawler
        ->filter('h1')
        ->reduce(function ($node, $i)
        {
            if (!$node->getAttribute('class')) {
                return false;
            }
        })
        ->first();

.. tip::

    ``count()`` 方法可以返回Crawler里结果集所包含的节点个数： ``count($crawler)``

其他用法
~~~~~~~~

Crawler还可以用来获取与节点有关的信息：

.. code-block:: php

    // 获得第一个节点的class属性值
    $crawler->attr('class');

    // 获得第一个节点的文本值
    $crawler->text();

    // 以数组形式获得所有节点的指定属性值（_text用来返回节点的文本值）
    // 例：获得Crawler里包含的所有节点的文本值和href。
    $info = $crawler->extract(array('_text', 'href'));

    // 通过一个回调函数获得所有节点的href属性值
    $data = $crawler->each(function ($node, $i)
    {
        return $node->attr('href');
    });

链接
~~~~

你可以使用前述的遍历查找方法来选中链接（链接是节点类型），或者使用工具方法： ``selectLink()`` 。

.. code-block:: php

    $crawler->selectLink('点我');

这个调用将选中所有包含指定文本的文本链接，或者替代文本（alt值）包含指定文本的图片链接。与其他所有的过滤方法类似，这个方法将返回一个 ``Crawler`` 对象。

当你成功选中了一个链接，你就可以访问与之对应的 ``Link`` 对象，这个对象包含了一些十分有用的方法，诸如 ``getMethod()`` 和 ``getUri()`` 。要“点击”这个链接，你可以调用Client对象的 ``click()`` 方法，并传入 ``Link`` 对象作为参数：

.. code-block:: php

    $link = $crawler->selectLink('Click here')->link();

    $client->click($link);

表单
~~~~

与链接类似的，你可以通过 ``selectButton()`` 方法来选中表单的提交按钮：

.. code-block:: php

    $buttonCrawlerNode = $crawler->selectButton('submit');

.. note::

    需要注意的是，这里我们选中的是某一个提交按钮，而不是具体的表单。因为一个表单可能包含多个提交按钮。如果你使用遍历查找API，请注意要针对按钮来编写规则。

``selectButton()`` 可以选中 ``button`` 标签和类型为 ``input`` 的提交按钮。查找的规则包括：

* ``value`` 属性的值

* ``id`` 或 ``alt`` 属性值

* ``button`` 标签的 ``id`` 或 ``name`` 属性值

如果你的Crawler对象已经选中了一个按钮，你可以通过 ``form()`` 方法来获得包含此按钮的 ``Form`` 对象：

.. code-block:: php

    $form = $buttonCrawlerNode->form();

``form()`` 方法允许你传入一组值来覆盖表单项的默认值：

.. code-block:: php

    $form = $buttonCrawlerNode->form(array(
        'name'              => 'Fabien',
        'my_form[subject]'  => 'Symfony rocks!',
    ));

该方法的第二个参数可以用来指定表单提交时使用的HTTP方法：

.. code-block:: php

    $form = $buttonCrawlerNode->form(array(), 'DELETE');

Client对象可以“提交” ``Form`` 实例：

.. code-block:: php

    $client->submit($form);

表单项的值也可以在 ``submit()`` 方法的第二个参数里以数组形式传入：

.. code-block:: php

    $client->submit($form, array(
        'name'              => 'Fabien',
        'my_form[subject]'  => 'Symfony rocks!',
    ));

对于更复杂的情形，你可以直接按照数组形式来操作 ``Form`` 实例来设定表单项的值：

.. code-block:: php

    // 改变表单项的值
    $form['name'] = 'Fabien';
    $form['my_form[subject]'] = 'Symfony rocks!';

另外还有一组API方法可以很方便地提供与表单项类型对应的操作：

.. code-block:: php

    // 选中下拉菜单项或者一个选择框
    $form['country']->select('France');

    // 选中复选框
    $form['like_symfony']->tick();

    // 上传文件
    $form['photo']->upload('/path/to/lucas.jpg');

.. tip::

    ``getValues()`` 方法可以用来获得 ``Form`` 对象所包含的所有表单项的值。待上传的文件由另一个方法（ ``getFiles()`` ）来获得。 ``getPhpValues()`` 和 ``getPhpFiles()`` 方法的作用类似，不过返回值的格式是PHP变量形式。

.. index::
   pair: Tests; Configuration

测试的参数
----------

单元测试Client创建的是运行在 ``test`` 环境下的Kernel。由于Symfony在 ``test`` 环境下会加载 ``app/config/config_test.yml`` 配置文件，你可以在这个文件里调整参数，以适应你测试的需要。

比如，默认情况下，swiftmailer在 ``test`` 环境下 *不会* 实际发送邮件。你可以在 ``swiftmailer`` 的配置项下找到如下的参数：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config_test.yml
        # ...

        swiftmailer:
            disable_delivery: true

    .. code-block:: xml

        <!-- app/config/config_test.xml -->
        <container>
            <!-- ... -->

            <swiftmailer:config disable-delivery="true" />
        </container>

    .. code-block:: php

        // app/config/config_test.php
        // ...

        $container->loadFromExtension('swiftmailer', array(
            'disable_delivery' => true
        ));

你甚至可以创建另外的测试环境，在调用 ``createClient()`` 方法时传入需要的参数：

.. code-block:: php

    $client = static::createClient(array(
        'environment' => 'my_test_env',
        'debug'       => false,
    ));

如果你的Web应用的行为依赖于某些HTTP头信息，你可以在 ``createClient()`` 的第二个参数里指定：

.. code-block:: php

    $client = static::createClient(array(), array(
        'HTTP_HOST'       => 'en.example.com',
        'HTTP_USER_AGENT' => 'MySuperBrowser/1.0',
    ));

当然，每个测试请求都可以单独指定：

.. code-block:: php

    $client->request('GET', '/', array(), array(), array(
        'HTTP_HOST'       => 'en.example.com',
        'HTTP_USER_AGENT' => 'MySuperBrowser/1.0',
    ));

.. tip::

    测试客户端在 ``test`` 环境里，以一个服务的形式存在，这意味着你可以按照你需要的方式对其进行重载。

.. index::
    pair: PHPUnit; Configuration

PHPUnit的配置
~~~~~~~~~~~~~

每个Web应用程序都有自己独立的PHPUnit配置，保存在 ``phpunit.xml.dist`` 文件里。你可以修改这个文件来改变一些默认值，或者根据你本地环境的需要做必要的修改。

.. tip::

    在代码仓库里保存 ``phpunit.xml.dist`` 文件，并在本地忽略 ``phpunit.xml`` 文件的变更。

默认的配置下，只有保存在“标准的”Symfony代码包里的测试才会被 ``phpunit`` 命令运行（即保存在 ``src/*/Bundle/Tests`` 或者 ``src/*/Bundle/*Bundle/Tests`` 目录里）。但要添加其他的目录很简单，下面的例子即说明了如何包含第三方代码包的测试用例：

.. code-block:: xml

    <!-- hello/phpunit.xml.dist -->
    <testsuites>
        <testsuite name="Project Test Suite">
            <directory>../src/*/*Bundle/Tests</directory>
            <directory>../src/Acme/Bundle/*Bundle/Tests</directory>
        </testsuite>
    </testsuites>

要在测试覆盖里增加其他目录，还需要修改 ``<filter>`` 配置段对应的值：

.. code-block:: xml

    <filter>
        <whitelist>
            <directory>../src</directory>
            <exclude>
                <directory>../src/*/*Bundle/Resources</directory>
                <directory>../src/*/*Bundle/Tests</directory>
                <directory>../src/Acme/Bundle/*Bundle/Resources</directory>
                <directory>../src/Acme/Bundle/*Bundle/Tests</directory>
            </exclude>
        </whitelist>
    </filter>

了解更多实用的技巧
------------------

* :doc:`/cookbook/testing/http_authentication`
* :doc:`/cookbook/testing/insulating_clients`
* :doc:`/cookbook/testing/profiling`


.. _`DemoControllerTest`: https://github.com/symfony/symfony-standard/blob/master/src/Acme/DemoBundle/Tests/Controller/DemoControllerTest.php
.. _`$_SERVER`: http://php.net/manual/en/reserved.variables.server.php
.. _`documentation`: http://www.phpunit.de/manual/3.5/en/
