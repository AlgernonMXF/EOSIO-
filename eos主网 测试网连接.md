# 搭建测试节点-连接EOS主网/测试网 
连接不同的eos网络，一般只需要修改`config.ini`和`genesis.json`文件；特殊情况需要重新获取源代码重新编译安装
需要重点注意的只有：`config.ini`和`genesis.json`文件

依次连接：
* EOS Force
* CryptoKylin-Testnet

## EOS Force
EOS Force是EOS的主网之一，
主页：https://www.eosforce.io/
具体介绍：https://eosforce.github.io/Documentation/#/zh-cn/what_is_eosforce
github：https://github.com/eosforce/eosforce
在该网络上执行的操作几乎都会消耗EOS（已经上市的数字货币），简单来说，创建账号、转账、投票等都是要花钱的。

### 部署服务器节点并连接EOS Force主网
服务器（eos_blockchain）配置：
阿里云服务器2-CPU 8G内存 1TB硬盘 Ubuntu 16.04.4版本

文件路径：
源代码：`/wyf/eosforce`
区块信息：`/opt/eosdata/eosforce`
配置文件：`~/.local/share/eosio/nodeos/config`


以下步骤参考官方文档https://eosforce.github.io/Documentation/#/zh-cn/eosforce_bp

#### 1.  下载源码
```C++
apt-get update && apt-get install -y git wget

//获取源代码
git clone https://github.com/eosforce/eosforce.git eosforce
```

#### 2. 安装EOS Force
```C++
cd eosforce && git submodule update --init --recursive 
./eosio_build.sh

//创建配置文件夹
mkdir -p ~/.local/share/eosio/nodeos/config
//虎获取genesis.json
curl https://raw.githubusercontent.com/eosforce/genesis/master/genesis.json -o ~/.local/share/eosio/nodeos/config/genesis.json

//将配置文件拷贝至config
cp build/contracts/eosio.token/eosio.token.abi build/contracts/eosio.token/eosio.token.wasm ~/.local/share/eosio/nodeos/config
cp build/contracts/System/System.abi build/contracts/System/System.wasm ~/.local/share/eosio/nodeos/config
cp build/contracts/eosio.bios/eosio.bios.abi build/contracts/eosio.bios/eosio.bios.wasm ~/.local/share/eosio/nodeos/config
cp build/contracts/eosio.msig/eosio.msig.abi build/contracts/eosio.msig/eosio.msig.wasm ~/.local/share/eosio/nodeos/config

//编译安装
cd build && make install
```

#### 3. 获取config.ini文件
```C++
//获取config.ini并拷贝至config
wget http://download.aitimeout.site/config.ini
cp config.ini ~/.local/share/eosio/nodeos/config/
```

#### 4. 修改config.ini
* p2p-server-address
`p2p-server-addree = 0.0.0.0:9876`

* genesis.json路径
`genesis-json = YOUR-GENESIS-PATH`

* p2p-peer-address
查看http://t.eosforce.io/p2p/  任选3个节点地址并添加即可

#### 5. 启动节点测试
```C++
cd build/programs/nodeos 
./nodeos
```
可同时打开另一个终端查看链信息和区块高度，对比EOS Force信息；EOS Force信息参考https://w1.eosforce.cn/v1/chain/get_info
```C++
cleso get info --data-dir /opt/eosdata/eosforce
```

## CryptoKylin
CryptoKylin是EOS的一个测试网络，可供开发人员测试DAPP或者测试如何运行BO节点。
官方网站：https://www.cryptokylin.io/
区块浏览器：https://tools.cryptokylin.io/#/tx

### 部署服务器节点连接kylin测试网
服务器配置同上。

文件路径：
源代码：`/wyf/eos`
区块信息：`/opt/eosdata/kylin`
配置文件：`~/.local/share/eosio/nodeos/config-kylin`

配置步骤如下：
#### 1. 在CryptoKylin Testnet上创建账户
将以下URL中的`new_account_name`替换成你想要的账户名然后打开，即可自动创建新账户
http://faucet.cryptokylin.io/create_account?new_account_name

获取free token
将以下URL中的`new_account_name`替换成你想要获取token的账户名然后打开，kylin网络就会给你打钱
http://faucet.cryptokylin.io/get_token?your_account_name

#### 2. 创建genesis.json文件
```C++
cd ~/.local/share/eosio/nodeos/config-kylin

//使用vim创建genesis.json文件
vim genesis.json
```
然后将网页https://github.com/cryptokylin/CryptoKylin-Testnet/blob/master/genesis.json 中genesis.json文件的内容拷贝到刚刚创建的文件中并保存退出

#### 3. 创建config.ini文件
```C++
cd ~/.local/share/eosio/nodeos/config-kylin

//使用vim创建config.ini文件
vim config.ini
```
* 将网页https://github.com/cryptokylin/Cryptokylin-Doc/blob/master/configure_desc/fullnode_config_demo.ini 中genesis.json文件的内容拷贝到刚刚创建的文件中
* 修改以下地方：
	* p2-peer-address：
	参考网页https://github.com/cryptokylin/CryptoKylin-Testnet 中列出的P2P LIST，选择3个以上加入到config.ini中
	
	* agent-name:
	修改成自己的账号名称
	
	* 添加genesis.json文件路径
	```C++
	genesis-json = "/root/.local/share/eosio/nodeos/config-kylin/genesis.json"
	```

（注: 该配置下此时节点只是全节点而非BP节点；若想成为BP节点还应先在社区中获得一定的投票）

#### 4. 启动节点测试
```C++
cd /wyf/eos/build/programs/nodeos

//指定config.ini文件路径以及区块数据存储路径
./nodeos --config-dir ~/.local/share/eosio/nodeos/config-kylin --data-dir /opt/eosdata/kylin
```

可同时打开另一个终端查看链信息和区块高度，对比CryptoKylin Testnet信息；
```C++
cleso get info --data-dir /opt/eosdata/eosforce
```
CryptoKylin Testnet chain id：
`5fff1dae8dc8e2fc4d5b23b2c7665c97f9e9d8edf2b6485a86ba311c25639191`
（参考https://github.com/cryptokylin/CryptoKylin-Testnet）

CryptoKylin Testnet区块高度信息参考:
https://tools.cryptokylin.io/#/blocks

