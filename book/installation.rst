.. index::
   single: Installation

Symfony2的安装和配置
====================

本章将说明如何上手Symfony来搭建Web应用。Symfony官方提供了预先配置好的“发行版”，包含供开发者借鉴的示例代码，。

.. tip::

    如果你想了解如何创建新的Symfony2项目，并对代码进行版本管理，请阅读： `代码版本管理`_ 。

下载Symfony2的发行版
--------------------

.. tip::

    首先，请确认你安装了Web服务器（如Apache），以及5.3.2以上版本的PHP。更多关于Symfony运行条件的细节，请参考：《 :doc:`/reference/requirements` 》。你可以访问 `Apache`_ 和 `Nginx`_ 的官方网站来了解如何配置站点的根目录。

Symfony2有不同的“发行版”供你选择，但它们都包含了：基于Symfony2框架的应用程序，一些有用的第三方代码包，推荐的文件目录结构以及默认的配置。下载一个Symfony2的发行版，相当于得到了一个可以立即运行的代码骨架，有了这个基础，你可以快速开发你自己的应用。

你可以访问Symfony2的下载页面来获得Symfony2的发行版： `http://symfony.com/download`_ 。你有两个选项：

* 下载 ``.tgz`` 或者 ``.zip`` 文件。这两种格式的文件所包含的内容是一致的，你可以任意选择。

* 下载不包含第三方代码的发行版。如果你的电脑上安装了 `Git`_ ，建议你下载这个版本。因为Symfony2提供了通过Git来管理第三方代码的便利，可以更灵活地实现定制。

把下载下来的压缩包，解压到你本地的站点根目录。如果使用UNIX命令行，你可以输入以下命令（注意将 ``###`` 替换为实际的文件名）：

.. code-block:: bash

    # .tgz 文件
    tar zxvf Symfony_Standard_Vendors_2.0.###.tgz

    # .zip 文件
    unzip Symfony_Standard_Vendors_2.0.###.zip

解压后的 ``Symfony/`` 目录内容如下：

.. code-block:: text

    www/ <- 你的站点根目录
        Symfony/ <- 压缩包里的Symfony目录
            app/
                cache/
                config/
                logs/
            src/
                ...
            vendor/
                ...
            web/
                app.php
                ...

升级第三方代码
~~~~~~~~~~~~~~

如果你下载的是不包含第三方代码的版本，你可以通过下面的命令来完成安装：

.. code-block:: bash

    php bin/vendors install

这个命令将在 ``vendor/`` 目录里安装第三方代码（包括Symfony框架本身的核心库文件）。要了解Symfony2是如何管理第三方代码的，请阅读：《 :ref:`cookbook-managing-vendor-libraries` 》。

配置与安装
~~~~~~~~~~

完成上一步骤，所有必须的第三方代码都应该已经保存在 ``vendor/`` 目录里了。另外，在 ``app/`` 里，有一个Web应用的常规配置， ``src/`` 里则有一套示例代码。

Symfony2自带一个服务器运行环境的检测脚本，用来确保你的服务器和PHP的参数是正确的。你可以通过下面的地址来访问这个页面：

.. code-block:: text

    http://localhost/Symfony/web/config.php

如果有问题，请修正。

.. sidebar:: 设置文件权限

    一个常见的问题是，Web服务器和你做开发使用的用户帐户都需要 ``app/cache`` 和 ``app/logs`` 目录的写权限。如果你使用的是UNIX系统，而你的Web服务器用户和开发帐户不是同一个，你可以运行下面的命令来确保有正确的文件权限。注意替换 ``www-data`` 为你实际的Web服务器用户：

    **1. 如果你的系统支持通过 chmod +a 来配置ACL（访问控制）**

    多数系统都允许你使用 ``chmod +a`` 命令，如果系统报错，你可以尝试方法2。
    
    .. code-block:: bash

        rm -rf app/cache/*
        rm -rf app/logs/*

        sudo chmod +a "www-data allow delete,write,append,file_inherit,directory_inherit" app/cache app/logs
        sudo chmod +a "`whoami` allow delete,write,append,file_inherit,directory_inherit" app/cache app/logs

    **2. 不支持 chmod +a 的系统**

    有的系统不支持 ``chmod +a`` ，但有 ``setfacl`` 工具可以用来完成同样的任务。你可能需要在相应的分区 `启用文件系统ACL`_ ，并安装setfacl工具（Ubuntu系统即是这个情况），然后运行如下命令：

    .. code-block:: bash

        sudo setfacl -R -m u:www-data:rwx -m u:`whoami`:rwx app/cache app/logs
        sudo setfacl -dR -m u:www-data:rwx -m u:`whoami`:rwx app/cache app/logs

    需要注意的是，并不是所有Web服务器都以 ``www-data`` 用户运行。你可以通过查看服务器进程来了解实际的帐户是什么。

    **3. 没有ACL**

    如果你没法启用ACL，你也可以通过修改umask，使cache和logs目录有用户组或全局可写的权限。即，在 ``app/console`` ， ``web/app.php`` 和 ``web/app_dev.php`` 文件里增加以下语句：

    .. code-block:: php

        umask(0002); // PHP脚本生成的文件权限为0775

        // 或者

        umask(0000); // PHP脚本生成的文件权限为0777

    更推荐使用ACL，因为修改umask不是线程安全的。

好了，所有的问题都解决了，点击”Go to the Welcome page”来访问Symfony的欢迎页吧：

.. code-block:: text

    http://localhost/Symfony/web/app_dev.php/

Symfony2这时应该会向你打招呼，辛苦了！

.. image:: /images/quick_tour/welcome.jpg

进行开发
--------

你的Symfony2应用程序运行起来了，你可以进行开发了！你的发行版可能包含了一些示例代码，你可以阅读 ``README.rst`` 文件来确认哪些代码被包含在你所使用的发行版里，并了解如何在不需要的时候移除它们。

如果你刚接触Symfony，《 :doc:`page_creation` 》将向你介绍如何在你新建的Web应用里基于Symfony来创建页面，修改配置等等。

代码版本管理
------------

如果你打算用 ``Git`` 或者 ``Subversion`` 来管理你的代码，你可以照常操作，并将Symfony2的标准发行版作为你项目的起点。

要了解如何更好地用Git来管理你的Symfony项目，请阅读： :doc:`/cookbook/workflow/new_project_git` 。

忽略 ``vendor/`` 目录
~~~~~~~~~~~~~~~~~~~~~

如果你使用的是不包含第三方代码的发行版，或者你了解如何使用Symfony2自带的脚本通过Git来管理代码依赖，你可以在 ``.gitignore`` 文件里忽略 ``vendor/`` 目录：

.. code-block:: text

    vendor/

这样，你的vendor目录不会被提交到Git。其他人如果需要参与项目，他只需要检出相对少很多的文件，并通过 ``php bin/vendors install`` 命令来下载必须的第三方代码。

.. _`启用文件系统ACL`: https://help.ubuntu.com/community/FilePermissionsACLs
.. _`http://symfony.com/download`: http://symfony.com/download
.. _`Git`: http://git-scm.com/
.. _`GitHub Bootcamp`: http://help.github.com/set-up-git-redirect
.. _`Apache`: http://httpd.apache.org/docs/current/mod/core.html#documentroot
.. _`Nginx`: http://wiki.nginx.org/Symfony
