# Raft

阅读本文你将了解以下内容：

1. raft是干什么
2. etcd是如何实现raft算法的




## raft简介



## ETCD中raft实现


### 角色

```go
const (
    StateFollower StateType = iota
    StateCandidate
    StateLeader
    StatePreCandidate
    numStates
)

type StateType uint64

var stmap = [...]string{
    "StateFollower",
    "StateCandidate",
    "StateLeader",
    "StatePreCandidate",
}

func (st StateType) String() string {
    return stmap[st]
}
```




### 节点



|     方法                                                              |       功能        |
| :-                                                                    |   :-              |
| Tick()                                                                | 递增一个逻辑时钟单元，选举和心跳都是基于这个时钟  |
| Campaign(ctx context.Context) error                                   | 触发节点角色转变为**候选人**事件                  |
| Propose(ctx context.Context, data []byte) error                       | 提议将`data`数据写入日志，可能会出现数据丢失      |
| ProposeConfChange(ctx context.Context, cc pb.ConfChangeI) error       | 提议配置变更，同样可能数据丢失或错误响应          |
| Step(ctx context.Context, msg pb.Message) error                       | 应用消息实习状态机转移                            |
| Ready() <-chan Ready                                                  | 返回一个用于获取节点实时状态的频道                |
| Advance()                                                             |   |
| ApplyConfChange(cc pb.ConfChangeI) \*pb.ConfState                     | 对节点应用一个配置变更                            |
| TransferLeadership(ctx context.Context, lead, transferee uint64)      | 尝试将领导人身份转移至`transferee`指定的节点      |
| ForgetLeader(ctx context.Context) error                               | 清除追随者节点中的领导人信息                      |
| ReadIndex(ctx context.Context, rctx []byte) error                     |   |
| Status() Status                                                       | 返回节点raft状态机当前的状态                      |
| ReportUnreachable(id uint64)                                          | 报告`id`节点上次消息发送时不可达                  |
| ReportSnapshot(id uint64, status SnapshotStatus)                      | 报告发送快照的状态，`id`是追随者的ID，状态是成功或失败    |
| Stop()                                                                | 处理节点停止                                      |

### 消息

用于事件的触发和响应。

#### 消息分类

| 名称                 | 域 | 说明                          |
| :-                   | :- | :-                            |
| MsgHup               | 0  | 用于选举，追随者或候选人选举心跳超时时通过MsgHup进入或保持候选人状态，`tick`函数指针指向`tickElection` |
| MsgBeat              | 1  | L内部消息，通知L发送`MsgHeartbeat`心跳消息给追随者，`tick`函数指针指向`tickHeartbeat`    |
| MsgProp              | 2  | 日志复制提议消息，走propc频道。L收到消息时，现追加日志到本地，再广播日志给其他节点；C收到消息直接丢弃；F收到消息先存于msgs数组中，稍后转发至L          |
| MsgApp               | 3  | 日志赋值消息，领导者调用`bcastAppend（sendAppend）`发送该类型消息。候选者收到消息后转变为追随者。候选者或追随者收到消息后会响应`MsgAppResp`类型消息。  |
| MsgAppResp           | 4  | 候选者或追随者收到`MsgApp`消息后调用`handleAppendEntries`方法回复`MsgAppResp`消息
| MsgVote              | 5  | 选票发起消息。候选者或追随者处理`MsgHup`消息时调用`campaign`方法使自身转变为候选者，并发送`MsgVote`消息给集群中其他节点给自己投票。<br/>1. 选票任期小于本地任期，拒绝投票(MsgVoteResp消息中Reject为真)<br/>2. 选票任期大于本地任期则转变为追随者 ...... |
| MsgVoteResp          | 6  | 选票响应消息。其他节点给候选者的选票结果。候选者统计投票结果：<br/>1. 获过半选票，转变为L，并调用bcastAppend方法<br/>2. 获过半否决票，重现转变为F |
| MsgSnap              | 7  | 快照同步请求。节点刚刚成为L或收到`MysgProp`消息，调用`bcastAppend`方法触发`sendAppend`，当无法获取任期或日志序列时，发送`MsgSnap`进行快照安装               |
| MsgSnapStatus        | 11 | 反馈快照安装结果。L会根据响应更新Progress中F的状态    |
| MsgHeartbeat         | 8  | L发送的心跳消息。<br/>1. C收到任期大的心跳时，重置为F状态，并设置心跳中的commit index到本地；<br/>2. F收到任期大的心跳，更新本地leaderID为心跳消息中的 |
| MsgHeartbeatResp     | 9  | 心跳消息的响应。L根据索引值判断是否触发sendAppend  |
| MsgUnreachable       | 10 |
| MsgCheckQuorum       | 12 | 
| MsgTransferLeader    | 13 |
| MsgTimeoutNow        | 14 |
| MsgReadIndex         | 15 |
| MsgReadIndexResp     | 16 |
| MsgPreVote           | 17 | 选举优化，预投票发起请求。`Config.PreVote`开启时，会使用2阶段选举，先通过该消息进入预候选人 |
| MsgPreVoteResp       | 18 | 预投票的响应消息。    |
| MsgStorageAppend     | 19 | 
| MsgStorageAppendResp | 20 |
| MsgStorageApply      | 21 |
| MsgStorageApplyResp  | 22 |
| MsgForgetLeader      | 23 |

#### 消息格式

```protobuf
message Message {
    optional MessageType type        = 1  [(gogoproto.nullable) = false];
    optional uint64      to          = 2  [(gogoproto.nullable) = false];
    optional uint64      from        = 3  [(gogoproto.nullable) = false];
    optional uint64      term        = 4  [(gogoproto.nullable) = false];
    optional uint64      logTerm     = 5  [(gogoproto.nullable) = false];
    optional uint64      index       = 6  [(gogoproto.nullable) = false];
    repeated Entry       entries     = 7  [(gogoproto.nullable) = false];
    optional uint64      commit      = 8  [(gogoproto.nullable) = false];
    optional uint64      vote        = 13 [(gogoproto.nullable) = false];
    optional Snapshot    snapshot    = 9  [(gogoproto.nullable) = true];
    optional bool        reject      = 10 [(gogoproto.nullable) = false];
    optional uint64      rejectHint  = 11 [(gogoproto.nullable) = false];
    optional bytes       context     = 12 [(gogoproto.nullable) = true];
    repeated Message     responses   = 14 [(gogoproto.nullable) = false];
}
```


### 选举


### 日志格式

```protobuf
enum EntryType {
    EntryNormal       = 0;
    EntryConfChange   = 1; // corresponds to pb.ConfChange
    EntryConfChangeV2 = 2; // corresponds to pb.ConfChangeV2
}

message Entry {
    optional uint64     Term  = 2 [(gogoproto.nullable) = false]; // must be 64-bit aligned for atomic operations
    optional uint64     Index = 3 [(gogoproto.nullable) = false]; // must be 64-bit aligned for atomic operations
    optional EntryType  Type  = 1 [(gogoproto.nullable) = false];
    optional bytes      Data  = 4;
}
```

### 日志复制

```protobuf
enum EntryType {
    EntryNormal       = 0;
    EntryConfChange   = 1; // corresponds to pb.ConfChange
    EntryConfChangeV2 = 2; // corresponds to pb.ConfChangeV2
}

message Entry {
    optional uint64     Term  = 2 [(gogoproto.nullable) = false]; // must be 64-bit aligned for atomic operations
    optional uint64     Index = 3 [(gogoproto.nullable) = false]; // must be 64-bit aligned for atomic operations
    optional EntryType  Type  = 1 [(gogoproto.nullable) = false];
    optional bytes      Data  = 4;
}
```



### 成员变更























-----


# 参考文献 


- [etcd raft模块解析](https://zhuanlan.zhihu.com/p/676995678)
