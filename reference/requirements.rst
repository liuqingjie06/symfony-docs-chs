.. index::
   single: Requirements
   
Symfony2的运行条件
==================

你的系统需要满足一定的条件，才能正确、高效地运行Symfony2。你可以通过访问 ``web/config.php`` 来确认这些条件是否满足。由于命令行的PHP通常使用另外的 ``php.ini`` 文件，所以你还应该在命令行下进行检查：

.. code-block:: bash

    php app/check.php

以下是运行条件的清单：

必须
----

* PHP的版本应是5.3.2以上
* 安装Sqlite3并启用PHP的支持
* 启用JSON
* 启用ctype
* PHP.ini里需要设置date.timezone（时区，中国可以用Asia/Chongqing）

可选
----

* 启用PHP-XML
* 2.6.21以上版本的libxml
* 启用PHP tokenizer
* 启用mbstring（开发中文的应用程序，这个是必须）
* 启用iconv
* 启用POSIX（仅限于\*nix类系统）
* 启用Intl，并安装ICU 4+
* 安装3.0.17以上版本的APC（或者其他的加速器）
* PHP.ini里的一些推荐配置

  * ``short_open_tag = Off``
  * ``magic_quotes_gpc = Off``
  * ``register_globals = Off``
  * ``session.autostart = Off``

Doctrine
--------

如果你打算使用Doctrine，你需要安装PDO，以及与你打算使用的数据库相应的PDO
驱动。
