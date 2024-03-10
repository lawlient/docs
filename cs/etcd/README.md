# [ETCD](https://github.com/etcd-io/etcd)

> 本文记录学习etcd项目笔记



## 系统架构

<img src="https://cdnjson.com/images/2024/03/16/etcd-clusterfb2345ead2cb34f1.md.webp" alt="etcd-cluster" border="1" />

上图展现了etcd集群整体的工作方式。

<img src="https://cdnjson.com/images/2024/03/16/etcd-node7fada74efc838737.jpg" alt="etcd-node7fada74efc838737.jpg" border="0" width="50%" height='auto' />

上图展示了etcd节点内部的核心模块以及模块之间的交互关系。 




## 组件

- [wal](cs/etcd/wal.md)
- [raft](cs/etcd/raft.md)




