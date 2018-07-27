# 搭建本地多节点测试网络
> 写在前面：                    
> 刚刚参考官方文档搭建好本地的多节点测试网络，在搭建的过程中依旧遇到了很多错误，在此总结我自己的解决方法，仅供参考           
> eosio源码版本：1.0                
> 官方文档地址：https://developers.eos.io/eosio-nodeos/v1.0.0/docs/local-multi-node-testnet
             
             
## 启动钱包管理器
`keosd --http-server-address 127.0.0.1:8899`                         
该命令使keosd监听本地8899端口                  


## 创建钱包
`cleos --wallet-url http://localhost:8899 wallet create -n mxf`                      
该命令在8899端口创建了一个钱包，名称为`mxf`              
你可以自定义钱包的名称                 
                     
创建成功后，cleos会返回如下信息：
```
Creating wallet: mxf
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5J8vDvyNxdaTPscMyatQCLftgoRvapkbxwAFUUUZvTYG1QRFioh"
```
该钥匙是用于打开钱包`mxf`的钥匙，要记得保存                   


## 导入私钥
`cleos wallet import 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3 -n mxf`                   
官方文档中是这么描述的:                 
> The private blockchain launched in the steps above is created with a default initial key which must be loaded into the wallet.
                          
也就是说，在本地运行的私链是通过一个默认的初始密钥创建的，要首先将该密钥导入本地钱包。                
具体原理我还不清楚，但是我们的确可以在`nodeos`中的`config.ini`文件中找到如下代码：                
文件路径：`Library/Application\ Support/eosio/nodeos/config/config.ini` 
```C++
signature-provider = EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV=KEY:5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```                  
         
## 启动第一个生产者节点
`nodeos --enable-stale-production --producer-name eosio --plugin eosio::chain_api_plugin --plugin eosio::net_api_plugin
`                  
其中producer-name为`eosio`，即启动nodeos默认的生产者账户                   
换句话说，若使用`nodeos`直接启动nodeos，使用的生产者账户即为`eosio`

## 启动第二个生产者节点
### 加载bios合约
要启动多个节点，首先要加载`bios`合约，该合约使得我们拥有控制其他账户资源分配的权限。                    
官方文档中描述的命令如下：             
`cleos --wallet-url http://localhost:8899 set contract eosio build/contracts/eosio.bios`               
但是我在执行该命令时出错了，错误信息如下：             
```
710381ms thread-0   main.cpp:2756                 main                 ] Failed with error: Assert Exception (10)
!wast.empty(): no wast file found eos/build/contracts/eosio.bios/eosio.bios.wast
```
于是我尝试修改路径，将命令改为：                 
`leos --wallet-url http://localhost:8899 set contract eosio ../../contracts/eosio.bios`                   
然后就成功了···               
成功后cleos会返回如下信息：
```
Reading WAST/WASM from ../../contracts/eosio.bios/eosio.bios.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: ab3367cea69264e07eab0ae9e70bd1396994c164e76081d05932f54df466c580  3720 bytes  6487 us
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001621260037f7e7f0060057f7e7e7e7e...
#         eosio <= eosio::setabi                {"account":"eosio","abi":"0e656f73696f3a3a6162692f312e30050c6163636f756e745f6e616d65046e616d650f7065...
warning: transaction executed locally, but may not be confirmed by the network yet
```            
### 创建第二个生产账号
* 获取用于创建账户的密钥

官方教程中直接使用`cleos create key`获取钥匙              
我使用的是             
`cleos --wallet-url http://localhost:8899 create key`
使用两次该命令，获取两对公钥私钥，一对作为owner的公私钥，一对作为active的公私钥                                   
结果如下：
```
Private key: 5KYydgmWa61UzN6h8WftvfJDtniGAGEphJe8rH4vfaN62YWfov8
Public key: EOS7fUFfx28CqGoC2i3EJLdvW8n4nhSL4qmW2qHuvLhkKA3yFdeQ7

Private key: 5Jgd8MptkNhK1zNTnvafi2EkfmNZq8gerr3WRhKX4cWpgqWBC6J
Public key: EOS6VWVdRd7cCeyiajDV6Jx8UAXYbgzXez6Msh3bHVgoLYhi7Nfdz
```

* 将两对密钥导入钱包
```
cleos --wallet-url http://localhost:8899 wallet import 5KYydgmWa61UzN6h8WftvfJDtniGAGEphJe8rH4vfaN62YWfov8 -n mxf

cleos --wallet-url http://localhost:8899 wallet import 5Jgd8MptkNhK1zNTnvafi2EkfmNZq8gerr3WRhKX4cWpgqWBC6J -n mxf
```

* 使用密钥创建账户
`cleos --wallet-url http://localhost:8899 create account eosio inita EOS7fUFfx28CqGoC2i3EJLdvW8n4nhSL4qmW2qHuvLhkKA3yFdeQ7 EOS6VWVdRd7cCeyiajDV6Jx8UAXYbgzXez6Msh3bHVgoLYhi7Nfdz`           
创建成功会显示如下信息：
```
executed transaction: dc44fa169c1309dc4f9748f1e01c07d452447ef32d866b1b5fcbd34d32721501  200 bytes  1317 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"inita","owner":{"threshold":1,"keys":[{"key":"EOS7fUFfx28CqGoC2i3EJLdvW8n4n...
warning: transaction executed locally, but may not be by the network yet
```

* 启动第二个节点
```
nodeos --producer-name mxf --plugin eosio::chain_api_plugin --plugin eosio::net_api_plugin --http-server-address 127.0.0.1:8890 --p2p-listen-endpoint 127.0.0.1:9878 --p2p-peer-address 127.0.0.1:9876 --config-dir node0 --data-dir node0 --private-key [\"EOS7fUFfx28CqGoC2i3EJLdvW8n4nhSL4qmW2qHuvLhkKA3yFdeQ7\",\"5KYydgmWa61UzN6h8WftvfJDtniGAGEphJe8rH4vfaN62YWfov8\"]
```
启动成功会发现有类似信息:
```
1807525ms thread-0   producer_plugin.cpp:294       on_incoming_block    ] Received block 4e955fd00b5b9c17... #3963 @ 2018-07-27T06:30:07.500 signed by eosio [trxs: 0, lib: 3962, conf: 0, latency: 25 ms]
1808008ms thread-0   producer_plugin.cpp:294       on_incoming_block    ] Received block 7da5fe81e8cef99e... #3964 @ 2018-07-27T06:30:08.000 signed by eosio [trxs: 0, lib: 3963, conf: 0, latency: 8 ms]
```

* 将第二个节点变为生产者节点
```
cleos --wallet-url http://localhost:8899 push action eosio setprods "{ \"schedule\": [{\"producer_name\": \"mxf\",\"block_signing_key\": \"EOS7fUFfx28CqGoC2i3EJLdvW8n4nhSL4qmW2qHuvLhkKA3yFdeQ7\"}]}" -p eosio@active
```
启动成功返回信息如下:          
```
executed transaction: f0c4dcab83f3c12629b288ae63665221f3e01f6a550ffa253193633000912695  136 bytes  2799 us
#         eosio <= eosio::setprods              {"schedule":[{"producer_name":"mxf","block_signing_key":"EOS7fUFfx28CqGoC2i3EJLdvW8n4nhSL4qmW2qHuvLh...
warning: transaction executed locally, but may not be confirmed by the network yet
```

同时，在第二个`nodeos`中，会发现从`Received block`变成了`Produced block`，这代表已经成功将第二个节点变成了生产者节点。


