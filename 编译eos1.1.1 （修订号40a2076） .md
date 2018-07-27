# 编译eos1.1.1

## 问题一
依照官方文档安装好依赖包，并获取源码之后，执行camke，遇到如下错误：       

`Could not find a package configuration file provided by "libmongoc-1.0”`

遇到这个问题之后google了一下 `eos libmongoc-1.0`，发现了有人也遇到了类似的问题，       
参见网页https://github.com/EOSIO/eos/issues/4719       
阅读这位朋友的列出的，使用 `./eosio_build.sh`自动编译eos的过程之后，          
发现自动安装过程中出现了如下信息：              
```
Checking MongoDB C++ driver installation.
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   617    0   617    0     0    866      0 --:--:-- --:--:-- --:--:--   865
100 6193k  100 6193k    0     0  1096k      0  0:00:05  0:00:05 --:--:-- 1475k
```
想一下意识到，这是要自己手动安装libmongoc-1.0和MongoDB C++ driver吧，于是搜索教程开始安装
参考文档：
* 官方：`http://mongodb.github.io/mongo-cxx-driver/mongocxx-v3/installation/`
* 别人写的教程：http://www.andybear.top/p/10ff99a0-28a9-11e7-bae0-d13403639de6


## 问题二

安装完毕之后重新执行cmake，又遇到如下错误：
```
CMake Error at plugins/mongo_db_plugin/CMakeLists.txt:22 (find_package):
  By not providing "Findlibbsoncxx-static.cmake" in CMAKE_MODULE_PATH this
  project has asked CMake to find a package configuration file provided by
  "libbsoncxx-static", but CMake did not find one.

  Could not find a package configuration file provided by "libbsoncxx-static"
  with any of the following names:

    libbsoncxx-staticConfig.cmake
    libbsoncxx-static-config.cmake

  Add the installation prefix of "libbsoncxx-static" to CMAKE_PREFIX_PATH or
  set "libbsoncxx-static_DIR" to a directory containing one of the above
  files.  If "libbsoncxx-static" provides a separate development package or
  SDK, be sure it has been installed.


-- Configuring incomplete, errors occurred!
```
意思是说我没有安装`libbsoncxx-static`?
到文件`plugins/mongo_db_plugin/CMakeLists.txt`中查看发现，哎原来这里面写了要怎么安装mongodb，不过是Ubuntu版的，sad，MacOS系统要怎么拯救呢              
文件中具体描述如下：     
```
sudo apt-get install pkg-config libssl-dev libsasl2-dev
wget https://github.com/mongodb/mongo-c-driver/releases/download/1.8.0/mongo-c-driver-1.8.0.tar.gz
tar xzf mongo-c-driver-1.8.0.tar.gz
cd mongo-c-driver-1.8.0
./configure --disable-automatic-init-and-cleanup --enable-static
make
sudo make install

git clone https://github.com/mongodb/mongo-cxx-driver.git --branch releases/stable --depth 1
cd mongo-cxx-driver/build
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local -DBUILD_SHARED_LIBS=OFF ..
sudo make EP_mnmlstc_core
make
sudo make install
sudo apt-get install mongodb
```

对比一下我自己在MacOS系统下安装mongodb的步骤如下：

参考官方文档：https://mongodb.github.io/mongo-cxx-driver/mongocxx-v3/installation/#installing-the-mongocxx-driver      
* 安装MongoDB C Driver
```
//下载最新版tarball
$ curl -LO https://github.com/mongodb/mongo-c-driver/releases/download/1.12.0/mongo-c-driver-1.12.0.tar.gz
$ tar xzf mongo-c-driver-1.12.0.tar.gz
$ cd mongo-c-driver-1.12.0

//编译并安装
$ mkdir cmake-build
$ cd cmake-build
$ cmake -DENABLE_AUTOMATIC_INIT_AND_CLEANUP=OFF ..
```
  
* 下载mongocxx driver
```
git clone https://github.com/mongodb/mongo-cxx-driver.git \
    --branch releases/stable --depth 1
cd mongo-cxx-driver/build

//配置cmake
cmake -DCMAKE_BUILD_TYPE=Release -DBSONCXX_POLY_USE_MNMLSTC=1 \
-DCMAKE_INSTALL_PREFIX=/usr/local ..

//编译安装MNMLSTC的核心组件
sudo make EP_mnmlstc_core

//编译安装
make && sudo make install
```


对比之后发现，编译安装MongoDB C Driver时没有执行
`./configure --disable-automatic-init-and-cleanup --enable-static`


## 然后我发现官网有官方教程！！
网址：https://developers.eos.io/eosio-nodeos/v1.0.0/docs/clean-install-macos-sierra-1012-and-higher                       
其中也有提到如何安装MongoDB C++ driver              
但是我在执行`make`时却出错了，错误如下：
```
./src/mongoc/mongoc-index.h:78:41: error: expected function body after function
      declarator
mongoc_index_opt_geo_get_default (void) BSON_GNUC_CONST;
                                        ^
./src/mongoc/mongoc-index.h:80:40: error: expected function body after function
      declarator
mongoc_index_opt_wt_get_default (void) BSON_GNUC_CONST;
                                       ^
In file included from src/mongoc/mongoc-async.c:21:
In file included from ./src/mongoc/mongoc-async-cmd-private.h:26:
In file included from ./src/mongoc/mongoc-client.h:36:
./src/mongoc/mongoc-ssl.h:47:35: error: expected function body after function
      declarator
mongoc_ssl_opt_get_default (void) BSON_GNUC_CONST;

4 errors generated.
make[1]: *** [src/mongoc/libmongoc_la-mongoc-apm.lo] Error 1
make[1]: *** Waiting for unfinished jobs....
4 errors generated.
make[1]: *** [src/mongoc/libmongoc_la-mongoc-async.lo] Error 1
4 errors generated.
make[1]: *** [src/mongoc/libmongoc_la-mongoc-async-cmd.lo] Error 1
make: *** [all-recursive] Error 1
```
看一下错误内容，好像是读取代码时出错···，初步猜测是编译器的问题。

