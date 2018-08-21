链码（Chaincode）教程
===================

什么是链码？
------------------


Chaincode是一个用`Go <https://golang.org>`_, `node.js <https://nodejs.org>`_ 编写的程序，
然后最终用Java等其他编程语言编写，来实现规定的接口。
Chaincode运行在与背书peer进程隔离的安全Docker容器中，其通过应用程序提交的事务初始化和管理分类帐状态。

链码通常处理网络成员同意的业务逻辑，因此可以将其视为“智能合约”。
由链码创建的状态仅限于该链码，并且不能由另一个链码直接访问。
但是，在同一网络中，给定适当的权限下，链码可以调用另一个链码来访问其状态。

两种人格
------------

我们对链码提供两种不同的观点。
+ 一个从应用程序开发人员的角度出发，开发一个名为 :doc:`chaincode4ade` 的区块链应用程序/解决方案
+ 另一个是面向区块链操作者的 :doc:`chaincode4noah`，该操作者负责管理区块链网络，
  并且还利用Hyperledger Fabric API安装，实例化和升级链代码，但可能不会涉及链代码应用程序的开发。

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
