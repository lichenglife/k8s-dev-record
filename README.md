#  k8s-dev-record  
《kubernetes开发指南》

## 内容简介

本项目旨在记录kubernetes 0.20.4版本源码学习及二次开发相关的文档

# 大纲目录

#####    第一章 开发环境与编译测试

- 1.1 编译kubernets源码

  - ​      linux 环境 编译

  - ​       docker 中编译               

-  1.2  Makefile 与编译脚本分析

#####  第二章  client-go 开发指南

- 2.1  环境搭建
- 2.2 简单示例
- 2.3 认识证书与kubeconfig
- 2.4 client-go  架构
- 2.5 dynamic client
- 2.6 rest client
- 2.7 cache client
- 2.8 informer 

##### 第三章  CRD 与  adminssion webhook 开发

- 3.1   kubebuilder cronjob 示例解析

- 3.2  API 版本

- 3.3 CRD  编码

  - 3.31  kubebuilder  

  - 3.3.2  operator-framwork

  -  3.3.3   operator 的全生命周期管理

    

##### 第四章  CRI 实现

- 4.1 pod 接口
- 4.2 容器接口 
- 4.3 镜像接口
- 4.4 实现简单的CRI 

##### 第四章  CNI  实现

- 5.1 CNI  概览
- 5.2 CNI   命令与参数
- 5.3 CNI  命令返回值
- 5.4  网络配置参数
- 5.5  IP 地址分配
- 5.6 动手实现 CNI

##### 第六章  CSI  实现

- 6.1   CSI   概览
- 6.3    CSI 具体分析
- 6.4    动手实现  CSI 

##### 第七章  扩展apiserver

##### 第八章 定制kube-sheduler

##### 第九章  定制kubelet

##### 第十章  开发 kubelet plugin 

##### 第十一章  kubernets 存储原理

##### 第十二章  kubernetes 网络原理

##### 第十三章  kubectl 



## 目标

整理完本书，可以达到以下目标：

1.  熟练使用client-go 进行开发，用于项目中PASS平台接口开发；
2.  熟练operator开发流程，可以使用kubebuilder 或者operator-sdk根据需求定制化operator；
3. 熟悉kubernetes 各组件代码架构，核心处理逻辑，掌握各个组件的扩展开发方式；
4. 具备 CNI  CSI CRI  开发的能力；
5. 定制化kubelet plugin开发；
6. 收集 圈内对kubernetes  二次开发的需求，模拟开发。





















