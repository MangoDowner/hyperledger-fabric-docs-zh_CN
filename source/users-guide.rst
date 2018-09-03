Fabric CA 用户手册
======================

Hyperledger Fabric CA是用于Hyperledger Fabric的证书颁发机构（CA）。

它提供了如下特征：

  * 注册身份，或者作为用户注册表连接到LDAP
  * 颁发注册证书（ECETS）
  * 证书更新与撤销

Hyperledger Fabric CA 由服务器和客户端组件组成，如本文后面所述。

对于有兴趣参与Hyperledger Fabric CA的开发人员，请参见
`Fabric CA repository <https://github.com/hyperledger/fabric-ca>`__
获取更多信息。

.. _回到顶端:

目录表
-----------------

1. `总览`_

2. `开始动手吧`_

   1. `前提条件`_
   2. `安装`_
   3. `探索Fabric CA 命令行`_

3. `配置设置`_

   1. `文件路径`_

4. `Fabric CA 服务器`_

   1. `初始化服务器`_
   2. `启动服务器`_
   3. `配置数据库`_
   4. `配置LDAP`_
   5. `Setting up a cluster`_
   6. `启动多CA`_
   7. `注册中间CA`_
   8. `升级服务器`_

5. `Fabric CA 客户端`_

   1. `注册启动身份`_
   2. `Registering a new identity`_
   3. `Enrolling a peer identity`_
   4. `Getting a CA certificate chain from another Fabric CA server`_
   5. `重新注册身份`_
   6. `吊销证书或身份`_
   7. `Generating a CRL (Certificate Revocation List)`_
   8. `Attribute-Based Access Control`_
   9. `Dynamic Server Configuration Update`_
   10. `启用TLS`_
   11. `Contact specific CA instance`_

6. `HSM`_

   1. `Configuring Fabric CA server to use softhsm2`_

7. `文件格式`_

   1. `Fabric CA 服务器的配置文件格式`_
   2. `Fabric CA 客户端的的配置文件格式`_

8. `故障排除`_

总览
--------

下面的图表说明了Hyperledger Fabric CA服务器如何适用于整个Hyperledger Fabric结构。

.. image:: ./images/fabric-ca.png

有两种方式与Hyperledger Fabric CA服务器交互：通过Hyperledger Fabric CA客户端或通过FabricSDK。
对Fabric CA CA服务器进行的所有通信都是通过REST API实现的。
参阅 `fabric-ca/swagger/swagger-fabric-ca.json` 来看看这些REST API的SWAGER文档。
您可以通过 http://editor2.swagger.io 在线编辑器查看此文档。

Hyperledger Fabric CA客户端或SDK可以连接到Hyperledger Fabric CA服务器集群中的服务器。
这在图的右上部分进行了说明。客户端路由到HA代理端点（Proxy endpoint），
该端点将会将流量平衡负载到一个fabric-ca-server集群成员。

集群中所有的Hyperledger Fabric CA服务器共享同一数据库以跟踪身份和证书。
如果配置LDAP，则将身份信息保存在LDAP中而不是数据库中。

服务器可以包含多个CAS。每个CA要么是根CA，要么是中间CA。
每个中间CA都有一个父CA，它要么是根CA，要么是另一个中间CA。

开始动手吧
---------------

前提条件
~~~~~~~~~~~~~~~

-  安装 Go 1.9+
-  正确设置 ``GOPATH`` 环境变量
-  安装 libtool 和 libtdhl-dev 包

下面的命令在Ubuntu上安装libtool依赖:

.. code:: bash

   sudo apt install libtool libltdl-dev

下面的命令在MacOSX上安装libtool依赖:

.. code:: bash

   brew install libtool

.. note:: 如果你通过Homebrew安装libtool，那么libtldl-dev便没必要安装了

想要了解libtool的更多内容，查阅 https://www.gnu.org/software/libtool.

想要了解libltdl-dev的更多内容，查阅 https://www.gnu.org/software/libtool/manual/html_node/Using-libltdl.html.

安装
~~~~~~~~~~~~~~~
接下来的命令在 $GOPATH/bin 里安装 `fabric-ca-server` 和 `fabric-ca-client` 程序

.. code:: bash

    go get -u github.com/hyperledger/fabric-ca/cmd/...

.. note:: 如果您已经克隆了fabric-ca库，那么在运行上面的“go get”命令之前，请确保您在master分支上。否则，您可能会看到以下错误：

::

    <gopath>/src/github.com/hyperledger/fabric-ca; git pull --ff-only
    There is no tracking information for the current branch.
    Please specify which branch you want to merge with.
    See git-pull(1) for details.

        git pull <remote> <branch>

    If you wish to set tracking information for this branch you can do so with:

        git branch --set-upstream-to=<remote>/<branch> tlsdoc

    package github.com/hyperledger/fabric-ca/cmd/fabric-ca-client: exit status 1

本地启动服务器
~~~~~~~~~~~~~~~~~~~~~

下面命令动以默认设置启 `fabric-ca-server`。

.. code:: bash

    fabric-ca-server start -b admin:adminpw

`-b` 选项为引导管理员提供了注册（enrollment）ID和密码；如果LDAP没有启用“ldap.enabled”设置，则需要这样做。

在本地目录中创建名为 `fabric-ca-server-config.yaml` 的配置文件，该目录也是可配置的。

通过Docker启动服务器
~~~~~~~~~~~~~~~~~~~~~~~

Docker Hub
^^^^^^^^^^^^

访问: https://hub.docker.com/r/hyperledger/fabric-ca/tags/

找到与你想拉取的fabric-ca的架构和版本相匹配的tag。

导航到 `$GOPATH/src/github.com/hyperledger/fabric-ca/docker/server` ，
并在编辑器中打开 `docker-compose.yml`。

更改 `image` 行以反映您先前找到的tag。对于X86架构的beta版本该文件可能是这样的。

.. code:: yaml

    fabric-ca-server:
      image: hyperledger/fabric-ca:x86_64-1.0.0-beta
      container_name: fabric-ca-server
      ports:
        - "7054:7054"
      environment:
        - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      volumes:
        - "./fabric-ca-server:/etc/hyperledger/fabric-ca-server"
      command: sh -c 'fabric-ca-server start -b admin:adminpw'

在与docker-compose.yml文件相同的目录中打开一个终端并执行以下操作：

.. code:: bash

    # docker-compose up -d

如果compose文件中指定的fabric-ca映像不存在，则将拉取该映像，并启动fabric-ca服务器的实例。

创建你自己的Docker镜像
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

您可以通过docker-compose创建并启动服务器，如下所示。

.. code:: bash

    cd $GOPATH/src/github.com/hyperledger/fabric-ca
    make docker
    cd docker/server
    docker-compose up -d

hyperledger/fabric-ca 镜像包含了fabric-ca-server和fabric-ca-client。



.. code:: bash

    # cd $GOPATH/src/github.com/hyperledger/fabric-ca
    # FABRIC_CA_DYNAMIC_LINK=true make docker
    # cd docker/server
    # docker-compose up -d

探索Fabric CA 命令行
~~~~~~~~~~~~~~~~~~~~~~~~~~~

本节简单地为织物Fabric CA服务器和客户端提供使用消息。
在下面的章节中将会提供附加的使用信息。

下面的链接显示 :doc:`Server Command Line <servercli>` 和
:doc:`Client Command Line <clientcli>`。

.. note:: 注意，作为字符串片（列表）的命令行选项，可以通过两种方式来指定：即使用逗号分隔的列表元素，或者多次指定选项，
          每个选项都具有组成列表的字符串值。例如，要为``csr.hosts``选项指定 ``host1`` 和 ``host2`` ，
          可以传递 ``--csr.hosts 'host1,host2'`` 或 ``--csr.hosts host1 --csr.hosts host2`` 。
          使用前一种格式时，请确保在逗号之前或之后没有空格。

`回到顶端`_

配置设置
---------------

Fabric CA提供3种方式来配置Fabric CA服务器和客户机上的设置。
优先顺序为：

  1. CLI标志
  2. 环境变量
  3. 配置文件

在本文档的其余部分中，我们提到对配置文件进行更改。
但是，配置文件更改可以通过环境变量或CLI标志重写。

例如，如果在客户端配置文件中有以下内容：

.. code:: yaml

    tls:
      # Enable TLS (default: false)
      enabled: false

      # TLS for the client's listenting port (default: false)
      certfiles:
      client:
        certfile: cert.pem
        keyfile:

下面的环境变量可用于覆盖配置文件中的 ``cert.pem`` 设置：

.. code:: bash

  export FABRIC_CA_CLIENT_TLS_CLIENT_CERTFILE=cert2.pem

如果我们想重写环境变量和配置文件，我们可以使用命令行标志。

.. code:: bash

  fabric-ca-client enroll --tls.client.certfile cert3.pem

同样的方法也适用于fabric-ca-server，除了使用了 ``FABRIC_CA_SERVER`` 而不是 ``FABIRC_CA_CLIENT`` 作为环境变量的前缀。

.. _server:

文件路径
~~~~~~~~~~~~~~~

Fabric CA服务器和客户端配置文件中指定文件名的所有属性都支持相对路径和绝对路径。
相对路径与配置文件所在的配置目录相对。例如，如果配置目录是 ``~/config``  ，并且tls部分如下所示，
则Fabric CA服务器或客户端将在``~/config`` 目录中查找 ``cert.pem``文件、
``~/config/certs`` 目录中的 ``cert.pem`` 文件和 ``/abs/path`` 目录中的 ``key.pem`` 文件

.. code:: yaml

    tls:
      enabled: true
      certfiles:
        - root.pem
      client:
        certfile: certs/cert.pem
        keyfile: /abs/path/key.pem

`回到顶端`_

Fabric CA 服务器
----------------

This section describes the Fabric CA server.

You may initialize the Fabric CA server before starting it. This provides an
opportunity for you to generate a default configuration file that can be
reviewed and customized before starting the server.

The Fabric CA server's home directory is determined as follows:
  - if the --home command line option is set, use its value
  - otherwise, if the ``FABRIC_CA_SERVER_HOME`` environment variable is set, use
    its value
  - otherwise, if ``FABRIC_CA_HOME`` environment variable is set, use
    its value
  - otherwise, if the ``CA_CFG_PATH`` environment variable is set, use
    its value
  - otherwise, use current working directory

For the remainder of this server section, we assume that you have set
the ``FABRIC_CA_HOME`` environment variable to
``$HOME/fabric-ca/server``.

The instructions below assume that the server configuration file exists
in the server's home directory.

.. _initialize:

初始化服务器
~~~~~~~~~~~~~~~~~~~~~~~

通过如下方式初始化abric CA服务器:

.. code:: bash

    fabric-ca-server init -b admin:adminpw

当禁用LDAP时，需要初始化``-b``（启动身份）选项。启动Fabric CA服务器需要至少一个引导身份；
该身份是服务器管理员。

服务器配置文件包含可配置的证书签名请求（CSR）部分。下面是CSR示例。

.. _csr-fields:

.. code:: yaml

   cn: fabric-ca-server
   names:
      - C: US
        ST: "North Carolina"
        L:
        O: Hyperledger
        OU: Fabric
   hosts:
     - host1.example.com
     - localhost
   ca:
      expiry: 131400h
      pathlength: 1

以上所有字段都属于X.509签名密钥和证书，该证书是由 ``fabric-ca-server init`` 生成的。
这对应于服务器配置文件中的 ``ca.certfile`` 和 ``ca.keyfile`` 文件。字段如下：

  -  **cn** 是公共名字
  -  **O** 是组织名字
  -  **OU** 是组织单元
  -  **L** 是城市位置
  -  **ST** 是洲（state）名
  -  **C** 是国家名

如果需要CSR的自定义值，则可以自定义配置文件，删除 ``ca.certfile`` 和 ``ca.keyfile`` 配置项指定的文件，
然后再次运行 ``fabric-ca-server init -b admin:adminpw`` 命令。

除非指定了 ``-u <parent-fabric-ca-server-URL>`` 选项，否则 ``fabric-ca-server init`` 命令将生成一个自签名的CA证书。
如果指定了 ``-u`` ，则服务器的CA证书由父结构CA服务器签名。

为了向父Fabric CA服务器进行身份验证，URL必须为 ``<scheme>://<enrollmentID>:<secret>@<host>:<port>`` ，
其中 <enrollmentID> 和 <secret> 对应于“hf.IntermediateCA”属性值为“true”的身份。

``fabric-ca-server init`` 命令还在服务器的主目录中生成名为 **fabric-ca-server-config.yaml** 的默认配置文件。

如果希望Fabric CA服务器使用您提供的CA签名证书和密钥文件，则必须将文件分别放在``ca.certfile``和 ``ca.keyfile`` 引用的位置。
两个文件必须是PEM编码的，且不能是已加密的。更具体地说，CA证书文件的内容必须以 ``-----BEGIN CERTIFICATE-----`` 开始，
而密钥文件的内容必须以 ``-----BEGIN PRIVATE KEY-----`` 开始，而不是 ``-----BEGIN ENCRYPTED PRIVATE KEY-----`` 开始。

算法和密钥尺寸
~~~~~~~~~~~~~~~~~~~

CSR可以定制生成X.509证书和支持椭圆曲线（ECDSA）的密钥。
以下设置是椭圆曲线数字签名算法(ECDSA)（用曲线素数 ``prime256v1``）和
签名算法 ``ecdsa-with-SHA256``的实现的示例：

.. code:: yaml

    key:
       algo: ecdsa
       size: 256

算法和密钥大小的选择是基于安全需求的。

椭圆曲线（ECDSA）提供以下密钥尺寸选择：

+--------+--------------+-----------------------+
| 尺寸    | ASN1 OID     | 签名算法               |
+========+==============+=======================+
| 256    | prime256v1   | ecdsa-with-SHA256     |
+--------+--------------+-----------------------+
| 384    | secp384r1    | ecdsa-with-SHA384     |
+--------+--------------+-----------------------+
| 521    | secp521r1    | ecdsa-with-SHA512     |
+--------+--------------+-----------------------+

启动服务器
~~~~~~~~~~~~~~~~~~~

按照下面方法启动Fabric CA server：

.. code:: bash

    fabric-ca-server start -b <admin>:<adminpw>

如果服务器没有被预先初始化，它将在第一次启动时初始化它自己。
在此初始化期间，如果还没有ca-cert.pem和ca-key.pem文件，服务器将生成它们，
如果它们不存在，服务器还将创建默认的配置文件。
请参见 `Initialize the Fabric CA server <#initialize>`__ 部分。

除非Fabric CA服务器被配置为使用LDAP，否则它必须配置有至少一个预先注册的引导身份，
以使您能够登记（register）和注册（enroll）其他标识。``-b``  选项指定引导身份的名称和密码。

要使Fabric CA服务器侦听 ``https`` 而不是 ``http``，将 ``tls.enabled`` 设定为 ``true``。

.. note:: 安全警告：该结构CA服务器应该总是以启用TLS（ ``tls.enabled`` 设置为true）开始。
          如果不这样做，服务器就容易受到攻击者访问网络流量的影响。

若要限制同一秘密（或密码）可用于注册（enroll）的次数，请将配置文件中的 ``registry.maxenrollments`` 设置为适当的值。
如果将值设置为1，则Fabric CA服务器只允许对特定注册ID使用一次密码。
如果将值设置为-1，则Fabric CA服务器对可重用秘密进行注册的次数没有限制。
默认值为-1。将值设置为0，Fabric CA服务器将禁用所有标识的登记和注册。

Fabric CA服务器现在应该监听端口7054。

如果不希望将Fabric CA服务器配置为在集群中运行或使用LDAP，则可以跳到
`Fabric CA Client <#fabric-ca-client>`__
部分。

配置数据库
~~~~~~~~~~~~~~~~~~~~~~~~

本节介绍如何配置Fabric CA服务器以连接到PostgreSQL或MySQL数据库。
默认的数据库是SQLite，默认的数据库文件是Fabric Ca服务器的主目录中的``fabric-ca-server.db``。

如果不关心在集群中运行Fabric CA服务器，则可以跳过本节；
否则，必须按照以下描述配置PostgreSQL或MySQL。
在集群设置中，结构CA支持以下数据库版本：

- PostgreSQL: 9.5.5 或者更高版本
- MySQL: 5.7 或者更高版本

PostgreSQL
^^^^^^^^^^

The following sample may be added to the server's configuration file in
order to connect to a PostgreSQL database. Be sure to customize the
various values appropriately. There are limitations on what characters are allowed
in the database name. Please refer to the following Postgres documentation
for more information: https://www.postgresql.org/docs/current/static/sql-syntax-lexical.html#SQL-SYNTAX-IDENTIFIERS

.. code:: yaml

    db:
      type: postgres
      datasource: host=localhost port=5432 user=Username password=Password dbname=fabric_ca sslmode=verify-full

Specifying *sslmode* configures the type of SSL authentication. Valid
values for sslmode are:

|

+----------------+----------------+
| Mode           | Description    |
+================+================+
| disable        | No SSL         |
+----------------+----------------+
| require        | Always SSL     |
|                | (skip          |
|                | verification)  |
+----------------+----------------+
| verify-ca      | Always SSL     |
|                | (verify that   |
|                | the            |
|                | certificate    |
|                | presented by   |
|                | the server was |
|                | signed by a    |
|                | trusted CA)    |
+----------------+----------------+
| verify-full    | Same as        |
|                | verify-ca AND  |
|                | verify that    |
|                | the            |
|                | certificate    |
|                | presented by   |
|                | the server was |
|                | signed by a    |
|                | trusted CA and |
|                | the server     |
|                | hostname       |
|                | matches the    |
|                | one in the     |
|                | certificate    |
+----------------+----------------+

|

If you would like to use TLS, then the ``db.tls`` section in the Fabric CA server
configuration file must be specified. If SSL client authentication is enabled
on the PostgreSQL server, then the client certificate and key file must also be
specified in the ``db.tls.client`` section. The following is an example
of the ``db.tls`` section:

.. code:: yaml

    db:
      ...
      tls:
          enabled: true
          certfiles:
            - db-server-cert.pem
          client:
                certfile: db-client-cert.pem
                keyfile: db-client-key.pem

| **certfiles** - A list of PEM-encoded trusted root certificate files.
| **certfile** and **keyfile** - PEM-encoded certificate and key files that are used by the Fabric CA server to communicate securely with the PostgreSQL server

PostgreSQL SSL Configuration
"""""""""""""""""""""""""""""

**Basic instructions for configuring SSL on the PostgreSQL server:**

1. In postgresql.conf, uncomment SSL and set to "on" (SSL=on)

2. Place certificate and key files in the PostgreSQL data directory.

Instructions for generating self-signed certificates for:
https://www.postgresql.org/docs/9.5/static/ssl-tcp.html

Note: Self-signed certificates are for testing purposes and should not
be used in a production environment

**PostgreSQL Server - Require Client Certificates**

1. Place certificates of the certificate authorities (CAs) you trust in the file root.crt in the PostgreSQL data directory

2. In postgresql.conf, set "ssl\_ca\_file" to point to the root cert of the client (CA cert)

3. Set the clientcert parameter to 1 on the appropriate hostssl line(s) in pg\_hba.conf.

For more details on configuring SSL on the PostgreSQL server, please refer
to the following PostgreSQL documentation:
https://www.postgresql.org/docs/9.4/static/libpq-ssl.html

MySQL
^^^^^^^

The following sample may be added to the Fabric CA server configuration file in
order to connect to a MySQL database. Be sure to customize the various
values appropriately. There are limitations on what characters are allowed
in the database name. Please refer to the following MySQL documentation
for more information: https://dev.mysql.com/doc/refman/5.7/en/identifiers.html

On MySQL 5.7.X, certain modes affect whether the server permits '0000-00-00' as a valid date.
It might be necessary to relax the modes that MySQL server uses. We want to allow
the server to be able to accept zero date values.

In my.cnf, find the configuration option *sql_mode* and remove *NO_ZERO_DATE* if present.
Restart MySQL server after making this change.

Please refer to the following MySQL documentation on different modes available
and select the appropriate settings for the specific version of MySQL that is
being used.

https://dev.mysql.com/doc/refman/5.7/en/sql-mode.html

.. code:: yaml

    db:
      type: mysql
      datasource: root:rootpw@tcp(localhost:3306)/fabric_ca?parseTime=true&tls=custom

If connecting over TLS to the MySQL server, the ``db.tls.client``
section is also required as described in the **PostgreSQL** section above.

MySQL SSL Configuration
""""""""""""""""""""""""

**Basic instructions for configuring SSL on MySQL server:**

1. Open or create my.cnf file for the server. Add or uncomment the
   lines below in the [mysqld] section. These should point to the key and
   certificates for the server, and the root CA cert.

   Instructions on creating server and client-side certficates:
   http://dev.mysql.com/doc/refman/5.7/en/creating-ssl-files-using-openssl.html

   [mysqld] ssl-ca=ca-cert.pem ssl-cert=server-cert.pem ssl-key=server-key.pem

   Can run the following query to confirm SSL has been enabled.

   mysql> SHOW GLOBAL VARIABLES LIKE 'have\_%ssl';

   Should see:

   +----------------+----------------+
   | Variable_name  | Value          |
   +================+================+
   | have_openssl   | YES            |
   +----------------+----------------+
   | have_ssl       | YES            |
   +----------------+----------------+

2. After the server-side SSL configuration is finished, the next step is
   to create a user who has a privilege to access the MySQL server over
   SSL. For that, log in to the MySQL server, and type:

   mysql> GRANT ALL PRIVILEGES ON *.* TO 'ssluser'@'%' IDENTIFIED BY
   'password' REQUIRE SSL; mysql> FLUSH PRIVILEGES;

   If you want to give a specific IP address from which the user will
   access the server change the '%' to the specific IP address.

**MySQL Server - Require Client Certificates**

Options for secure connections are similar to those used on the server side.

-  ssl-ca identifies the Certificate Authority (CA) certificate. This
   option, if used, must specify the same certificate used by the server.
-  ssl-cert identifies MySQL server's certificate.
-  ssl-key identifies MySQL server's private key.

Suppose that you want to connect using an account that has no special
encryption requirements or was created using a GRANT statement that
includes the REQUIRE SSL option. As a recommended set of
secure-connection options, start the MySQL server with at least
--ssl-cert and --ssl-key options. Then set the ``db.tls.certfiles`` property
in the server configuration file and start the Fabric CA server.

To require that a client certificate also be specified, create the
account using the REQUIRE X509 option. Then the client must also specify
proper client key and certificate files; otherwise, the MySQL server
will reject the connection. To specify client key and certificate files
for the Fabric CA server, set the ``db.tls.client.certfile``,
and ``db.tls.client.keyfile`` configuration properties.

配置LDAP
~~~~~~~~~~~~~~~~

The Fabric CA server can be configured to read from an LDAP server.

In particular, the Fabric CA server may connect to an LDAP server to do
the following:

-  authenticate an identity prior to enrollment
-  retrieve an identity's attribute values which are used for authorization.

Modify the LDAP section of the Fabric CA server's configuration file to configure the
server to connect to an LDAP server.

.. code:: yaml

    ldap:
       # Enables or disables the LDAP client (default: false)
       enabled: false
       # The URL of the LDAP server
       url: <scheme>://<adminDN>:<adminPassword>@<host>:<port>/<base>
       userfilter: <filter>
       attribute:
          # 'names' is an array of strings that identify the specific attributes
          # which are requested from the LDAP server.
          names: <LDAPAttrs>
          # The 'converters' section is used to convert LDAP attribute values
          # to fabric CA attribute values.
          #
          # For example, the following converts an LDAP 'uid' attribute
          # whose value begins with 'revoker' to a fabric CA attribute
          # named "hf.Revoker" with a value of "true" (because the expression
          # evaluates to true).
          #    converters:
          #       - name: hf.Revoker
          #         value: attr("uid") =~ "revoker*"
          #
          # As another example, assume a user has an LDAP attribute named
          # 'member' which has multiple values of "dn1", "dn2", and "dn3".
          # Further assume the following configuration.
          #    converters:
          #       - name: myAttr
          #         value: map(attr("member"),"groups")
          #    maps:
          #       groups:
          #          - name: dn1
          #            value: orderer
          #          - name: dn2
          #            value: peer
          # The value of the user's 'myAttr' attribute is then computed to be
          # "orderer,peer,dn3".  This is because the value of 'attr("member")' is
          # "dn1,dn2,dn3", and the call to 'map' with a 2nd argument of
          # "group" replaces "dn1" with "orderer" and "dn2" with "peer".
          converters:
            - name: <fcaAttrName>
              value: <fcaExpr>
          maps:
            <mapName>:
                - name: <from>
                  value: <to>

Where:

  * ``scheme`` is one of *ldap* or *ldaps*;
  * ``adminDN`` is the distinquished name of the admin user;
  * ``pass`` is the password of the admin user;
  * ``host`` is the hostname or IP address of the LDAP server;
  * ``port`` is the optional port number, where default 389 for *ldap*
    and 636 for *ldaps*;
  * ``base`` is the optional root of the LDAP tree to use for searches;
  * ``filter`` is a filter to use when searching to convert a login
    user name to a distinguished name. For example, a value of
    ``(uid=%s)`` searches for LDAP entries with the value of a ``uid``
    attribute whose value is the login user name. Similarly,
    ``(email=%s)`` may be used to login with an email address.
  * ``LDAPAttrs`` is an array of LDAP attribute names to request from the
    LDAP server on a user's behalf;
  * the attribute.converters section is used to convert LDAP attributes to fabric
    CA attributes, where
    * ``fcaAttrName`` is the name of a fabric CA attribute;
    * ``fcaExpr`` is an expression whose evaluated value is assigned to the fabric CA attribute.
    For example, suppose that <LDAPAttrs> is ["uid"], <fcaAttrName> is 'hf.Revoker',
    and <fcaExpr> is 'attr("uid") =~ "revoker*"'.  This means that an attribute
    named "uid" is requested from the LDAP server on a user's behalf.  The user is
    then given a value of 'true' for the 'hf.Revoker' attribute if the value of
    the user's 'uid' LDAP attribute begins with 'revoker'; otherwise, the user
    is given a value of 'false' for the 'hf.Revoker' attribute.
  * the attribute.maps section is used to map LDAP response values.  The typical
    use case is to map a distinguished name associated with an LDAP group to an
    identity type.

The LDAP expression language uses the govaluate package as described at
https://github.com/Knetic/govaluate/blob/master/MANUAL.md.  This defines
operators such as "=~" and literals such as "revoker*", which is a regular
expression.  The LDAP-specific variables and functions which extend the
base govaluate language are as follows:

  * ``DN`` is a variable equal to the user's distinguished name.
  * ``affiliation`` is a variable equal to the user's affiliation.
  * ``attr`` is a function which takes 1 or 2 arguments.  The 1st argument
    is an LDAP attribute name.  The 2nd argument is a separator string which is
    used to join multiple values into a single string; the default separator
    string is ",". The ``attr`` function always returns a value of type
    'string'.
  * ``map`` is a function which takes 2 arguments.  The 1st argument
    is any string.  The second argument is the name of a map which is used to
    perform string substitution on the string from the 1st argument.
  * ``if`` is a function which takes a 3 arguments where the first argument
    must resolve to a boolean value.  If it evaluates to true, the second
    argument is returned; otherwise, the third argument is returned.

For example, the following expression evaluates to true if the user has
a distinguished name ending in "O=org1,C=US", or if the user has an affiliation
beginning with "org1.dept2." and also has the "admin" attribute of "true".

  **DN =~ "*O=org1,C=US" || (affiliation =~ "org1.dept2.*" && attr('admin') = 'true')**

NOTE: Since the ``attr`` function always returns a value of type 'string',
numeric operators may not be used to construct expressions.
For example, the following is NOT a valid expression:

.. code:: yaml

     value: attr("gidNumber) >= 10000 && attr("gidNumber) < 10006

Alternatively, a regular expression enclosed in quotes as shown below may be used
to return an equivalent result:

.. code:: yaml

     value: attr("gidNumber") =~ "1000[0-5]$" || attr("mail") == "root@example.com"

The following is a sample configuration section for the default setting
for the OpenLDAP server whose docker image is at
``https://github.com/osixia/docker-openldap``.

.. code:: yaml

    ldap:
       enabled: true
       url: ldap://cn=admin,dc=example,dc=org:admin@localhost:10389/dc=example,dc=org
       userfilter: (uid=%s)

See ``FABRIC_CA/scripts/run-ldap-tests`` for a script which starts an
OpenLDAP docker image, configures it, runs the LDAP tests in
``FABRIC_CA/cli/server/ldap/ldap_test.go``, and stops the OpenLDAP
server.

When LDAP is configured, enrollment works as follows:


-  The Fabric CA client or client SDK sends an enrollment request with a
   basic authorization header.
-  The Fabric CA server receives the enrollment request, decodes the
   identity name and password in the authorization header, looks up the DN (Distinguished
   Name) associated with the identity name using the "userfilter" from the
   configuration file, and then attempts an LDAP bind with the identity's
   password. If the LDAP bind is successful, the enrollment processing is
   authorized and can proceed.

Setting up a cluster
~~~~~~~~~~~~~~~~~~~~

You may use any IP sprayer to load balance to a cluster of Fabric CA
servers. This section provides an example of how to set up Haproxy to
route to a Fabric CA server cluster. Be sure to change hostname and port
to reflect the settings of your Fabric CA servers.

haproxy.conf

.. code::

    global
          maxconn 4096
          daemon

    defaults
          mode http
          maxconn 2000
          timeout connect 5000
          timeout client 50000
          timeout server 50000

    listen http-in
          bind *:7054
          balance roundrobin
          server server1 hostname1:port
          server server2 hostname2:port
          server server3 hostname3:port


Note: If using TLS, need to use ``mode tcp``.

启动多CA
~~~~~~~~~~~~~~~~~~~~~~~

默认情况下，fabric-ca服务器由一个默认的CA组成。
但是，可以使用 `cafiles` 或 `cacount` 配置选项向单个服务器添加额外的CA。
每个附加的CA都有自己的主目录。

cacount:
^^^^^^^^
CCANUT。主目录将与服务器目录相对应。使用此选项，目录结构如下：

`cacount` 提供了启动X个默认附加CA的快速方法。 主目录将与服务器目录相对应。使用此选项，目录结构如下：

.. code:: yaml

    --<Server Home>
      |--ca
        |--ca1
        |--ca2

每个附加的CA将获得在其主目录中生成的默认配置文件，在配置文件中它将包含唯一的CA名称。

例如，下面的命令将启动2个缺省CA实例：

.. code:: bash

   fabric-ca-server start -b admin:adminpw --cacount 2

cafiles:
^^^^^^^^

如果使用cafiles配置选项时没有提供绝对路径，则CA主目录将相对于服务器目录。

若要使用此选项，必须为每个要启动的CA生成和配置CA配置文件。
每个配置文件必须具有唯一的CA名称和公共名称（CN），否则服务器将无法启动，因为这些名称必须是唯一的。
CA配置文件将覆盖任何默认的CA配置，并且CA配置文件中的任何缺失选项都将由默认CA的值替换。

优先顺序如下：

  1. CA配置文件
  2. 默认CA CLI标志
  3. 默认CA环境变量
  4. 默认CA配置文件

CA配置文件必须至少包含以下内容：

.. code:: yaml

    ca:
    # Name of this CA
    name: <CANAME>

    csr:
      cn: <COMMONNAME>

您可以将目录结构配置如下：

.. code:: yaml

    --<Server Home>
      |--ca
        |--ca1
          |-- fabric-ca-config.yaml
        |--ca2
          |-- fabric-ca-config.yaml

例如，下面的命令将启动两个定制的CA实例：

.. code:: bash

    fabric-ca-server start -b admin:adminpw --cafiles ca/ca1/fabric-ca-config.yaml
    --cafiles ca/ca2/fabric-ca-config.yaml


注册中间CA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

为了给中间CA创建CA签名证书，中间CA必须以fabric-ca-client向CA注册相同的方式，向父CA注册。
这是通过使用 -u 选项指定父CA的URL以及注册ID和密码来完成的，正如下所示。
与此注册ID相关联的标识必须具有名为“hf.IntermediateCA”的属性和“true”的值。
颁发证书的CN（或公共名称）将被设置为注册ID（enrollment ID）。
如果中间CA试图显式指定CN值，则将发生错误。

.. code:: bash

    fabric-ca-server start -b admin:adminpw -u http://<enrollmentID>:<secret>@<parentserver>:<parentport>

对于其他中间CA标志，请参见 `Fabric CA server's configuration file format`_ 部分。

升级服务器
~~~~~~~~~~~~~~~~~~~~

在Fabric CA客户端之前，必须对Fabric CA服务器进行升级。
在升级之前，建议备份当前数据库：

- 如果使用sqlite3，则备份当前数据库文件（默认为命名为fabric-ca-server.db）。
- 对于其他数据库类型，使用适当的备份/复制机制。

升级织物CA服务器的单个实例：

启动织物CA服务器进程。通过以下命令验证.-ca-server进程可用，其中<host>是启动服务器的主机名：


1. 停止Fabric CA服务器进程。
2. 确保备份当前数据库。
3. 用升级版本替换以前的fabric-ca-server二进制文件。
4. 启动fabric-ca-server进程。
5. 通过以下命令验证fabric-ca-server进程是否可用，其中<host>是启动服务器的主机名::

      fabric-ca-client getcainfo -u http://<host>:7054

Upgrading a cluster:
^^^^^^^^^^^^^^^^^^^^
To upgrade a cluster of fabric-ca-server instances using either a MySQL or Postgres database, perform the following procedure. We assume that you are using haproxy to load balance to two fabric-ca-server cluster members on host1 and host2, respectively, both listening on port 7054. After this procedure, you will be load balancing to upgraded fabric-ca-server cluster members on host3 and host4 respectively, both listening on port 7054.

In order to monitor the changes using haproxy stats, enable statistics collection. Add the following lines to the global section of the haproxy configuration file:

::

    stats socket /var/run/haproxy.sock mode 666 level operator
    stats timeout 2m

Restart haproxy to pick up the changes::

    # haproxy -f <configfile> -st $(pgrep haproxy)

To display summary information from the haproxy "show stat" command, the following function may prove useful for parsing the copious amount of CSV data returned:

.. code:: bash

    haProxyShowStats() {
       echo "show stat" | nc -U /var/run/haproxy.sock |sed '1s/^# *//'|
          awk -F',' -v fmt="%4s %12s %10s %6s %6s %4s %4s\n" '
             { if (NR==1) for (i=1;i<=NF;i++) f[tolower($i)]=i }
             { printf fmt, $f["sid"],$f["pxname"],$f["svname"],$f["status"],
                           $f["weight"],$f["act"],$f["bck"] }'
    }


1) Initially your haproxy configuration file is similar to the following::

      server server1 host1:7054 check
      server server2 host2:7054 check

   Change this configuration to the following::

      server server1 host1:7054 check backup
      server server2 host2:7054 check backup
      server server3 host3:7054 check
      server server4 host4:7054 check

2) Restart the HA proxy with the new configuration as follows::

      haproxy -f <configfile> -st $(pgrep haproxy)

   ``"haProxyShowStats"`` will now reflect the modified configuration,
   with two active, older-version backup servers and two (yet to be started) upgraded servers::

      sid   pxname      svname  status  weig  act  bck
        1   fabric-cas  server3   DOWN     1    1    0
        2   fabric-cas  server4   DOWN     1    1    0
        3   fabric-cas  server1     UP     1    0    1
        4   fabric-cas  server2     UP     1    0    1

3) Install upgraded binaries of fabric-ca-server on host3 and host4. The new
   upgraded servers on host3 and host4 should be configured to use the same
   database as their older counterparts on host1 and host2. After starting
   the upgraded servers, the database will be automatically migrated. The
   haproxy will forward all new traffic to the upgraded servers, since they
   are not configured as backup servers. Verify using the ``"fabric-ca-client getcainfo"``
   command that your cluster is still functioning appropriately before proceeding.
   Also, ``"haProxyShowStats"`` should now reflect that all servers are active,
   similar to the following::

      sid   pxname      svname  status  weig  act  bck
        1   fabric-cas  server3    UP     1    1    0
        2   fabric-cas  server4    UP     1    1    0
        3   fabric-cas  server1    UP     1    0    1
        4   fabric-cas  server2    UP     1    0    1

4) Stop the old servers on host1 and host2. Verify using the
   ``"fabric-ca-client getcainfo"`` command that your new cluster is still
   functioning appropriately before proceeding. Then remove the older
   server backup configuration from the haproxy configuration file,
   so that it looks similar to the following::

      server server3 host3:7054 check
      server server4 host4:7054 check

5) Restart the HA proxy with the new configuration as follows::

      haproxy -f <configfile> -st $(pgrep haproxy)

   ``"haProxyShowStats"`` will now reflect the modified configuration,
   with two active servers which have been upgraded to the new version::

      sid   pxname      svname  status  weig  act  bck
        1   fabric-cas  server3   UP       1    1    0
        2   fabric-cas  server4   UP       1    1    0


`回到顶端`_



.. _client:

Fabric CA 客户端
----------------

本节介绍如何使用fabric-ca-client命令。

Fabric CA客户端的主目录确定如下：


This section describes how to use the fabric-ca-client command.

The Fabric CA client's home directory is determined as follows:
  - 如果设置了 --home 命令行选项，则使用它的值
  - 否则，如果设置了 ``FABRIC_CA_CLIENT_HOME`` 环境变量，则使用其值
  - 否则，如果设置了 ``FABRIC_CA_HOME`` 环境变量，则使用其值。
  - 否则，如果设置了 ``CA_CFG_PATH`` 环境变量，则使用其值。
  - 否则，使用 ``$HOME/.fabric-ca-client``

下面的说明，假定客户端配置文件存在于客户端的主目录中。

注册启动身份
~~~~~~

首先，如果需要，在客户端配置文件中自定义CSR（证书签名请求）部分。
注意，必须将 ``csr.cn`` 字段设置为引导标识的ID。默认CSR值如下所示：

.. code:: yaml

    csr:
      cn: <<enrollment ID>>
      key:
        algo: ecdsa
        size: 256
      names:
        - C: US
          ST: North Carolina
          L:
          O: Hyperledger Fabric
          OU: Fabric CA
      hosts:
       - <<hostname of the fabric-ca-client>>
      ca:
        pathlen:
        pathlenzero:
        expiry:

CSR字段来描述字段。

参见 `CSR fields <#csr-fields>`__ 来查看这些字段的描述。

然后运行 ``fabric-ca-client enroll`` 命令来注册身份。
例如，以下命令通过调用本地在7054端口运行的Fabric CA服务器来注册ID为 **admin** 和密码为 **adminpw** 的身份。

.. code:: bash

    export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
    fabric-ca-client enroll -u http://admin:adminpw@localhost:7054

注册命令在Fabric CA客户端的 ``msp`` 目录的子目录中存储注册证书（ECert）、相应的私钥和CA证书链PEM文件。
您将看到指示存储PEM文件的位置的消息。

登记一个新的身份
~~~~~~~~

执行注册请求的身份必须当场（currently）注册，并且还必须具有注册正在注册的身份类型的适当权限。

特别地，在注册期间，由Fabric CA服务器进行的三个授权检查如下：

1. The registrar (i.e. the invoker) must have the "hf.Registrar.Roles" attribute with a
   comma-separated list of values where one of the values equals the type of
   identity being registered; for example, if the registrar has the
   "hf.Registrar.Roles" attribute with a value of "peer,app,user", the registrar
   can register identities of type peer, app, and user, but not orderer.

2. The affiliation of the registrar must be equal to or a prefix of
   the affiliation of the identity being registered.  For example, an registrar
   with an affiliation of "a.b" may register an identity with an affiliation
   of "a.b.c" but may not register an identity with an affiliation of "a.c".
   If root affiliation is required for an identity, then the affiliation request
   should be a dot (".") and the registrar must also have root affiliation.
   If no affiliation is specified in the registration request, the identity being
   registered will be given the affiliation of the registrar.

3. The registrar can register a user with attributes if all of the following conditions
   are satisfied:

   - Registrar can register Fabric CA reserved attributes that have the prefix 'hf.'
     only if the registrar possesses the attribute and it is part of the value of the
     hf.Registrar.Attributes' attribute. Furthermore, if the attribute is of type list
     then the value of attribute being registered must be equal to or a subset of the
     value that the registrar has. If the attribute is of type boolean, the registrar
     can register the attribute only if the registrar's value for the attribute is 'true'.
   - Registering custom attributes (i.e. any attribute whose name does not begin with 'hf.')
     requires that the registrar has the 'hf.Registar.Attributes' attribute with the value of
     the attribute or pattern being registered. The only supported pattern is a string with
     a "*" at the end. For example, "a.b.*" is a pattern which matches all attribute names
     beginning with "a.b.". For example, if the registrar has hf.Registrar.Attributes=orgAdmin,
     then the only attribute which the registrar can add or remove from an identity is the
     'orgAdmin' attribute.
   - If the requested attribute name is 'hf.Registrar.Attributes', an additional
     check is performed to see if the requested values for this attribute are equal
     to or a subset of the registrar's values for 'hf.Registrar.Attributes'. For this
     to be true, each requested value must match a value in the registrar's value for
     'hf.Registrar.Attributes' attribute. For example, if the registrar's value for
     'hf.Registrar.Attributes' is 'a.b.*, x.y.z' and the requested attribute
     value is 'a.b.c, x.y.z', it is valid because 'a.b.c' matches 'a.b.*' and 'x.y.z'
     matches the registrar's 'x.y.z' value.

Examples:
   Valid Scenarios:
      1. If the registrar has the attribute 'hf.Registrar.Attributes = a.b.*, x.y.z' and
         is registering attribute 'a.b.c', it is valid 'a.b.c' matches 'a.b.*'.
      2. If the registrar has the attribute 'hf.Registrar.Attributes = a.b.*, x.y.z' and
         is registering attribute 'x.y.z', it is valid because 'x.y.z' matches the registrar's
         'x.y.z' value.
      3. If the registrar has the attribute 'hf.Registrar.Attributes = a.b.*, x.y.z' and
         the requested attribute value is 'a.b.c, x.y.z', it is valid because 'a.b.c' matches
         'a.b.*' and 'x.y.z' matches the registrar's 'x.y.z' value.
      4. If the registrar has the attribute 'hf.Registrar.Roles = peer,client' and
         the requested attribute value is 'peer' or 'peer,client', it is valid because
         the requested value is equal to or a subset of the registrar's value.

   Invalid Scenarios:
      1. If the registrar has the attribute 'hf.Registrar.Attributes = a.b.*, x.y.z' and
         is registering attribute 'hf.Registar.Attributes = a.b.c, x.y.*', it is invalid
         because requested attribute 'x.y.*' is not a pattern owned by the registrar. The value
         'x.y.*' is a superset of 'x.y.z'.
      2. If the registrar has the attribute 'hf.Registrar.Attributes = a.b.*, x.y.z' and
         is registering attribute 'hf.Registar.Attributes = a.b.c, x.y.z, attr1', it is invalid
         because the registrar's 'hf.Registrar.Attributes' attribute values do not contain 'attr1'.
      3. If the registrar has the attribute 'hf.Registrar.Attributes = a.b.*, x.y.z' and
         is registering attribute 'a.b', it is invalid because the value 'a.b' is not contained in
         'a.b.*'.
      4. If the registrar has the attribute 'hf.Registrar.Attributes = a.b.*, x.y.z' and
         is registering attribute 'x.y', it is invalid because 'x.y' is not contained by 'x.y.z'.
      5. If the registrar has the attribute 'hf.Registrar.Roles = peer,client' and
         the requested attribute value is 'peer,client,orderer', it is invalid because
         the registrar does not have the orderer role in its value of hf.Registrar.Roles
         attribute.
      6. If the registrar has the attribute 'hf.Revoker = false' and the requested attribute
         value is 'true', it is invalid because the hf.Revoker attribute is a boolean attribute
         and the registrar's value for the attribute is not 'true'.

下表列出了可以为身份注册的所有属性。属性的名称是区分大小写的。

+-----------------------------+------------+------------------------------------------------------------------------------------------------------------+
| 名称                         | 类型        | 描述                                                                                                |
+=============================+============+============================================================================================================+
| hf.Registrar.Roles          | List       | List of roles that the registrar is allowed to manage                                                      |
+-----------------------------+------------+------------------------------------------------------------------------------------------------------------+
| hf.Registrar.DelegateRoles  | List       | List of roles that the registrar is allowed to give to a registree for its 'hf.Registrar.Roles' attribute  |
+-----------------------------+------------+------------------------------------------------------------------------------------------------------------+
| hf.Registrar.Attributes     | List       | List of attributes that registrar is allowed to register                                                   |
+-----------------------------+------------+------------------------------------------------------------------------------------------------------------+
| hf.GenCRL                   | Boolean    | Identity is able to generate CRL if attribute value is true                                                |
+-----------------------------+------------+------------------------------------------------------------------------------------------------------------+
| hf.Revoker                  | Boolean    | Identity is able to revoke a user and/or certificates if attribute value is true                           |
+-----------------------------+------------+------------------------------------------------------------------------------------------------------------+
| hf.AffiliationMgr           | Boolean    | Identity is able to manage affiliations if attribute value is true                                         |
+-----------------------------+------------+------------------------------------------------------------------------------------------------------------+
| hf.IntermediateCA           | Boolean    | Identity is able to enroll as an intermediate CA if attribute value is true                                |
+-----------------------------+------------+------------------------------------------------------------------------------------------------------------+

Note: When registering an identity, you specify an array of attribute names and values. If the array
specifies multiple array elements with the same name, only the last element is currently used. In other words,
multi-valued attributes are not currently supported.

The following command uses the **admin** identity's credentials to register a new
user with an enrollment id of "admin2", an affiliation of
"org1.department1", an attribute named "hf.Revoker" with a value of "true", and
an attribute named "admin" with a value of "true".  The ":ecert" suffix means that
by default the "admin" attribute and its value will be inserted into the user's
enrollment certificate, which can then be used to make access control decisions.

.. code:: bash

    export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
    fabric-ca-client register --id.name admin2 --id.affiliation org1.department1 --id.attrs 'hf.Revoker=true,admin=true:ecert'

The password, also known as the enrollment secret, is printed.
This password is required to enroll the identity.
This allows an administrator to register an identity and give the
enrollment ID and the secret to someone else to enroll the identity.

Multiple attributes can be specified as part of the --id.attrs flag, each
attribute must be comma separated. For an attribute value that contains a comma,
the attribute must be encapsulated in double quotes. See example below.

.. code:: bash

    fabric-ca-client register -d --id.name admin2 --id.affiliation org1.department1 --id.attrs '"hf.Registrar.Roles=peer,user",hf.Revoker=true'

or

.. code:: bash

    fabric-ca-client register -d --id.name admin2 --id.affiliation org1.department1 --id.attrs '"hf.Registrar.Roles=peer,user"' --id.attrs hf.Revoker=true

You may set default values for any of the fields used in the register command
by editing the client's configuration file.  For example, suppose the configuration
file contains the following:

.. code:: yaml

    id:
      name:
      type: user
      affiliation: org1.department1
      maxenrollments: -1
      attributes:
        - name: hf.Revoker
          value: true
        - name: anotherAttrName
          value: anotherAttrValue

The following command would then register a new identity with an enrollment id of
"admin3" which it takes from the command line, and the remainder is taken from the
configuration file including the identity type: "user", affiliation: "org1.department1",
and two attributes: "hf.Revoker" and "anotherAttrName".

.. code:: bash

    export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
    fabric-ca-client register --id.name admin3

To register an identity with multiple attributes requires specifying all attribute names and values
in the configuration file as shown above.

Setting `maxenrollments` to 0 or leaving it out from the configuration will result in the identity
being registered to use the CA's max enrollment value. Furthermore, the max enrollment value for
an identity being registered cannot exceed the CA's max enrollment value. For example, if the CA's
max enrollment value is 5. Any new identity must have a value less than or equal to 5, and also
can't set it to -1 (infinite enrollments).

Next, let's register a peer identity which will be used to enroll the peer in the following section.
The following command registers the **peer1** identity.  Note that we choose to specify our own
password (or secret) rather than letting the server generate one for us.

.. code:: bash

    export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
    fabric-ca-client register --id.name peer1 --id.type peer --id.affiliation org1.department1 --id.secret peer1pw

Note that affiliations are case sensitive except for the non-leaf affiliations that are specified in
the server configuration file, which are always stored in lower case. For example, if the affiliations
section of the server configuration file looks like this:

.. code:: bash

    affiliations:
      BU1:
        Department1:
          - Team1
      BU2:
        - Department2
        - Department3

`BU1`, `Department1`, `BU2` are stored in lower case. This is because Fabric CA uses Viper to read configuration.
Viper treats map keys as case insensitive and always returns lowercase value. To register an identity with
`Team1` affiliation, `bu1.department1.Team1` would need to be specified to the
`--id.affiliation` flag as shown below:

.. code:: bash

    export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
    fabric-ca-client register --id.name client1 --id.type client --id.affiliation bu1.department1.Team1

Enrolling a peer identity
~~~~~~~~~~~~~~~~~~~~~~~~~

Now that you have successfully registered a peer identity, you may now
enroll the peer given the enrollment ID and secret (i.e. the *password*
from the previous section).  This is similar to enrolling the bootstrap identity
except that we also demonstrate how to use the "-M" option to populate the
Hyperledger Fabric MSP (Membership Service Provider) directory structure.

The following command enrolls peer1.
Be sure to replace the value of the "-M" option with the path to your
peer's MSP directory which is the
'mspConfigPath' setting in the peer's core.yaml file.
You may also set the FABRIC_CA_CLIENT_HOME to the home directory of your peer.

.. code:: bash

    export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/peer1
    fabric-ca-client enroll -u http://peer1:peer1pw@localhost:7054 -M $FABRIC_CA_CLIENT_HOME/msp

Enrolling an orderer is the same, except the path to the MSP directory is
the 'LocalMSPDir' setting in your orderer's orderer.yaml file.

All enrollment certificates issued by the fabric-ca-server have organizational
units (or "OUs" for short) as follows:

1. The root of the OU hierarchy equals the identity type
2. An OU is added for each component of the identity's affiliation

For example, if an identity is of type `peer` and its affiliation is
`department1.team1`, the identity's OU hierarchy (from leaf to root) is
`OU=team1, OU=department1, OU=peer`.

Getting a CA certificate chain from another Fabric CA server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In general, the cacerts directory of the MSP directory must contain the certificate authority chains
of other certificate authorities, representing all of the roots of trust for the peer.

The ``fabric-ca-client getcainfo`` command is used to retrieve these certificate chains from other
Fabric CA server instances.

For example, the following will start a second Fabric CA server on localhost
listening on port 7055 with a name of "CA2".  This represents a completely separate
root of trust and would be managed by a different member on the blockchain.

.. code:: bash

    export FABRIC_CA_SERVER_HOME=$HOME/ca2
    fabric-ca-server start -b admin:ca2pw -p 7055 -n CA2

The following command will install CA2's certificate chain into peer1's MSP directory.

.. code:: bash

    export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/peer1
    fabric-ca-client getcainfo -u http://localhost:7055 -M $FABRIC_CA_CLIENT_HOME/msp

By default, the Fabric CA server returns the CA chain in child-first order. This means that each CA
certificate in the chain is followed by its issuer's CA certificate. If you need the Fabric CA server
to return the CA chain in the opposite order, then set the environment variable ``CA_CHAIN_PARENT_FIRST``
to ``true`` and restart the Fabric CA server. The Fabric CA client will handle either order appropriately.

重新注册身份
~~~~~~~~~~~~~~~~~~~~~~~

假设你的入学证书即将到期。您可以发布 reenroll 命令来更新您的注册证书，就像下面这样操作：

.. code:: bash

    export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/peer1
    fabric-ca-client reenroll

吊销证书或身份
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

可以取消身份或证书。撤销身份将撤销该身份所拥有的所有证书，并且还将阻止该身份获得任何新证书。

吊销证书将使单个证书无效。为了撤销证书或身份，调用身份必须具有 ``hf.Revoker` 和 ``hf.Registrar.Roles`` 属性。
撤消身份只能撤消具有与撤消身份所属关系相等或前缀的附属关系的证书或身份。
此外，撤消者只能撤销在撤消者的 ``hf.Registrar.Roles`` 角色属性中列出的类型的身份。

例如，具有关联 **orgs.org1** 和 'hf.Registrar.Roles=peer,client' 属性的撤销器，
可以撤销与　**orgs.org1**  或 **orgs.org1.department1** 相关联的 **peer** 或 **client** 类型身份，
但不能撤销与 **orgs.org2**  或任何其他类型相关联的标识。下

面的命令禁用身份并撤销与该身份相关联的所有证书。
所有Fabric CA服务器接收到的来自该身份的请求都将被拒绝。

.. code:: bash

    fabric-ca-client revoke -e <enrollment_id> -r <reason>

以下是可以使用 ``-r`` 标志指定的支持的原因：

  1. unspecified
  2. keycompromise
  3. cacompromise
  4. affiliationchange
  5. superseded
  6. cessationofoperation
  7. certificatehold
  8. removefromcrl
  9. privilegewithdrawn
  10. aacompromise

例如，与关联树的根关联的bootstrap admin，可以按照如下方式撤销 **peer1** 的身份：

.. code:: bash

    export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
    fabric-ca-client revoke -e peer1

通过指定其AKI（Authority Key Identifier：授权密钥标识符）和序列号，
可以撤销属于某身份的注册证书（enrollment certificate）：

.. code:: bash

    fabric-ca-client revoke -a xxx -s yyy -r <reason>

例如，可以使用openssl命令获得证书的AKI和序列号，并将其传递给 ``revoke`` 命令，
以便按以下方式撤销所述证书：

.. code:: bash

   serial=$(openssl x509 -in userecert.pem -serial -noout | cut -d "=" -f 2)
   aki=$(openssl x509 -in userecert.pem -text | awk '/keyid/ {gsub(/ *keyid:|:/,"",$1);print tolower($0)}')
   fabric-ca-client revoke -s $serial -a $aki -r affiliationchange

`--gencrl` 标志可用于生成包含所有撤销证书的CRL（证书吊销列表）。
例如，下面的命令将撤销标识对等点1，生成一个CRL并将其存储在 **<msp 文件夹>/crls/crl.pem** 文件中。

.. code:: bash

    fabric-ca-client revoke -e peer1 --gencrl

还可以使用 `gencrl` 命令生成CRL。有关 `gencrl` 命令的更多信息，请参阅
`Generating a CRL (Certificate Revocation List)`_
部分。

生成CRL(证书吊销列表：Certificate Revocation List)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
在Fabric CA服务器中撤销证书之后，还必须更新Hyperledger Fabric中的对应MSP。
这既包括peer的本地MSP，也包括适当通道配置块中的MSP。
为此，必须将PEM编码的CRL（证书吊销列表）文件放置在MSP的 `crls` 文件夹中。
可以使用Fabric CA客户端 ``fabric-ca-client gencrl`` 命令生成CRL。
任何具有 ``hf.GenCRL`` 属性的身份都可以创建一个CRL，该CRL包含某个时期内撤销的所有证书的序列号。
创建的CRL存储在 `<msp 文件夹>/crls/crl.pem` 文件中。

下面的命令将创建一个包含所有撤销的证书（过期和未到期）的CRL，并将CRL存储在 `~/msp/crls/crl.pem` 文件中。

.. code:: bash

    export FABRIC_CA_CLIENT_HOME=~/clientconfig
    fabric-ca-client gencrl -M ~/msp

下一个命令将创建包含所有特定证书（过期和未过期）的CRL，这些证书在2017～0913T16:39:55-0800（由 `--revokedafter` 标志指定）之后，
在2017～0921T16:39:55-0800（由 `--revokedbefore` 指定）之前。CRL存储在 `~/msp/crls/crl.pem` 文件中。

.. code:: bash

    export FABRIC_CA_CLIENT_HOME=~/clientconfig
    fabric-ca-client gencrl --caname "" --revokedafter 2017-09-13T16:39:57-08:00 --revokedbefore 2017-09-21T16:39:57-08:00 -M ~/msp

`--caname` 标识指明了命令被发送往的CA的名称。在这个例子里，gencrl请求被发送到默认的CA。

`--revokedafter` 和 `--revokedbefore` 标识指明了一个时间段的上限和下限。
生成的CRL将会包含这段时间内吊销的证书。
值必须是以RFC3339格式表示的UTC时间戳。 `--revokedafter` 不能比 `--revokedbefore` 时间戳大.

默认, 'Next Update' CRL日期被设定为下一天。 `crl.expiry` CA 配置属性可以同来指定一个自定义值。

gencrl命令还将接受 `--expireafter` 和 `--expirebefore` 标记，
这些标记可用于生成具有特定撤销证书的CRL，这些证书在这些标记指定的期间过期。
例如，以下命令将生成一个CRL，该CRL包含在 2017-09-13T16:39:57-08:00 之后和 2017-09-21T16:39:57-08:00 之前被撤销，
并在 2017-09-13T16:39:57-08:00 之后和 2018-09-13T16:39:57-08:00 之前过期的证书。

.. code:: bash

    export FABRIC_CA_CLIENT_HOME=~/clientconfig
    fabric-ca-client gencrl --caname "" --expireafter 2017-09-13T16:39:57-08:00 --expirebefore 2018-09-13T16:39:57-08:00  --revokedafter 2017-09-13T16:39:57-08:00 --revokedbefore 2017-09-21T16:39:57-08:00 -M ~/msp

`fabric-samples/fabric-ca <https://github.com/hyperledger/fabric-samples/blob/master/fabric-ca/scripts/run-fabric.sh>`_
示例演示如何生成包含被撤销的用户所拥有证书的CRL并更新通道msp。
然后，将证明使用撤销的用户凭据来查询通道，将导致授权错误。

启用TLS
~~~~~

本节将更详细地描述如何为Fabric CA客户端配置TLS。
以下部分可以配置在 ``fabric-ca-client-config.yaml`` 中。

.. code:: yaml

    tls:
      # Enable TLS (default: false)
      enabled: true
      certfiles:
        - root.pem
      client:
        certfile: tls_client-cert.pem
        keyfile: tls_client-key.pem

**certfiles** 选项是客户端信任的根证书的集合。
这通常就是服务器home目录中找到的根Fabric CA服务器证书，即**ca-cert.pem**文件。

只有在服务器上配置mutual TLS时才需要 **client** 选项。

Attribute-Based Access Control
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Access control decisions can be made by chaincode (and by the Hyperledger Fabric runtime)
based upon an identity's attributes.  This is called
**Attribute-Based Access Control**, or **ABAC** for short.

In order to make this possible, an identity's enrollment certificate (ECert)
may contain one or more attribute name and value.  The chaincode then
extracts an attribute's value to make an access control decision.

For example, suppose that you are developing application *app1* and want a
particular chaincode operation to be accessible only by app1 administrators.
Your chaincode could verify that the caller's certificate (which was issued by
a CA trusted for the channel) contains an attribute named *app1Admin* with a
value of *true*.  Of course the name of the attribute can be anything and the
value need not be a boolean value.

So how do you get an enrollment certificate with an attribute?
There are two methods:

1.   When you register an identity, you can specify that an enrollment certificate
     issued for the identity should by default contain an attribute.  This behavior
     can be overridden at enrollment time, but this is useful for establishing
     default behavior and, assuming registration occurs outside of your application,
     does not require any application change.

     The following shows how to register *user1* with two attributes:
     *app1Admin* and *email*.
     The ":ecert" suffix causes the *appAdmin* attribute to be inserted into user1's
     enrollment certificate by default, when the user does not explicitly request
     attributes at enrollment time.  The *email* attribute is not added
     to the enrollment certificate by default.

.. code:: bash

     fabric-ca-client register --id.name user1 --id.secret user1pw --id.type user --id.affiliation org1 --id.attrs 'app1Admin=true:ecert,email=user1@gmail.com'

2. When you enroll an identity, you may explicitly request that one or more attributes
   be added to the certificate.
   For each attribute requested, you may specify whether the attribute is
   optional or not.  If it is not requested optionally and the identity does
   not possess the attribute, an error will occur.

   The following shows how to enroll *user1* with the *email* attribute,
   without the *app1Admin* attribute, and optionally with the *phone*
   attribute (if the user possesses the *phone* attribute).

.. code:: bash

   fabric-ca-client enroll -u http://user1:user1pw@localhost:7054 --enrollment.attrs "email,phone:opt"

The table below shows the three attributes which are automatically registered for every identity.

===================================   =====================================
     Attribute Name                               Attribute Value
===================================   =====================================
  hf.EnrollmentID                        The enrollment ID of the identity
  hf.Type                                The type of the identity
  hf.Affiliation                         The affiliation of the identity
===================================   =====================================

To add any of the above attributes **by default** to a certificate, you must
explicitly register the attribute with the ":ecert" specification.
For example, the following registers identity 'user1' so that
the 'hf.Affiliation' attribute will be added to an enrollment certificate if
no specific attributes are requested at enrollment time.  Note that the
value of the affiliation (which is 'org1') must be the same in both the
'--id.affiliation' and the '--id.attrs' flags.

.. code:: bash

    fabric-ca-client register --id.name user1 --id.secret user1pw --id.type user --id.affiliation org1 --id.attrs 'hf.Affiliation=org1:ecert'

For information on the chaincode library API for Attribute-Based Access Control,
see `https://github.com/hyperledger/fabric/tree/release-1.1/core/chaincode/lib/cid/README.md <https://github.com/hyperledger/fabric/tree/release-1.1/core/chaincode/lib/cid/README.md>`_

For an end-to-end sample which demonstrates Attribute-Based Access Control and more,
see `https://github.com/hyperledger/fabric-samples/tree/release-1.1/fabric-ca/README.md <https://github.com/hyperledger/fabric-samples/tree/release-1.1/fabric-ca/README.md>`_

Dynamic Server Configuration Update
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This section describes how to use fabric-ca-client to dynamically update portions
of the fabric-ca-server's configuration without restarting the server.

All commands in this section require that you first be enrolled by executing the
`fabric-ca-client enroll` command.

Dynamically updating identities
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This section describes how to use fabric-ca-client to dynamically update identities.

An authorization failure will occur if the client identity does not satisfy all of the following:

 - The client identity must possess the "hf.Registrar.Roles" attribute with a comma-separated list of
   values where one of the values equals the type of identity being updated; for example, if the client's
   identity has the "hf.Registrar.Roles" attribute with a value of "client,peer", the client can update
   identities of type 'client' and 'peer', but not 'orderer'.

 - The affiliation of the client's identity must be equal to or a prefix of the affiliation of the identity
   being updated.  For example, a client with an affiliation of "a.b" may update an identity with an affiliation
   of "a.b.c" but may not update an identity with an affiliation of "a.c". If root affiliation is required for an
   identity, then the update request should specify a dot (".") for the affiliation and the client must also have
   root affiliation.

The following shows how to add, modify, and remove an affiliation.

Getting Identity Information
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A caller may retrieve information on a identity from the fabric-ca server as long as the caller meets
the authorization requirements highlighted in the section above. The following command shows how to get an
identity.

.. code:: bash

    fabric-ca-client identity list --id user1

A caller may also request to retrieve information on all identities that it is authorized to see by
issuing the following command.

.. code:: bash

    fabric-ca-client identity list

Adding an identity
"""""""""""""""""""

The following adds a new identity for 'user1'. Adding a new identity performs the same action as registering an
identity via the 'fabric-ca-client register' command. There are two available methods for adding a new identity.
The first method is via the `--json` flag where you describe the identity in a JSON string.

.. code:: bash

    fabric-ca-client identity add user1 --json '{"secret": "user1pw", "type": "user", "affiliation": "org1", "max_enrollments": 1, "attrs": [{"name": "hf.Revoker", "value": "true"}]}'

The following adds a user with root affiliation. Note that an affiliation name of "." means the root affiliation.

.. code:: bash

    fabric-ca-client identity add user1 --json '{"secret": "user1pw", "type": "user", "affiliation": ".", "max_enrollments": 1, "attrs": [{"name": "hf.Revoker", "value": "true"}]}'

The second method for adding an identity is to use direct flags. See the example below for adding 'user1'.

.. code:: bash

    fabric-ca-client identity add user1 --secret user1pw --type user --affiliation . --maxenrollments 1 --attrs hf.Revoker=true

The table below lists all the fields of an identity and whether they are required or optional, and any default values they might have.

+----------------+------------+------------------------+
| Fields         | Required   | Default Value          |
+================+============+========================+
| ID             | Yes        |                        |
+----------------+------------+------------------------+
| Secret         | No         |                        |
+----------------+------------+------------------------+
| Affiliation    | No         | Caller's Affiliation   |
+----------------+------------+------------------------+
| Type           | No         | client                 |
+----------------+------------+------------------------+
| Maxenrollments | No         | 0                      |
+----------------+------------+------------------------+
| Attributes     | No         |                        |
+----------------+------------+------------------------+


Modifying an identity
""""""""""""""""""""""

There are two available methods for modifying an existing identity. The first method is via the `--json` flag where you describe
the modifications in to an identity in a JSON string. Multiple modifications can be made in a single request. Any element of an identity that
is not modified will retain its original value.

NOTE: A maxenrollments value of "-2" specifies that the CA's max enrollment setting is to be used.

The command below make multiple modification to an identity using the --json flag.

.. code:: bash

    fabric-ca-client identity modify user1 --json '{"secret": "newPassword", "affiliation": ".", "attrs": [{"name": "hf.Regisrar.Roles", "value": "peer,client"},{"name": "hf.Revoker", "value": "true"}]}'

The commands below make modifications using direct flags. The following updates the enrollment secret (or password) for identity 'user1' to 'newsecret'.

.. code:: bash

    fabric-ca-client identity modify user1 --secret newsecret

The following updates the affiliation of identity 'user1' to 'org2'.

.. code:: bash

    fabric-ca-client identity modify user1 --affiliation org2

The following updates the type of identity 'user1' to 'peer'.

.. code:: bash

    fabric-ca-client identity modify user1 --type peer


The following updates the maxenrollments of identity 'user1' to 5.

.. code:: bash

    fabric-ca-client identity modify user1 --maxenrollments 5

By specifying a maxenrollments value of '-2', the following causes identity 'user1' to use
the CA's max enrollment setting.

.. code:: bash

    fabric-ca-client identity modify user1 --maxenrollments -2

The following sets the value of the 'hf.Revoker' attribute for identity 'user1' to 'false'.
If the identity has other attributes, they are not changed.  If the identity did not previously
possess the 'hf.Revoker' attribute, the attribute is added to the identity. An attribute may
also be removed by specifying no value for the attribute.

.. code:: bash

    fabric-ca-client identity modify user1 --attrs hf.Revoker=false

The following removes the 'hf.Revoker' attribute for user 'user1'.

.. code:: bash

    fabric-ca-client identity modify user1 --attrs hf.Revoker=

The following demonstrates that multiple options may be used in a single `fabric-ca-client identity modify`
command. In this case, both the secret and the type are updated for user 'user1'.

.. code:: bash

    fabric-ca-client identity modify user1 --secret newpass --type peer

Removing an identity
"""""""""""""""""""""

The following removes identity 'user1' and also revokes any certificates associated with the 'user1' identity.

.. code:: bash

    fabric-ca-client identity remove user1

Note: Removal of identities is disabled in the fabric-ca-server by default, but may be enabled
by starting the fabric-ca-server with the `--cfg.identities.allowremove` option.

Dynamically updating affiliations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This section describes how to use fabric-ca-client to dynamically update affiliations. The
following shows how to add, modify, remove, and list an affiliation.

Adding an affiliation
"""""""""""""""""""""""

An authorization failure will occur if the client identity does not satisfy all of the following:

  - The client identity must possess the attribute 'hf.AffiliationMgr' with a value of 'true'.
  - The affiliation of the client identity must be hierarchically above the affiliation being updated.
    For example, if the client's affiliation is "a.b", the client may add affiliation "a.b.c" but not
    "a" or "a.b".

The following adds a new affiliation named ‘org1.dept1’.

.. code:: bash

    fabric-ca-client affiliation add org1.dept1

Modifying an affiliation
"""""""""""""""""""""""""

An authorization failure will occur if the client identity does not satisfy all of the following:

  - The client identity must possess the attribute 'hf.AffiliationMgr' with a value of 'true'.
  - The affiliation of the client identity must be hierarchically above the affiliation being updated.
    For example, if the client's affiliation is "a.b", the client may add affiliation "a.b.c" but not
    "a" or "a.b".
  - If the '--force' option is true and there are identities which must be modified, the client
    identity must also be authorized to modify the identity.

The following renames the 'org2' affiliation to 'org3'.  It also renames any sub affiliations
(e.g. 'org2.department1' is renamed to 'org3.department1').

.. code:: bash

    fabric-ca-client affiliation modify org2 --name org3

If there are identities that are affected by the renaming of an affiliation, it will result in
an error unless the '--force' option is used. Using the '--force' option will update the affiliation
of identities that are affected to use the new affiliation name.

.. code:: bash

    fabric-ca-client affiliation modify org1 --name org2 --force

Removing an affiliation
"""""""""""""""""""""""""

An authorization failure will occur if the client identity does not satisfy all of the following:

  - The client identity must possess the attribute 'hf.AffiliationMgr' with a value of 'true'.
  - The affiliation of the client identity must be hierarchically above the affiliation being updated.
    For example, if the client's affiliation is "a.b", the client may remove affiliation "a.b.c" but not
    "a" or "a.b".
  - If the '--force' option is true and there are identities which must be modified, the client
    identity must also be authorized to modify the identity.

The following removes affiliation 'org2' and also any sub affiliations.
For example, if 'org2.dept1' is an affiliation below 'org2', it is also removed.

.. code:: bash

    fabric-ca-client affiliation remove org2

If there are identities that are affected by the removing of an affiliation, it will result
in an error unless the '--force' option is used. Using the '--force' option will also remove
all identities that are associated with that affiliation, and the certificates associated with
any of these identities.

Note: Removal of affiliations is disabled in the fabric-ca-server by default, but may be enabled
by starting the fabric-ca-server with the `--cfg.affiliations.allowremove` option.

Listing affiliation information
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An authorization failure will occur if the client identity does not satisfy all of the following:

  - The client identity must possess the attribute 'hf.AffiliationMgr' with a value of 'true'.
  - Affiliation of the client identity must be equal to or be hierarchically above the
    affiliation being updated. For example, if the client's affiliation is "a.b",
    the client may get affiliation information on "a.b" or "a.b.c" but not "a" or "a.c".

The following command shows how to get a specific affiliation.

.. code:: bash

    fabric-ca-client affiliation list --affiliation org2.dept1

A caller may also request to retrieve information on all affiliations that it is authorized to see by
issuing the following command.

.. code:: bash

    fabric-ca-client affiliation list

管理证书
~~~~~~~~~~~~~~~~~~~~

本节介绍如何使用Fabric CA客户端管理证书。调用方可见的证书包括：

列出证书信息
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^



调用方可见的证书包括：

  - 属于调用者的证书
  - 如果调用方拥有值为true的 ``hf.Registrar.Roles`` 属性或 ``hf.Revoker`` 属性，
    则所有属于调用方从属关系之内及之下的身份的证书。例如，如果客户的关联是 ``a.b``，
    则客户可以获得属于 ``a.b`` 或 ``a.b.c``，
    但不是 ``a`` 或 ``a.c`` 的身份证书。

如果执行请求多个身份（identity）的证书的列表命令，则将只列出具有与调用者的附属（affiliation）相等或下属的附属的身份证书。

将列出的证书可以基于ID、AKI、序列号、过期时间、撤销时间、notrevoked和notexpired标志进行筛选。

* ``id``: 列出这个注册ID的证书
* ``serial``: 列出具有这个序列号的证书
* ``aki``: 列出具有这个AKI的证书
* ``expiration``: 列出到期日期在该到期时间内的证书
* ``revocation``: 列出在该吊销时间内撤销的证书
* ``notrevoked``: 列出尚未被撤销的证书
* ``notexpired``: 列出尚未过期的证书

可以使用 ``notexpired`` 和 ``notrevoked`` 标志作为筛选器，从结果集中排除撤销的证书 和/或 过期证书。
例如，如果只关心已经过期但尚未撤销的证书，则可以使用 ``expiration`` 标志和 ``notrevoked`` 标志来返回这样的结果。

下面提供了这种情况的一个例子。时间应根据RFC3339指定。
例如，为了列出在2018年3月1日下午1点到2018年6月15日上午2点之间到期的证书，
输入的时间串就看起来像 2018-03-01T13:00:00z 和2 2018-06-15T02:00:00z 。
如果具体时分秒不是考虑事项，只有日期才重要，那么时间部分便被去掉，
字符串成为 2018—03-01 和 2018—06—15。

字符串 ``now`` 现在可以用来表示当前时间，而空字符串来表示任何时间。
例如，``now::`` 表示从现在到将来的任何时间的时间范围，而 `::now` 表示从过去到现在的任何时间的时间范围。

下面的命令演示如何使用各种筛选器列出证书。

列出所有证书：

.. code:: bash

 fabric-ca-client certificate list

按ID列出所有证书：

.. code:: bash

 fabric-ca-client certificate list --id admin

通过serial和AKI列出证书：te by serial and aki:

.. code:: bash

 fabric-ca-client certificate list --serial 1234 --aki 1234

用ID和serial/AKI列出证书：

.. code:: bash

 fabric-ca-client certificate list --id admin --serial 1234 --aki 1234

通过ID列出既没撤销，也没过期的证书：

.. code:: bash

 fabric-ca-client certificate list --id admin --notrevoked --notexpired

列出某个ID（管理员）没取消的所有证书：

.. code:: bash

 fabric-ca-client certificate list --id admin --notrevoked

列出某个ID（管理员）没有过期的证书：

"--notexpired" 标志相当于 "--expiration now::" ，这意味着证书将在未来某个时间过期。

.. code:: bash

 fabric-ca-client certificate list --id admin --notexpired

列出某个ID（admin）在某个时间段内吊销的所有证书：

.. code:: bash

 fabric-ca-client certificate list --id admin --revocation 2018-01-01T01:30:00z::2018-01-30T05:00:00z

列出某个ID(admin)在的某时间范围内吊销但未过期的所有证书（管理员）：

.. code:: bash

 fabric-ca-client certificate list --id admin --revocation 2018-01-01::2018-01-30 --notexpired

列出某个ID（admin）在某段时间（在30天和15天前过期）内吊销的证书：

.. code:: bash

 fabric-ca-client certificate list --id admin --revocation -30d::-15d

列出在某个时间前过期的所有证书

.. code:: bash

 fabric-ca-client certificate list --revocation ::2018-01-30

列出在某个时间之后吊销的所有证书

.. code:: bash

 fabric-ca-client certificate list --revocation 2018-01-30::

列出在在今天和之前某个时间点之间过吊销的所有证书

.. code:: bash

 fabric-ca-client certificate list --id admin --revocation 2018-01-30::now

列出某个ID（admin）在某个时间段内过期但未吊销的所有证书：

.. code:: bash

 fabric-ca-client certificate list --id admin --expiration 2018-01-01::2018-01-30 --notrevoked

列出某个ID（admin）在某段时间（在30天和15天前过期）内过期的证书：

.. code:: bash

 fabric-ca-client certificate list --expiration -30d::-15d

列出已经过期，或在某个特定时间前会过期的所有证书：

.. code:: bash

 fabric-ca-client certificate list --expiration ::2058-01-30

列出已经过期，或在某个特定时间后会过期的所有证书：

.. code:: bash

 fabric-ca-client certificate list --expiration 2018-01-30::

列出在此刻之前在某刻之后过期的所有证书：

.. code:: bash

 fabric-ca-client certificate list --expiration 2018-01-30::now

列出在接下来的十天内会过期的证书：

.. code:: bash

 fabric-ca-client certificate list --id admin --expiration ::+10d --notrevoked

列表证书命令也可用于在文件系统上存储证书。这是在MSP中填充admins文件夹的简便方法，
"-store" 标志指向文件系统上存储证书的位置。

通过在MSP中存储身份标识，将身份配置为管理员（admin）：
.. code:: bash

 export FABRIC_CA_CLIENT_HOME=/tmp/clientHome
 fabric-ca-client certificate list --id admin --store msp/admincerts

接触特定CA实例
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

当服务器运行多个CA实例时，请求可以指向特定的CA。
默认情况下，如果在客户端请求中没有指定CA名称，则请求将被指向fabric-ca服务器上的默认CA。
可以像下面这样使用 ``caname`` 筛选器在客户端命令的命令行上指定CA名称：

.. code:: bash

    fabric-ca-client enroll -u http://admin:adminpw@localhost:7054 --caname <caname>

`回到顶端`_

HSM
---
By default, the Fabric CA server and client store private keys in a PEM-encoded file,
but they can also be configured to store private keys in an HSM (Hardware Security Module)
via PKCS11 APIs. This behavior is configured in the BCCSP (BlockChain Crypto Service Provider)
section of the server’s or client’s configuration file.

Configuring Fabric CA server to use softhsm2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This section shows how to configure the Fabric CA server or client to use a software version
of PKCS11 called softhsm (see https://github.com/opendnssec/SoftHSMv2).

After installing softhsm, create a token, label it “ForFabric”, set the pin to ‘98765432’
(refer to softhsm documentation).

You can use both the config file and environment variables to configure BCCSP
For example, set the bccsp section of Fabric CA server configuration file as follows.
Note that the default field’s value is PKCS11.

.. code:: yaml

  #############################################################################
  # BCCSP (BlockChain Crypto Service Provider) section is used to select which
  # crypto library implementation to use
  #############################################################################
  bccsp:
    default: PKCS11
    pkcs11:
      Library: /usr/local/Cellar/softhsm/2.1.0/lib/softhsm/libsofthsm2.so
      Pin: 98765432
      Label: ForFabric
      hash: SHA2
      security: 256
      filekeystore:
        # The directory used for the software file-based keystore
        keystore: msp/keystore

And you can override relevant fields via environment variables as follows:

FABRIC_CA_SERVER_BCCSP_DEFAULT=PKCS11
FABRIC_CA_SERVER_BCCSP_PKCS11_LIBRARY=/usr/local/Cellar/softhsm/2.1.0/lib/softhsm/libsofthsm2.so
FABRIC_CA_SERVER_BCCSP_PKCS11_PIN=98765432
FABRIC_CA_SERVER_BCCSP_PKCS11_LABEL=ForFabric

`回到顶端`_

文件格式
------------

Fabric CA 服务器的配置文件格式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

默认配置文件是在服务器的主目录中创建的
（请参阅`Fabric CA Server <#server>`__ 部分以获取更多信息）。
下面的链接显示了一个示例 :doc:`Server configuration file <serverconfig>`。

Fabric CA 客户端的的配置文件格式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

默认配置文件是在客户端的主目录中创建的（
请参阅`Fabric CA Server <#server>`__ 部分以获取更多信息）。
下面的链接显示了一个示例 :doc:`Server configuration file <serverconfig>`。
`回到顶端`_

故障排除
---------------

1. 如果您在试图执行 ``fabric-ca-client`` 或 ``fabric-ca-server`` 时在OSX上看到 ``Killed: 9`` 错误，
   那么在 https://github.com/golang/go/issues/19734. 有一个长线程描述这个问题。
   简短的答案是，为了解决这个问题，您可以运行以下命令::

    # sudo ln -s /usr/bin/true /usr/local/bin/dsymutil

2. 如果下面的事情发生，那么错误 ``[ERROR] No certificates found for provided serial and aki`` 就会出现：

   a. 你使用了 `fabric-ca-client enroll` 命令，创建了一个注册证书, (即ECert)。
      这将在fabric-ca-server的数据库里存储一个ECert的拷贝。
   b. 如果删除并重建fabric-ca-server的数据库，就会丢失步骤'a'里的ECert。
      比如说，如果你停止并重新启动了承载fabric-ca-server的docker容器，
      fabric-ca-server却使用了默认的sqlite数据库，但是数据库文件却没有存储在卷上，因此未持久化存储，
      这样一来，错误就发生了。
   c. 你使用了 `fabric-ca-client register` 命令，或者其他命令，来尝试使用步骤'a'里的ECert 。
      在这种情况下，因为数据库不再包含ECert,
      ``[ERROR] No certificates found for provided serial and aki`` 就发生了。

例如，如果停止并重新启动承载fabric-ca-server的docker容器，但是.-ca-server正在使用默认sqlite数据库，并且数据库文件没有存储在卷上，因此不持久，则可能发生这种情况。
0
   若要解决此错误，必须通过重复步骤“a”再次注册。这将发布一个新的ECert，并且被存储在当前数据库中。

3. 当向使用共享sqlite3数据库的Fabric CA Server集群发送多个并行请求时，
   服务器偶尔会返回一个'database locked'错误。这很可能是因为在等待释放数据库锁（由另一个集群成员持有）时，
   数据库事务超时。这是一个无效的配置，因为SQLite是一个嵌入式数据库，
   这意味着，Fabric C CA服务器集群必须通过共享文件系统共享相同的文件，
   该文件引入了SPOF（single point of failure：单点故障），这与集群拓扑的目的相矛盾。
   最好的做法是在集群拓扑中使用Postgres或MySQL数据库。

4. 假设这样的错误
   ``Failed to deserialize creator identity, err The supplied identity is not valid, Verify() returned x509: certificate signed by unknown authority``
   由peer或排序节点在使用Fabric CA服务器颁发的注册证书时返回。
   这表示Fabric CA 服务器用于颁发证书的签名CA证书，与用于进行授权检查的MSP的 `cacerts` 或 `intermediatecerts` 文件夹中的证书不匹配。

   用于授权检查的MSP取决于您在错误发生时执行的操作。
   例如，如果试图在peer上安装chaincode，则使用peer文件系统上的本地MSP；
   否则，如果正在执行某些特定于通道的操作（例如在特定通道上实例化chaincode），那么则使用genesis块中的MSP或最新使用的N个通道配置块。

   为了确认确实是这个问题，比较下面两者：
   + 注册证书的AKI（授权密钥标志符）
   + 适当MSP内 `cacerts` 和 `intermediatecerts` 文件夹下的证书SKI(Subject Key Identifier)

   命令 `openssl x509 -in <PEM-file> -noout -text | grep -A1 "Authority Key Identifier"` 将会展示AKI，而
   `openssl x509 -in <PEM-file> -noout -text | grep -A1 "Subject Key Identifier"` 将会展示 SKI。
   如果他们不相同，你就可以肯定这确实就是问题所在了。

   可能发生的多种原因包括：

   a. 你使用了 `cryptogen` 来生成你的密钥材料，但是却没有使用其生成的签名密钥和证书来启动 `fabric-ca-server`。

      为了解决问题 (假定 `FABRIC_CA_SERVER_HOME` 设定为你 `fabric-ca-server` 的home目录):

      1. 关闭 `fabric-ca-server`.
      2. 拷贝 `crypto-config/peerOrganizations/<orgName>/ca/*pem` 到 `$FABRIC_CA_SERVER_HOME/ca-cert.pem`.
      3. 拷贝 `crypto-config/peerOrganizations/<orgName>/ca/*_sk` 到 `$FABRIC_CA_SERVER_HOME/msp/keystore/`.
      4. 启动 `fabric-ca-server`.
      5. 删除所有之前发行的注册证书并且重新注册来获取新证书。

   b. 在生成创世纪块之后，您删除并重新创建了由Fabric CA服务器使用的CA签名密钥和证书。
      如果Fabric CA Server在docker容器中运行，容器被重新启动，并且其主目录不在卷挂载上，则可能发生这种情况。
      在这种情况下，Fabric CA服务器将创建新的CA签名密钥和证书。

      假设您无法恢复原始CA签名密钥，从此场景恢复的唯一方法是将适当MSP的 `cacerts`（或 `intermediatecerts` ）中的证书更新为新的CA证书。

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
