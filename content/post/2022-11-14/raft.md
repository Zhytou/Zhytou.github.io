---
title: "Raft 笔记"
date: 2022-11-14T10:38:10+08:00
draft: true
author: Zhytou
---

> 简单记录一下学习和实现Raft共识算法（6.824）的笔记。

`Raft`是一种易于理解的共识算法。它通过保证`复制状态机Replicated State Machine`中的日志一致来保证一个分布式系统可靠、复制、冗余和容错。

## Basics

在`Raft`算法中，每一个服务器都是一个复制状态机。每个服务器存储着一份包含命令序列的日志文件，状态机会按顺序执行这些命令。而保证这些日志一致是`Raft`算法的职责。

一个`Raft`集群包括若干服务器。服务器一定会处于以下三种状态中的一种：

+ `跟随者/Follower`：跟随者是被动的，它们不会发送任何请求，只是响应来自领导者和参选者的请求。

+ `参选者/Candidate`：参选者是参加选举领导者的一台服务器。它是由在一段时间内没有收到任何消息的跟随者转变成的。

+ `领导者/Leader`：领导者处理所有来自客户端的请求。领导者在它们宕机之前会一直保持领导者的状态。

时间被分为一个个任意不同长度的`任期Term`，每一个任期的开始都是领导者选举。任期用连续的数字进行表示。

基本的`Raft`一致性算法仅需要 2 种`RPC`。

+ `RequestVote RPC`是参选者在选举过程中触发的。

+ `AppendEntries RPC`是领导者触发的，为的是复制日志条目和提供一种`心跳（Heartbeat）机制`。

此外，`Raft`算法中主要涉及到两种超时：

+ `选举超时 Election Timeout`：每个服务器（跟随者）都有长短不一的`选举超时 Election Timeout`。一旦超过这个时间没有接受到来自领导者的消息，那么该跟随者会转化成参选者，进行一次选举。

+ `心跳间隔（超时） Heartbeat Timeout`：领导者每间隔固定时间向集群中每个服务器发送消息，维护自己领导者地位，避免开启一次新选举。

对于`Raft`服务器中`日志 Log Entry`的状态，主要分成两种：

+ `已提交 Committed`：当领导者接收到超过半数服务器复制该日志成功，则会将这条日志记录设置为`已提交 Committed`状态。

+ `已执行 Applied`：当跟随者更新`已提交 Committed`日志后，跟随者会将日志应用到状态机上执行，此时将这条日志设置为`已执行 applied`状态。

`Raft`服务器中`日志 Log Entry`至少会记录该日志条目的发生`任期 Term`、在整个日志记录中的`序号 Index`（从1开始）和状态机需要执行的`命令 Command`。

当且仅当两个服务器内，所有日志条目的`任期 Term`和`序号 Index`均相等时，称两个服务器日志匹配。

## Leader election

### Basic leader election procedure

每当一个`Raft`集群初始化（所有服务器被初始化为跟随者）或该集群中领导者宕机或网络阻塞时，会开始一次新领导者选举：

首先，先到达`选举超时 Election Timeout`的跟随者转化为参选者，并将自己当前的任期加一，并向整个集群所有服务器发送`RequestVote RPC`，请求选票。

接着，接受到`RequestVote RPC`的服务器，根据任期和日志匹配以及是否还有选票确认是否投票给参选者。具体来说，做以下判断：

+ 如果接受到投票请求的服务器任期大于参选者任期，拒绝投票请求，返回false；如果接受到投票请求的服务器任期小于或等于参选者任期，则作进一步判断（注意：如果小于，则还要更新自己的任期并将选票状态刷新为有选票，因为选票和任期同步）；

+ 如果日志不匹配参选者日志（参选者日志更旧），拒绝投票请求，返回false；反之，作进一步判断；

+ 如果已经投出票，拒绝投票请求，返回false；反之，投票，重置选举超时计时器，并更新已无选票，返回true。

然后，参选者此时根据`RequestVote RPC`的回复，更新自己的状态。具体来说，做以下判断：

+ 如果接受到任期大于自己任期的回复，退出选举、更新自己的任期并重置自己的选举超时计时器。

+ 统计有效选票数量，如果数量超过集群服务器总数一半，则选举成果；否则，重置自己的选举超时计时器（重新随机生成一个选举超时时间，避免一直`分裂选票 split vote`）并更新自己的任期，进入新一轮选举。

经过一轮或多轮选举后，成功得到一个领导者，领导者向集群中其余服务器发送`AppendEntries RPC`请求，确认自己领导地位，保证所有参选者退出选举、跟随者重置选举超时计时器。

除此之外，应当注意的是，在整个选举过程中的任何时间点，如果一个参选者收到大于自身任期的`RequestVote RPC`请求回复或有效领导者的`AppendEntries RPC`请求，都将重置选举超时计时器，并退出选举。

### Corner case

整个选举流程中可能出现的边界情况，包括：

`分裂选票 split vote`：多个参选者同时发送`RequestVote RPC`请求，导致始终没有一个参选者能获得超过半数的选票，赢得选举。

`Raft`算法通过随机选举超时时间来解决`分裂选票 split vote`的问题。

`领导者掉线，形成多领导者`：

`跟随者掉线，重新加入集群`：

## Log replication

### Basic log replication procedure

首先，客户端向`Raft`集群的领导者发送需要执行的日志条目。领导者接受到该请求后，会将该条目作为参数向其他服务器发送`AppendEntries RPC`请求。

接着，接受到`AppendEntries RPC`请求的服务器会根据请求参数，确认当前是否领导者日志匹配。具体做以下判断：

+ 如果prevLogIndex大于当前服务器日志长度，则返回false，将conflictIndex设置为当前服务器日志长度，并请求领导者重新发送一次`AppendEntries RPC`。

+ 如果prevLogIndex小于等于当前服务器日志长度且prevLogTerm不等于同位置日志条目的任期，则返回fasle，将conflictIndex设置为当前服务器日志上一任期的最后一条日志序号，并要求领导者重新发送一次`AppendEntries RPC`请求。

+ 如果prevLogIndex小于等于当前服务器日志长度且prevLogTerm等于同位置日志条目的任期，则二者日志匹配。删除后续所有日志条目，并在尾部增添新日志条目。最后，异步将新复制的且领导者已提交的日志条目在状态机上执行。

然后，领导者会根据回复，更新自身状态。具体如下：

+ 若返回复制成功（匹配肯定也成功），更新匹配位置；若返回复制失败（匹配也失败），则根据conflictIndex和conflictTerm返回上一个任期的最后一条日志判断。

+ 统计成功复制某条目日志的服务器数量，若超过半数服务器成功复制某日志条目，则将其设置为`已提交 Committed`，并异步将其在状态机上执行。

## Optimization

`批处理复制请求 Batch and Pipeline`

`异步执行 Asynchronous Apply`

## Reference

+ [Raft 动画演示](http://www.kailing.pub/raft/index.html)

+ [Raft Gitbook](https://knowledge-sharing.gitbooks.io/raft/content/)

+ [Extended Raft](https://pdos.csail.mit.edu/6.824/papers/raft-extended.pdf)

+ [Raft 优化](https://cn.pingcap.com/blog/optimizing-raft-in-tikv)
