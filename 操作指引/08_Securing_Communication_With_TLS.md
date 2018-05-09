# 基于TLS的安全通信

Fabric支持在各节点间进行TLS安全通信. Fabric TLS通信可以使用单向（服务器-服务器）认证, 也可以使用双向（服务器和客户端）认证。

## 为Peer节点配置TLS

Peer节点既是TLS服务器，又是TLS客户端。 当另外一个节点(Peer节点, Appliction, 或者CLI)尝试连接它时, 它作为TLS服务器的角色存在; 当它尝试连接其他节点时,它作为客户端角色存在.

如果需要在Peer节点上启用TLS, 你可以通过设置Peer属性的方式完成:

* ````peer.tls.enabled```` = ````true````
* ````peer.tls.cert.file```` = TLS服务器证书文件的完整路径
* ````peer.tls.key.file```` = TLS服务器证书秘钥文件的完整路径
* ````peer.tls.rootcert.file```` = 包含TLS服务器证书颁发机构（CA）的证书链文件的完整路径(根证书)

当在Peer节点上启用TLS时，TLS客户端身份验证默认是关闭的。这意味着Peer节点在TLS握手期间不会验证客户端（另一个Peer节点、Application或CLI）的证书.如果需要启用客户端身份验证,可以通过同时设置以下属性来完成:

* ````peer.tls.clientAuthRequired```` = ````true````
* ````peer.tls.clientRootCAs.files```` = 为你的组织颁发证书的CA链文件(该文件包含CA证书链)

默认情况下，Peer节点在充当TLS服务器和客户端角色时使用相同的证书和私钥对。如果需要为客户端角色使用不同的证书和私钥对,可以通过同时设置以下属性来完成:

* ````peer.tls.clientCert.file```` = 客户端证书文件 
* ````peer.tls.clientKey.file```` = 客户端证书私钥文件

TLS客户端认证也可以通过设置环境变量的方式启用:

* ````CORE_PEER_TLS_ENABLED```` = ````true````
* ````CORE_PEER_TLS_CERT_FILE```` = 服务器证书完整路径
* ````CORE_PEER_TLS_KEY_FILE```` = 服务器私钥完整路径
* ````CORE_PEER_TLS_ROOTCERT_FILE```` = CA链文件完整路径
* ````CORE_PEER_TLS_CLIENTAUTHREQUIRED```` = ````true````
* ````CORE_PEER_TLS_CLIENTROOTCAS_FILES```` = CA链文件完整路径
* ````CORE_PEER_TLS_CLIENTCERT_FILE```` = 客户端证书完整路径
* ````CORE_PEER_TLS_CLIENTKEY_FILE```` = 客户端私钥完整路径

当Peer节点启用客户端认证时，其它节点(Peer节点,Application, CLI)需要在TLS握手期间发送它们的证书。如果其它节点不发送证书，握手将失败，并且Peer节点将关闭连接。

当Peer节点加入Channel时, Fabric会从Channel的Config Block中读取Channel成员的根CA证书链,并把它们添加Peer节点的TLS根CAs的数据结构中, 这样Peer和Peer之间的TLS通信,以及Peer和Orderer之间的TLS通信就可以无缝的工作.

> 注意: 默认情况下,Peer节点在充当TLS服务器和客户端角色时使用相同的证书和私钥对, 这可能存在一些安全风险. 建议采用不同的证书和私钥对!

## 为Orderer节点配置TLS

如果需要在Orderer节点启用TLS, 可以通过为Orderer节点配置以下属性的方式来完成:

* ````General.TLS.Enabled```` = ````true````
* ````General.TLS.PrivateKey```` = 服务器私钥文件完整路径
* ````General.TLS.Certificate```` = 服务器证书文件完整路径
* ````General.TLS.RootCAs```` = CA证书链文件完整路径

默认情况下, Orderer节点的TLS客户端认证是关闭的(和Peer节点类似). 如果需要启用Orderer节点的TLS客户端认证, 可以通过同时设置以下属性的方式完成:

* ````General.TLS.ClientAuthRequired```` = ````true````
* ````General.TLS.ClientRootCAs````= 其它客户端发来证书的根证书链

TLS客户端认证,也可以通过设置环境变量的方式启用:

* ````ORDERER_GENERAL_TLS_ENABLED```` = ````true````
* ````ORDERER_GENERAL_TLS_PRIVATEKEY```` = 服务器私钥文件完整路径
* ````ORDERER_GENERAL_TLS_CERTIFICATE```` = 服务器证书完整路径
* ````ORDERER_GENERAL_TLS_ROOTCAS````  = 服务器根证书链文件完整路径
* ````ORDERER_GENERAL_TLS_CLIENTAUTHREQUIRED```` = ````true````
* ````ORDERER_GENERAL_TLS_CLIENTROOTCAS```` = 客户端证书的根证书链文件完整路径

## 为Peer CLI配置TLS

如果需要通过Peer CLI的方式,和启用了TLS的远程Peer节点进行通信, 则必须设置以下环境变量:

* ````CORE_PEER_TLS_ENABLED```` = ````true````
* ````CORE_PEER_TLS_ROOTCERT_FILE``` = 本地CA证书链文件

如果远程Peer节点启用了客户端认证, 则还必须在本地设置以下环境变量:

* ````CORE_PEER_TLS_CLIENTAUTHREQUIRED```` = ````true````
* ````CORE_PEER_TLS_CLIENTCERT_FILE```` = 本地客户端证书路径
* ````CORE_PEER_TLS_CLIENTKEY_FILE```` = 本地客户端私钥

当需要通过Peer CLI连接到Orderer节点时, 如, *peer channel &lt;create|update|fetch&gt;* 或者 *peer chaincode &lt;invoke|instantiate&gt;*, 如果Orderer节点启用了TLS, 则必须为Peer CLI
指定以下参数:

* ````-tls````
* ````-cafile <包含Orderer CA证书链文件的完全限定路径>````

如果远程Orderer节点启用了TLS客户端认证, 则还必须为Peer CLI指定以下参数:

* ````-clientauth````
* ````-keyfile <包含客户端私钥的文件的完全限定路径>````
* ````-certfile <包含客户端证书的文件的完全限定路径>````

## 调试TLS

在调试TLS之前，建议在TLS客户端和服务器端启用````GRPC debug````以获取更多附加信息。可以通过设置环境变量````CORE_LOGGING_GRPC=DEUBG````的方式启用````GRPC debug````。

如果您在客户端上看到错误消息````remote error: tls: bad certificate````，这通常意味着TLS服务器已经启用了客户端身份验证，但服务器没有接收到正确的客户端证书，或者接收到了它不信任的客户端证书。碰到这种问题,通常需要:

* 确保客户端正常发送证书
* 确保客户端发送的证书已被远程服务器信任的CA机构签名

如果您在````Chaincode````日志中看到错误消息```` remote error: tls: bad certificate````. 请确保

* 您的Chaincode是基于Fabric v1.1或更高版本的````Chaincode shim````构建的
* 如果您的Chaincode不包含一个被允许的````shim````副本，则需要删除链Chaincode容器并重新启动相应Peer节点. 这可以基于当前````shim````版本重建Chaincode容器。



