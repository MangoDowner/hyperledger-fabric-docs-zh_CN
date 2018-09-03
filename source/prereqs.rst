先决条件
=============

在我们开始之前，如果你还没这样做的话，最好检查下你是否安装了下面提到的所有先决条件，
他们需要被安装在开发区块链应用或操作Hyperledger Fabric的平台上。

安装cURL
------------

下载最新版本的 `cURL <https://curl.haxx.se/download.html>`__ 工具，
如果还没安装或者运行文档中的curl命令报错的话。

.. 提示:: 如果你在Windows下，请看下面的 `Windows extras`_ .

Docker 和 Docker Compose
-------------------------

你将需要在操作或开发Hyperledger Fabric的平台上安装以下这些：

  - MacOSX, \*nix, 或者 Windows 10: `Docker <https://www.docker.com/get-docker>`__
    需要Docker 17.06.2-ce 或者更高版本。
  - 老版本的Windows: `Docker
    Toolbox <https://docs.docker.com/toolbox/toolbox_install_windows/>`__ -
    同样, 需要Docker 17.06.2-ce 或者更高版本。

你可以在命令行中输入以下命令来确认你安装的Docker版本。

.. code:: bash

  docker --version

.. note:: 在Mac或者Windows上安装Docker会同时安装Docker Compose。如果里已经安装了Docker，
          你需要检查下你是不是也安装了Docker Compose 1.14.0或者更高版本。
          如果没有，我们建议你安装一个更近一点的Docker版本。

你可以在命令行中输入以下命令来确认你安装的ocker Compose版本

.. code:: bash

  docker-compose --version

.. _Golang:

Go 编程语言
-----------------------

Hyperledger Fabric使用Go编程语言来编写它的很多组件。

  - `Go <https://golang.org/dl/>`__ 1.10.x 版本是需要的。

既然我们要用Go来编写链码（chaincode）程序，有两个环境变量就需要被正确的设置；
你可以将他们放在合适的启动文件里，这样就可以永久设定了，比如说你使用Linux的话，
你就可以使用你的 ``~/.bashrc`` 文件。

首先，你需要设置环境变量 ``GOPATH`` 以指定Go的工作空间（workspace），它包含了下载的Fabric代码。
比如可以这么设定：


.. code:: bash

  export GOPATH=$HOME/go

.. note:: 你 **必须** 设置GOPATH变量

  尽管在Linux中，Go的 ``GOPATH`` 变量可以是一个冒号分隔的目录列表，并且在未设置的情况下使用默认值
  ``$HOME/go``，当前的Fabric构建框架仍然要求你设置并且导出（export）这个变量。并且它必须 **只** 包含
  Go的工作空间目录。(这个限制可能会在将来的版本中被移除。)

其二，你应该（再次，在正确的启动文件里）扩展你的命令查询路径（PATH）来将Go ``bin`` 目录包含进来，比如说下面的
Linux ``bash`` 例子:

.. code:: bash

  export PATH=$PATH:$GOPATH/bin

虽然这个目录可能在新安装的Go工作空间中不存在，但它随后将由Fabric构建系统填充，其中包含了少量的go可执行文件，
这些文件由构建系统的其他部分使用。所以即使你现在没有这个目录，还请如上所述那样扩展你的命令行查询路径。

Node.js 运行时（Runtime） 和 NPM
-----------------------

如果你要使用Hyperledger Fabric SDK的Node.js版本来开发Hyperledger Fabric应用, 你就需要安装Node.js的8.9.x 版本。

.. note:: Node.js version 9.x is not supported at this time.

  - `Node.js <https://nodejs.org/en/download/>`__ - 8.9.x 或者更高版本

.. note:: 安装Node.js同时会安装NPM，然而你最好确认下安装的NPM版本。你可以使用下面的命令来更新 ``npm`` 工具。

.. code:: bash

  npm install npm@5.6.0 -g

Python
^^^^^^

.. note:: 下面的内容只对Ubuntu 16.04用户有效。

  译者注：MacOS默认安装了Python 2.7。

默认情况下，Ubuntu 16.04会安装好Python 3.5.1版本来作为 ``python3`` 的可执行文件。
Fabric Node.js SDK需要Python 2.7以执行 ``npm install`` 操作，从而成功安装。
通过下面的命令可以获取2.7版本：

.. code:: bash

  sudo apt-get install python

检查你的版本:

.. code:: bash

  python --version

.. _windows-extras:

Windows 额外需要安装的东西
--------------

如果你在Windows 7上面开发，你需要使用Docker Quickstart Terminal，它使用 `Git Bash
<https://git-scm.com/downloads>`__ 并且提供了Windows内置shell的更好替代物。

然而经验表明，这是个功能受限不好的开发环境。它适用于运行基于Docker的场景，比如说 :doc:`getting_started`,
但是在使用包含``make`` 及 ``docker`` 命令在内的操作的时候，可能会困难重重。

在Windows 10上面，你应该使用原生的Docker发行版，而且最好使用Windows PowerShell。
然而，为了让 ``binaries`` 命令能够成功运行，你还需要 ``uname`` 命令是可用的。
你可以将它作为Git的一部分获取到，但是请注意，只有64位的版本才支持。


在运行任何 ``git clone`` 命令之前，运行下面的命令：

::

    git config --global core.autocrlf false
    git config --global core.longpaths true

你可以使用下面的命令来检查这些参数的设定：

::

    git config --get core.autocrlf
    git config --get core.longpaths
它们需要分别为 ``false`` 和 ``true``。

Git和 Docer Toolbox中的 ``curl`` 命令已经陈旧了，而且不能够正确处理 :doc:`getting_started` 中的跳转。
确保你通过 `cURL downloads page <https://curl.haxx.se/download.html>`__ 安装并使用了更新的版本。

为了Node.js,你还需要不可或缺的Visual Studio C++构建工具，它可以免费获取而且可以通过以下命令安装：

.. code:: bash

	  npm install --global windows-build-tools

查看 `NPM windows-build-tools page
<https://www.npmjs.com/package/windows-build-tools>`__ 来获取更多细节。

一旦这些都做到了，你应该使用下面的命令来安装NPM GRPC模块：

.. code:: bash

	  npm install --global grpc

现在，你的环境应该能够走通 :doc:`getting_started` 中的例子和教程了。

.. note:: 如果你还有本文未能提到的问题，或者遇到了任何教程中的问题，请访问 :doc:`questions`
页面来获取如何找到更多帮助的方法。


.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
