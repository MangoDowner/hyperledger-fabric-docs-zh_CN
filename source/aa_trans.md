macOS下整个流程及问题的解决
===========================

.. note::  以下问题仅限于macOS


## 在fabric的scripts目录下运行bootstrap.sh
```
./bootstrap.sh
```
该脚本下载了例子和所需的镜像，镜像列表如下：
```
limengdeMBP:fabric limeng$ docker images
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
hyperledger/fabric-ca          1.2.0               66cc132bd09c        6 weeks ago         252MB
hyperledger/fabric-ca          latest              66cc132bd09c        6 weeks ago         252MB
hyperledger/fabric-tools       1.2.0               379602873003        6 weeks ago         1.51GB
hyperledger/fabric-tools       latest              379602873003        6 weeks ago         1.51GB
hyperledger/fabric-ccenv       1.2.0               6acf31e2d9a4        6 weeks ago         1.43GB
hyperledger/fabric-ccenv       latest              6acf31e2d9a4        6 weeks ago         1.43GB
hyperledger/fabric-orderer     1.2.0               4baf7789a8ec        6 weeks ago         152MB
hyperledger/fabric-orderer     latest              4baf7789a8ec        6 weeks ago         152MB
hyperledger/fabric-peer        1.2.0               82c262e65984        6 weeks ago         159MB
hyperledger/fabric-peer        latest              82c262e65984        6 weeks ago         159MB
hyperledger/fabric-zookeeper   0.4.10              2b51158f3898        6 weeks ago         1.44GB
hyperledger/fabric-zookeeper   latest              2b51158f3898        6 weeks ago         1.44GB
hyperledger/fabric-kafka       0.4.10              936aef6db0e6        6 weeks ago         1.45GB
hyperledger/fabric-kafka       latest              936aef6db0e6        6 weeks ago         1.45GB
hyperledger/fabric-couchdb     0.4.10              3092eca241fc        6 weeks ago         1.61GB
hyperledger/fabric-couchdb     latest              3092eca241fc        6 weeks ago         1.61GB
hyperledger/fabric-baseimage   amd64-0.4.10        62513965e238        6 weeks ago         1.39GB
hyperledger/fabric-baseos      amd64-0.4.10        52190e831002        6 weeks ago         132MB
limengdeMBP:fabric limeng$
```
## 运行byfn.sh
进入fabric-examples目录下的first-network,运行byfn.sh
```
./byfn.sh generate
./byfn.sh up
```
生成了下面的容器
```
limengdeMBP:first-network limeng$ docker ps
CONTAINER ID        IMAGE                                                                                                  COMMAND                  CREATED              STATUS              PORTS                                              NAMES
9338ffe1bb11        dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "chaincode -peer.add…"   17 seconds ago       Up 15 seconds                                                          dev-peer1.org2.example.com-mycc-1.0
feb0b1cbafee        dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "chaincode -peer.add…"   32 seconds ago       Up 30 seconds                                                          dev-peer0.org1.example.com-mycc-1.0
d74bc68b96d4        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "chaincode -peer.add…"   46 seconds ago       Up 44 seconds                                                          dev-peer0.org2.example.com-mycc-1.0
cf9b9bcb2785        hyperledger/fabric-tools:latest                                                                        "/bin/bash"              About a minute ago   Up About a minute                                                      cli
c3f2264192d9        hyperledger/fabric-orderer:latest                                                                      "orderer"                About a minute ago   Up About a minute   0.0.0.0:7050->7050/tcp                             orderer.example.com
a2d68f9aba26        hyperledger/fabric-peer:latest                                                                         "peer node start"        About a minute ago   Up About a minute   0.0.0.0:8051->7051/tcp, 0.0.0.0:8053->7053/tcp     peer1.org1.example.com
427245b9c05d        hyperledger/fabric-peer:latest                                                                         "peer node start"        About a minute ago   Up About a minute   0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp     peer0.org1.example.com
21255d335612        hyperledger/fabric-peer:latest                                                                         "peer node start"        About a minute ago   Up About a minute   0.0.0.0:10051->7051/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com
627a4a9473f7        hyperledger/fabric-peer:latest                                                                         "peer node start"        About a minute ago   Up About a minute   0.0.0.0:9051->7051/tcp, 0.0.0.0:9053->7053/tcp     peer0.org2.example.com
limengdeMBP:first-network limeng$ docker ps
CONTAINER ID        IMAGE                                                                                                  COMMAND                  CREATED              STATUS              PORTS                                              NAMES
9338ffe1bb11        dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "chaincode -peer.add…"   21 seconds ago       Up 19 seconds                                                          dev-peer1.org2.example.com-mycc-1.0
feb0b1cbafee        dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "chaincode -peer.add…"   36 seconds ago       Up 34 seconds                                                          dev-peer0.org1.example.com-mycc-1.0
d74bc68b96d4        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "chaincode -peer.add…"   50 seconds ago       Up 48 seconds                                                          dev-peer0.org2.example.com-mycc-1.0
cf9b9bcb2785        hyperledger/fabric-tools:latest                                                                        "/bin/bash"              About a minute ago   Up About a minute                                                      cli
c3f2264192d9        hyperledger/fabric-orderer:latest                                                                      "orderer"                About a minute ago   Up About a minute   0.0.0.0:7050->7050/tcp                             orderer.example.com
a2d68f9aba26        hyperledger/fabric-peer:latest                                                                         "peer node start"        About a minute ago   Up About a minute   0.0.0.0:8051->7051/tcp, 0.0.0.0:8053->7053/tcp     peer1.org1.example.com
427245b9c05d        hyperledger/fabric-peer:latest                                                                         "peer node start"        About a minute ago   Up About a minute   0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp     peer0.org1.example.com
21255d335612        hyperledger/fabric-peer:latest                                                                         "peer node start"        About a minute ago   Up About a minute   0.0.0.0:10051->7051/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com
627a4a9473f7        hyperledger/fabric-peer:latest                                                                         "peer node start"        About a minute ago   Up About a minute   0.0.0.0:9051->7051/tcp, 0.0.0.0:9053->7053/tcp     peer0.org2.example.com
```




如果出现以下错误：

.. code:: bash

    Error processing tar file(bzip2 data invalid: bad magic value in continuation file):

解决办法 ,更新gnu-tar

.. code:: bash

    brew install gnu-tar --with-default-names


B. 在 mac os 下出现’ltdl.h’ file not found错误

brew install libtool openssl
1
C. 出现goimports: command not found错误

make linter 
出现如下错误：

LINT: Running code checks..
Checking ./bccsp
./scripts/golinter.sh: line 22: goimports: command not found
make: *** [linter] 错误 127
1
2
3
4
但是执行$GOPATH/src/github.com/hyperledger/fabric/scripts/golinter.sh 
是可以正常执行的

make gotools

#如果无法科学上网,需要使用代理
git config --global http.proxy "192.168.0.50:8016"
git config --global https.proxy "192.168.0.50:8016"

#或者在可以 git 的地方，把 gotools 的源码 git 下来，然后复制到相应目录下

git clone https://go.googlesource.com/tools 
#当前目录下生成 tools 目录，
#把目录复制到$GOPATH/src/github.com/hyperledger/fabric/gotools/build/gopath/src/golang.org/x/目录下。
#重新 make gotools
cd $GOPATH/src/github.com/hyperledger/fabric
make gotools
1
2
3
4
5
6
7
8
9
10
11
12
13
14
D. make 过程中出现 IPv4 forwarding is disabled. Networking will not work. 这个警告。

sysctl -w net.ipv4.ip_forward=1
#重启 docker 服务
systemctl restart docker
1
2
3
E. 运行 e2e 例子的时候出现如下错误:

1.无法创建 channel

vim  $GOPATH/src/github.com/hyperledger/fabric/examples/e2e_cli/docker-compose.yaml

#修改 docker-compose.yaml 文件。在 cli 部分增加 host 映射
    extra_hosts:
      - "orderer0:10.211.55.8"
      - "peer0:10.211.55.8"
      - "peer1:10.211.55.8"
      - "peer2:10.211.55.8"
      - "peer3:10.211.55.8"

#修改 script.sh 脚本,进行 peer 的判断，映射端口号：

if [ $1 -eq 0 ]; then
     CORE_PEER_ADDRESS=peer$1:7051
elif [ $1 -eq 1 ]; then
        ORE_PEER_ADDRESS=peer$1:8051
elif [ $1 -eq 2 ]; then
        ORE_PEER_ADDRESS=peer$1:9051
elif [ $1 -eq 3 ]; then
        ORE_PEER_ADDRESS=peer$1:10051
fi
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
F. 相关知识点

1. git 拉取某个特定版本

git reset --hard <commitlevel号>