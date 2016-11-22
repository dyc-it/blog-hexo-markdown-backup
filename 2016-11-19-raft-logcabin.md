---
title: raft-logcabin
date: 2016-11-19 13:24:52
categories: distributed-system
tags: raft logcabin
---

分析Raft的实现之一——[LogCabin](https://github.com/logcabin/logcabin)。

<!-- more -->
# 1 简介
&emsp;&emsp;Logcabin是Raft的设计者之一Diego Ongaro使用C++对Raft的实现。[Github地址](https://github.com/logcabin/logcabin)
## 1.1 依赖工具


```
# CentOS
yum install gcc gcc-c++ protobuf protobuf-compiler cryptopp doxygen
```
### 1.1.1 scons
&emsp;&emsp;[scons](http://scons.org/)（software construction）是一个项目构建工具。与make相比，Makefile的编写规则比较复杂，scons通过SConstruct（实际上是一个Python脚本）来管理依赖关系，更为简单。  
### 1.1.2 g++
&emsp;&emsp;gcc和g++的关系是什么？gcc最开始的时候是GNC C Compiler，是一个C语言编译器。但是后来这个项目里面集成了更多其他不同语言的编译器，GCC就代表GNU Compiler Collection。g++则是GCC的C++编译器。 [参考](https://www.zhihu.com/question/20940822)  
### 1.1.3 protobuf
&emsp;&emsp;[Protocol Buffers](https://developers.google.com/protocol-buffers/)（简称Protobuf）是Google为序列化格式化的数据开发的语言中立、平台中立并且可扩展的数据交换格式。截止2016年11月，支持C++、C#、GO、JAVA、PTYHON共五种语言。 详细使用请参考[另一篇博文](/2016/11/19/protobuf/)。
### 1.1.4 crypto++
&emsp;&emsp;[crypto++](https://www.cryptopp.com/)是使用C++开发的开源密码算法库。
### 1.1.5 doxygen
[doxygen](http://www.stack.nl/~dimitri/doxygen/)是一个C++、C、Java等语言的文档生成器。 




