步骤一：初始化

chaincode的Init方法

1. 向worldstate里面写入 CONF:MainChannel-And-Chaincode => mainChannel & mainChaincode的key-value值；
2. 向worldstate里面写入 CONF:CurrentChaincode => curChaincode
3. 如果还有最后一个参数，也就是全局配置的值，则向worldstate里面写入 CONF-V2:SYS-ON-CHAIN => config



LocalStub的Init方法参数：(name string, args []string)，name就是Funcname，剩余的参数含义：args[0] -> mainChannel, args[1] -> mainChaincode, args[2] -> curChaincode，最后一个参数时全局配置，也就是args[3]



