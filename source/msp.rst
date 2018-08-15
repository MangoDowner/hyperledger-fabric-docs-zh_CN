成员服务提供商 (MSP)
==================================

该文档提供了MSP设置和最佳实践的详细信息。

成员服务提供商(MSP)是一个组件，其旨在提供成员操作架构的抽象。

特别是，MSP抽象了颁发和验证证书及用户身份验证背后所有的加密机制和协议。
一个MSP可以定义自己的身份概念，以及这些身份的管理规则（身份验证）和认证(签名生成和验证)。

Hyperledger Fabric区块链网络可以由一个或多个MSP控制。
这提供了成员操作的模块化，以及不同的成员标准和架构之间的互操作性。

在本问的其他们部分，我们将详细介绍Hyperledger Fabric支持的MSP实现，以及它的最佳使用实践。

MSP配置
-----------------

如要启动一个MSP的实例，需要在每一个peer和排序节点上给予特定的配置（以启用peer和排序节点签名）；
还要在通道上设置来启用peer，排序节点，客户端身份验证，以及所有通道成员各自的签名验证（认证）。

首先，每个MSP需要指定一个名称，这样就可以在网络中引用该MSP(比如说，``msp1``, ``org2``, 还有 ``org3.divA``)。
这是一个用来在通道里被引用的名字，其满足了MSP成员规则，代表一个财团（consortium），组织或者组织分割。
这也被称为*MSP 标识符* 或者 *MSP ID*。MSP标识符对于每个MSP实例来说都是唯一的。
比如说，如果在系统通道gensis发现两个MSP有着一样的标志符，排序节点就会启动失败。

在默认MSP实现里，需要指定一系列参数来启用身份（证书）验证和签名验证。  这些参数由
`RFC5280 <http://www.ietf.org/rfc/rfc5280.txt>`_ 推导出来, 并且包括:

- 一个自签名 (X.509) 证书列表来构成 * 信任根基（root of trust） *

一个X.509证书列表，用于表示该提供者考虑用于证书验证的中间CAs


- 一个 X.509 证书列表用于代表中间CA,该提供商考虑将这些中间CA用于证书验证;
  这些证书应该由具体的某个根证书签发;中间证书是可选的参数。
- 一个拥有可验证证书路径的 X.509 证书列表，该路径指向了某个具体的根证书以代表MSP的管理员。
  该证书的拥有者有权利去发起改变MSP配置（比如根CA和中间CA）的请求。
- 一个组织单元列表,MSP的有效成员都应该在他们的X.509证书里包含该列表;
  这是一个可选的配置参数，只在一些场景下使用，
  比如：多个组织使用同一信任根基和中间CA，并为他们的成员保留了一个OU字段。
- 一个撤销证书列表（CRL），每个CRL恰好对应于某个所列出的(中间或根)MSP证书颁发机构； 这是一个可选参数。
- 一个自签名 (X.509) 证书列表，来构成TLS证书的 **TLS 信任根基（root of
  trust）** 。
- 一个 X.509 证书列表用于代表中间TLS CA，该提供商考虑将这些中间CA用于证书验证；
  这些证书应该由具体的某个TLS根证书签发；中间证书是可选参数。

*验证* MSP实例的身份需要满足下面的条件：

- 他们以 X.509 证书的形式存在，具有可验证的证书路径，该路径指向某个根证书；
- 他们不在CRL里;
- 并且他们在 X.509 证书结构的 ``OU`` 字段中 *列出* 了MSP配置的一个或多个组织单元。


要获取更多关于当前MSP实现下身份验证的内容，我们推荐阅读 :doc:`msp-identity-validity-rules`。

除了验证相关参数外，如果要为MSP启用签名或者验证的节点，我们需要指定：

- 用于节点签名的签名密钥(当前只支持ECDSA密钥), 还有
- 节点的 X.509 证书, 该证书需是是MSP的验证参数下一个合法的身份。

值得注意的是，MSP身份永不过期，他们只能通过被添加到合法的CRL里来实现撤销。
此外，目前还不支持强行撤销TLS证书。

如果生成MSP证书和他们的签名密钥？
--------------------------------------------------------

为了生成 X.509 证书以提供MSP配置，应用可以使用  `Openssl <https://www.openssl.org/>`_.
我们强调下，Hyperledger Fabric不支持包含RSA密钥的证书。

作为替代，我们可以使用 ``cryptogen`` 工具，它的操作说明见 :doc:`getting_started`。

`Hyperledger Fabric CA <http://hyperledger-fabric-ca.readthedocs.io/en/latest/>`_
也可以用来生成迷药和证书以配置MSP。

在peer&排序节点这里启动MSP
------------------------------------

为了设置一个本地的MSP（为了给peer或者排序节点），管理员应该创建一个文件夹(比如说 ``$MY_PATH/mspconfig``，
其包含了6个子文件夹和1个文件：

1. 一个 ``admincerts`` 文件夹用于包含与管理员证书相对应的 PEM 文件
2. 一个 ``cacerts`` 文件夹用于对应根CA证书的 PEM 文件
3. (可选) 一个 ``intermediatecerts`` 文件夹用于包含与中间CA证书相对应的PEM文件
4. (可选) 一个 ``config.yaml`` 文件来配置支持的组织单元还有身份分类 (阅读下面对应的部分)
5. (可选) 一个 ``crls`` 文件夹来包含CRL
6. 一个 ``keystore`` 文件夹来包含带有节点签名密钥的PEM文件;
   强调下，目前RSA密钥不支持
7. 一个 ``signcerts`` 文件夹来包含带有节点X.509证书的 PEM 文件
8. (可选) 一个 ``tlscacerts`` 文件夹来包含与TLS根CA证书对应的 PEM 文件
9. (可选) 一个 ``tlsintermediatecerts`` 文件夹来包含与LS中间CA证书相对应的PEM文件

在节点(peer的core.yaml,还有排序节点的orderer.yaml)的配置文件里，需要指定到mspconfig文件夹的路径以及节点MSP的MSP标志符。
到mspconfig文件夹的路径应该是相对于FABRIC_CFG_PATH的，
并作为参数 ``mspConfigPath`` 的值提供给peer，作为 ``LocalMSPDir``的值提供给排序节点。
节点MSP的标志符可以作为参数 ``localMspId`` 值提供给peer，作为 ``LocalMSPID``提供给排序节点。
这些变量可以通过环境使用peer的核心前缀(例如CORE_PEER_LOCALMSPID)和排序节点的ORDERER前缀(例如ORDERER_GENERAL_LOCALMSPID)来重写。
注意，对于排序节点设置，需要生成并向排序节点提供系统通道的genesis块。下一节将详细介绍此块的MSP配置需求。

*重新配置*  "本地" MSP只能通过手动进行，而且要求重启peer或者排序节点进程。
在随后的版本里，我们目前是提供在线/动态重新配置（即不需要通过使用节点管理的系统链码来终止该节点）

组织单元
--------------------

为了配置组织单元列表，这些列表会被MSP中合法的成员在其X.509 证书中囊括,
``config.yaml`` 文件需要指明组织单元描述符。 这里就是一个例子:

::

   OrganizationalUnitIdentifiers:
     - Certificate: "cacerts/cacert1.pem"
       OrganizationalUnitIdentifier: "commercial"
     - Certificate: "cacerts/cacert2.pem"
       OrganizationalUnitIdentifier: "administrators"

上面的例子申明了两个组织单元描述符： **commercial** 还有 **administrators**。
如果一个MSP描述符带有至少一个组织单元描述符，它便是合法的。
``Certificate`` 字段指向CA或者中间CA证书路径，在该路径下，具有该特定OU的身份应该是合法的。
路径是相对MSP根目录而言的，而且不能为空。

身份分类
-----------------------

默认的MSP实现允许根据身份的x509证书，进一步将其划分为client和peer。
一个身份应该被分类为 **client**，如果它进行递交（submit）交易，请求peer等操作。
一个身份应该被分类为 **peer** 如果它进行背书或者提交（commit）交易等操作。
为了给指定MSP定义client和peer，需要正确设置 ``config.yaml`` 文件。下面就是例子：

::

   NodeOUs:
     Enable: true
     ClientOUIdentifier:
       Certificate: "cacerts/cacert.pem"
       OrganizationalUnitIdentifier: "client"
     PeerOUIdentifier:
       Certificate: "cacerts/cacert.pem"
       OrganizationalUnitIdentifier: "peer"

如上所示,  ``NodeOUs.Enable`` 被设置为 ``true``, 这就开启了身份分类。
然后，client (peer) 标志符可以通过为`NodeOUs.ClientOUIdentifier`` (``NodeOUs.PeerOUIdentifier``) 密钥设定下面的属性来定义：

a. ``OrganizationalUnitIdentifier``: 将其设置为与client(peer)的x509证书应该包含的OU相匹配的值。
b. ``Certificate``: 将其设置为用来验证client（peer）身份的CA或者中间CA。 这个字段是相对与MSP根目录的。
   它可以是空的，这意味着身份的x509证书可以在MSP配置中定义的任何CA下进行验证。

当启用分类时，MSP管理员需要成为该MSP的client，这意味着他们的x509证书需要携带识别client的OU。
还要注意，身份可以是client，也可以是peer。这两种分类相互排斥。如果标识既不是客户端也不是对等端，则验证将失败。

最后请注意，对于升级的环境，在使用标识分类之前，需要启用1.1通道功能。

设置通道 MSP
-----------------

在系统创建时，需要指定网络中出现的所有MSP的验证参数，并将其包含在系统通道的genesis块中。
回想一下，MSP验证参数包括MSP标识符、信任证书的根、中间CA和管理证书以及OU规范和CRL。
系统genesis块在排序节点的设置阶段提供给排序节点，并允许他们验证通道创建请求。
如果系统genesis块包含两个具有相同标识符的MSP，则排序节点将拒绝系统genesis块，从而导致网络引导失败。

对于应用程序通道，只有管理通道的MSP的验证组件需要驻留在通道的genesis块中。
我们强调，**应用程序有责任** 在指示一个或多个peer加入通道之前，
确保通道的genesis块(或最近的配置块)中包含了正确的MSP配置信息。

在使用configtxgen工具引导通道时，可以在mspconfig文件夹中包含MSP的验证参数，
并在 ``configtx.yaml`` 的相关部分里设置通道MSP。

通道上MSP的**重新配置**，包括与该MSP的CA关联的CRL（证书撤销列表）的公告，
都是MSP的一个管理员证书的所有者通过创建 ``config_update`` 对象来实现的。
然后，管理员管理的client应用程序将向出现MSP的通道宣布这个更新。

最佳实践
--------------

在本节中，我们将详细介绍在常见场景中MSP配置的最佳实践。

**1) 在组织/公司（organizations/corporations）和MSP之间进行映射**

我们建议在组织和MSP之间存在一对一的映射。如果选择了不同类型的映射，则需要考虑以下问题:

- **一个组织使用多个MSP。**这对应于一个组织的情况，该组织包括了由其MSP代表的各种部门，
  之所以除了分割，或者出于管理独立的原因，或者出于隐私的原因。
  在这种情况下，一个对等点只能由单个MSP拥有，并且不会将具有来自其他MSP身份的对等点识别为同一组织。
  这意味着，对等点可以通过gossip，来与属于同一细分下的对等点共享组织范围内的数据，
  而不是与构成实际组织的全部提供者共享。
- **多个组织使用一个MSP。** 这对应多个组织联合体的情况，这些组织由类似的成员架构管理。
  这里需要知道的是，peer将组织作用域（organization-scoped）的消息传播给具有相同MSP身份的peer，而不管它们是否属于相同的实际组织。
  这是对于MSP定义力度以及对等点配置的限制。

**2) 一个组织有不同的部门(比如组织单位)， **
**它希望授予这些部门访问不同渠道的权利。**

有两种方法来处理这个：

- **定义一个MSP以容纳所有组织成员的成员资格**。
  该MSP的配置将包括根CA、中间CA和管理证书的列表;成员身份将包括成员所属的组织单位 (``OU``)。
  然后可以定义策略来获取特定 ``OU`` 的成员，这些策略可能构成通道的读/写策略或链码的背书策略。
  这种方法的一个限制是，gossip pee 会将在其本地MSP下具有成员身份的peer视为同一组织的成员，
  因此会与他们讨论组织范围内的数据(例如他们的状态)。
- **为每个部门定义一个MSP**.
  这将为每个分区指定一组证书，它们是根CAs、中间CAs和管理证书的集合，
  这样MSP之间就不会有重叠的认证路径。这将意味着，比如说，每个部门使用不同的中间CA。
  这里的缺点是管理多个MSP而不是一个，但是这回避了前面方法中出现的问题。
  还可以通过利用MSP配置的OU扩展为每个分部定义一个MSP。

**3) 分开同个组织中的client和peer**

在许多情况下，需要从identity本身检索identity的“类型”
(例如，可能需要保证背书是由peer派生的，而不是client或节点仅充当排序节点的角色)。

对这种要求的支持有限。

实现这种分离的一种方法是为每个节点类型创建一个独立的中间CA——一个用于client，一个用于peer/排序节点;
配置两个不同的MSP——一个用于client，另一个用于peer/排序节点。
这个组织应该访问的通道需要包含两个MSP，而背书策略将使用指向peer的MSP。
这最终会导致组织被映射到两个MSP实例，并且会对peer和client交互的方式产生一定的影响。

gossip不会受到很大的影响，因为同一组织的所有peer仍然属于一个MSP。
peer可以根据本地MSP的策略来限制某些系统链码的执行。
例如，如果请求是由其本地MSP的管理员签署的（管理员只能是client，因为最终用户应该位于请求的起点），则peer端将只执行“joinChannel”请求。
如果我们接受这点，即唯一能成为peer/排序节点MSP成员的client，只能是MSP的管理员 ，那么我们就可以绕过这种不一致。

这种方法需要考虑的另一点是，peer根据自己本地的MSP中请求发起者的成员身份，来进行事件注册请求的授权。
显然，由于请求的发起者是client，因此请求的发起者总是被认为属于与请求的peer不同的MSP，而对等方会拒绝请求。

**4) 管理（Admin）和CA证书。**

将MSP管理证书与MSP用于``信任根基``(或中间CA)的任何证书区别对待是很重要的。
这是一种常见的(安全)做法，将管理成员组成部分的职责与颁发/验证证书的职责分开。

**5) 将中间CA列入黑名单。**

如前几节所述，MSP的重新配置是通过重新配置机制实现的
(本地MSP实例使用手动重新配置；通道的MSP实例则通过构造适当的 ``config_update`` 消息)。
显然，有两种方法可以确保MSP的中间CA不再用于MSP的身份验证:

1. 重新配置MSP，使其不再将该中间CA的证书包含在可信的中间CA证书列表中。
   对于本地配置的MSP，这意味着该CA的证书将从 ``intermediatecerts`` 文件夹中删除。

2. 重新配置MSP，使其包含由信任根基生成的CRL，它通告废除了上述中间CA的证书。

在当前MSP实现里，我们只支持方法（1），因为其更为简单，而且不需要将不再需要的中间CA列入黑名单。

**6) CA 和 TLS CA**

MSP身份的根CA和MSP TLS证书的根CA(以及相关的中间CA)需要在不同的文件夹中声明。
这是为了避免不同级别证书之间的混淆。并不禁止对MSP身份和TLS证书使用相同的CA，但最佳实践建议在生产中避免这种情况。


.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
