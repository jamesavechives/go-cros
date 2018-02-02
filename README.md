# go-cros
CROS chain in golang implementation based on Ethereum protocol 
摘要:
以太坊分叉主要是通过修改chainId和networkId实现。目前在本机建立的以太坊私有链本质上也是一种分叉，但它具有以下几个缺点: 一、要手动设置创世区块参数以创建创世区块；二、每次运行geth需要手动设置datadir, networkid, rpcdomainanme等参数；三、需要将节点设置成nodiscover以避免被主链发现，但在这种情况下如果要在私链上加入节点需要手动进行配置。总之缺点就是：与运行主链的节点相比非常麻烦。
本文从源码上直接修改chainId和neworkId，编译生成可执行文件之后，可以直接运行，不需要设置创世区块，不需要设置运行参数，而且节点可以自发现，跟主链的易用性是一模一样的。

前提条件:
1.下载githum.com/ethereum/go-ethereum源码；
2.下载并安装Golang,并将可执行文件路径设置成环境变量,设置GoPath；
3.如果是windows，还需要下载mingw作为编译环境并将可执行文件路径设置成环境变量。


一、设置chainId
源码目录: params/config.go
修改MainnetChainConfig的ChainId 为1-4以外的整数。(因为1-4是以太坊基金会保留的官方ID，分别代表主链和几条不同的测试链)

二、设置种子节点
源码目录:params/bootnodes.go
将MainnetBootnodes所包含的种子节点都去掉（如果有稳定的节点运行新链，可以将这些新节点放入MainnetBootnodes中）

三、设置创世区块
源码目录: core/genesis.go
修改DefaultGenesisBlock()函数体里面的参数设置，包括Nonce、ExtraData、GasLimit、Difficulty。

四、设置networkId
源码目录: eth/config.go
修改DefaultConfig中的NetworkId为1-4以外的整数。

五、编译，生成可执行文件
在代码根目录下面运行:
Windows环境下运行 go install -v ./cmd/...
Ubuntu环境下运行 make geth
生成的可执行文件路径:
Windows环境下 %GoPath%/bin/geth.exe
Ubuntu 环境下 代码根目录/build/bin/geth

六、设置种子节点
运行geth并进入控制台后运行命令 admin.nodeInfo.enode,将获取的enode加入文件static-nodes.json中。格式如下:
[
“enode:{enodeId1}[::]@{port1}”,
“enode:{enodeId2}[::]@{port2}”,
“enode:{enodeId3}[::]@{port3}”,
...
]
将[::]替换成本机的ip。
在windows环境下：将static-nodes.json文件放入%APPDATA%/ethereum/文件夹下面
在ubuntu环境下：将static-nodes.json文件放入~/.ethereum文件夹下面
保证每一个种子节点下面的static-nodes.json的内容一致，再重启geth。
七、测试节点的连通性
运行admin.peers命令，如果包含一个或多个节点信息，代表新链上的本节点已经连接至其它节点了。也可以通过不同节点账号之间的转账等操作进一步测试多个节点的通信。