# thrift 

## 安装

> https://github.com/apache/thrift/releases 下载所需版本

按需安装减少错误发生率

```shell
./bootstrap.sh
```
查看参数
```shell
./configure --help
```
我使用的

```shell
./configure \ 
 --without-python  \
 --without-csharp  \
 --without-ruby \
 --without-perl  \
 --without-php  \
 --without-haskell  \
 --without-erlang  
```

```shell
make & make install
```