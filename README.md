# 功能要求

## 背景

znsq项目是按照nsq的功能，自己实现的一个分布式队列。功能有所精简，比如去除掉可视化的nsq管理，但是核心功能保持不变。

znsq 0.1 版本将是完全按照自己的想法设计实现。后续版本会对比nsq的性能和代码实现，进行自我优化。

zsnq系列项目以学习为目标，经过这一系列项目，本猿可以达到nsq作者的层次。

## 功能

高效分布式消息队列，实现以下功能：

1. 做到生产者和消耗者解耦
2. 水平伸缩，动态扩缩容
3. 负载均衡和组播方式的消息路由分发
4. 支持基于流式（高吞吐）和基于任务（低吞吐）工作负载方式
5. 内存存储（超过阈值的消息保存在磁盘中）
6. 消费者对生产者的自动服务发现
7. 与数据格式无关

## 限制

1. 没有副本机制，如果需要容错，可以使用其他方法，比如使用slave集群做备份
2. 消息至少会被传递一次，由于网络连接，传递超时等原因，消息可能会被传递多次，client需要保证幂等性或者重复数据的删除
3. 消息传递不保证顺序
4. 消费者从生产者获取所有的topic信息，满足最终一致性

## 性能要求

100byte消息，3个topic，性能满足每秒200w次请求

参考：[Performance-nsq](https://nsq.io/overview/performance.html)


# 方案分析

> 生产者和消耗者解耦

zlookup负责做服务发现，以及生产者，消费者，以及具体的node节点解耦。zlookup顾名思义，负责维护topic对应的node节点。

> 水平伸缩，动态扩缩容

添加新节点后，会向zlookup汇报心跳，计入到空闲节点中。

> 负载均衡和组播方式的消息路由分发

采取topic和channel结构，一个集群topic全局唯一，channel为一个topic的多个内存拷贝，可以被多个client接收。同时一个channel接收的多个client，随机选择一个client。

>  支持基于流式（高吞吐）和基于任务（低吞吐）工作负载方式

支持单个消息写入，也支持多个消息一起写入

> 服务发现

zq节点向zlookup建立tcp长连接，定期发送心跳。zlookup内存维护存货node，做服务发现，以及topic的任务派发。

对死亡的节点，zlookup要进行topic的转移，以杜绝单点问题。

## CAP权衡

nsq其实不是一个分布式分布式服务，第一不能容忍单点故障，zlookup单点，zq单点，未消费完的数据会丢失。

没有同步问题，不存在一致性。

也不存在网络分区的问题。

还有很大的改进空间，第二版可以参考有赞的架构。

## 容错

二期考虑人工干预功能，可以实现topic迁移。避免节点异常。

# 总体设计

## 架构

![](http://media.tumblr.com/edb403d38fc2bcc727b8655ea70eb3a7/tumblr_inline_mf8sfr2sp41qj3yp2.png)

## 流程

1. 心跳汇报
2. 创建topic
3. 删除topic
4. 逻辑删除topic
5. 读取消息

## 部署

见架构图

# 详细设计

## 数据库设计

## 关键数据结构

## 关键算法
