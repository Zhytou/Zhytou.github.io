---
title: "基于Raft搭建一个简单KV存储服务"
date: 2023-06-03T19:15:28+08:00
draft: false
author: Zhytou
---


虽然一结束实习就想着要把Raft捡起来做完，但还是拖拖拉拉地磨洋工。最近才勉强过了lab3的测试，早知道当时做完lab2就应该一鼓作气把6.824的4个lab全部做完的。

lab3要求我们在Raft基础上实现一个高可用的KV存储服务，包括对客户端和服务端的实现。而lab3又根据是否支持快照，将其分成A和B两部分。其中，lab3-B需要使用lab2-D留下的接口CondInstallSnapshot。需要注意是，即使lab2成功通过了，也应该注意该函数是否只是简单return true。如果是的话，可能就需要重新理解并实现InstallSnapshot和CondInstallSnapshot。我就是因为这一点卡了很久。

## lab3-A

### Client

仔细读完实验说明文档，客户端需要记录服务端领导者ID、自身ID和请求ID，并且在发送请求时，自增请求ID。

值得注意的是，为了代码可读性和复用性，我的服务端实际上只提供了一种API，即：Command。原本的Get、Put以及Append实际上都是调用该接口来完成功能的。

``` golang
type Clerk struct {
 servers []*labrpc.ClientEnd
 leaderId  int
 clientId  int64
 commandId int64
}

func (ck *Clerk) Get(key string) string {
 return ck.Command(&CommandArgs{Key: key, OpType: "Get"})
}

func (ck *Clerk) PutAppend(key string, value string, op string) {
 ck.Command(&CommandArgs{Key: key, Value: value, OpType: op})
}

func (ck *Clerk) Put(key string, value string) {
 ck.PutAppend(key, value, "Put")
}

func (ck *Clerk) Append(key string, value string) {
 ck.PutAppend(key, value, "Append")
}

func (ck *Clerk) Command(args *CommandArgs) string {
 args.ClientId, args.CommandId = ck.clientId, ck.commandId
 for {
  reply := CommandReply{}
  if ok := ck.servers[ck.leaderId].Call("KVServer.Command", args, &reply); !ok || reply.Err == ErrWrongLeader || reply.Err == ErrTimeout {
   ck.leaderId = (ck.leaderId + 1) % len(ck.servers)
   continue
  }
  ck.commandId++
  return reply.Value
 }
}
```

### Server

根据实验要求，服务端需要存储键值对，并支持Get、Put以及Append三种接口。为了代码的可读性，我就将其封装在一个自定义的MemoryKV类型中了。

``` golang
type MemoryKV struct {
 KV map[string]string
}

func NewMemoryKV() *MemoryKV {
 return &MemoryKV{make(map[string]string)}
}

func (memoryKV *MemoryKV) Get(key string) (string, Err) {
 if value, ok := memoryKV.KV[key]; ok {
  return value, OK
 }
 return "", ErrNoKey
}

func (memoryKV *MemoryKV) Put(key, value string) Err {
 memoryKV.KV[key] = value
 return OK
}

func (memoryKV *MemoryKV) Append(key, value string) Err {
 memoryKV.KV[key] += value
 return OK
}
```

整个实验的难点主要集中在服务端，尤其需要注意线性一致性。而为了达成这一点，lab3要求服务端将所有请求，无论读写，都先交给Raft层达成一致之后，再将其执行在其本身的状态机上。

简单来说，服务端的工作流程是：

- 接收到来自客户端的请求，
  - 首先，调用Raft层接口判断当前节点是不是领导者。若不是领导者，则直接返回；反之，则继续。
  - 其次，根据请求ID和客户端ID判断是不是重复请求。若重复请求，则直接返回历史结果；反之，则继续。
- 创建消息队列（通道，用于获取Raft层执行结果），向Raft层发送请求，要求对当前请求达成一致。若Raft层达成一致超时，则返回超时；反之，处理Raft层返回结果，并回复客户端。（记得最后异步销毁消息队列）

``` golang
type KVServer struct {
 mu      sync.RWMutex
 me      int
 rf      *raft.Raft
 applyCh chan raft.ApplyMsg
 dead    int32 // set by Kill()

 maxraftstate int // snapshot if log grows this big
 persister    *raft.Persister

 lastApplied    int
 lastOperations map[int64]OperationContext // last operation for each client
 stateMachine   *MemoryKV                  // KV stateMachine
 notifyChans    map[int]chan *CommandReply // notify client goroutine by applier goroutine to response
}

func (kv *KVServer) Command(args *CommandArgs, reply *CommandReply) {
 kv.mu.RLock()
 if args.OpType != "Get" && kv.isDuplicateRequest(args.ClientId, args.CommandId) {
  lastReply := kv.lastOperations[args.ClientId].LastReply
  reply.Value, reply.Err = lastReply.Value, lastReply.Err
  kv.mu.RUnlock()
  return
 }
 kv.mu.RUnlock()
 // do not hold lock to improve throughput
 // when KVServer holds the lock to take snapshot, underlying raft can still commit raft logs
 index, _, isLeader := kv.rf.Start(Op{OpType: args.OpType, ClientId: args.ClientId, CommandId: args.CommandId, Key: args.Key, Value: args.Value})
 if !isLeader {
  reply.Err = ErrWrongLeader
  return
 }
 kv.mu.Lock()
 ch := kv.getNotifyChan(index)
 kv.mu.Unlock()
 select {
 case result := <-ch:
  reply.Value, reply.Err = result.Value, result.Err
 case <-time.After(ExecuteTimeout):
  reply.Err = ErrTimeout
 }
 // release notifyChan to reduce memory footprint
 // why asynchronously? to improve throughput, here is no need to block client request
 go func() {
  kv.mu.Lock()
  kv.removeOutdatedNotifyChan(index)
  kv.mu.Unlock()
 }()
}
```

此外，为了保持服务端代码结构清晰，服务端往往单独起一个守护线程（applier）处理异步返回的Raft层结果。在不考虑Snapshot的情况下，applier的工作逻辑：

``` golang
func (kv *KVServer) applier() {
 for !kv.killed() {
  message := <-kv.applyCh
  DPrintf("{Node %v} tries to apply message %v", kv.rf.Me(), message)
  if message.CommandValid {
   kv.mu.Lock()
   if message.CommandIndex <= kv.lastApplied {
    DPrintf("{Node %v} discards outdated message %v because lastApplied is %v", kv.rf.Me(), message, kv.lastApplied)
    kv.mu.Unlock()
    continue
   }

   var cmd Op = message.Command.(Op)
   kv.lastApplied = message.CommandIndex
   var reply *CommandReply
   // no need to apply to stateMachine if request is duplicated
   if cmd.OpType != "Get" && kv.isDuplicateRequest(cmd.ClientId, cmd.CommandId) {
    DPrintf("{Node %v} doesn't apply duplicated message %v to stateMachine because maxAppliedCommandId is %v for client %v", kv.rf.Me(), message, kv.lastOperations[cmd.ClientId], cmd.ClientId)
    reply = kv.lastOperations[cmd.ClientId].LastReply
   } else {
    // apply log to stateMachine
    reply = &CommandReply{}
    switch cmd.OpType {
    case "Get":
     {
      reply.Value, reply.Err = kv.stateMachine.Get(cmd.Key)
     }
    case "Put":
     {
      reply.Err = kv.stateMachine.Put(cmd.Key, cmd.Value)
     }
    case "Append":
     {
      reply.Err = kv.stateMachine.Append(cmd.Key, cmd.Value)
     }
    }
    // add to lastOperations
    if cmd.OpType != "Get" {
     kv.lastOperations[cmd.ClientId] = OperationContext{CommandId: cmd.CommandId, LastReply: reply}
    }
   }

   // only notify related channel for currentTerm's log when node is leader
   if currentTerm, isLeader := kv.rf.GetState(); isLeader && message.CommandTerm == currentTerm {
    ch := kv.getNotifyChan(message.CommandIndex)
    ch <- reply
   }

   kv.mu.Unlock()
  } else {
   panic(fmt.Sprintf("unexpected Message %v", message))
  }
 }

}
```

## lab3-B

### Comprehend lab2-D

lab2-D为服务层提供了两个方法，分别是：

- `func (rf *Raft) Snapshot(index int, snapshot []byte)`
- `func (rf *Raft) CondInstallSnapshot(lastIncludedTerm int, lastIncludedIndex int, snapshot []byte) bool`。

其中，服务可以通过调用前者使Raft记录当前状态并持久化当前服务器状态（快照），而调用后者则是尝试使Raft状态机接收该快照。此外，lab2-D还实现了另一个方法`func (rf *Raft) InstallSnapshot(args *InstallSnapshotArgs, reply *InstallSnapshotReply)`。

该方法与`CondInstallSnapshot`联系紧密，它的工作流程如下：

- Raft领导者发现有服务器日志落后过多，尝试调用`InstallSnapshot`的RPC，要求它下载领导者的快照；
- 该服务器到接受`InstallSnapshot`的请求后进行处理判断，若快照包含日志序号比该服务器的提交序号小，说明快照过期，拒绝接受快照；反之，可以接受快照，向服务层发送ApplyMsg；
- 服务层applier线程接受到该消息后，调用Raft层`CondInstallSnapshot`方法，若返回为真且执行序号小于快照包含日志序号，则服务层接受快照并更新服务器状态机；反之，拒绝快照。

也就说，`InstallSnapshot`并没实际更新Raft状态机，而是发送消息通知服务层，直到服务层调用`CondInstallSnapshot`才尝试更新Raft状态机，并在更新成功后返回true。至于为什么不能像实现lab2时，`InstallSnapshot`在满足条件就直接更新Raft状态机，而`CondInstallSnapshot`直接返回true去实现？

这是因为将实际安装快照放在`CondInstallSnapshot`中可以让Service层去主动调用，进而在避免死锁的情况下确保Service层与Raft层安装快照的原子性，具体可以参考[博客](https://www.cnblogs.com/sun-lingyu/p/14591757.html)。

``` golang
func (rf *Raft) Snapshot(index int, snapshot []byte) {
 rf.mu.Lock()
 // rf.log[0] is the place where snapshot info is kept
 snapshotIndex := rf.getFirstLogIndex()
 if index <= snapshotIndex {
  DPrintf("Log Compaction: Server %d has already compacted log entries before %d", rf.me, index)
  return
 }
 DPrintf("Log Compaction: Server %d compacted log entries (%d , %d] successfully", rf.me, rf.getFirstLogIndex(), index)
 rf.log = append([]LogEntry{}, rf.log[index-snapshotIndex:]...)
 rf.log[0].Command = nil
 rf.mu.Unlock()
 rf.persister.SaveStateAndSnapshot(rf.encodeState(), snapshot)
}


func (rf *Raft) CondInstallSnapshot(lastIncludedTerm int, lastIncludedIndex int, snapshot []byte) bool {
 rf.mu.Lock()
 defer rf.mu.Unlock()
 DPrintf("{Node %v} service calls CondInstallSnapshot with lastIncludedTerm %v and lastIncludedIndex %v to check whether snapshot is still valid in term %v", rf.me, lastIncludedTerm, lastIncludedIndex, rf.currentTerm)

 // outdated snapshot
 if lastIncludedIndex <= rf.commitIndex {
  DPrintf("{Node %v} rejects the outdated snapshot because lastIncludedIndex = %v,commitIndex = %v, firstLogIndex = %v", rf.me, lastIncludedIndex, rf.commitIndex, rf.getFirstLogIndex())
  return false
 }

 if lastIncludedIndex > rf.getLastLogIndex() {
  rf.log = make([]LogEntry, 1)
 } else {
  rf.log = rf.getSubLogFrom(lastIncludedIndex)
  rf.log[0].Command = nil
 }
 // update dummy entry with lastIncludedTerm and lastIncludedIndex
 rf.log[0].Term, rf.log[0].Index = lastIncludedTerm, lastIncludedIndex
 rf.lastApplied, rf.commitIndex = lastIncludedIndex, lastIncludedIndex
 rf.persister.SaveStateAndSnapshot(rf.encodeState(), snapshot)

 DPrintf("{Node %v}'s state is {state %v,term %v,commitIndex %v,lastApplied %v,firstLog %v,lastLog %v} after accepting the snapshot which lastIncludedTerm is %v, lastIncludedIndex is %v", rf.me, rf.state, rf.currentTerm, rf.commitIndex, rf.lastApplied, rf.getFirstLogIndex(), rf.getLastLogIndex(), lastIncludedTerm, lastIncludedIndex)
 return true
}

func (rf *Raft) InstallSnapshot(args *InstallSnapshotArgs, reply *InstallSnapshotReply) {
 rf.mu.Lock()
 defer rf.mu.Unlock()
 reply.Term = rf.currentTerm

 if args.Term < rf.currentTerm {
  return
 }

 if args.Term > rf.currentTerm {
  rf.currentTerm, rf.votedFor = args.Term, -1
  rf.persist()
 }

 rf.state = FOLLOWER
 rf.electionTimer.Reset(randomElectionTimeout())

 // outdated snapshots
 if args.LastIncludedIndex <= rf.commitIndex {
  return
 }

 // asynchronously send info to clients(service layer)
 go func() {
  rf.applyCh <- ApplyMsg{
   CommandValid:  false,
   SnapshotValid: true,
   Snapshot:      args.Data,
   SnapshotIndex: args.LastIncludedIndex,
   SnapshotTerm:  args.LastIncludedTerm,
  }
 }()

}
```

### Supprt Snapshot in Service Layer

lab3-B要求在part A的基础上支持快照功能，即：拍快照和读取快照。

根据实验说明，我们可以得到拍快照的时机，`Whenever your key/value server detects that the Raft state size is approaching this threshold, it should save a snapshot by calling Raft's Snapshot. If maxraftstate is -1, you do not have to snapshot.`。

至于读取快照，主要发生在两种情况下：

- 服务器重启初始化时。
- 作为跟随者的服务器，因网络或其他问题日志落后过多，接受领导者快照。

因此我们参考lab2-C持久化代码完成`readSnapshot`和`takeSnapshot`两个函数。具体实现如下：

``` golang
// take a snapshot(log compaction) when current size of persisent Raft state in bytes bigger than maxraftstate
func (kv *KVServer) takeSnapshot(commandIndex int) {
 w := new(bytes.Buffer)
 e := labgob.NewEncoder(w)
 e.Encode(kv.lastOperations)
 e.Encode(*(kv.stateMachine))
 e.Encode(kv.lastApplied)
 data := w.Bytes()
 kv.rf.Snapshot(commandIndex, data)
}

// read a snapshot and restore stateMachine
func (kv *KVServer) readSnapshot(snapshot []byte) {
 if snapshot == nil || len(snapshot) < 1 {
  return
 }
 r := bytes.NewBuffer(snapshot)
 d := labgob.NewDecoder(r)
 var lastOperations map[int64]OperationContext
 var stateMachine MemoryKV
 var lastApplied int
 if d.Decode(&lastOperations) != nil || d.Decode(&stateMachine) != nil || d.Decode(&lastApplied) != nil {
  DPrintf("error to read the snapshot data")
 } else {
  kv.lastOperations = lastOperations
  kv.stateMachine = &stateMachine
  kv.lastApplied = lastApplied
 }
}
```

至于KVServer初始化和applier调用上述函数的修改如下：

``` golang
func StartKVServer(servers []*labrpc.ClientEnd, me int, persister *raft.Persister, maxraftstate int) *KVServer {
 // call labgob.Register on structures you want
 // Go's RPC library to marshall/unmarshall.
 labgob.Register(Op{})

 kv := new(KVServer)
 kv.me = me
 kv.maxraftstate = maxraftstate

 kv.dead = 0
 kv.lastApplied = 0
 kv.lastOperations = make(map[int64]OperationContext)
 kv.notifyChans = make(map[int]chan *CommandReply)
 kv.stateMachine = NewMemoryKV()

 kv.persister = persister
 kv.readSnapshot(kv.persister.ReadSnapshot())

 kv.applyCh = make(chan raft.ApplyMsg)
 kv.rf = raft.Make(servers, me, persister, kv.applyCh)

 go kv.applier()
 return kv
}

func (kv *KVServer) applier() {
 for !kv.killed() {
  message := <-kv.applyCh
  // DPrintf("{Node %v} tries to apply message %v", kv.rf.Me(), message)
  if message.CommandValid {
   kv.mu.Lock()
   if message.CommandIndex <= kv.lastApplied {
    DPrintf("{Node %v} discards outdated message %v because lastApplied is %v", kv.rf.Me(), message, kv.lastApplied)
    kv.mu.Unlock()
    continue
   }

   var cmd Op = message.Command.(Op)
   kv.lastApplied = message.CommandIndex
   var reply *CommandReply
   // no need to apply to stateMachine if request is duplicated
   if cmd.OpType != "Get" && kv.isDuplicateRequest(cmd.ClientId, cmd.CommandId) {
    DPrintf("{Node %v} doesn't apply duplicated message %v to stateMachine because maxAppliedCommandId is %v for client %v", kv.rf.Me(), message, kv.lastOperations[cmd.ClientId], cmd.ClientId)
    reply = kv.lastOperations[cmd.ClientId].LastReply
   } else {
    // apply log to stateMachine
    reply = &CommandReply{}
    switch cmd.OpType {
    case "Get":
     {
      reply.Value, reply.Err = kv.stateMachine.Get(cmd.Key)
     }
    case "Put":
     {
      reply.Err = kv.stateMachine.Put(cmd.Key, cmd.Value)
     }
    case "Append":
     {
      reply.Err = kv.stateMachine.Append(cmd.Key, cmd.Value)
     }
    }
    // add to lastOperations
    if cmd.OpType != "Get" {
     kv.lastOperations[cmd.ClientId] = OperationContext{CommandId: cmd.CommandId, LastReply: reply}
    }
   }

   // only notify related channel for currentTerm's log when node is leader
   if currentTerm, isLeader := kv.rf.GetState(); isLeader && message.CommandTerm == currentTerm {
    ch := kv.getNotifyChan(message.CommandIndex)
    ch <- reply
   }

   // need to take a snapshot
   if kv.maxraftstate != -1 && kv.persister.RaftStateSize() > kv.maxraftstate {
    DPrintf("{Node %v} tries to take snapshot for message %v, kv.stateMachine = %v", kv.rf.Me(), message, kv.stateMachine.KV)
    kv.takeSnapshot(message.CommandIndex)
   }

   kv.mu.Unlock()
  } else if message.SnapshotValid {
   kv.mu.Lock()
   if kv.rf.CondInstallSnapshot(message.SnapshotTerm, message.SnapshotIndex, message.Snapshot) && message.SnapshotIndex > kv.lastApplied {
    DPrintf("{Node %v} tries to install snapshot for message %v", kv.rf.Me(), message)
    kv.readSnapshot(message.Snapshot)
    kv.lastApplied = message.SnapshotIndex
   }
   kv.mu.Unlock()
  } else {
   panic(fmt.Sprintf("unexpected Message %v", message))
  }
 }
}
```

## 参考

[lab2-D CondInstallSnapshot的理解](https://www.cnblogs.com/sun-lingyu/p/14591757.html)
