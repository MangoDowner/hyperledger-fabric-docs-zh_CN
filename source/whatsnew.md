# 1.2版本新增内容

快速浏览Hyperledger Fabric V1.2版本中新特性和文档：

访问控制：如何配置哪些客户端标识可以在每个通道的基础上与对等函数交互。可插拔背书和验证：利用可插入背书和验证逻辑每链码。

## 新的主要特征

* **[私有数据集合](https://hyperledger-fabric.readthedocs.io/en/release-1.2/private-data/private-data.html)**:
  一种在通道成员"子集"中保持某些数据/交易保密的方法。我们还有一个关于这个主题的架构文档，可以在这里找到
  [here](https://hyperledger-fabric.readthedocs.io/en/release-1.2/private-data-arch.html).

* **[服务发现](https://hyperledger-fabric.readthedocs.io/en/release-1.2/discovery-overview.html)**:
  动态地发现网络服务，包括排序节点、对等体、链表和背书策略，以简化客户端应用程序。

* **[访问控制](https://hyperledger-fabric.readthedocs.io/en/release-1.2/access_control.html)**:
  配置哪些客户端实体可以在每个通道的基础上与peer函数交互的方法。

* **[可插拔背书和验证](https://hyperledger-fabric.readthedocs.io/en/release-1.2/pluggable_endorsement_and_validation.html)**:
  在每个链码上使用可插拔背书和验证逻辑。

### 新的教程

CouCHDB：私有数据：演示如何使用BYFN设置集合。基于各种过滤标准（FraseCA）的查询证书：描述如何使用FraseCA客户端管理证书。

* **[更新到 v1.2版本](https://hyperledger-fabric.readthedocs.io/en/release-1.2/upgrade_to_newest_version.html)**:
  利用BYFN网络来显示升级流应该如何工作。既包含脚本（也可以用作升级模板），也包括单独的命令。

* **[CouchDB](https://hyperledger-fabric.readthedocs.io/en/release-1.2/couchdb_tutorial.html)**:
  如何建立一个CouchDB数据存储（允许丰富查询）。

* **[私密数据](https://hyperledger-fabric.readthedocs.io/en/release-1.2/private_data_tutorial.html)**:
  演示如何使用BYFN设置集合。

* **[基于各种过滤标准（Fabric CA）的查询证书](https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#manage-certificates)**:
  描述如何使用 `fabric-ca-client` 管理证书。

### 新概念文档

* **[作为一个概念的Fabric网络](https://hyperledger-fabric.readthedocs.io/en/release-1.2/network/network.html)**:
  看看织物网络的各个部分的结构以及它们如何相互作用。

* **私人数据**: 见上文。

### 其他新的文档

* **[Service Discovery CLI](https://hyperledger-fabric.readthedocs.io/en/release-1.2/discovery-cli.html)**:
  使用CLI配置发现服务。

## 发行说明

要了解更多信息，包括问题的 `FAB` 编号和构成这些更改的代码审查（除了我们没有明确记录的其他卫生\[hygiene\]/性能/bug修复之外），
请查看：

* [Fabric release notes](https://github.com/hyperledger/fabric/releases/tag/v1.2.0).
* [Fabric CA release notes](https://github.com/hyperledger/fabric-ca/releases/tag/v1.2.0).

<!--- Licensed under Creative Commons Attribution 4.0 International License
https://creativecommons.org/licenses/by/4.0/ -->
