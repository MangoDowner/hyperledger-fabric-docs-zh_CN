创建你的第一个网络
===========================

.. note::  该教程已经被验证过，基于最新的稳定版本Docker镜像和预编译的安装程序（包含在提供的tar文件里）。
           如果你使用当前的master分支中的镜像或工具来运行指令，有可能会看到配置和panic错误。

“构建你的第一个网络（BYFN）”场景提供了一个示例性的 Hyperledger Fabric 网络，
该网络包括了两个组织机构(Organization)，每个组织机构(Organization)都维护了两个对等节点(Peer)，
并提供“单独”的排序服务(Ordering Service)。

安装先决条件（Install prerequisites）
---------------------

在我们开始之前，如果你还没这样做的话，最好检查下你是否安装了所有的 :doc:`prereqs`，
他们需要被安装在开发区块链应用或操作Hyperledger Fabric的平台上。

你还需要 :doc:`install`。你会发现在 ``fabric-samples`` 仓库里已经有很多例子了。
我们会使用 ``first-network`` 这个例子。现在让我们打开那个子目录吧。

.. code:: bash

  cd fabric-samples/first-network

.. note:: 本文章中提供的命令 **必须** 在你的 ``fabric-samples`` 仓库的 ``first-network`` 子目录下运行。
如果你选择在其他位置运行命令，那么提供的各种脚本就无法找到可执行文件。

想要现在就运行吗？
-------------------

我们提供了一个完整注释的脚本 - ``byfn.sh`` - 它利用这些Docker映像快速引导一个Hyperledger Fabric网络，
该网络由4个对等节点（peers）组成，代表两个不同的组织（organization），以及一个排序节点（orderer node）。
它还将启动一个容器来运行脚本化的执行，该执行将将对等节点连接到通道、部署和实例化链码，并驱动执行针对已部署链码的事务。

这里是 ``byfn.sh`` 脚本的帮助内容:

.. code:: bash

  Usage:
    byfn.sh <mode> [-c <channel name>] [-t <timeout>] [-d <delay>] [-f <docker-compose-file>] [-s <dbtype>] [-l <language>] [-i <imagetag>] [-v]
      <mode> - one of 'up', 'down', 'restart', 'generate' or 'upgrade'
        - 'up' - bring up the network with docker-compose up
        - 'down' - clear the network with docker-compose down
        - 'restart' - restart the network
        - 'generate' - generate required certificates and genesis block
        - 'upgrade'  - upgrade the network from v1.0.x to v1.1
      -c <channel name> - channel name to use (defaults to "mychannel")
      -t <timeout> - CLI timeout duration in seconds (defaults to 10)
      -d <delay> - delay duration in seconds (defaults to 3)
      -f <docker-compose-file> - specify which docker-compose file use (defaults to docker-compose-cli.yaml)
      -s <dbtype> - the database backend to use: goleveldb (default) or couchdb
      -l <language> - the chaincode language: golang (default) or node
      -i <imagetag> - the tag to be used to launch the network (defaults to "latest")
      -v - verbose mode
    byfn.sh -h (print this message)

  Typically, one would first generate the required certificates and
  genesis block, then bring up the network. e.g.:

	  byfn.sh generate -c mychannel
	  byfn.sh up -c mychannel -s couchdb
          byfn.sh up -c mychannel -s couchdb -i 1.1.0-alpha
	  byfn.sh up -l node
	  byfn.sh down -c mychannel
          byfn.sh upgrade -c mychannel

  Taking all defaults:
	  byfn.sh generate
	  byfn.sh up
	  byfn.sh down

如果你还没提供通道名称，脚本将会使用默认名称``mychannel``。
CLI超时参数(用-t标记指定)是可选的值；如果你不设置它，CLI将默认在10s后放弃请求。

生成网络构建（Artifacts）
^^^^^^^^^^^^^^^^^^^^^^^^^^

准备好了吗？好的！执行下面的命令：
.. code:: bash

  ./byfn.sh generate

你会看到一个简短描述，告诉你将会发生什么，同时还有个是/否命令行提醒。
回复``y`` 或者按下回车键来执行描述的动作。

.. code:: bash

  Generating certs and genesis block for with channel 'mychannel' and CLI timeout of '10'
  Continue? [Y/n] y
  proceeding ...
  /Users/xxx/dev/fabric-samples/bin/cryptogen

  ##########################################################
  ##### Generate certificates using cryptogen tool #########
  ##########################################################
  org1.example.com
  2017-06-12 21:01:37.334 EDT [bccsp] GetDefault -> WARN 001 Before using BCCSP, please call InitFactories(). Falling back to bootBCCSP.
  ...

  /Users/xxx/dev/fabric-samples/bin/configtxgen
  ##########################################################
  #########  Generating Orderer Genesis block ##############
  ##########################################################
  2017-06-12 21:01:37.558 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
  2017-06-12 21:01:37.562 EDT [msp] getMspConfig -> INFO 002 intermediate certs folder not found at [/Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/intermediatecerts]. Skipping.: [stat /Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/intermediatecerts: no such file or directory]
  ...
  2017-06-12 21:01:37.588 EDT [common/configtx/tool] doOutputBlock -> INFO 00b Generating genesis block
  2017-06-12 21:01:37.590 EDT [common/configtx/tool] doOutputBlock -> INFO 00c Writing genesis block

  #################################################################
  ### Generating channel configuration transaction 'channel.tx' ###
  #################################################################
  2017-06-12 21:01:37.634 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
  2017-06-12 21:01:37.644 EDT [common/configtx/tool] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
  2017-06-12 21:01:37.645 EDT [common/configtx/tool] doOutputChannelCreateTx -> INFO 003 Writing new channel tx

  #################################################################
  #######    Generating anchor peer update for Org1MSP   ##########
  #################################################################
  2017-06-12 21:01:37.674 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
  2017-06-12 21:01:37.678 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
  2017-06-12 21:01:37.679 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update

  #################################################################
  #######    Generating anchor peer update for Org2MSP   ##########
  #################################################################
  2017-06-12 21:01:37.700 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
  2017-06-12 21:01:37.704 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
  2017-06-12 21:01:37.704 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update

第一步生成了以下东西：为不同的网络实体生成了以下东西所有的证书和密钥；
``genesis block``，是用来启动排序服务的；
一个配置事务集合，用来配置一个 :ref:`Channel`。

打开网络
^^^^^^^^^^^^^^^^^^^^

接下来，你可以用下面的一个命令来打开网络：

.. code:: bash

  ./byfn.sh up


上面的命令会编译Golang的链码镜像并启动相应的容器。Go是默认的链码语言，
然而链码也支持 `Node.js <https://fabric-shim.github.io/>`__ 。
如果你想用node链码来走通整个教程，传递下面的命令作为替代：

.. code:: bash

  # we use the -l flag to specify the chaincode language
  # forgoing the -l flag will default to Golang

  ./byfn.sh up -l node

.. note:: 查看 `Hyperledger Fabric Shim <https://fabric-shim.github.io/ChaincodeStub.html>`__
          文档来找到更多关于node.js chaincode shim APIs的内容

再次，你又看到了一个提醒，问你是否要继续或者放弃。
输入``y`` 或者按下回车：

.. code:: bash

  Starting with channel 'mychannel' and CLI timeout of '10'
  Continue? [Y/n]
  proceeding ...
  Creating network "net_byfn" with the default driver
  Creating peer0.org1.example.com
  Creating peer1.org1.example.com
  Creating peer0.org2.example.com
  Creating orderer.example.com
  Creating peer1.org2.example.com
  Creating cli


   ____    _____      _      ____    _____
  / ___|  |_   _|    / \    |  _ \  |_   _|
  \___ \    | |     / _ \   | |_) |   | |
   ___) |   | |    / ___ \  |  _ <    | |
  |____/    |_|   /_/   \_\ |_| \_\   |_|

  Channel name : mychannel
  Creating channel...

日志将会在这里继续。这将启动所有的容器，然后启动一个完全的端到端场景。
一旦成功安装了，它将在你的命令窗口里报告以下内容：

.. code:: bash

    Query Result: 90
    2017-05-16 17:08:15.158 UTC [main] main -> INFO 008 Exiting.....
    ===================== Query successful on peer1.org2 on channel 'mychannel' =====================

    ===================== All GOOD, BYFN execution completed =====================


     _____   _   _   ____
    | ____| | \ | | |  _ \
    |  _|   |  \| | | | | |
    | |___  | |\  | | |_| |
    |_____| |_| \_| |____/

你可以滚动这些日志来看到各种各样的事务。如果你没能得到上述结果，
那么就去 :ref:`Troubleshoot` 部分，让我们看看是不是能帮忙找到哪儿出了问题。

关闭网络
^^^^^^^^^^^^^^^^^^^^^^

最后，我们来关闭所有东西，这样一来，我们可以逐步探索整个网络启动的过程。
下面的操作将清除镜像并且移除加密材料和四个构建（artifacts），并且从你的Docker注册表里删除链码镜像。

.. code:: bash

  ./byfn.sh down

再次，你有碰到了是否继续的提醒，输入``y`` 或者按下回车：

.. code:: bash

  Stopping with channel 'mychannel' and CLI timeout of '10'
  Continue? [Y/n] y
  proceeding ...
  WARNING: The CHANNEL_NAME variable is not set. Defaulting to a blank string.
  WARNING: The TIMEOUT variable is not set. Defaulting to a blank string.
  Removing network net_byfn
  468aaa6201ed
  ...
  Untagged: dev-peer1.org2.example.com-mycc-1.0:latest
  Deleted: sha256:ed3230614e64e1c83e510c0c282e982d2b06d148b1c498bbdcc429e2b2531e91
  ...

如果你想要学习更多关于潜在的工具和启动机制的内容，继续阅读。
在接下来的部分，我们将会从头到尾走一遍创建具备完整功能的Hyperledger Fabric network网络的流程。

.. note:: 下面突出的手册步骤是基于这样的前提的，即 ``cli`` 容器的 ``CORE_LOGGING_LEVEL`` 设置为 ``DEBUG``。
          你可以通过修改 ``first-network`` 目录里的 ``docker-compose-cli.yaml`` 文件来设置它。
          比如说：

          .. code::

            cli:
              container_name: cli
              image: hyperledger/fabric-tools:$IMAGE_TAG
              tty: true
              stdin_open: true
              environment:
                - GOPATH=/opt/gopath
                - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
                - CORE_LOGGING_LEVEL=DEBUG
                #- CORE_LOGGING_LEVEL=INFO

加密生成器
----------------

我们将使用 ``cryptogen`` 工具为各种网络实体生成加密材料(x509 证书和签名密钥) 。
这些证书是网络实体的代表，他们允许实体交流和事务的时候进行身份的签名/验证。

它是怎么工作的呢?
^^^^^^^^^^^^^^^^^

Cryptogen假定文件 - ``crypto-config.yaml`` -
其包含网络拓扑并让我们可以为组织和组织下属的组件生成一系列证书和密钥。
每个组织都又一个唯一的根证书 (``ca-cert``)，这个证书将特定的组件（对等节点和排序节点）绑定到组织里。
通过给每个组织分配唯一的根证书，我们模拟了一个典型的网络，该网络中参与 :ref:`Member` 将会使用自己的证书授权。
Hyperledger Fabric里的事务和交流，是通过实体的私钥 (``keystore``) 签名的，然后通过公钥 (``signcerts``) 来验证。

你会发现该文件里有一个 ``count`` 变量。我们使用它来指明每个组织中的对等节点的数量；
在我们的例子里，每个组织有2个对等节点。
我们现在不会去钻研 `x.509 certificates and public key
infrastructure <https://en.wikipedia.org/wiki/Public_key_infrastructure>`__
的细节。如果你感兴趣的话，可以私下去细读这些主题。

在运行工具之前，我们先快速浏览下 ``crypto-config.yaml`` 的一个片段.
特别注意``OrdererOrgs`` 头部下的 "Name", "Domain" 还有 "Specs" 参数：

.. code:: bash

  OrdererOrgs:
  #---------------------------------------------------------
  # Orderer
  # --------------------------------------------------------
  - Name: Orderer
    Domain: example.com
    CA:
        Country: US
        Province: California
        Locality: San Francisco
    #   OrganizationalUnit: Hyperledger Fabric
    #   StreetAddress: address for org # default nil
    #   PostalCode: postalCode for org # default nil
    # ------------------------------------------------------
    # "Specs" - See PeerOrgs below for complete description
  # -----------------------------------------------------
    Specs:
      - Hostname: orderer
  # -------------------------------------------------------
  # "PeerOrgs" - Definition of organizations managing peer nodes
   # ------------------------------------------------------
  PeerOrgs:
  # -----------------------------------------------------
  # Org1
  # ----------------------------------------------------
  - Name: Org1
    Domain: org1.example.com
    EnableNodeOUs: true

网络实体的命名规范如下所述 - "{{.Hostname}}.{{.Domain}}"。
所以使用我们的排序节点作为参考点，我们就剩下了个名为 ``orderer.example.com`` 的排序节点，
它被绑定到了一个叫做 ``Orderer`` 的MSP ID上了。
该文件包含了关于定义和语法的扩展文档。你也可以查看 :doc:`msp` 文档来深入了解MSP.

我们运行 ``cryptogen`` 工具后, 生成的证书和密钥就被保存进了一个名为``crypto-config``的文件夹。

配置事务生成器
-----------------------------------

``configtxgen tool`` 用来生成四个配置构建（artifacts）:

  * orderer ``genesis block``,
  * channel ``configuration transaction``,
  * 还有两个 ``anchor peer transactions`` - 每个Peer Org各有一个。

请查看 :doc:`commands/configtxgen` ，其包含了工具功能的完整叙述。

orderer块是订购服务的Genesis块，通道配置事务文件在通道创建时向订购者广播。锚点对等事务(顾名思义)在这个通道上指定每个Org的锚点。

排序节点的块是排序服务的:ref:`Genesis-Block`，通道配置事务文件在 :ref:`Channel` 创建时像排序节点广播。
锚点peer事务，顾名思义，在这个通道上指定每个Org的: ref:`Anchor-Peer`。

它是怎么工作的呢?
^^^^^^^^^^^^^^^^^

Configtxgen 用到了一个文件 - ``configtx.yaml`` - 其包含了例子网络的定义。
存在三个成员 - 一个排序节点组织 (``OrdererOrg``) 还有两个Peer组织 (``Org1`` & ``Org2``) 各自管理和维护了两个peer节点。
该文件还指明了一个联盟（consortium） - ``SampleConsortium`` - 包含了我们的两个peer组织；
请特别注意下该文件顶部的"Profiles"部分。你会发现我们有两个独一无二的首部。一个是给排序节点创世块的 - ``TwoOrgsOrdererGenesis`` -
还有一个是给我们的通道的 - ``TwoOrgsChannel``.

这些首部很重要，我们将会在创建构建（artifacts）的时候将他们作为参数传递过去。

.. note:: 注意我们的 ``SampleConsortium`` 在系统级别的概要文件中定义，然后在通道级别的概要文件里被引用。
          渠道存在于联盟的权限范围内，所有联盟必须在整个网络的范围内进行定义。

值得注意的是，这个文件还包含了两个额外的规范。
首先，我们为每个Peer组织(``peer0.org1.example.com`` & ``peer0.org2.example.com``)指定锚点peer。
其次，我们指出每个成员的MSP目录的位置，从而允许我们在排序节点的创世块中存储每个Org的根证书。
这是一个关键的概念。现在任何与排序服务通信的网络实体都可以验证其数字签名。

运行工具
-------------

你可以通过``configtxgen`` 和 ``cryptogen``命令来手工生成证书/密钥以及各种配置构建（artifacts）。
作为替代，你可以试着使用 byfn.sh 脚本来达成目标。

手动生成构建（artifacts）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

您可以在byfn.sh脚本中引用``generateCerts``函数用于生成证书所需的命令，这些证书将用于您的网络配置，
正如 ``crypto-config.yaml`` 中定义的那样。但是，为了方便起见，我们在这里也提供一个参考。

首先，我们来运行 ``cryptogen`` 工具。我们的二进制文件在 ``bin`` 目录里，所以我们需要提供工具所在的相对路径。

.. code:: bash

    ../bin/cryptogen generate --config=./crypto-config.yaml

你会在终端里看到下面的东西:

.. code:: bash

  org1.example.com
  org2.example.com

证书和密钥(即 MSP material) 将会输出到一个目录里 - ``crypto-config`` -
其就在 ``first-network`` 的根目录里面.

接下来，我们需要告诉``configtxgen`` 工具在哪里查找它需要摄取的文件 ``configtx.yaml`` 。
我们会让它在我们的当前工作目录里找:

.. code:: bash

    export FABRIC_CFG_PATH=$PWD

然后，我们调用 ``configtxgen`` 工具来创建排序节点的创世区块：

.. code:: bash

    ../bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block

你会在终端里看到类似下面的输出：

.. code:: bash

  2017-10-26 19:21:56.301 EDT [common/tools/configtxgen] main -> INFO 001 Loading configuration
  2017-10-26 19:21:56.309 EDT [common/tools/configtxgen] doOutputBlock -> INFO 002 Generating genesis block
  2017-10-26 19:21:56.309 EDT [common/tools/configtxgen] doOutputBlock -> INFO 003 Writing genesis block

.. note:: 我们将要创建的orderer genesis块和后续构建将输出到项目根目录下的``channel-artifacts``目录中。

.. _createchanneltx:

创建一个通道配置事务
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

接下来，我们需要创建通道事务构建。
请确保将 ``$CHANNEL_NAME`` 设置或者替换为一个可以在整个指令中都能使用的环境变量。

.. code:: bash

    # channel.tx 构建包含了我们整个示例通道的定义

    export CHANNEL_NAME=mychannel  && ../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME

你会在终端里看到类似下面的输出：

.. code:: bash

  2017-10-26 19:24:05.324 EDT [common/tools/configtxgen] main -> INFO 001 Loading configuration
  2017-10-26 19:24:05.329 EDT [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
  2017-10-26 19:24:05.329 EDT [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 003 Writing new channel tx

接下来，我们将在正构建的通道上为Org1定义锚点peer。
同样，请确保为接下来的命令替换或设置``$CHANNEL_NAME``。终端输出将模拟通道事务构建的输出:

.. code:: bash

    ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP

现在我们将为在同一个通道上为Org2定义锚点peer：

.. code:: bash

    ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP

启动网络
-----------------

.. note:: 如果你之前跑过了 ``byfn.sh`` 例子,请确保在行动前你已经关闭了测试网络
          (查看 `Bring Down the Network`_).

我们会使用一个脚本来启动来运转我们的网络。
docker-compose会引用我们之前下载的镜像,并且通过我们之前生成的 ``genesis.block`` 来启动排序节。

我们希望手动遍历这些命令，以便公开每个调用的语法和功能。

首先，让我们开始我们的网络：

.. code:: bash

    docker-compose -f docker-compose-cli.yaml up -d

如果你想要看到你的网络的实时日志，需要提供 ``-d`` 标记.
如果想要看到日志流，那么需要打开第二个终端来执行CLI调用。

.. _peerenvvars:

环境变量
^^^^^^^^^^^^^^^^^^^^^

为了让下面针对 ``peer0.org1.example.com`` 的CLI命令起作用，我们需要使用下面给出的四个环境变量作为命令的前言。
``peer0.org1.example.com`` 的这些变量被包含了到CLI容器中，因此我们可以在不传递它们的情况下操作。
**然而**，如果您想要向其他peer或排序节点发送调用，那么您可以通过在启动容器前先编辑``docker-compose-base.yaml``来相应地提供这些值。
修改以下四个环境变量以使用不同的peer和org。

.. code:: bash

    # PEER0的环境变量

    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    CORE_PEER_LOCALMSPID="Org1MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt

.. _createandjoin:

创建&加入通道
^^^^^^^^^^^^^^^^^^^^^

回想一下，我们在上面的 :ref:`createchanneltx` 部分中使用 ``configtxgen`` 工具创建了通道配置事务。
您可以重复这个过程，通过使用传递给 ``configtxgen`` 工具的不同的 ``configtx.yaml`` 配置来创建额外的通道配置事务。
您可以重复本节中定义的过程，以在您的网络中建立其他通道。

我们将会通过 ``docker exec`` 命令来进入CLI容器：

.. code:: bash

        docker exec -it cli bash

如果成功的话，你会看到下面的东西：

.. code:: bash

        root@0d78bb69300d:/opt/gopath/src/github.com/hyperledger/fabric/peer#

如果你不想使用默认的 ``peer0.org1.example.com`` peer来运行CLI命令，
替换四个环境变量中的 ``peer0`` 或者 ``org1` 来运行指令:

.. code:: bash

    # PEER0的环境变量

    export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    export CORE_PEER_LOCALMSPID="Org1MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt


接下来，我们将把我们在 :ref:`createchanneltx` 部分
(我们称之为 ``channel.tx`` )中创建的通道配置事务构建作为创建通道请求的一部分传递给排序节点。

我们用 ``-c`` 标记指定通道名称，用 ``-f`` 标记指定通道配置事务。
在这种情况下，它就是 ``channel.tx`` 。
但是，您可以使用不同的名称挂载您自己的配置事务。
同样，我们将在CLI容器中设置 ``CHANNEL_NAME`` 环境变量，这样我们就不必显式传递这个参数。
通道名称必须是小写的，长度小于250个字符，并且匹配正则表达式 ``[a-z][a-z0-9.-]*``。

.. code:: bash

        export CHANNEL_NAME=mychannel

        # the channel.tx 构件
        # 因此，我们传递文件的完整路径
        # 我们还为排序节点的ca-cert传递路径，以验证TLS握手
        # 一定要适当地导出或替换$CHANNEL_NAME变量

        peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

.. note:: 注意我们在这个命令中作为一部分传递的 ``--cafile``。它是排序节点根证书的本地路径，让我们能够验证TLS握手。

这个命令返回一个创世区块 - ``<channel-ID.block>`` ——我们将使用它加入通道。
它包含 ``channel.tx`` 指定的配置信息。
如果您没有对默认的通道名称进行任何修改，那么命令将返回一个名为``mychannel.block``的原型。

.. note:: 您将在CLI容器中继续执行这些手动命令的其余部分。
          在面对``peer0.org1.example.com``以外的peer时，必须记住使用相应的环境变量为所有命令作为前置条件。

现在，让我们把 ``peer0.org1.example.com`` 加入通道。

.. code:: bash

        # 默认，这仅仅加入了 ``peer0.org1.example.com``
        # <channel-ID.block> 会由之前的命令会返回
        # 如果你没有改过通道名, 你会通过 mychannel.block 加入
        # 如果你是用不同的通道名加入, 那么传递合理命名的区块吧

         peer channel join -b mychannel.block

您可以通过对我们在上面的 :ref:`peerenvvars` 部分中使用的四个环境变量进行适当的更改，使其他对等点在必要时加入通道。


与其加入每个peer，我们只需加入 ``peer0.org2.example.com``，这样我们就可以适当地更新我们通道中的锚点peer定义。
要重写CLI容器里的默认环境变量，完整的命令如下所示:

.. code:: bash

  CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer channel join -b mychannel.block

或者，你可以选择单独设置每个环境变量，而不是传递整个字符串。
设置完成后，你只要再次运行 ``peer channel join`` 命令，CLI容器就会代表``peer0.org2.example.com`` 执行操作.

更新锚点peer
^^^^^^^^^^^^^^^^^^^^^^^

以下命令是通道更新，它们将传播到通道的定义上。
实际上，我们在通道的genesis块上添加了额外的配置信息。
注意，我们不是修改genesis块，而是简单地将增量（deltas）添加到将定义锚点peer的链中。

更新通道定义，将Org1的锚点peer定义为``peer0.org1.example.com``：
.. code:: bash

  peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

现在更新通道定义，将Org2的锚点peer定义为 ``peer0.org2.example.com``。
与Org2 peer的 ``peer channel join`` 命令相同，我们需要使用适当的环境变量作为这个调用的前置。

.. code:: bash

  CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org2MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

安装&实例化链码头
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note:: 我们将使用一个简单的现有链码。
          要了解如何编写自己的链码，请参阅 :doc:`chaincode4ade`。


应用程序通过 ``链码（chaincode）`` 与区块链账本交互。
因此，我们需要在每个将要执行和背书我们的事务的peer上安装链码，然后在通道上实例化链码。

首先，在四个peer节点之一上安装Go或Node.js的样例样例链码。这些命令将指定的源代码放到我们peer文件系统中

.. note:: 每个链码名称和版本只能安装一个版本的源代码。源代码存在于peer的文件系统的链码名称和版本上下文中;它是语言无关的。
          同样，实例化的链码容器将反映peer上安装的任何语言。

**Golang**

.. code:: bash

    # 这个安装Go的链码
    peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/

**Node.js**

.. code:: bash

    # 这个安装 Node.js 链码
    # 注意下 -l 标记; 我们使用它来表明所用语言
    peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node/


接下来，在通道上实例化链码。这将在通道上初始化链码，为链码设置背书策略，并为目标peer启动链码容器。
注意 ``-P`` 参数。这是我们的策略，在此策略中，我们针对要验证的链码指定事务所需的背书级别。

在下面的命令中，您会注意到我们将策略指定为``-P "AND ('Org1MSP.peer','Org2MSP.peer')"``。
这意味着我们需要来自Org1 ***和** Org2的peer的“背书”(即两个背书)。
如果我们将语法更改为``OR``，那么我们只需要一个背书。

**Golang**

.. code:: bash

    # 如果没有导出$CHANNEL_NAME环境变量，请确保替换它
    # 如果没有以mycc的名称安装链接代码，那么也要修改这个参数

    peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"

**Node.js**

.. note::  Node.js链码实例化大约需要一分钟。命令没有挂起;而是在镜像编译时安装fabric-shim层。

.. code:: bash

    # 确保替换了 $CHANNEL_NAME 环境变量，如果你之前未export它
    # 如果你以mycc的为名字安装链码，那么还需要修改那个参数
    # 注意我们必须在链码名字后面传递-l标记来表明语言

    peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -l node -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"

参阅 `endorsement
policies <http://hyperledger-fabric.readthedocs.io/en/latest/endorsement-policies.html>`__
文档来获取关于策略实现的更多细节。

如果您想要其他的peer与账本交互，那么您需要将它们连接到通道，并将相同名称、版本和语言的链码源码安装到适当peer的文件系统中。
当每个peer尝试与特定的链码交互时，就会为每个peer启动一个链码容器。
再次，要认识到这一个事实，即Node.js镜像编译相对比较慢。

在通道上实例化链码之后，我们可以放弃``l``标志。我们只需要传入通道标识符和链码的名称。

查询
^^^^^

让我们查询 ``a`` 的值，以确保正确地实例化了链码并填充了state DB。查询的语法如下:

.. code:: bash

  # 确保正确设置了 -C 和 -n 标记

  peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

调用
^^^^^^

现在让我们将 ``10`` 从 ``a`` 移到``b``。
这个事务将删除一个新的块并更新state DB。调用的语法如下:

.. code:: bash

    # 确保正确设置了 -C 和 -n 标记

    peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'

查询
^^^^^

让我们确认前面的调用是否正确执行。我们初始化了值为 `100`` 的键 ``a``，并在前面的调用中删除了 ``10``。
因此，对a的查询应该显示 ``90``。查询的语法如下所示。

.. code:: bash

  # 确保正确设置了 -C 和 -n 标记

  peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

我们应该看到下面的内容：

.. code:: bash

   Query Result: 90

你可以随意重新开始并操作键值对和随后的调用。

.. _behind-scenes:

该场景背后发生了什么?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note:: 这些步骤描述了由'./byfn.sh up'运行的``script.sh``脚本中的场景。
          用``./byfn.sh down``清理你的网络，并确保此命令是活跃的。
          然后使用相同的docker-compose提示再次启动网络。


-  一个脚本 - ``script.sh`` - 内嵌入了CLI容器。该脚本根据提供的通道名称驱动``createChannel``命令并使用channel.tx文件来配置通道。

-  ``createChannel`` 的输出是一个创世区块 -
   ``<your_channel_name>.block`` - 它被存储在peer的文件系统中，并包含了从channel.tx指定的通道配置

-  对所有四个peer执行 ``joinChannel`` 命令，该命令将前面生成的genesis块作为输入。
   该命令指示peer加入 ``<你的通道名称>`` 并创建一个以``<你的通道名称>.block``开始的链。

-  现在我们有了一个由四个peer和两个组织组成的渠道。
   这是我们的两份简历。

-  ``peer0.org1.example.com`` 和 ``peer1.org1.example.com`` 属于 Org1;
   ``peer0.org2.example.com`` 和 ``peer1.org2.example.com`` 属于 to Org2

-  这些关系是通过 ``crypto-config.yaml`` 定义的。MSP路径是在我们的docker compose中指定的。

然后在peer0.org2.example.com上“实例化”链码。实例化将链代码添加到通道中，启动目标对等点的容器，并初始化与链代码关联的键值对。这个示例的初始值是[" a "、" 100 "、" b "、" 200 "]。这个“实例化”导致一个名为dev-peer0.org2.example.com-mycc-1.0的容器启动。
实例化还传递支持策略的参数。策略被定义为-P "和('Org1MSP.peer'，'Org2MSP.peer')，这意味着任何事务都必须得到与Org1和Org2相关联的对等方的支持。
向peer0.org1.example.com发出针对“A”值的查询。之前在peer0. Org1 .example.com上安装了chaincode，因此这将启动一个名为dev-peer0.org1.example.com-mycc-1.0的Org1 peer0容器。查询的结果也会返回。没有发生写操作，因此对“a”的查询仍然返回值“100”。
调用被发送到peer0.org1.example.com，以将“10”从“a”移动到“b”

-  随后更新Org1MSP (``peer0.org1.example.com``)和Org2MSP (``peer0.org2.example.com``)的锚点。
   我们通过传递 ``Org1MSPanchors.tx`` 和 ``Org2MSPanchors.tx`` 给订购节点来实现，同时传过去的还有我们的通道名。

-  一个链码 - **chaincode_example02** - 被安装在``peer0.org1.example.com`` 和 ``peer0.org2.example.com`` 上

-  然后在 ``peer0.org2.example.com`` 上“实例化”链码。实例化将链码添加到通道中，启动目标peer的容器，并初始化与链码关联的键值对。
   这个示例的初始值是["a","100" "b","200"]。这个“实例化”启动了一个名为``dev-peer0.org2.example.com-mycc-1.0``的容器。

-  实例化还传递支持策略的参数。策略被定义为``-P "AND ('Org1MSP.peer','Org2MSP.peer')"``，这意味着任何事务都必须得到与Org1和Org2相关联的peer的背书。

-  向 ``peer0.org1.example.com`` 发出针对“A”值的查询。
   之前在 ``peer0.org1.example.com`` 安装了chaincode，因此这将为Org1 peer0启动一个名为 ``dev-peer0.org1.example.com-mycc-1.0`` 的容器。
   查询的结果也会返回。没有发生写操作，因此对“a”的查询仍然返回值“100”。

-  调用被发送到``peer0.org1.example.com``，以将“10”从“a”移动到“b”

-  然后在 ``peer1.org2.example.com`` 上安装链码

-  查询被发送到 ``peer1.org2.example.com`` 以获取“A”的值。这将启动名为 ``dev-peer1.org2.example.com-mycc-1.0`` 的第三个链码容器。
   返回一个值90，正确地反映了之前的事务，在该事务中键“A”的值被减少了10。

这说明了什么?
^^^^^^^^^^^^^^^^^^^^^^^^^^^

**必须** 在对等点上安装链码，才能成功地账本执行读/写操作。
此外，直到对链代码行 ``init`` 或传统事务(读/写)时(例如查询“a”的值)，链码容器才会为peer启动。事务导致容器启动。
此外，通道中的所有peer都维护一个完整的账本副本，该副本包括区块链，用于在块中存储不可变的、有顺序的记录，以及一个用于维护当前状态的快照的状态数据库。
这包括了那些没有在其上安装链码的peer(如上面示例中的``peer1.org1.example.com``)。
最后，chaincode在安装之后(如上面示例中的peer1.org2.example.com)就可以被访问了，因为它已经被实例化了。

我怎么看到这些事务?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

检查CLI Docker容器的日志。

.. code:: bash

        docker logs -f cli

你应该看到下面的输出：

.. code:: bash

      2017-05-16 17:08:01.366 UTC [msp] GetLocalMSP -> DEBU 004 Returning existing local MSP
      2017-05-16 17:08:01.366 UTC [msp] GetDefaultSigningIdentity -> DEBU 005 Obtaining default signing identity
      2017-05-16 17:08:01.366 UTC [msp/identity] Sign -> DEBU 006 Sign: plaintext: 0AB1070A6708031A0C08F1E3ECC80510...6D7963631A0A0A0571756572790A0161
      2017-05-16 17:08:01.367 UTC [msp/identity] Sign -> DEBU 007 Sign: digest: E61DB37F4E8B0D32C9FE10E3936BA9B8CD278FAA1F3320B08712164248285C54
      Query Result: 90
      2017-05-16 17:08:15.158 UTC [main] main -> INFO 008 Exiting.....
      ===================== Query successful on peer1.org2 on channel 'mychannel' =====================

      ===================== All GOOD, BYFN execution completed =====================


       _____   _   _   ____
      | ____| | \ | | |  _ \
      |  _|   |  \| | | | | |
      | |___  | |\  | | |_| |
      |_____| |_| \_| |____/

你可以滚动日志看到各种事务：

我怎么看到链码的日志?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

检查各个chaincode容器，以查看针对每个容器执行的独立事务。以下是每个容器的联合输出:

.. code:: bash

        $ docker logs dev-peer0.org2.example.com-mycc-1.0
        04:30:45.947 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
        ex02 Init
        Aval = 100, Bval = 200

        $ docker logs dev-peer0.org1.example.com-mycc-1.0
        04:31:10.569 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
        ex02 Invoke
        Query Response:{"Name":"a","Amount":"100"}
        ex02 Invoke
        Aval = 90, Bval = 210

        $ docker logs dev-peer1.org2.example.com-mycc-1.0
        04:31:30.420 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
        ex02 Invoke
        Query Response:{"Name":"a","Amount":"90"}

理解Docker Compose拓扑
-----------------------------------------

BYFN示例提供了两种Docker Compose文件，它们都是从``docker-compose-base.yaml``(位于 ``base`` 文件夹中)基础上扩展而来的。
我们的第一个方案, ``docker-compose-cli.yaml`` 为我们提供了一个CLI容器，以及一个排序节点，四个peer。我们在这一页的所有说明中都使用这个文件。

.. note:: 本节的其余部分将介绍为SDK设计的docker-compose文件。有关运行这些测试的详细信息，请参阅
          `Node SDK <https://github.com/hyperledger/fabric-sdk-node>`__

第二种方案, ``docker-compose-e2e.yaml``, 是构造来运用Node.js SDK以运行端到端测试。
除了与SDK一起工作外，它的主要区别是是有fabric-ca服务器的容器。
因此，我们能够将REST调用发送到组织CA以进行用户注册和注册。


如果您想使用``docker-compose-e2e.yaml``却不先运行byfn.sh脚本，然后我们需要做四个小的修改。
我们需要指向我们组织的CA的私钥。您可以在加密配置(crypto-config)文件夹中找到这些值。
例如，要找到Org1的私钥，我们将追寻以下路径- ``crypto-config/peerOrganizations/org1.example.com/ca/``。
私钥是一个后面跟着``_sk``的长哈希值。Org2的路径是 - ``crypto-config/peerOrganizations/org2.example.com/ca/``。

``docker-compose-e2e.yaml``为ca0和ca1更新 FABRIC_CA_SERVER_TLS_KEYFILE 变量。
您还需要编辑命令中提供的路径以启动ca服务器。您将两次为每个CA容器提供相同的私钥。

使用 CouchDB
-------------

状态数据库可以从默认值(goleveldb)切换到CouchDB。
使用CouchDB可以使用相同的chaincode函数，但是还添加了对状态数据库数据内容执行丰富复杂查询的功能，
前提是要将chaincode数据建模为JSON。

要使用CouchDB而不是默认数据库(goleveldb)，除了启动网络时传递``docker-compose-couch.yaml``外，
还需遵循前面描述的生成构件的相同过程:

.. code:: bash

    docker-compose -f docker-compose-cli.yaml -f docker-compose-couch.yaml up -d

**chaincode_example02** 应该能够在CouchDB支持下运行了。

.. note::  如果您选择实现fabric-couchdb容器端口到主机端口的映射，请确保您知道它的安全含义。
           开发环境中端口的映射使CouchDB REST API可用，并允许通过CouchDB web接口(Fauxton)来实现数据库可视化。
           为了限制对CouchDB容器的外部访问，生产环境可能不会实现端口映射。

您可以使用上面列出的步骤对CouchDB状态数据库使用 **chaincode_example02** 链码，
但是为了执行CouchDB查询功能，您将需要使用具有JSON建模数据的链码(例如 **marbles02**)。
您可以在 ``fabric/examples/chaincode/go`` 目录中找到 **marbles02** 链码。

我们将按照相同的过程来创建并加入 :ref:`createandjoin` 部分列出的通道。
一旦你把你的peer加入了通道，使用以下步骤与 **marbles02** 链码交互:

-  在 ``peer0.org1.example.com``上安装并且实例化链码：

.. code:: bash

       # 确保为实例化的命令相应地修改 $CHANNEL_NAME 变量
       peer chaincode install -n marbles -v 1.0 -p github.com/chaincode/marbles02/go
       peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org0MSP.peer','Org1MSP.peer')"

-  创建一些大理石（marbles）并且转移他们:

.. code:: bash

        # 确保相应地修改 $CHANNEL_NAME 变量

        peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble1","blue","35","tom"]}'
        peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble2","red","50","tom"]}'
        peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble3","blue","70","tom"]}'
        peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarble","marble2","jerry"]}'
        peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarblesBasedOnColor","blue","jerry"]}'
        peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["delete","marble1"]}'


-  如果您选择在docker-compose中映射CouchDB端口，
   现在您可以通过CouchDB web interface (Fauxton)查看状态数据库，方法是打开浏览器并导航到以下URL:

   ``http://localhost:5984/_utils``

你应该看见一个叫做 ``mychannel`` (或者你特有的通道名)的数据库，还有里面的文档。

.. note:: 对于下面的命令，确保恰当更新了 $CHANNEL_NAME 变量。

您可以从CLI中运行常规查询 (比如说 reading ``marble2``):

.. code:: bash

      peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["readMarble","marble2"]}'

输出应该展示了 ``marble2`` 的细节:

.. code:: bash

       Query Result: {"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}

你可以获得特定marble的历史 - 例如说 ``marble1``:

.. code:: bash

      peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["getHistoryForMarble","marble1"]}'

输出应该显示了 ``marble1`` 上的事务:

.. code:: bash

      Query Result: [{"TxId":"1c3d3caf124c89f91a4c0f353723ac736c58155325f02890adebaa15e16e6464", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"tom"}},{"TxId":"755d55c281889eaeebf405586f9e25d71d36eb3d35420af833a20a2f53a3eefd", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"jerry"}},{"TxId":"819451032d813dde6247f85e56a89262555e04f14788ee33e28b232eef36d98f", "Value":}]

您还可以对数据内容执行富查询，例如通过所有者``jerry``查询大理石（marble）字段:

.. code:: bash

      peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarblesByOwner","jerry"]}'

输出应该显示``jerry``拥有的两个大理石（marbles）：

.. code:: bash

       Query Result: [{"Key":"marble2", "Record":{"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}},{"Key":"marble3", "Record":{"color":"blue","docType":"marble","name":"marble3","owner":"jerry","size":70}}]


为什么要使用 CouchDB
-------------

CouchDB是一种NoSQL解决方案。它是一个面向文档的数据库，文档字段存储为键-值映射。
字段可以是简单的键-值对、列表或映射。除了LevelDB支持的键控/组合键/键范围查询（keyed/composite-key/key-range）之外，
CouchDB还支持完整的数据丰富查询功能，比如对整个区块链数据进行非键查询，因为它的数据内容以JSON格式存储，完全可以查询。
因此，CouchDB可以满足许多未被LevelDB支持的用例的链码、审计和报告需求。

CouchDB还可以增强区块链中的遵从性和数据保护的安全性。因为它能够通过过滤和屏蔽事务中的单个属性来实现字段级安全性（field-level security），
并且仅在需要时授权只读权限。

此外，CouchDB符合CAP定理的ap类型(可用性和分区容忍性)。它使用一个最终具有一致性的主控复制（master-master replication）模型。
更多信息可以在CouchDB文档的
`Eventual Consistency page of the CouchDB documentation <http://docs.couchdb.org/en/latest/intro/consistency.html>`__
页面找到。
然而，在每个fabric peer下，都没有数据库副本，对数据库的写入保证了一致性和持久性(而不是``最终一致性``)。

CouchDB是Fabric的第一个外部插件化状态数据库，可以也应该有其他外部数据库可选才对。
例如，IBM为其区块链启用了关系数据库。而可能也需要CP-type (一致性和分区容忍性)数据库，以便在没有应用层保证的情况下实现数据一致性。

关于数据持久化的说明
--------------------------

如果希望在peer容器或CouchDB容器上实现数据持久性，一种选择是将docker-host中的目录挂载到容器中的相关目录中。
例如，您可以在``docker-compose-base.yaml`` 文件中peer容器规范里添加以下两行：

.. code:: bash

       volumes:
        - /var/hyperledger/peer0:/var/hyperledger/production

对于CouchDB容器，您可以在CouchDB容器规范中添加以下两行：

.. code:: bash

       volumes:
        - /var/hyperledger/couchdb0:/opt/couchdb/data

.. _Troubleshoot:

故障排除
---------------

-  总是以全新的方式开始你的网络。使用以下命令删除构件、密码、容器和链码容器:

   .. code:: bash

      ./byfn.sh down

   .. note:: 如果你没删除旧的容器和镜像，**就会**遇到错误

-

   如果你看到Docker错误，首先检查你的Docker版本(:doc:`prereqs`)，
   然后尝试重新启动Docker进程。Docker的问题通常不能被立即辨认出来。
   例如，您可能会看到由于无法访问安装在容器中的加密材料（crypto material）而导致的错误。

   如果你简直从零开始地移除你的镜像:

   .. code:: bash

       docker rm -f $(docker ps -aq)
       docker rmi -f $(docker images -q)

-  如果在创建、实例化、调用或查询命令时看到错误，请确保正确更新了通道名称和链码名称。
   在提供的示例命令中有占位符值。

-  如果你看到下面的错误:

   .. code:: bash

       Error: Error endorsing chaincode: rpc error: code = 2 desc = Error installing chaincode code mycc:1.0(chaincode /var/hyperledger/production/chaincodes/mycc.1.0 exits)

   您可能有以前运行的链接代码映像(例如 ``dev-peer1.org2.example.com-mycc-1.0`` 或 ``dev-peer0.org1.example.com-mycc-1.0`` )。删除它们，再试一次。

   .. code:: bash

       docker rmi -f $(docker images | grep peer[0-9]-peer[0-9] | awk '{print $3}')

-  如果你看到了类似下面的东西：

   .. code:: bash

      Error connecting: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure
      Error: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure

  请确保您的网络是针对“1.0.0”映像运行的，这些映像已被重新标记为“最新”。

-  如果你看到了下面的错误：

   .. code:: bash

     [configtx/tool/localconfig] Load -> CRIT 002 Error reading configuration: Unsupported Config Type ""
     panic: Error reading configuration: Unsupported Config Type ""

   那么你没有正确设置 ``FABRIC_CFG_PATH`` 环境变量。configtxgen工具需要这个变量来定位config .yaml。
   返回并执行``export FABRIC_CFG_PATH=$PWD``，然后重新创建通道工件。

-  如要清理网络，使用 ``down`` 选项:

   .. code:: bash

       ./byfn.sh down

-  如果发现声明仍然有“活动端点（active endpoints）”的错误，则删除你的Docker网络。
   这将清除您以前的网络，并以新的环境运行。

   .. code:: bash

        docker network prune

   你会看到下面的信息:

   .. code:: bash

      WARNING! This will remove all networks not used by at least one container.
      Are you sure you want to continue? [y/N]

   选择 ``y``.

-  如果你看到类似下面的错误：

   .. code:: bash

      /bin/bash: ./scripts/script.sh: /bin/bash^M: bad interpreter: No such file or directory

   确保有问题的文件(本例中就是 **script.sh**）使用了Unix格式编码。
   这很可能是由于在你的Git配置中设置``core.autocrlf`` 为 ``false``导致的(参阅 :ref:`windows-extras`)。
   有几种方法可以解决这个问题。比如如果你可以访问vim编辑器，打开文件:

   .. code:: bash

      vim ./fabric-samples/first-network/scripts/script.sh

   然后执行下面的vim命令来改变格式：

   .. code:: bash

      :set ff=unix

.. note:: 如果你还是看到错误，请将你的日志分享到
          `Hyperledger Rocket Chat <https://chat.hyperledger.org/home>`__ 或者
          `StackOverflow <https://stackoverflow.com/questions/tagged/hyperledger-fabric>`__ 上的
          **fabric-questions** 频道。

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
