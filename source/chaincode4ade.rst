开发人员的链码
========================

链码（Chaincode）是什么？
------------------

链码是一个程序，用`Go <https://golang.org>`_, `node.js <https://nodejs.org>`_
编写，实现一个规定的接口。最终，将支持其他编程语言，如Java。
链码运行在安全的Docker容器中，与背书peer进程隔离开来。
链码通过应用程序提交的事务初始化和管理分类帐状态。

链码通常处理网络成员同意的业务逻辑，因此类似于“智能合约”。
可以调用链码来更新或查询交易提案中的分类帐。
给定适当的权限，链码可以调用另一个链码（在同一通道或不同通道）来访问其状态。
注意，如果被调用链码位于与调用链码不同的信道上，则只允许读取查询。
也就是说，不同通道上的被调用链码只是一个 `Query`，它不参与后续提交阶段的状态验证检查。

在下面的部分中，我们将通过应用程序开发人员的眼睛来探索链码。
我们将介绍一个简单的链码示例应用程序，并通览下链码Shim API中的每个方法的目的。

链码 API
-------------

所有的链码程序必须实现 ``Chaincode interface``:

  - `Go <https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#Chaincode>`__
  - `node.js <https://fabric-shim.github.io/ChaincodeInterface.html>`__

其方法被调用来接收交易。特别地，当链码接收到 ``instantiate`` 或 ``upgrade`` 交易时，调用 ``Init`` 方法，
以便链码可以执行任何必要的初始化，包括应用程序状态的初始化。
当接收``invoke``交易以处理交易提案时，调用``Invoke`` 方法。

链表“SHIM”API的另一个接口是 ``ChaincodeStubInterface``：

  - `Go <https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStubInterface>`__
  - `node.js <https://fabric-shim.github.io/ChaincodeStub.html>`__

用于访问和修改分类帐，并在链码之间进行调用。

在本教程中，我们将通过实现一个管理简单“资产”的简单链码应用程序来演示这些API的使用。

.. _Simple Asset Chaincode:

简单资产链码
----------------------

我们的应用程序是一个基本的链码，用于创建分类帐上的资产（键值对）。

选择代码的位置
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如果您没有用Go进行过编程，那么您可能希望确保安装了 :ref:`Golang` 并正确配置了系统。

现在，您将希望为你的链码程序创建一个目录，其作为 ``$GOPATH/src/`` 的子目录。

为了让事情简单一些，我们使用下面的命令：

.. code:: bash

  mkdir -p $GOPATH/src/sacc && cd $GOPATH/src/sacc

现在，让我们创建源文件，用来存放我们的代码：

.. code:: bash

  touch sacc.go

家务（Housekeeping）
^^^^^^^^^^^^

首先，让我们开始做一些家务。
与其他链码一样，它实现了
`Chaincode interface <https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#Chaincode>`_
，特别是 ``Init`` 和 ``Invoke`` 函数。
因此，让我们为我们的链码添加必要依赖项的GO导入语句。
我们将导入链码shim包和
`peer protobuf package <https://godoc.org/github.com/hyperledger/fabric/protos/peer>`_.
包。接下来，让我们添加``SimpleAsset`` 结构来作为链码shim函数的接收器。

.. code:: go

    package main

    import (
    	"fmt"

    	"github.com/hyperledger/fabric/core/chaincode/shim"
    	"github.com/hyperledger/fabric/protos/peer"
    )

    // SimpleAsset implements a simple chaincode to manage an asset
    type SimpleAsset struct {
    }

初始化链码
^^^^^^^^^^^^^^^^^^^^^^^^^^

下面，我们实现 ``Init`` 函数。

.. code:: go

  // Init is called during chaincode instantiation to initialize any data.
  func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {

  }

.. note:: 注意，链码升级也调用这个函数。当编写一个升级现有代码的链码时，请确保适当修改 ``Init`` 函数。
          特别地，如果没有“迁移”或作为升级的一部分没有要初始化的内容，则提供一个空的“Init”方法。

接下来，我们将使用
`ChaincodeStubInterface.GetStringArgs <https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.GetStringArgs>`_
函数检索 ``Init`` 调用的参数，并检查其有效性。
在我们的例子中，我们需要一个键值对。

  .. code:: go

    // Init is called during chaincode instantiation to initialize any
    // data. Note that chaincode upgrade also calls this function to reset
    // or to migrate data, so be careful to avoid a scenario where you
    // inadvertently clobber your ledger's data!
    func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
      // Get the args from the transaction proposal
      args := stub.GetStringArgs()
      if len(args) != 2 {
        return shim.Error("Incorrect arguments. Expecting a key and a value")
      }
    }

接下来，既然我们已经确定调用是有效的，我们将把初始状态存储在分类帐中。
要做到这一点，我们将调用
`ChaincodeStubInterface.PutState <https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.PutState>`_
作为传入参数的键值和值。假设一切顺利，将返回一个peer。响应对象指示初始化是成功的。

.. code:: go

  // Init is called during chaincode instantiation to initialize any
  // data. Note that chaincode upgrade also calls this function to reset
  // or to migrate data, so be careful to avoid a scenario where you
  // inadvertently clobber your ledger's data!
  func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
    // Get the args from the transaction proposal
    args := stub.GetStringArgs()
    if len(args) != 2 {
      return shim.Error("Incorrect arguments. Expecting a key and a value")
    }

    // Set up any variables or assets here by calling stub.PutState()

    // We store the key and the value on the ledger
    err := stub.PutState(args[0], []byte(args[1]))
    if err != nil {
      return shim.Error(fmt.Sprintf("Failed to create asset: %s", args[0]))
    }
    return shim.Success(nil)
  }

调用链码
^^^^^^^^^^^^^^^^^^^^^^

首先，让我们添加 ``Invoke` 函数的签名。

.. code:: go

    // Invoke is called per transaction on the chaincode. Each transaction is
    // either a 'get' or a 'set' on the asset created by Init function. The 'set'
    // method may create a new asset by specifying a new key-value pair.
    func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {

    }

与上面的 ``Init`` 函数一样，我们需要从 ``ChaincodeStubInterface`` 提取参数。
``Invoke`` 函数的参数将是调用的链码应用程序函数的名称。
在我们的示例中，我们的应用程序将仅仅具有两个函数：``set`` 和 ``get``，这两个函数允许设置资产的值或检索资产的当前状态。
我们首先调用
`ChaincodeStubInterface.GetFunctionAndParameters <https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.GetFunctionAndParameters>`_
来提取该链码应用程序函数的函数名和参数。

.. code:: go

    // Invoke is called per transaction on the chaincode. Each transaction is
    // either a 'get' or a 'set' on the asset created by Init function. The Set
    // method may create a new asset by specifying a new key-value pair.
    func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
    	// Extract the function and args from the transaction proposal
    	fn, args := stub.GetFunctionAndParameters()

    }

接下来，我们将验证函数名是否被 ``set`` 或 ``get``，并调用这些链码应用程序函数，
通过 ``shim.Success`` 或 ``shim.Error`` 函数返回适当的响应，
这些函数将响应序列化为gRPC protobuf消息。

.. code:: go

    // Invoke is called per transaction on the chaincode. Each transaction is
    // either a 'get' or a 'set' on the asset created by Init function. The Set
    // method may create a new asset by specifying a new key-value pair.
    func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
    	// Extract the function and args from the transaction proposal
    	fn, args := stub.GetFunctionAndParameters()

    	var result string
    	var err error
    	if fn == "set" {
    		result, err = set(stub, args)
    	} else {
    		result, err = get(stub, args)
    	}
    	if err != nil {
    		return shim.Error(err.Error())
    	}

    	// Return the result as success payload
    	return shim.Success([]byte(result))
    }

实现链码应用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如前所述，我们的链码应用程序实现了两个可以通过 ``Invoke`` 函数调用的函数。现在让我们实现这些功能。
注意，如前所述，为了访问分类账的状态，我们将利用链码 shim API中的
`ChaincodeStubInterface.PutState <https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.PutState>`_
和
`ChaincodeStubInterface.GetState <https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.GetState>`_
函数。

.. code:: go

    // Set stores the asset (both key and value) on the ledger. If the key exists,
    // it will override the value with the new one
    func set(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    	if len(args) != 2 {
    		return "", fmt.Errorf("Incorrect arguments. Expecting a key and a value")
    	}

    	err := stub.PutState(args[0], []byte(args[1]))
    	if err != nil {
    		return "", fmt.Errorf("Failed to set asset: %s", args[0])
    	}
    	return args[1], nil
    }

    // Get returns the value of the specified asset key
    func get(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    	if len(args) != 1 {
    		return "", fmt.Errorf("Incorrect arguments. Expecting a key")
    	}

    	value, err := stub.GetState(args[0])
    	if err != nil {
    		return "", fmt.Errorf("Failed to get asset: %s with error: %s", args[0], err)
    	}
    	if value == nil {
    		return "", fmt.Errorf("Asset not found: %s", args[0])
    	}
    	return string(value), nil
    }

.. _Chaincode Sample:

把一切合在一起
^^^^^^^^^^^^^^^^^^^^^^^

最后，我们需要添加 ``main`` 函数，它将调用
`shim.Start <https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#Start>`_
函数。下面是完整的链码程序源码。

.. code:: go

    package main

    import (
    	"fmt"

    	"github.com/hyperledger/fabric/core/chaincode/shim"
    	"github.com/hyperledger/fabric/protos/peer"
    )

    // SimpleAsset implements a simple chaincode to manage an asset
    type SimpleAsset struct {
    }

    // Init is called during chaincode instantiation to initialize any
    // data. Note that chaincode upgrade also calls this function to reset
    // or to migrate data.
    func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
    	// Get the args from the transaction proposal
    	args := stub.GetStringArgs()
    	if len(args) != 2 {
    		return shim.Error("Incorrect arguments. Expecting a key and a value")
    	}

    	// Set up any variables or assets here by calling stub.PutState()

    	// We store the key and the value on the ledger
    	err := stub.PutState(args[0], []byte(args[1]))
    	if err != nil {
    		return shim.Error(fmt.Sprintf("Failed to create asset: %s", args[0]))
    	}
    	return shim.Success(nil)
    }

    // Invoke is called per transaction on the chaincode. Each transaction is
    // either a 'get' or a 'set' on the asset created by Init function. The Set
    // method may create a new asset by specifying a new key-value pair.
    func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
    	// Extract the function and args from the transaction proposal
    	fn, args := stub.GetFunctionAndParameters()

    	var result string
    	var err error
    	if fn == "set" {
    		result, err = set(stub, args)
    	} else { // assume 'get' even if fn is nil
    		result, err = get(stub, args)
    	}
    	if err != nil {
    		return shim.Error(err.Error())
    	}

    	// Return the result as success payload
    	return shim.Success([]byte(result))
    }

    // Set stores the asset (both key and value) on the ledger. If the key exists,
    // it will override the value with the new one
    func set(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    	if len(args) != 2 {
    		return "", fmt.Errorf("Incorrect arguments. Expecting a key and a value")
    	}

    	err := stub.PutState(args[0], []byte(args[1]))
    	if err != nil {
    		return "", fmt.Errorf("Failed to set asset: %s", args[0])
    	}
    	return args[1], nil
    }

    // Get returns the value of the specified asset key
    func get(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    	if len(args) != 1 {
    		return "", fmt.Errorf("Incorrect arguments. Expecting a key")
    	}

    	value, err := stub.GetState(args[0])
    	if err != nil {
    		return "", fmt.Errorf("Failed to get asset: %s with error: %s", args[0], err)
    	}
    	if value == nil {
    		return "", fmt.Errorf("Asset not found: %s", args[0])
    	}
    	return string(value), nil
    }

    // main function starts up the chaincode in the container during instantiate
    func main() {
    	if err := shim.Start(new(SimpleAsset)); err != nil {
    		fmt.Printf("Error starting SimpleAsset chaincode: %s", err)
    	}
    }

创建链码
^^^^^^^^^^^^^^^^^^

现在让我们编译你的链码。

.. code:: bash

  go get -u github.com/hyperledger/fabric/core/chaincode/shim
  go build

假设没有错误，现在我们可以进入下一步，测试您的链码。

使用DEV模式测试
^^^^^^^^^^^^^^^^^^^^^^
通常链表是由peer启动和维护的。然而，在“DEV模式”中，链码是由用户构建和启动的。
这种模式在链码开发阶段是有用的，用于快速代码/构建/运行/调试循环周转。

我们利用为样本dev网络预生成的排序和通道构件，来开始使用“DEV模式”。
这样一来，用户可以立即开始编译链码和驱动。

安装Hyperledger Fabric例子
----------------------------------

如果你还没这么做的话，请I :doc:`install`.

导航到 ``fabric-samples`` 的 ``chaincode-docker-devmode``目录：

.. code:: bash

  cd chaincode-docker-devmode

现在打开三个终端并导航到您的 ``chaincode-docker-devmode`` 目录中。

终端 1 - 开启网络
------------------------------

.. code:: bash

    docker-compose -f docker-compose-simple.yaml up

上面用 ``SingleSampleMSPSolo`` 排序配置文件启动网络，并在“DEV模式”中启动peer。
它还启动两个额外的容器——一个用于链码环境，一个CLI容器用于与链码交互。
创建和加入通道的命令被嵌入了CLI容器中，因此我们可以立即跳转到链码调用。

终端 2 - 创建&启动链码
----------------------------------------

.. code:: bash

  docker exec -it chaincode bash

你应该看到下面的东西:

.. code:: bash

  root@d2629980e76b:/opt/gopath/src/chaincode#

现在，编译你的链码:

.. code:: bash

  cd sacc
  go build

现在，运行链码:

.. code:: bash

  CORE_PEER_ADDRESS=peer:7052 CORE_CHAINCODE_ID_NAME=mycc:0 ./sacc

链码从peer和链码日志开始，这些日志表明与peer成功注册。
注意，在这个阶段，链码与任何通道不相关。这是在使用 ``instantiate`` 命令的后续步骤中完成的。

终端 3 - 使用链码
------------------------------

Even though you are in ``--peer-chaincodedev`` mode, you still have to install the
chaincode so the life-cycle system chaincode can go through its checks normally.
This requirement may be removed in future when in ``--peer-chaincodedev`` mode.

We'll leverage the CLI container to drive these calls.

即使您处于 ``--peer-chaincodedev`` 模式，您仍然必须安装链码，以便生命周期系统链码可以正常地通过检查。
这种需求可能会在未来的 ``--peer-chaincodedev`` 式中移除。

我们将利用CLI容器来驱动这些调用。

.. code:: bash

  docker exec -it cli bash

.. code:: bash

  peer chaincode install -p chaincodedev/chaincode/sacc -n mycc -v 0
  peer chaincode instantiate -n mycc -v 0 -c '{"Args":["a","10"]}' -C myc

现在发出一个调用，将“a”的值改为“20”。

.. code:: bash

  peer chaincode invoke -n mycc -c '{"Args":["set", "a", "20"]}' -C myc

最后，查询 ``a``，我们应该看到一个 ``20`` 的值。

.. code:: bash

  peer chaincode query -n mycc -c '{"Args":["query","a"]}' -C myc

测试新链码
---------------------

默认情况下，我们只安装 ``sacc``。
但是，您可以通过将它们添加到 ``chaincode`` 子目录，并重新启动网络来轻松地测试不同的链接代码。
在这一点上，它们可以在您的 ``chaincode`` 容器中访问。

链码加密
--------------------

在某些场景中，加密与密钥相关联的值是完全有用的，或者仅仅是部分地加密。
例如，如果某人的社会保险号码或地址被写入分类账，那么您可能不希望该数据以明文显示。
链码加密是通过利用
`entities extension <https://github.com/hyperledger/fabric/tree/master/core/chaincode/shim/ext/entities>`__
来实现的，实体扩展（Entities extension）是具有商品工厂和功能的BCCSP包装器，用于执行加密和椭圆曲线数字签名等加密操作。
例如，为了加密，链码的调用程序通过瞬态字段（transient field）在密码密钥中传递。
然后，相同的密钥可用于随后的查询操作，从而允许对加密的状态值进行适当的解密。

想了解更多信息和示例，请参见 ``fabric/examples`` 目录中的
`Encc Example <https://github.com/hyperledger/fabric/tree/master/examples/chaincode/go/enccc_example>`__
示例。
特别注意` `utils.go`` 辅助程序。这个实用程序加载链码shim API和实体扩展，
并构建一类新的函数（例如，`encryptAndPutState`` & ``getStateAndDecrypt``），这些函数之后将由由示例加密链码利用。
因此，链码现在可以将shim API中 ``Get`` 和 ``Put`` ，与额外的功能函数 ``Encrypt`` and ``Decrypt``相结合使用。



管理用Go写的链码的外部依赖
----------------------------------------------------------
如果您的链码需要Go标准库不提供的包，则需要将这些包与链码一起包含进来。
有许多工具可用于管理（或“销货[vendoring]”）这些依赖关系。下面演示如何使用 ``govendor``：

.. code:: bash

  govendor init
  govendor add +external  // Add all external package, or
  govendor add github.com/external/pkg // Add specific external package

这将外部依赖项导入本地 ``vendor`` 目录。
``peer chaincode package`` 和 ``peer chaincode install`` 操作随后将与依赖项关联的代码包括在chaincode包中。

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
