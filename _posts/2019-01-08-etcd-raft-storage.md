---
layout: post
title: etcd-raft-storage
thread: 1
categories: etcd
tags: 技术文章
---

#### 日志压缩

Raft 的日志正常操作中不断增长，但实际系统中，日志不能无限增长，随着日志不断增长,他会占用越来越多的空间，如果没有清楚陈旧日志的技术
则会带来性能问题

快照是最简单的压缩方法，在整个系统中状态读可以快照的新式写入到稳定的持久存储中,然后那个时间点之前的日志全部丢弃。

增量压缩方法


领导人发送快照给跟随者，领导人总是安顺序发送的

|参数 |解释|
|---|:---|
|term| 领导人的任期号|
|leaderid|领导人的id，以便跟随着重定向请求|
|lastincludeindex| 快照中包含的最后日志的条目索引值|
|lastincludeterm|快照中包含最后日志条目的任期号|
|offset|分块在快照中的偏移量|
|date[]|原始数据|
|done|如果这是最后一个分快则为true|

接受者实现

1 如果term < currentterm 就立即回复

2 如果是第一个分快（offset为0）就创建一个新的快照

3 指定偏移量写入数据

4 日过done是false，则继续等待更多的数据

5 保存快照文件，丢弃索引值小于快照的日志

6 如果现存的日志拥有相同的最后的任期号和索引值，值后面的数据继续保持

7 丢弃整个日志

8 使用快照重制状态机


#### 主要是对以下接口的实现，这里面基本一些简单的业务逻辑

````go
type Storage interface {
	// InitialState returns the saved HardState and ConfState information.
	InitialState() (pb.HardState, pb.ConfState, error)
	// Entries returns a slice of log entries in the range [lo,hi).  //Entries 项
	// MaxSize limits the total size of the log entries returned, but
	// Entries returns at least one entry if any.
	Entries(lo, hi, maxSize uint64) ([]pb.Entry, error)
	// Term returns the term of entry i, which must be in the range
	// [FirstIndex()-1, LastIndex()]. The term of the entry before
	// FirstIndex is retained for matching purposes even though the
	// rest of that entry may not be available.
	Term(i uint64) (uint64, error)
	// LastIndex returns the index of the last entry in the log.
	LastIndex() (uint64, error)
	// FirstIndex returns the index of the first log entry that is
	// possibly available via Entries (older entries have been incorporated
	// into the latest Snapshot; if storage only contains the dummy entry the
	// first log entry is not available).
	FirstIndex() (uint64, error)
	// Snapshot returns the most recent snapshot.
	// If snapshot is temporarily unavailable, it should return ErrSnapshotTemporarilyUnavailable,
	// so raft state machine could know that Storage needs some time to prepare
	// snapshot and call Snapshot later.
	Snapshot() (pb.Snapshot, error)
}
````

