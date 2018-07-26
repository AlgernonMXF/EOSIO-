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

