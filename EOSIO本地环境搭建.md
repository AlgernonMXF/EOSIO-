# EOS本地环境搭建

>写在前面:
>由于EOS技术尚在发展中，EOSIO代码更新较快，参考本文安装时可能会出现问题，推荐读者同时参考EOS.IO官方给出的开发文档https://developers.eos.io/eosio-nodeos/docs/overview-1 
>注：该官方教程中依旧可能存在与代码不符合的地方，具体问题可参考下载的EOS源代码目录自行解决

## 软件安装
* Homebrew
访问https://brew.sh/, 按照**Install Homebrew**指示下载


* 安装依赖包
【请在安装前参考https://developers.eos.io/eosio-nodeos/docs/overview 网页中的**Manually Install Dependencies**】

	```
	* Clang 4.0.0
	* CMake 3.5.1
	* Boost 1.6.6
	* OpenSSL
	* LLVM 4.0
	* secp256k1-zkp(Crypotonomex branch)
	```
  
	执行以下命令，安装编译环境

	```
  brew install git cmake autoconf automake libtool boost openssl llvm@4 gmp pkg-config
	```
	
  手动安装secp256k1-zkp
  
	```
  git clone https://github.com/cryptonomex/secp256k1-zkp.git
  cd secp256k1-zkp
  ./autogen.sh
  ./configure
  make
  sudo make install
	```

## 配置WASM环境
```
mkdir  ~/wasm-compiler
cd ~/wasm-compiler
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/llvm.git
cd llvm/tools
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/clang.git
cd ..
mkdir build
cd build
cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=.. -DLLVM_TARGETS_TO_BUILD= -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=WebAssembly -DCMAKE_BUILD_TYPE=Release ../
make -j4 install
```

## 获取EOS源代码
【请在下载前参考https://developers.eos.io/eosio-nodeos/docs/getting-the-code 】
```
git clone https://github.com/EOSIO/eos --recursive
```

如果下载时未添加 `--recursive` ，可通过运行以下命令在代码中添加子模块
```
git submodule update --init --recursive
```


## 使用WASM编译器完整编译EOS源代码
<!-- more -->
注：新版mac移除了设置Openssl环境变量，需手动设置OPENSSL_ROOT_DIR和OPENSSL_LIBRARIES，读者在设置时应注意自己电脑上openssl的子目录是否为`1.0.21`
```
mkdir eos/build/
cd eos/build/

配置环境变量
export WASM_LLVM_CONFIG=~/wasm-compiler/llvm/bin/llvm-config
export LLVM_DIR=/usr/local/Cellar/llvm/4.0.1/lib/cmake/llvm

设置WASM_ROOT以及Openssl变量
cmake -DWASM_ROOT=~/opt/wasm-compiler/llvm -DOPENSSL_ROOT_DIR=/usr/local/Cellar/openssl/1.0.2l -DOPENSSL_LIBRARIES=/usr/local/Cellar/openssl/1.0.2l/lib ..
```

## 配置完成，检查目录
此时可查看eo/build/programs中生成的几个目录：

* nodeos - 服务端区块链节点组件，可以启动EOSIO的核心守护进程，可生产区块
* cleos	- 命令行界面，与区块链交互并管理钱包
* keosd - 将EOSIO密钥安全存储在钱包中

## 运行本地单节点测试网络
【请先阅读参考官方文档https://developers.eos.io/eosio-nodeos/docs/local-single-node-testnet】
### 测试
此时可在`eos/build/programs/nodeos`中找到的二进制可执行文件`nodeos`
执行如下命令
```
cd build/programs/nodeos
./nodeos -e -p eosio --plugin eosio::chain_api_plugin --plugin eosio::history_api_plugin
```

得到类似的消息，则表示已经可以正确生产区块
```
1575001ms thread-0   chain_controller.cpp:235      _push_block          ] initm #1 @2017-09-04T04:26:15  | 0 trx, 0 pending, exectime_ms=0
1575001ms thread-0   producer_plugin.cpp:207       block_production_loo ] initm generated block #1 @ 2017-09-04T04:26:15 with 0 trxs  0 pending
1578001ms thread-0   chain_controller.cpp:235      _push_block          ] initc #2 @2017-09-04T04:26:18  | 0 trx, 0 pending, exectime_ms=0
1578001ms thread-0   producer_plugin.cpp:207       block_production_loo ] initc generated block #2 @ 2017-09-04T04:26:18 with 0 trxs  0 pending
...
eosio generated block 046b9984... #101527 @ 2018-04-01T14:24:58.000 with 0 trxs
eosio generated block 5e527ee2... #101528 @ 2018-04-01T14:24:58.500 with 0 trxs
...
```

### 修改nodeos配置文件
可在如下路径处找到`nodeos`的配置文件

* Mac OS: `~/Library/Application Support/eosio/nodeos/config`
* Linux: `~/.local/share/eosio/nodeos/config`

在终端执行如下命令，打开finder中的文件`config.ini`
```
cd ~/Library/Application\ Support/eosio/nodeos/config
open .
```

修改`config.ini`，添加/修改如下设置
```c++
# Enable production on a stale chain, since a single-node test chain is pretty much always stale
enable-stale-production = true
# Enable block production with the testnet producers
producer-name = eosio
# Load the block producer plugin, so you can produce blocks
plugin = eosio::producer_plugin
# As well as API and HTTP plugins
plugin = eosio::chain_api_plugin
plugin = eosio::http_plugin
# This will be used by the validation step below, to view history
plugin = eosio::history_api_plugin
```

此时可通过以下命令执行nodeos并产生区块
```
 ./programs/nodeos/nodeos
```

`nodeos`将运行结果数据存储在如下位置

* Mac OS: `~/Library/Application Support/eosio/nodeos/data`
* Linux: `~/.local/share/eosio/nodeos/data`

可通过`--data-dir`设置数据存储位置
