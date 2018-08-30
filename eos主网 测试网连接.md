# EOS主网/测试网 连接
连接不同的eos网络，一般只需要修改`config.ini`和`genesis.json`文件；特殊情况需要重新获取源代码重新编译安装
需要重点注意的只有：`config.ini`和`genesis.json`文件

依次连接：
* EOS Force
* 

## EOS Force
EOS Force是EOS的主网之一，
主页：https://www.eosforce.io/
具体介绍：https://eosforce.github.io/Documentation/#/zh-cn/what_is_eosforce
github：https://github.com/eosforce/eosforce

### 部署节点并连接EOS Force主网
服务器（eos_blockchain）配置：
阿里云服务器2-CPU 8G内存 1TB硬盘 Ubuntu 16.04.4版本

文件路径：
源代码：`/wyf/eosforce`
区块信息：`/opt/eosforce`
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

#### 5. 启动节点并测试
```C++
cd build/programs/nodeos 
./nodeos
```
可同时打开另一个终端查看链信息和区块高度，对比EOS Force信息；EOS Force信息参考https://w1.eosforce.cn/v1/chain/get_info
```C++
cleso get info
```

