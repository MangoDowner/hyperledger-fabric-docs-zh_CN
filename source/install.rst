安装例子，二进制文件还有Docker镜像
===========================================

在开始为Hyperledger Fabric二进制文件开发真正的安装程序前,
我们提供了一个脚本用以在你的系统上下载和安装例子及二进制文件。
我们认为，您将发现安装的示例应用程序是有用的，
可以了解更多关于Hyperledger Fabric的功能和操作。

.. note:: 如果你在 **Windows** 下运行，你将需要使用Docker Quickstart Terminal来执行接下来的终端命令。
          如果你之前没有安装它的话，请访问P :doc:`prereqs`。

          如果你在Windows 7或者macOS上使用Docker Toolbox,当你安装和运行例子的时候，你将需要使用
          ``C:\Users`` (Windows 7) 或者 ``/Users`` (macOS) 下的某个位置。

          如果你使用Mac下的Docker，你需要在这些目录下面寻找合适的位置：
          ``/Users``, ``/Volumes``, ``/private``, 或者 ``/tmp``。如果要使用不同的位置，
          请查询Docker文档 `file sharing <https://docs.docker.com/docker-for-mac/#file-sharing>`__ 。

          If you are using Docker for Windows, please consult the Docker
          documentation for `shared drives <https://docs.docker.com/docker-for-windows/#shared-drives>`__
          and use a location under one of the shared drives.

在你的机器上确定一个位置来放置 `fabric-samples` 目录，并且在命令行窗口里访问那个目录。
下面的命令将会执行以下步骤：

#. 如果需要的话，克隆 `hyperledger/fabric-samples` 仓库
#. Checkout正确的版本tag
#. 安装 Hyperledger Fabric 基于平台的二进制文件，并且在fabric-samples仓库的根目录里配置版本文件。
#. 下载特定版本的Hyperledger Fabric的Docker镜像

一旦你准备好了，并且正位于你准备安装Fabric例子和二进制的目录里，那么继续执行下面的命令吧：

.. code:: bash

  curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2.0

.. note:: 如果你想要下载Fabric，Fabric-ca和第三方的Docker镜像，
          你必须把版本标志符传给脚本。

.. code:: bash

  curl -sSL http://bit.ly/2ysbOFE | bash -s <fabric> <fabric-ca> <thirdparty>
  curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2.0 1.2.0 0.4.10

.. note:: 如果你执行上述命令遇到了错误, 你可能使用了旧版本的curl，
          其不能处理跳转或者不支持的环境。you may


	  请访问 :doc:`prereqs` 页面以获取额外的信息，关于哪里得到最新的curl和正确的环境。
      作为替代，你可以赚到下面的这个URL：
	  https://github.com/hyperledger/fabric/blob/master/scripts/bootstrap.sh

.. note:: 你可以为任何发行版本的Hyperledger Fabric执行上面的命令。
          只需要将 `1.2.0` 替换为你要安装的版本标识符就好。

上面的命令下载并执行了一个bash脚本，该脚本会下载并且解压所有的平台相关的二进制文件，
这些二进制文件将用于创建你的网络，并且将他们放置在你上面创建的克隆仓库。
它获取下面这些平台相关的二进制文件

  * ``cryptogen``,
  * ``configtxgen``,
  * ``configtxlator``,
  * ``peer``,
  * ``orderer``,
  * ``idemixgen``, 还有
  * ``fabric-ca-client``

并且将他们放置在当前工作目录的 ``bin`` 子目录下。

你可能需要将那个放在你的PATH环境变量里，这样不用输入全拼路径，就可以访问到这些文件。比如说：

.. code:: bash

  export PATH=<path to download location>/bin:$PATH

最后，这个脚本会下载 `Docker Hub <https://hub.docker.com/u/hyperledger/>`__ 下的
Hyperledger Fabric的Docker镜像，然后放置在你的本地Docker档案里，并打上'latest'标记。

一言以蔽之，该脚本列举了安装的Docker镜像。

看看每个镜像的名字；这些将会是最终组成我们Hyperledger Fabric网络的组件。
你还会发现两个有着相同镜像ID的实例 - 一个标签为 "amd64-1.x.x"，另一个标签为 "latest"。
在1.2.0之前，下载的镜像由 ``uname -m`` 决定并展示为 "x86_64-1.x.x"。

.. note:: 在不同的系统架构下, x86_64/amd64 会被定义你架构的文本所替换。

.. note:: 如果你还有本文未能提到的问题，或者遇到了任何教程中的问题，请访问 :doc:`questions`
页面来获取如何找到更多帮助的方法。

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
