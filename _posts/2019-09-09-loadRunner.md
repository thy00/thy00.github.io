---
title:  loadRunner
categories: 
 - 压测
tags: 学习

---



## 负载测试&压力测试&性能测试

性能调优

> 通过对处在特定环境下的系统进行测试以完成相关的验证，而且往往要根据测试的结果，对系统的设计、代码和配置等进行调整，提高系统的性能。许多时候，系统性能的改善是测试、调整、再测试、再调整、……一个持续改进的过程



### 负载测试

获得系统正常工作时所能承受的最大负荷（容量）

### 压力测试

系统奔溃的极限条件以及系统是否能自我恢复

### 性能测试

获取系统在特定条件下的性能指标数据



## 步骤

### 创建脚本



### 中央控制器调度虚拟用户

### 运行并获得结果



## 性能需求

**多少用户**

做

**什么业务**

的

**响应时间控制在某个范围**

**tps不低于**

**系统资源占用**

整体平稳



## socket_io脚本



***

我想用loadrunner测试websocket

lr12写好脚本，然而只支持50的并发，根本不够用。然后换回11,11和12的脚本不兼容，并且11也支持100的websocket测试，，，，

所以做了一个决定，抛弃各种限制的loadrunner，改用jmeter测试



pv page view



内存泄露

连接泄露

性能瓶颈



top









