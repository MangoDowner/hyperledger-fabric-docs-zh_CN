八卦(Gossip)数据传播协议
==================================

Hyperledger Fabric通过在事务执行（背书和提交）peer和事务排序节点之间划分工作负载来优化区块链网络性能，安全性和可伸缩性。
这种网络操作的分离需要安全，可靠和可扩展的数据传播协议，从而确保数据的完整性和一致性。
为了满足这些要求，Fabric实现了一个**八卦数据传播协议（gossip data dissemination protocol）**。


Hyperledger Fabric optimizes blockchain network performance, security,
and scalability by dividing workload across transaction execution
(endorsing and committing) peers and transaction ordering nodes. This
decoupling of network operations requires a secure, reliable and
scalable data dissemination protocol to ensure data integrity and
consistency. To meet these requirements, Fabric implements a
**gossip data dissemination protocol**.

gossip协议
---------------

Peers利用gossip以可扩展的方式广播分类帐和通道数据。
Gossip消息传递是连续的，并且通道上的每个peer不断地从多个peer接收即时且一致的分类帐数据。
每个gossip消息都被签名，从而允许拜占庭参与者轻而易地举识别伪造消息，并防止将消息分发到不想要的目标。
受延迟，网络分区或其他原因影响，从而导致错过块peer，最终将通过联系拥有这些丢失块的peer方，同步到当前分类帐状态。

基于八卦的数据传播协议在Fabric网络上执行三个主要功能：
+ 1.通过不断识别可用的成员peer,并最终检测已脱机的peer,从而管理peer发现和通道成员资格。
+ 2.在通道中的所有peer之间传播分类帐数据。任何peer，如果具有与通道其余部分不同步的数据，将识别丢失的块并通过复制正确的数据来同步自身。
+ 3.通过允许分类帐数据的peer-to-peer状态传输更新，使新连接的peer加速同步。

基于八卦的广播由peer操作，该peer从该信道上的其他peer接收消息，
然后将这些消息转发到该信道上的多个随机选择的peer，其中该数量是可配置的常数。
peer也可以使用拉动机制（pull mechanism）而不是等待传递消息。
这个循环重复进行，从而通道成员资格，分类帐和状态信息的结果将不断保持最新和同步。
为了传播新的块，通道上的**领导者**从排序服务中提取数据，并向其自己组织中的peer发起八卦传播。

领导选举
---------------

领导者选举机制用于为每个组织**选举**一个peer，该peer将维持与排序服务的连接，并且在其自己的组织的peer上分发新到达的块。
利用领导者选举为系统提供了有效利用排序服务带宽的能力。领导者选举模块有两种可能的操作模式：

+ **1.静态** -- 系统管理员手动将组织中的一个peer配置为领导者，例如某一个与排序服务保持开放连接。
+ **2.动态** -- peer执行领导者选举程序，以选择组织中的一个peer成为领导者，从排序服务中提取块，并将块传播给组织中的其他peer。

静态领导人选举
~~~~~~~~~~~~~~~~~~~~~~

使用静态领导者选举允许在组织内手动定义一组领导者peer，可以将单个节点或所有可用的peer定义为领导者，
应该考虑到这一点 - 使太多的peer连接到排序服务可能导致带宽利用效率低下。
要启用静态领导者选举模式，请在 ``core.yaml`` 部分中配置以下参数：

::

    peer:
        # Gossip related configuration
        gossip:
            useLeaderElection: false
            orgLeader: true

或者，可以使用环境变量配置和覆盖这些参数：
::

    export CORE_PEER_GOSSIP_USELEADERELECTION=false
    export CORE_PEER_GOSSIP_ORGLEADER=true


.. note:: 以下配置将使peer处于待机模式，即peer不会尝试成为领导者：:

::

    export CORE_PEER_GOSSIP_USELEADERELECTION=false
    export CORE_PEER_GOSSIP_ORGLEADER=false

2. 将 ``CORE_PEER_GOSSIP_USELEADERELECTION`` 和 ``CORE_PEER_GOSSIP_USELEADERELECTION`` 设置为 ``true`` 值是不明确的，将导致错误。
3. 在静态配置组织中，admin负责在出现故障或崩溃时提供领导节点的高可用性。

动态领导人选举
~~~~~~~~~~~~~~~~~~~~~~~

动态领导者选举使组织peer能够**选举**一个peer,其将连接到订购服务并提取新块。
每个组织peers独立地选举领导者。

当选的领导者有责任将 **心跳** 信息发送给其他peer，作为存在并活跃的证据。
如果一个或多个peer在一段时间内没有获得心跳更新，他们将启动新一轮的领导者选举程序，最终选择新的领导者。
如果在最坏的情况下进行网络分区，则组织中将有多个活跃的领导者，从而保证弹性和可用性，从而允许组织的peer继续取得进展。
在网络分区得到修复后，其中一个领导者将放弃其领导地位，因此在稳定状态下，每个组织都没有网络分区，**只有一个**活跃的领导者连接到排序服务。

以下配置控制领导**心跳**消息的频率：

::

    peer:
        # Gossip related configuration
        gossip:
            election:
                leaderAliveThreshold: 10s

为了启用动态领导者选举，需要在``core.yaml``中配置以下参数：

::

    peer:
        # Gossip related configuration
        gossip:
            useLeaderElection: true
            orgLeader: false

或者，可以使用环境变量配置和覆盖这些参数：
::

    export CORE_PEER_GOSSIP_USELEADERELECTION=true
    export CORE_PEER_GOSSIP_ORGLEADER=false

锚点peers
------------
锚点peer用于促进来自不同组织的peer之间的八卦通信。
为了使跨组织八卦工作，来自一个组织的peer需要知道来自其他组织的peer的至少一个地址（来自该peer，它可以找到该组织中的所有peer）。
该地址是锚点peer，它在通道配置中定义。

具有peer的每个组织将在通道配置中将至少一个peer（尽管可以多于一个）定义为锚点peer。
请注意，锚点peer不需要与领导者peer是同一个peer。

八卦消息
----------------

在线peer通过不断广播“活动”消息来指示其可用性，每个消息包含 **公钥基础结构（PKI）** ID和发送者对消息的签名。
peer通过收集这些有效信息来维护频道成员资格;如果没有peer收到来自特定peer的活动消息，则该“死”peer最终将从通道成员资格中清除。
由于“活动”消息是加密签名的，因此恶意peer永远不会冒充其他peer，因为它们缺少由根证书颁发机构（CA）授权的签名密钥。

除了自动转发所接收的消息之外，状态协调过程还在每个通道上跨peer同步世界状态。
每个peer不断从通道上的其他peer中提取块，以便在识别出差异时修复其自身状态。
由于不需要固定连接来维护基于八卦的数据传播，因此该过程可靠地为共享分类帐提供数据一致性和完整性，包括容忍节点崩溃。

由于通道是隔离的，因此一个信道上的peer不能在任何其他信道上发送信息或共享信息。
虽然任何peer都可以属于多个通道，但是分区消息传递，通过应用基于peer的通道订阅的消息路由策略，
来防止块被传播到不在通道中的peer。

.. note:: 1.peer-to-peer消息的安全性由peer TLS层处理，不需要签名。
          peer通过其证书进行身份验证，这些证书由CA分配。虽然也使用了TLS证书，但它是在八卦层中进行身份验证的peer证书。
          账本块由排序服务签署，然后在渠道上交付给领导者。

          2.身份验证由peer的成员资格服务提供商（MSP）管理。当peer第一次连接到该通道时，TLS会话将与成员身份绑定。
          对于网络和信道中的成员资格，这基本上验证了连接peer的每个peer。

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
