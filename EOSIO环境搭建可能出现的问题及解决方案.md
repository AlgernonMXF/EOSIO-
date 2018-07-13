# EOSIO环境搭建可能出现的问题及解决方案

## 问题1
```
ERROR: Failed to find Gettext libintl (missing: Intl_INCLUDE_DIR)
需要执行以下命令
brew unlink gettext && brew link --force gettext
参考链接
https://github.com/EOSIO/eos/issues/2028?ref=tokendaily
```
