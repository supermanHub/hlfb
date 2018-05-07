# 成员服务提供者(Membership Service Providers, MSP)

本章提供关于MSP的设置和最佳实践的细节。

成员服务提供程序（MSP）是一个旨在提供成员操作架构的高层抽象组件。

特别的，MSP对发布和验证证书和用户身份验证之后密码机制和协议进行了高层抽象。一个MSP可以定义他们自己的身份概念，以及这些身份被支配的规则（身份验证）和认证（签名生成和验证）。

Hyperledger Fabric 区块链网络可以由一个或多个MSP来管理。这提供了成员服务操作的模块化，以及跨不同的成员标准和体系结构的互操作性。

在本文档的其余部分中，我们详细介绍了Hyperledger Fabric支持的MSP的设置，并讨论了有关其使用的最佳实践

## MSP 配置 (MSP Configuration)

要设置一个MSP的实例，需要在每个Peer和Orderer本地指定其配置（以启用peer和orderer签名），并在Channel上启用peer、orderer、客户端身份验证、以及所有Channel成员的的相应签名验证（认证）。

当然，对于每个MSP，需要指定一个名称，以便在网络中进行引用（例如msp1、org2）。这是MSP成员规则下的名称, 这些规则代表了联盟、组织或组织部门在一个channel中的引用。这个名称也被称为*MSP Identifier*或*MSP ID*。每个MSP实例需要的MSP ID是唯一的。例如，如果在system channel中检测到两个具有相同标识符的MSP实例，则genesis 和 orderer设置将失败。

在MSP的默认实现下，需要指定一组参数以允许身份（证书）验证和签名验证。这些参数由[RCF5280](http://www.ietf.org/rfc/rfc5280.txt)定义，并包括：

* 一组用以构成信任根源(*root of trust*)的自签名证书(X.509) - ROOT CAs
* 一组表示中间CAs的X.509证书列表, 这些证书应该由信任根源中的一个证书来验证；中间CAs是可选的。 - Intermediate CAs
* 一组可验证的MSP管理员X.509证书列表, 这些证书的所有者有权请求修改MSP的配置(如,ROOT CA, Intermediate CAs)
* 一组可验证其成员的组织单元X.509证书列表,这是一个可选的配置参数，用于当多个组织使用相同的信任根和中间的CAs时，为他们的成员保留了一个OU字段。
* 证书撤销列表（CRL）的列表，每个列表对应于MSP证书颁发机构中的一个；这是一个可选参数。
* 一个自签名（X.509）证书的列表，以构成TLS证书的TLS信任根。
* 一个X.509证书的列表，用于表示该提供者认为的中间TLS CAS；这些证书应该完全由TLS信任根中的一个证书来验证；中间CAs是可选参数。

MSP实例中有效的身份有效标识需要满足以下条件:
* 它们的X.509证书，具有可验证的证书路径，指向任一信任根
* 它们不包括在任何CRL中
* 它们列出了X.509证书结构中的OU字段中的一个或多个MSP配置的组织单元。

有关当前MSP实现中身份的有效性的更多信息，我们向读者介绍[MSP身份有效性规则](http://hyperledger-fabric.readthedocs.io/en/release-1.1/msp-identity-validity-rules.html)。

除了验证相关参数之外，对于MSP，使其实例化的节点能够进行签名或认证，需要指定：
* 用于由节点签名的签名密钥（目前仅支持ECDSA密钥），
* 以及节点的X.509证书，这是在这个MSP的验证参数下的有效身份。

重要的是要注意，MSP身份永远不会过期；它们只能通过将其添加到适当的CRL中进行撤销。此外，目前还没有支持强制取消TLS证书的支持。

## 如何生成MSP证书和签名秘钥

您可以使用[Openssl](https://www.openssl.org/)生成X.509证书。我们强调，在Hyperledger Fabric中，不支持包括RSA密钥的证书。

您也可以使用我们提供的````cryptogen````工具生成证书, 请参考相应[操作指引](http://linkto)

Hyperledger Fabric CA也可以用来生成配置MSP所需的密钥和证书。

## Peer 和 Orderer MSP设置(Peer & Orderer MSP Setup)

为了建立本地MSP（针对peer 和 orderer），管理员应该创建一个文件夹,其中包含6个子文件夹和一个文件：
1. ````admincerts```` 文件夹, 其中包含PEM文件，每个PEM文件对应于一个管理员证书。
2. ````cacerts```` 文件夹, 其中包含PEM文件,每个PEM文件对应一个ROOT CA 证书
3. ````intermediatecerts```` 文件夹(可选的),其中包含PEM文件,每个PEM文件对应一个Intermediate CA
4. ````config.yaml````文件（可选）, 用来配置对组织单元和身份分类的支持（见下面各节）。
5. ````crls````文件夹
6. ````keystore````文件夹,包含的节点(Peer or Orderer)签名密钥和PEM文件；不支持目前的RSA密钥
7. ````signcerts````包含节点X.509证书的PEM文件的文件夹标识
8. ````tlscacerst````（可选)包括每个与TLS根CA证书对应的PEM文件。
9. ````tlsintermediatecerts````（可选）其中包括对应于中间TLS CA证书的PEM文件。

在节点的配置文件（Peer的core.yaml和Orderer的orderer.yaml）中，需要指定MSP配置文件夹的路径，以及节点MSP标识符。MSP配置件夹的路径应是相对于````FABRIC_CFG_PATH````的路径，可以通过````mspConfigPath````为Peer节点进行配置,通过````LocalMSPDir````为Orderer节点配置. 这些参数可以通过环境变量进行覆盖. 如,通过以CORE开头的前缀,覆盖Peer的配置(如, CORE_PEER_LOCALMSPID) 和以ORDERER开头的前缀覆盖Orderer配置(如, ORDERER_GENERAL_LOCALMSPID). 注意,在配置Orderer节点的时候,还需要为System channel生成Genesis block,并在Orderer节点配置中,指定Genesis Block.

重新配置“本地(Peer或Orderer 本地)”MSP只能手动完成，并要求重新启动Peer或Orderer进程。在随后的版本中，我们的目标是提供在线/动态重构(例如, 即不用通过节点管理的系统链码来停止节点。)。

## 组织单元 ( Organizational Units)
在配置MSP有效组织单元时, 需要在````config.yaml````文件中指定组织单元标识符。下面是一个例子：
````
OrganizationalUnitIdentifiers:
  - Certificate: "cacerts/cacert1.pem"
    OrganizationalUnitIdentifier: "commercial"
  - Certificate: "cacerts/cacert2.pem"
    OrganizationalUnitIdentifier: "administrators"
````

上面的示例声明了两个组织单元标识符：````commercial````和````administrators````。如果MSP标识携带这些组织单元标识符中的至少一个，则是有效的。````Certificate````字段指的是CA或中间CA证书路径，在该路径下，应该验证具有特定OU的身份。路径与MSP根文件夹相对应，不能为空。
## 身份分类 (Identity Classification)
默认的MSP实现允许基于X509证书的OU来进一步将身份分类到Clients和Peers中。如果提交事务、查询Peer等，则身份应该被分类为Clients。如果它认可或提交事务，则身份应该被分类为Peers。为了定义给定MSP的Client和Peer，需要适当设置config.yaml文件。下面是一个例子：
````
NodeOUs:
  Enable: true
  ClientOUIdentifier:
    Certificate: "cacerts/cacert.pem"
    OrganizationalUnitIdentifier: "client"
  PeerOUIdentifier:
    Certificate: "cacerts/cacert.pem"
    OrganizationalUnitIdentifier: "peer"
````
如上所示，通过将````NodeOUs````被设置为````true````来启用主体分类。然后，Clients(Peer）标识符通过为````NodeOUs.ClientOUIdentifier````(````NodeOUs.PeerOUIdentifier````）设置以下属性来定义：

1. ````OrganizationalUnitIdentifier````, 将此值设置为与Client（Peer）X509证书包含的OU匹配的值。
2. ````Certificate````, 将其设置为CA或中间CA，在该CA下，Client（Peer）身份应该被验证。字段与MSP根文件夹相对。它可以是空的，这意味着身份验证的X509证书可以在MSP配置中定义的任何CA下进行验证。

当启用分类时，MSP管理员需要是该MSP的客户端，这意味着它们的X509证书需要携带标识客户端的OU。还要注意，身份可以是Client或Peer。这两种分类是互斥的。如果标识既不是Client也不是Peer，验证将失败。

最后，注意，在升级的时候，如果想要使用主体分类功能,需要先启用Channel 1.1的相关特性.
## Channel MSP设置 (Channel MSP Setup)
在系统的起源时，必须指定网络中所有MSP的验证参数，并且把这些信息包含在Geneis block中。回想一下MSP验证参数由MSP标识符、信任证书的根、中间CA和管理员证书以及OU规范和CRLS组成.系统Genesis block在Orderers节点设置阶段提供给Orderer节点，并允许它们认证Channel创建请求。如果系统包含两个具有相同标识符的MSP，则订购者会拒绝系统Genesis block，因此网络的启动将失败。

对于Application Channel，仅管理Channel的MSP的验证组件需要驻留在Channel的Genesis Block中。我们强调，应用程序的责任是确保正确的MSP配置信息被包含在一个Channel的Genesis block（或最新的配置块）中，在指示一个或多个它们的对等方加入信道之前。

当在````configtxgen````工具的帮助下引导信道时，可以通过在MSPCONFIG文件夹中包括MSP的验证参数来配置信道MSPS，并在config.yaml中的相关部分中设置该路径。

通过在MSP的管理员证书的所有者创建Office更新对象来实现对信道上的MSP的重新配置，包括与该MSP的CAS相关联的证书撤销列表的通知。由管理员管理的客户端应用程序将向MSP出现的通道中宣布此更新。
## 最佳实践 (Best Practices)
在这一节中，我们将详细说明MSP配置的最佳实践。

1. 组织/企业和MSP之间的映射

    我们建议在组织和MSP之间进行一对一映射。如果采用不同的映射类型，则需要考虑以下内容：
    1. 一个组织使用多个MSP

        这对应于一个组织多个部门使用独立的MSP(无论是出于独立管理的原因，还是出于隐私原因).在这种情况下，Peer只能由单个MSP拥有，并且不会被同一个组织下的其它MSP识别。这意味着Peer可以通过Gossip组织范围内的数据与一组相同部门的成员共享，而不能与构成实际组织的提供者的集合共享。
    
    2. 多个组织使用同一个MSP

        这对应于由类似的成员体系结构支配的组织联盟。这里需要知道的是，Peer会将组织范围消息传播到具有相同MSP下具有相同身份的Peer，而不管它们是否属于同一个实际组织。这是MSP定义粒度和/或Peer配置的限制。

2. 一个组织有不同的部门（组织单位, Organisational units），希望对不同部门对不同Channel的访问进行控制

    两种实现方法

    1. 为所有成员设置一个MSP

        该MSP的配置将包括根CAs、中间CAs和管理证书的列表；成员身份标识符需要包括成员所属的组织单元.然后可以定义策略来捕获特定的OU的成员，并且这些策略可以构成信道的读/写策略或链码的背书策略。这种方法的局限性是，Gossip peer会考虑在其本地MSP下归属于同一组织的作为成员身份的peer，并向他们广播组织范围内的数据。

    2. 为每个部门设置一个MSP

        这将涉及为每个分区指定一组证书，用于根CAS、中间CAs和管理证书，从而在MSPS上不存在重叠的证书路    `1230-AZ;\7U

在许多情况下，从主体身份中获取身份类型是必须的. 这种需求有一定的局限性:

允许这种分离的一种方法是为每个节点类型创建一个单独的中间CA，一个用于客户端，另一个用于Peer/Orderer；并且配置两个不同的MSP——一个用于客户端，另一个用于Peer/Orderer。访问该组织的渠道需要同时包含这两个MSP，背书政策使用Peer MSP.这最终会导致组织映射到两个MSP实例，并会对对Peer和Client交互的方式产生一定的影响。

当同一组织的所有Peer仍然属于一个MSP时，Gossip就不会受到严重影响。Peer可以将某些系统链码的执行限制为基于本地MSP的策略。例如，如果请求是由本地MSP客户端管理员签名，则只能执行“Join Channel”请求。

另外需要考虑的是，Peer基于其本地MSP中的请求始发者的成员来授权事件注册请求。显然，由于请求的始发者是客户端，请求始发者总是注定属于不同于请求的对等体的MSP，并且对等体会拒绝请求。
4. 管理和CA证书

设置MSP管理证书与MSP为信任根或中间CAS所考虑的任何证书不同是很重要的。这是一种常见的（安全）实践，将成员资格管理的职责与颁发新证书和/或验证现有证书分开。


5. 中间CA 黑名单
6. CA 和 TLS CA

MSP身份的根CAS和MSP TLS证书的根CAS（和相对中间CAS）需要在不同的文件夹中声明。这是为了避免不同类别的证书之间的混淆。对于MSP身份和TLS证书，不允许重用相同的CAS，但最佳实践建议在生产中避免这种情况。