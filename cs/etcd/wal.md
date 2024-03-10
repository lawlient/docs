# WAL

阅读本文你将了解：

- wal文件如何命名
- wal文件内部结构
- 日志记录格式
- 如何通过crc32进行日志完整性校验


## WAL文件命名规则

```golang
func walName(seq, index uint64) string {
    return fmt.Sprintf("%016x-%016x.wal", seq, index)
}
```

示例：`0000000000000000-0000000000000000.wal`.

前半部分是**递增的序号**，后半部分是文件**起始日志的索引值**。


## WAL 文件内容部结构




图中不同颜色填充部分称之为：Frame（帧），整个wal文件由连续的帧组成。


### 帧


![帧结构示意图]()


帧数据由2部分组成：帧头 + 帧体。


### 帧头

帧头占8字节，按小端序方式存储。

- 高1字节。高5bits不用，低3bits存储帧体尾部用于对齐的补齐bit数(0 ~ 7)。
- 低7字节。存储帧体部分总字节数。帧体最大长度（65535 TB）。


### 帧体

帧体由2部分组成：


- 数据域。日志内容CRC校验值(细节下面日志部分解析)。
- 对齐域(0 ~ 7bits)。为了解析日志时区分是否数据撕裂，wal存储时对每一帧数据进行了对齐（保证长度是8的整数倍）.



## 日志结构

日志定义如下:
```protobuf
message Record {
    optional int64 type  = 1 [(gogoproto.nullable) = false];
    optional uint32 crc  = 2 [(gogoproto.nullable) = false];
    optional bytes data  = 3;
}
```


### 日志类型

日志第一个域存储类型数字，类型定义如下：

```golang
const (
    MetadataType int64 = iota + 1
    EntryType
    StateType
    CrcType
    SnapshotType
)
```

共5种类型:

1. 元数据
2. raft 日志
3. raft 状态
4. crc数据
5. 快照数据


### CRC校验值


日志第二个域存储crc校验值。对整个record进行crc32计算所得值。

校验值计算主要由encoder对象完成，重点看下encoder的构造函数：

```golang
func newEncoder(w io.Writer, prevCrc uint32, pageOffset int) *encoder {
    return &encoder{
        bw:  ioutil.NewPageWriter(w, walPageBytes, pageOffset),
        crc: crc.New(prevCrc, crcTable),
        // 1MB buffer
        buf:       make([]byte, 1024*1024),
        uint64buf: make([]byte, 8),
    }
}
```

注意构造crc对象时第一个参数：`prevCrc`，即每条日志的crc计算是基于前一条日志的crc值。

整个文件的日志流通过crc形成了一条链表，解析时可以校验整个文件数据的完整性。


### 日志内容

日志第三个域存储日志核心内容，其实就是变长字节数组。


## 文件头

创建新日志文件时，文件头会先写入特定帧数据。


1. crc帧。此时prevCrc = 0。
2. 元数据帧。
3. 快照帧。快照字段均为默认值。
