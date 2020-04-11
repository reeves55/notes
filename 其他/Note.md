# Note

```go
directory:
  cathaya/(加密和身份验证相关的东西)
  config/(提供方法将系统配置put进state database或者get链上配置)
  constants/
  core/(业务逻辑核心代码，业务逻辑)
  db/(聚合shim包的相关操作)
  event/(也是put state，但是key有特定的prefix)
  model/(二级节点的CUR，二级节点信息存储在state database当中)
  node/()
  pb/(定义一些需要序列化的数据结构，写在.proto文件里，并利用protoc生成.pb.go文件，用protocol buffer序列化的数据长度更小，解析效率更高)
  utils/()
main.go(主程序，调用shim.Start())

package:
  pingan.com/chaincode/log
  github.com/hyperledger/fabric/core/chaincode/shim
  github.com/segmentio/errors-go
  github.com/sirupsen/logrus
  flatten
  

--------------------------------------------------------------
github.com/segmentio/errors-go: (系统builtin/error的增强版)
* errors.New()
* errors.WithTypes()
* errors.Wrap()

github.com/sirupsen/logrus (系统log包的增强版)
* logrus.Entry
* log.Debug()
* log.WithError()
* log.Warning()

base64.StdEncoding.DecodeString
```



gPPC框架使用的HTTP/2（应用层协议）+ Protocol Buffer（数据序列化）来进行通信，gRPC具有多语言的特性，在java中的实现，也就是grpc-java，技术栈上面通信层是基于netty来实现的，使用了netty的http2.0实现。但是go版本的rpc库并不是基于netty来实现的，毕竟netty是java里面的东西。



ProtocolBuffer（protobuf）其实是一种数据格式，和json，xml属于同一类，但是它有自己的特点，就是它支持数据格式编译成各个语言版本中的数据类型。开发者使用protobuf这种数据格式的时候直接使用protoc编译出来的类或者其他已有的数据结构就行了，非常方便，不像json一样，需要自己直接处理json字符串，需要专门处理json字符串的包。protobuf都帮我们实现了，只要你编写.proto文件，还有一个protoc编译器即可。protobuf只是一种数据格式规范，所以gRPC包括了HTTP/2和protobuf两个东西。

![image-20190606174010092](http://129.211.70.60:8080/image-20190606174010092.png)



##go语言基于shim的智能合约开发

智能合约程序需要实现go定义的 **<u>Chaincode接口</u>**

```go
type Chaincode interface {
    // Init is called during Instantiate transaction after the chaincode container
    // has been established for the first time, allowing the chaincode to
    // initialize its internal data
    Init(stub ChaincodeStubInterface) pb.Response

    // Invoke is called to update or query the ledger in a proposal transaction.
    // Updated state variables are not committed to the ledger until the
    // transaction is committed.
    Invoke(stub ChaincodeStubInterface) pb.Response
}
```

也就是我们会自己写一个MyChaincode，**<u>实现这个接口</u>**，引入

```go
import(
  "github.com/hyperledger/fabric/core/chaincode/shim"
  pb "github.com/hyperldger/fabric/protos/peer"
)
```

上面那个pb.Response就是pb package里面定义的chaincode和peer之间进行grpc的通信数据格式，shim包当中也使用到了pb，shim提供了一组API，用来让chaincode在运行时候可以调用read、write  state database的方法，shim之所以要用pb是因为shim里面的API方法并非真正操作数据库，而是基于gRPC调用peer的方法。**<u>另外还需要一个main方法才能使得智能合约可以安装运行</u>**

```
err := shim.Start(new(MyChaincode))
```

另外，在 Init 和 Invoke方法当中，都是通过

```go
stub.GetFunctionAndParameters()
```

来 <u>**获取调用这个方法的参数**</u> 的。



state database是一个key-value数据库，shim提供的接口就是get、put这种类型的，但是我们可以往链上put各种类型的数据，最好还是 <u>**将put到链上的数据使用protocol buffer进行序列化**</u>，不要使用json，因为json序列化出来的数据量还是比较大的。使用protocol buffer就需要我们自己新建一个.proto文件，然后在里面定义message类型，使用的时候，就是新建这种数据结构，然后调用

```go
import "github.com/golang/protobuf/proto"
proto.Marshal(&MyMessage{...}) // 注意这里要使用指针作为参数
```

将数据序列化，然后取出来的时候就使用

```go
proto.Unmarshal(buf []byte, &MyMessage)
```

来反序列化就行了。

**保存在链上数据（DataOnChain）的格式** 为：

```go
{
  Value: {
    Enc int32,
    Owner string,
    Value bytes,
    Info string
    Version int32,
    Timestamp int64
  },
  History: {
    Owner string,
    Type string,
    DataId string,
    Action string,
    History string,
    Time int64
  },
  Sup: {
    Type int32,
    Pubkey bytes,
    Entity: {
      Cypher bytes,
      SupIdArr []string
    }
  }
}
```

**链上一个数据项的数据（SingleChainItem）结构表示** 为：( 和存储在链上的数据相比，多了**Key**和**Sig**属性 )，这个在使用当中是当做参数的

```go
{
  Key: string,
  Value: {
    Enc int32,
    Owner string,
    Value bytes,
    Info string
    Version int32,
    Timestamp int64
  },
  History: {
    Owner string,
    Type string,
    DataId string,
    Action string,
    History string,
    Time int64
  },
  Sup: {
    Type int32,
    Pubkey bytes,
    Entity: {
      Cypher bytes,
      SupIdArr []string
    }
  },
  Sig: bytes
}
```



区块链当中根据key删除数据的方法，就是把原来的key-value删除掉（实际上调用的是**shim.ChaincodeStubInterface.DelState(key)这个方法**），然后再往区块链当中添加一个代表历史数据的key-value对，其实也就是historyPrefix+key对应着原来的value，但是只有原来的链上数据的History属性。删除操作是需要进行验证的，要不然用户没权删除数据然后我把数据删除不就完了吗，怎么验证呢？用户在进行key-vaue删除的时候，需要传入一个需要删除的对象的信息（包括<Key,Value,History<Owner,Type,DataId,Action,History,Time>,Supervise<Type,Pubkey,SupIdEntity>,Sig>），然后我去区块链上获取这个key对应的value信息（包括<Value,History,Supervise<Type,Pubkey,SupIdEntity>>），然后使用区块链当中存储的这个数据的Supervise中的Pubkey来验证传入参数中的Sig，其中Sig是用户使用自己的私钥对(key+value的protobuf序列化数据的base64编码格式字符串)进行加密。然后才会使用Supervise中的Pubkey来验证签名是否正确。除了数据的删除操作，数据的添加，数据的更新都需要验证参数当中的签名是否正确。数据添加的时候，是根据传入参数中的owner来在区块链上找到owner的证书，然后用证书来验证添加数据的请求是否是真正的二级节点的owner发起的，但是需要在运行前把二级节点的证书都先存储到区块链上面去。

接口调用的时候，客户端传入的参数都是字符串格式，这个字符串格式其实就是使用base64.StdEncoding.EncodeString来得到的，链码在拿到这个参数的时候，首先需要对它进行解码，也就是base64.StdEncoding.DecodeString，得到的是protobuf序列化的字节数据，然后我们再proto.Unmarshal，把字节数据转成对应的数据类型即可。

* 有的参数是json格式，然后拿到的时候直接json.Unmarshal就可以了。

* 应当重新定义一个包用来专门返回pb.Response，把返回信息当做参数传入进去。
* Cathaya提供了根据公钥来验证签名和根据证书来验证签名的方式

> shim.ChaincodeStubInterface.GetHistoryForKey(key) 这个方法究竟是如何把key的history查出来的，key的history在statedatabase当中是怎么存储的？



> 你在向别人讲解项目的情况，分配任务的时候，思路要非常清晰，让别人对现在这个项目大致是什么样子，有什么问题，接下来的任务是什么，每个人应该做什么，有什么需要注意的的事儿

区块链上也 **存储着二级节点的信息**：key是prefix + owner的id，value就是下面的这个节点信息数据结构

```go
{
  ID: string,
  URL: string,
  Name: string,
  CertFile: bytes,
  G1Pub: bytes,
  G2Pub: bytes
}
```



**Chaincode的核心功能**

Chaincode在初始化，也就是Init方法里面，可以传入一些参数来初始化chaincode，fabric里面都是通过chaincode来向区块链插入数据，查询数据，一个chaincode其实就代表了一个业务系统，从最底层来讲，chaincode并不关心存储的数据格式是什么，只提供数据的通用性操作就行。在初始化时，第一个参数是根证书，然后还可以添加其他的配置项，配置项都是以stub.PutState的方式添加到区块链上的，只不过配置项的key是有固定的prefix。

从区块链当中**查询**到的数据是DataOnChain，然后**一般会转换成KeyAndValue格式**（和DataOnChain相比，就多了个Key）：

```go
{
  Key: string,
  Value: {
    Enc int32,
    Owner string,
    Value bytes,
    Info string
    Version int32,
    Timestamp int64
  },
  History: {
    Owner string,
    Type string,
    DataId string,
    Action string,
    History string,
    Time int64
  },
  Sup: {
    Type int32,
    Pubkey bytes,
    Entity: {
      Cypher bytes,
      SupIdArr []string
    }
  }
}
```



**chaincode提供的核心功能 **包括：

* **queryData**( 参数是key，查询出来KeyAndValue，然后返回Value的protobuf base64编码格式)
* **batchExistKey**( 参数是json格式的key字符串数组，先转换成[]string，然后遍历查询key对应的Value是否存在，如果存在那么这个key是有效的，最终返回有效的key组成的数组结构的protobuf base64编码格式数据)
* **batchQueryData**( 接收到的参数是json数组字符串格式，然后转换成key的字符串数组，然后遍历这些key，查询到KeyAndValue，之后添加到结果数组里面，最后返回结果数组的protobuf base64编码后的数据字符串 )
* **batchSuperviseQuery**( 参数也是json数组字符串格式，首先也是要转换成[]string，里面都是Key，然后遍历这些Key，查询到KeyAndValue，特别的一点是，这个KeyAndValue是包括区块链当中的DataOnChain的Sup属性的，然后添加到结果数组里面，最后返回结果数组的protobuf base64编码后的数据字符串 )
* **queryHistory**( 首先参数是key，也就是args[0]，根据key调用shim.ChaincodeStubInterface.GetHistoryForKey(key)，得到一个迭代器，运行这个迭代器，然后在运行过程中，可以获取DataOnChain对象，然后判断这个迭代获取出来的对象是否已经被删除，如果没被删除则将对象以HistoryOut格式（包括Enc,Owner,Value,Info,Version,IsDelete:0,Record字段）添加到结果数组当中，如果迭代出来的对象已经被删除了，则将被删除的对象以HistoryOut格式（isDelete:1,Record）添加到结果数组当中 )
* **queryTxId**( 参数是Key，然后调用shim.ChaincodeStubInterface.GetHistoryForKey(key)，得到一个迭代器，运行这个迭代器，获取到 github.com/hyperledger/fabric/protos/ledger/queryresult.KeyModification 对象，然后不停的迭代，知道获取到最后一个对象，然后调用KeyModification对象的GetTxId方法，返回结果 )
* **queryRangeKeys**( 参数是json字符串，然后需要先转换成RangeKey struct {Start: string, End: string}，然后调用shim.ChaincodeStubInterface.GetStateByRange(startKey, endKey)得到一个迭代器，运行这个迭代器，拿到迭代器的结果的Key字段，把迭代器返回的所有Key添加到字符串数组中，然后返回数组的protobuf base64编码后的数据字符串 )
* **queryRangeData**( 参数是json字符串，然后需要先转换成RangeKey struct {Start: string, End: string}，然后调用shim.ChaincodeStubInterface.GetStateByRange(startKey, endKey)得到一个迭代器，运行这个迭代器，拿到迭代器的结果的Value值，然后Unmarshal到DataOnChain结构里面去，然后返回所有DataOnChain组成的数组 )
* **batchQueryRangeData**( 参数是一个json字符串，需要先json.Unmarshal转换成[{StartKey, EndKey}…]数组，然后遍历这个数组，在循环里面调用shim.ChaincodeStubInterface.GetStateByRange(startKey, endKey)得到一个迭代器，运行这个迭代器，拿到迭代器的结果的Value值，然后Unmarshal到DataOnChain结构里面去，一次迭代获得一个数组结果，然后把这些数组组合成一个新的数组结果，然后返回数组的protobuf base64编码后的数据字符串 )
* **createData**( 参数是一个字符串，这个字符串是SignChainItem的protobuf base64编码格式，拿到之后肯定是先转成SignChainItem，然后对SignChainItem进行一些验证，检查时间戳，然后这个SignChainItem的key是DB01:owner:type:key这样的形式，需要根据key里面的owner找到owner的二级节点信息，然后用二级节点的证书去验证SignChainItem中的签名，然后再判断区块链当中是否已经存在了参数当中的Key，有了则返回错误，如果没有，则向state database Put数据 )
* **batchCreateData**( 传入的参数其实是SignChainItem数组的protobuf base64编码格式，先把参数做转换，然后对数组当中的每一个SignChainItem进行验证，验证过程和createData类似，然后判断Key是否已经存在，最后再循环把Key-Value Put进区块链上 )
* **updateData**( 首先对传入的参数用parseChainValue进行处理，这个方法将会处理字符串的参数，返回SignChainItem对象，然后检查是否有权限修改Key对应的Value，验证通过则就可以PutData了 )
* **batchUpdateData**( 和update每多大差别，无非就是传入的参数是一个SignChainItem数组的序列化并且编码后的数据，先解析参数，然后循环对数据做检查验证，然后PutData )
* **deleteData**( 传入的参数是一个SignChainItem数组的序列化并且编码后的数据，先解析参数，然后验证参数中的签名是否有权限删除指定的数据，如果有权限，就调用shim.ChaincodeStubInterface.DelState方法删除数据 )
* **batchDeleteData**( 解析数组类型的参数，然后循环遍历数据，针对数组当中的每个元素先做deleteData类似的操作，然后再新建一个DataOnChain对象，设置DataOnChain对象的History属性，然后向区块链上Put进去Key和原来相同，Value就是刚刚新建的这个DataOnChain )
* **batchSaveData**( 这个操作可以批量存入数据，但是支持数据新添和数据更新，所以在这个方法当中，首先解析参数，然后用判断当前Key是否已经存在在区块链上了，判断的方式就是直接GetState(Key)，看看有没有Value，如果没有Value，那save操作就是插入新的数据，就走插入数据的操作流程，如果有Value，那save操作就是更新已有的数据，就走updateData的操作流程 )
* **batchOperateData**( 传入的参数是BatchOperatorIn的protobuf base64编码字符串，所以首先要转换参数，因为这个参数里包含了需要插入到链上的数据，更新的数据和要从链上删除的数据，因此， )
* **newL2Node**(  )
* **getL2Node**(  )
* **getAllL2Nodes**(  )
* **updateL2Node**(  )
* **sendToChain**(  )
* **rawInsert**(  )
* **rawQuery**(  )
* **rawDel**(  )
* **getConfig**(  )
* **newOrUpdateConfig**(  )



**HistoryOut**

```go
{
  Enc: int32,
  Owner: string,
  Value: bytes,
  Info: string,
  Version: int32,
  IsDelete: int32,
  Record: {
    Owner: string,
    Type: string,
    DataId: string,
    Action: string,
    History: string,
    Time: int64
  },
  Sup: {
    Type int32,
    Pubkey bytes,
    Entity: {
      Cypher bytes,
      SupIdArr []string
    }
  }
}
```



**SignChainItem**( 比SingleChainItem多了一个Sig字段)

```go
{
  Key: string,
  Value: {
    Enc int32,
    Owner string,
    Value bytes,
    Info string
    Version int32,
    Timestamp int64
  },
  History: {
    Owner string,
    Type string,
    DataId string,
    Action string,
    History string,
    Time int64
  },
  Sup: {
    Type int32,
    Pubkey bytes,
    Entity: {
      Cypher bytes,
      SupIdArr []string
    }
  },
  Sig: bytes
}
```



**BatchOperatorIn**

```go
{
  BatchInsert:[
    {
      // 就是SignChainItem
    },
  ],
  BatchUpdate:[
    {
      // 就是SignChainItem
    },
  ],
  BatchDelete:[
    {
      // 就是SignChainItem
    },
  ],
}
```





## 区块链+金融的应用场景

**现有的问题**

从金融 IT 基础设施的角度，仍存在一些操作风险、道德风险、 信用风险、信息保护风险等方面的不足与痛点。 

* 金融 IT 系统数据仍存在被篡改、被伪造或一致性差异的可能；

* 不同金融机构间的基础设施架构、业务流程各不相同，甚至仍涉及较多人工处理的环节，极大地增加了运营成本，也容易出现 操作风险与道德风险；

* 金融业务与金融合作或涉及多个参与方或中间方，容易提 升信任成本与摩擦成本，也存在一定的互信、协作或合作对等性问题；
* 金融业务往往复杂度较高，容易遗漏对业务全要素的记录， 有时较难对业务全流程进行追溯，无法满足监管和审计需求；
* 不同金融机构间的数据相对独立，难以实现安全高效的交互，导致进行重复的 KYC、反洗钱、反欺诈的成本较高，也间接带来 了用户数据被某些中介机构泄露的风险；
* 集中式的 IT 基础架构的可用性与适应性较弱，需要采用 分布式技术以提高健壮性或自适应性。

**能够解决的问题**

在金融场景中有不同的角色，对不不同角色，区块链的应用方式是不同的，概况来说：

* 银行机构：利用区块链技术降低清算、结算成本、提高中后台运营效率、提升流程自动化程度，降低运营成本。
* 非银行类金融机构：提升权益登记、信息存证的权威性、消减交易对手方风险、解决数据追踪与防伪问题、降低审核审计的操作成本。
* 金融监管机构：区块链为监管机构提供了一致且易于审计的数据，通过对机构间区块链的数据分析，能够比传统审计流程更快更精确地监管金融业务。