# Fabric

## 玩家身份

peers，orderers，client应用，administrators等等，都拥有X.509证书代表的数字身份。身份代表了资源访问的权限。在Fabric当中，MSP是组织管理组织下identities的方式，也就是哪些身份是组织认可的，是属于这个组织的。PKI是将身份和数字证书联系到一起，而MSP是将身份和组织机构联系在一起，哪些身份是属于哪个组织机构的。

一个节点想要参与到Fabric的区块链网络当中，**是需要合规的数字身份**的，这个数字身份可以由网络信赖的组织机构来颁发。

CA(颁发证书的机构)也是有公钥和私钥的，网络中其他节点都知道CA的公钥，这样才能对CA私钥签发的证书进行验证。

一个组织中的CA分为两类，一个是ROOT CA（可以有多个），一个是Intermediate CA（可以有多个），InterCA也能颁发合规的证书，且InterCA的证书必须是ROOT CA颁发的或者其他InterCA颁发的（InterCA颁发的证书追本溯源可以到ROOT CA）。这种设计**第一个是安全**，因为如果只用一个ROOT CA的话，一旦ROOT CA发生安全问题，那么影响的是整个组织，但是如果InterCA发生安全问题，影响范围要小得多。**第二个**就是可拓展性大大增强。

## Fabric CA

Fabric CA的功能有：

* 身份注册
* 颁发注册证书
* 证书续签和撤销

包括两个核心componet，一个是server端，一个是client端。

![_images/fabric-ca.png](https://hyperledger-fabric-ca.readthedocs.io/en/release-1.4/_images/fabric-ca.png)

client端可以是Fabric-CA client也可以是SDK，这两个和CA server通信都是通过调用server端的REST API来完成的，server端存储数据使用的是数据库或者LDAP。

Hyperledger Fabric CA的**client端和server端是使用go语言来编写**的，Github地址为：https://github.com/hyperledger/fabric-ca，编译出的client端和server端二进制命令文件可以用来启动Fabric CA server和CA client，