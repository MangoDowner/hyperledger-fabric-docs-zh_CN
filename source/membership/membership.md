# 成员管理

如果您已经阅读了关于 [身份](../identity/identity.html) 的文档，您就已经看到了PKI如何通过信任链提供可验证的身份。
现在让我们看看这些身份如何被用来表示区块链网络的可信成员。

这就是**成员服务**提供者（MSP）发挥作用的地方 --- 
它通过列出其成员的身份，或者通过识别哪些CA被授权为他们的成员发布有效的身份，又或者（通常情况下）通过两者的结合
**来识别哪些根CA和中间CA被信任，从而定义信任域（例如，组织）的成员**。

MSP的功能不仅仅是列出谁是一个网络参与者或一个通道的成员。
MSP可以识别参与者可能在MSP所代表的组织（例如，管理员，或作为子组织组的成员）的范围内扮演的特定**角色**，
并为在网络和通道（例如，通道管理员、读者，作者）上下文中定义**访问特权**设置基础。

MSP的配置被通告给相应组织的成员参与的所有通道(以**通道MSP**的形式)。
除通道MSP之外，peer、排序节点和客户机还维护**本地MSP**，以在通道的上下文之外认证成员消息并定义对特定组件
（例如，具有在peer上安装链码能力的组件）的权限。

此外，MSP可以允许识别已被撤销的身份列表（如[身份](../identity/identity.html)文档中所讨论的），
但我们将讨论该过程如何扩展到MSP。

我们将在后面更多地谈论本地和通道的MSP。现在让我们看看MSP一般做什么。

### 将MSP映射到组织

**组织**是一个管理的成员群体。这可以像跨国公司一样大，或者像花店一样小。
组织（**orgs**）中最重要的是他们在单个MSP下管理他们的成员。
请注意，这与X.509证书中定义的组织概念不同，我们稍后将对此进行讨论。

组织与其MSP之间的独占关系使得以组织名称命名MSP是明智的，在大多数策略配置中，您将发现采用这种约定。
例如，组织 `ORG1` 可能会有一个称为 `ORG1-MSP` 的MSP。
在某些情况下，组织可能需要多个成员组——例如，其中通道用于在组织之间执行非常不同的业务功能。
在这些情况下，具有多个MSP并相应地命名它们是有意义的，例如，`ORG2-MSP-NATIONAL` 和 `ORG2-MSP-GOVERNMENT`，
反映了 `ORG2` 内部在 `NATIONAL` 销售渠道和 `GOVERNMENT` 监管渠道里的不同成员信任根源。

![MSP1](./membership.diagram.3.png)

*组织的两种不同的MSP配置。
第一个配置显示了MSP和组织之间的典型关系——单个MSP定义组织的成员列表。
在第二种配置中，不同的MSP用于代表具有国家、国际和政府附属关系的不同组织团体。*

#### 组织单位与MSP

一个组织通常被划分为多个**组织单元**（OU），每个组织单元都有一定的职责集。
例如，`ORG1` 组织可能同时具有 `ORG1-MANUFACTURING` 和 `ORG1-DISTRIBUTION` ，以反映这些单独的业务线。
当CA颁发X.509证书时，证书中的 `OU` 字段指定身份所属的业务线。

稍后我们将看到OU如何帮助控制被认为是区块链网络成员的组织的各个部分。
例如，只有来自 `ORG1-MANUFACTURING` 的OU的身份可能能够访问信道，而 `ORG1-DISTRIBUTION` 不能。

最后，尽管这是对OU的轻微误用，但是有时候它们可以被一个联盟中的不同组织用来相互区分。
在这种情况下，不同的组织使用相同的根CA和中间CA作为它们的信任链，但是分配 `OU` 字段来标识每个组织的成员。
我们还将看到如何配置MSPS来实现这一点。

### 本地和通道MSP

MSP出现在区块链网络中的两个地方：通道配置（**通道MSP**）和本地在一行动者的前提下（**本地MSP**）。
**本地MSP是为客户端（用户）和节点（peer和排序节点）定义的。**
节点本地MSP定义了该节点的权限（例如，peer管理者是谁）。
用户的本地MSP允许用户侧在其交易中作为通道成员（例如在链码交易中）进行身份验证，或者作为特定角色的所有者进入系统（例如，在配置交易中的ORG管理）。

**每个节点和用户必须定义一个本地MSP**，因为它定义了谁在该级别具有管理或参与权限（peer管理员不一定是渠道管理员，反之亦然）。

相反，**通道MSP在通道级定义管理和参与权利**。
参与通道的每个组织都必须为它定义一个MSP。
通道上的peer和排序节点都将共享通道MSP的相同视图，因此将能够正确地验证通道参与者。
这意味着，如果一个组织希望加入该通道，则需要将包含该组织成员的信任链的MSP加入到通道配置中。
否则，来自该组织的身份的交易将被拒绝。

本地MSP和通道MSP之间的关键区别不在于它们如何工作（两者都将身份转换为角色）而是它们的**范围**。

<a name="msp2img"></a>

![MSP2](./membership.diagram.4.png)

*本地和通道MSP。每个peer的信任域（例如，组织）由peer的本地MSP（例如，OR1或ORG2）定义。
通过在通道配置中添加组织的MSP来实现组织在通道上的表示。
例如，该图的通道由Org1和Org2两者管理。类似的原则适用于网络、排序节点和用户，但为了简单起见，这里没有显示这他们。*

您可能会发现，查看区块链管理员安装和实例化智能合约时发生的情况，对了解如何使用本地和通道MSP很有帮助，
如[上图](#msp2img)所示。

管理员 `B` 以 `RCA1` 发布的身份连接到peer并存储在本地MSP中。
当 `B` 试图在peer上安装智能合约时，peer检查其本地MSP `ORG1-MSP`，以验证 `B` 的身份是否确实是 `ORG1` 的成员。
成功的验证将允许安装命令成功完成。
随后，`B` 希望在通道上实例化智能合同。因为这是一个通道操作，通道上的所有组织都必须同意它。
因此，peer必须在成功提交此命令之前检查信道的MSP。（其他的事情也必须发生，但是现在就集中在上面的事情。）

**本地MSP只定义在它们所应用的节点或用户的文件系统上**。
因此，在物理和逻辑上，每个节点或用户只有一个本地MSP。
然而，当通道MSP对通道中的所有节点可用时，它们在通道配置中被逻辑地定义一次。
**然而，通道MSP也在信道中的每个节点的文件系统中被实例化，并通过共识保持同步。**
因此，虽然在每个节点的本地文件系统上有每个通道MSP的副本，但在逻辑上，通道SP驻留在通道或网络上，并由通道或网络维护。

### MSP级别

通道和本地MSP之间的划分反映了组织管理其本地资源（如peer或排序节点）及其信道资源（如账本、智能合约和联盟）的需要，
这些资源在信道或网络级别上运行。
将这些MSP设想为处于不同**级别**是有帮助的，其中**与网络管理相关的MSP处于较高级别**，
而**用于管理私有资源身份的MSP处于较低级别**。
MSP在每一级管理中都是强制性的，它们必须为网络、通道、peer、排序节点和用户定义。

![MSP3](./membership.diagram.2.png)

*MSP级别。peer和排序节点的MSP是本地的，而通道（包括网络配置信道）的MSP在该通道的所有参与者之间共享。
在这个图中，网络配置通道由ORG1管理，但是另一个应用程序通道可以由ORG1和ORG2管理。
peer是ORG2的成员，由ORG2管理，而ORG1管理图中的的排序节点。
ORG1信任来自RCA1的身份，而ORG2信任来自RCA2的身份。
请注意，这些是管理身份，反映谁可以管理这些组件。
因此，当ORG1管理网络时，Org2.MSP确实也存在于网络定义中。*

Orther-MSP：

 * **网络MSP:** 网络的配置通过定义参与组织的MSP来定义谁是网络中的成员，以及这些成员中的哪些被授权执行管理任务（例如，创建通道）

 * **通道MSP:** 通道独立地维护成员的MSP是很重要的。通道在特定的一组组织之间提供私有通信，这些组织反过来又对它进行管理控制。
   在该通道的MSP上下文中解释的通道策略，定义了谁有能力参与该通道上的某些操作，例如，添加组织或实例化链码。
   注意，管理通道的权限与管理网络配置通道（或任何其他通道）的能力之间没有必要的关系。
   管理权限存在于所管理的内容范围内（除非规则已经被编写成其他内容，请参阅下面对 `ROLE` 属性的讨论）。

 * **Peer MSP:** 这个本地MSP是在每个peer的文件系统上定义的，每个peer都有一个单个的MSP实例。
   在概念上，它执行与通道MSP完全相同的功能，但它仅限于应用于定义它的peer。
   使用peer的本地MSP评估其授权的操作的一个示例是在peer上安装链码。

 * **排序节点 MSP:** 像peer MSP一样，在节点的文件系统中也定义了一个排序节点本地MSP，并且只适用于该节点。
   与peer一样，排序节点也由单个组织拥有，因此只有一个MSP列出它信任的行动者或节点。

### MSP 结构

到目前为止，您已经看到，MSP最重要的元素是，是用于形成根或中间CA的规范，这些CA用于建立参与者或节点在各自组织中的成员资格。
然而，有更多的元素与这两个元素一起使用，来辅助成员关系功能。

![MSP4](./membership.diagram.5.png)

*上面的图显示了本地MSP是如何存储在本地文件系统上的。尽管通道MSP不是以这样的方式物理构造的，但这仍然是一种有益的思考方式。*

正如您所看到的，MSP有九个元素。
在目录结构中考虑这些元素是最容易的，其中MSP名称是根文件夹名称，每个子文件夹表示MSP配置的不同元素。

让我们更详细地描述这些文件夹，看看它们为什么很重要。

* **根CA:** 此文件夹包含由此MSP代表的组织信任的自签名X.509根CA证书的列表。
  在这个MSP文件夹中必须至少有一个根 CA x.509证书。
           
  这是最重要的文件夹，因为它标识了其他中间CA，所有证书都要从这些中间CA派生以便将其视为相应组织的成员。

* **中间CA:** 此文件夹包含由该组织信任的中间CA的X.509证书列表。
  每个证书必须由MSP中的一个根CA或由中间CA签名，中间CA的发布CA链最终指向可信的根CA。

  中间CA可以表示组织的不同细分（如用于`ORG1` 的 `ORG1-MANUFACTURING` 和`ORG1-DISTRIBUTION`），
  或者组织本身（如果利用商业CA用于组织的身份管理，情况可能就是这样）。
  在后一种情况下，中间CA可以用来表示组织次级细分。
  在[这里](../msp.html)，您可以找到更多关于MSP配置的最佳实践的信息。
  注意，可能有一个没有中间CA的功能网络，在这种情况下，该文件夹将是空的。
  
  与根CA文件夹一样，该文件夹定义了必须从中颁发证书才能被视为组织成员的CA。
  
* **组织单元 (OUs):** 这些列在 `$FABRIC_CFG_PATH/msp/config.yaml`
  文件中，并且包含组织单元的列表，组织单元的成员被认为是由该MSP表示的组织的一部分。
  当您希望将组织的成员限制为具有特定OU的身份（由MSP指定的CA之一签名）的成员时，这尤其有用。

  指定OE是可选的。如果没有列出OU，则作为MSP一部分的所有身份（由根CA和中间CA文件夹颁发的身份）将被视为组织的成员。
  
* **管理员:** 此文件夹包含一个身份列表，该列表定义具有此组织管理员角色的角色。
  对于标准的MSP类型，在这个列表中应该有一个或多个X.509证书。
  
  值得注意的是，仅仅因为参与者具有管理员的角色，并不意味着他们可以管理特定的资源！
  给定身份在管理系统方面所具有的实际能力由管理系统资源的策略决定。
  例如，通道策略可以指定 `ORG1-MANUFACTURING` 管理员有权向通道添加新组织，
  而 `ORG1-DISTRIBUTION` 管理员没有这样的权限。

  尽管X.509证书具有 `ROLE` 属性（例如，指定参与者是 `admin` ），但这指的是参与者在其组织内的角色，而不是在区块链网络上的角色。
  这与 `OU` 属性的用途相似，如果定义了 `OU` 属性，则它指的是参与者在组织中的位置。
  
  如果该通道的策略已被写入，以允许来自组织（或某些组织）的任何管理员执行某些通道功能（例如实例化链码），
  则**可以**使用`ROLE`属性来授予通道级的管理权限。
  通过这种方式，组织角色可以被赋予网络角色。
    
* **吊销证书:** 如果一个参与者的身份被吊销，则在该文件夹中保存有关身份的信息而不是身份本身。
  对于基于X.509的身份，这些标识符是称为主题密钥标识符（SKI）和权限访问标识符（AKI）的字符串对，
  每当X.509证书被用于确保证书没有被撤销时，都会对其进行检查。

  此列表在概念上与CA证书撤销列表（CRL）相同，但也涉及撤销组织的成员资格。
  因此，MSP（本地或频道）的管理员可以通过宣传（颁发被撤销证书的）CA的CRL，来快速地从组织撤消参与者或节点。
  这个“列表列表”是可选的。它只会因为证书被吊销而变得拥挤不堪。
 
* **节点身份：** 此文件夹包含节点的身份，即，与 `KeyStore` 的内容结合的密码材料（cryptographic material），
  将允许节点在发送给其通道和网络的其他参与者的消息中，对自身进行身份验证。
  对于基于X.509的身份，此文件夹包含**X.509证书**。
  例如，这是对peer在交易提案响应中放置的证书，以指示peer已对它进行背书，随后可以在验证时根据所得交易的背书策略对其进行检查。
  
  此文件夹对于本地MSP是必需的，并且节点必须有一个X.509证书。它不用于信道MSP。
  

* **私钥的`KeyStore`:** 这个文件夹是为peer或排序节点的本地MSP（或在客户端的本地MSP）定义的，并且包含节点的**签名密钥**。
  此密钥与节点身份文件夹中包括的**节点身份**进行密码匹配，并用于对数据进行签名，
  例如，作为背书阶段的一部分，用于对交易提案响应进行签名。
    
  此文件夹对于本地MSP是必需的，并且必须包含一个私钥。
  显然，对该文件夹的访问必须仅限于peer负有管理责任的用户的身份。
    
  **通道MSP**的配置不包括此文件夹，因为通道MSP仅旨在提供身份验证功能而不是签名功能。

* **TLS根CA:** 该文件夹包含一个自签名的X.509证书的列表，这些证书是本组织为**TLS通信**所信任的根CA。
  TLS通信的一个例子是当一个peer要连接到一个排序节点，以便接收账本更新。
  
  MSP TLS信息涉及网络内部的节点 --- peer和排序节点，换句话说，而不是消耗网络的应用和管理。
  
  在该文件夹中必须至少有一个TLS根CA X.509证书。
 
* **TLS中间CA:** 该文件夹包含由此MSP所表示的，用于TLS通信的组织信任的中间CA证书CA的列表。
  当商业CA用于组织的TLS证书时，该文件夹特别有用。
  类似于成员中间CA，指定中间TLS CA是可选的。

  有关TLS的更多信息，请单击[此处](../enable_tls.html)。
  
如果您已经阅读过这个文档以及我们的 [身份](../identity/identity.html) ，
您应该对身份和成员资格如何在Hyperledger Fabric中工作有很好的理解。
您已经看到了如何使用PKI和MSP来标识区块链网络中的参与者进行协作。
除了MSP的物理和逻辑结构之外，您还了解了证书、公钥/私钥以及信任根是如何工作的。

<!---
Licensed under Creative Commons Attribution 4.0 International License https://creativecommons.org/licenses/by/4.0/
-->
