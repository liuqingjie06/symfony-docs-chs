.. index::
   single: Tests

性能优化
========

得益于优良的架构设计，以及对缓存机制的充分运用，默认配置下的Symfony2就已经能提供足够好的性能，但如果仍想要进一步提升性能，则可以参考本文介绍的几个优化方法。

.. index::
   single: Performance; Byte code cache

安装中间代码缓存组件
--------------------

中间代码（intermediate code 或 opcode）缓存组件，被开发人员亲切地称作“加速器”，其安装与使用对PHP执行效率的提升效果最明显，实施起来也较容易。\ `加速器`_\ 中，又以\ `APC`_\ 的使用最为广泛，其共同的工作原理是，在处理PHP请求时，将中间代码缓存起来，从而可以省去解释器反复编译相同代码的开销。

Symfony2针对安装了加速器的运行环境做了专门的设计和优化，有着出色的性能表现。

更进一步
~~~~~~~~

通常情况下，加速器组件会扫描PHP源文件的变动，从而保证中间代码的缓存能够自动更新。在开发、调试过程中，这是一项十分有用的功能，但如果你的代码较为稳定（比如是在生产环境中运行），对文件变动的检查也就成了不必要的开销。

所以，有些加速器组件提供了关闭文件变动检查的选项。如果关闭自动检查，源代码的变动将不会自动生效，需要由系统管理员重启PHP服务来清空并重建缓存。

以APC为例，在php.ini里关闭自动检查的配置是：\ ``apc.stat=0``\ 。

.. index::
   single: Performance; Autoloader

使用带缓存的类自动加载器
------------------------

Symfony2在\ `autoloader.php`_\ （位于/app文件夹内）里默认使用\ ``UniversalClassLoader``\ 作为类加载器。这个加载器很方便，因为它被设计成可以在代码目录里发现和自动加载新添的类文件。

但是，这个便利性会牺牲一定的性能，因为\ ``UniversalClassLoader``\ 必须遍历所有配置了类自动加载的命名空间所对应的文件目录，并调用\ ``file_exists``\ 方法根据类名来确认PHP源文件是否存在。

如果可以把类所在的PHP源文件的路径缓存起来，就可以提高框架的运行效率。Symfony2提供了一个\ ``UniversalClassLoader``\ 的扩展类\ ``ApcUniversalClassLoader``\ ，其使用APC来缓存类文件的路径。

要使用这个加载器，只需要对\ ``autoloader.php``\ 文件做如下修改：

.. code-block:: php

    // app/autoload.php
    require __DIR__.'/../vendor/symfony/src/Symfony/Component/ClassLoader/ApcUniversalClassLoader.php';

    use Symfony\Component\ClassLoader\ApcUniversalClassLoader;

    $loader = new ApcUniversalClassLoader('some caching unique prefix');
    // ...

.. note::

    如果使用\ ``ApcUniversalClassLoader``\ ，Symfony2框架依然可以自动发现并实现对新增PHP类的自动加载。但是，如果你改变了某个命名空间下源文件的路径，或者修改了命名空间的前缀，你就必须手动清空APC缓存，否则，类加载器仍然会去旧的文件位置查找代码。

.. index::
   single: Performance; Bootstrap files

使用预初始化文件
----------------

为了达到最大程度的灵活性和复用性，Symfony2引用了大量的第三方代码，如果在每次处理PHP请求时都要重新加载这些文件会带来一定的开销。针对这个问题，Symfony2的标准版本（Standard Edition）提供了用于生成预初始化文件（\ `bootstrap file`_\ ）的脚本，可以将分散在多个源文件里的类，合并到这个单一的PHP源文件里。通过调用这个预初始化文件，Symfony2框架就不再需要逐个去加载包含各个类的源文件，从而减少对磁盘IO的使用。

如果你使用的是标准版本的Symfony2，那可能你应该已经在享受预初始化文件（bootstrap文件）带来的好处了。如何进行确认？你只需要打开你的入口控制器文件（一般情况下是\ ``app.php``\ ），确认包含以下代码就可以了：

.. code-block:: php

    require_once __DIR__.'/../app/bootstrap.php.cache';

需要注意的是，使用预初始化文件会导致以下两个问题：

* 在源代码发生改变时，预初始化文件必须要重新生成；

* 在调试代码时，开发人员需要在预初始化文件里加断点，因为它才是实际被加载的文件。

如果你使用Symfony2的标准版本，预初始化文件会在使用\ ``php bin/vendors install``\ 脚本命令来安装或者更新第三方组件时，自动重新生成。

预初始化文件与中间代码缓存组件
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在安装了加速器组件的基础上，使用预初始化文件也依然会使框架有更好的性能表现，因为需要由加速器监测代码变动的文件的数量减少了。当然，如果你关闭了对文件变动的监测（如APC的\ ``apc.stat=0``\ ），预初始化文件也就没有必要了。

.. _`加速器`: http://en.wikipedia.org/wiki/List_of_PHP_accelerators
.. _`APC`: http://php.net/manual/en/book.apc.php
.. _`autoloader.php`: https://github.com/symfony/symfony-standard/blob/master/app/autoload.php
.. _`bootstrap file`: https://github.com/sensio/SensioDistributionBundle/blob/2.0/Resources/bin/build_bootstrap.php
