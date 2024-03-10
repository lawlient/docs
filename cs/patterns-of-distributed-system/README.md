# 分布式系统模式

> 记录分布式领域常见问题及解决方案。


## 主题


### 日志

- [ ] Write-Ahead Log: 预写日志，提升数据库性能，实现事务的原子性
- [ ] Replicated Log: 复制日志，保障集群一致性，节点间进行状态同步的手段
- [ ] Segmented Log : 分割日志，防止单日至文件过大影响读取性能

### 时钟

- [ ] Clock-Bound Wait: 时钟同步相关，待进一步了解。
- [ ] Hybrid Clock: 混合时钟
- [ ] Generation Clock：时钟生成，本质是单调递增，偏序特性
- [ ] Lamport Clock: 时钟的一种，本质还是希望获取单调值来计算偏序关系
- [ ] [HeartBeat](cs/patterns-of-distributed-system/heartbeat.md): 心跳，服务器保证自己活力的一种手段
- [ ] [Lease](cs/patterns-of-distributed-system/lease.md)： 租约。一种活性检测的手段，由于分布式节点可能出现故障，防止资源泄露，通过租约的方式进行资源回收
- [ ] State Watch： 状态观察，例如服务发现中服务的上下线，主动轮询浪费性能，通知方式更高效

### 分布式一致性

- [ ] Consistent Core: 一致性的核心
- [ ] Fixed Partitions: 有点分布式数据分片的感觉，一致性哈希算法之类的
- [ ] Key-Range Partitions： 键范围划分，本质是一种分片的手段，数据库中也有用到
- [ ] Gossip Dissemination: Gossip协议，保障AP性，以冗余消息的简单方式保证集群的最终一致性
- [ ] Paxos: 一种强一致性共识算法。
- [ ] Emergent Leader: 紧急情况领导者，不同于一致性共识算法的强领导者，此处主要是集群内需要协同的情况下选出的领导者，更加简单。
- [ ] Leader and Followers : 集群中节点角色，通常用于一致性算法中
- [ ] Majority Quorum: 多数法定人，一致性共识算法中领导者选举就是依赖多数法定人投票的结果，防止少数节点异常做出错误操作影响系统状态
- [ ] High-Water Mark: commitid 就是一种高水位线，一种保障
- [ ] Low-Water Mark : WAL中的一个分割线，表示此前的额数据可以安全丢弃

### rpc

- [ ] Request Batch: 批量请求，一种性能提升的手段
- [ ] Request Pipeline: 管道请求，提升网络请求的手段
- [ ] Request Waiting List： 共识算法要求多数法定人通过决策
- [ ] Single-Socket Channel ：单路通道，领导者和追随者通信依赖请求有效，通过tcp单链接保障
- [ ] Singular Update Queue ：队列的作用主要是异步解耦
- [ ] Follower Reads: 强一致性和最终一致性，追随者开启读提升集群性能，但放弃了强一致性
- [ ] Idempotent Receiver: 幂等请求，通常依赖唯一id进行去重

### 事务

- [ ] Two-Phase Commit：分布式事务
- [ ] Version Vector：版本向量，检测值变化的手段
- [ ] Versioned Value：带版本值，可追踪历史值


