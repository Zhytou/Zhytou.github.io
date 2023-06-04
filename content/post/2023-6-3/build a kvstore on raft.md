---
title: "基于Raft搭建一个简单KV存储服务"
date: 2023-06-03T19:15:28+08:00
draft: false
author: Zhytou
---


虽然一结束实习就想着要把Raft捡起来做完，但还是拖拖拉拉地磨洋工。最近才勉强过了lab3的测试，早知道当时做完lab2就应该一鼓作气把6.824的4个lab全部做完的。

lab3要求我们在Raft基础上实现一个高可用的KV存储服务，包括对客户端和服务端的实现。而lab3又根据是否支持快照，将其分成A和B两部分。其中，lab3-B需要使用lab2-D留下的接口CondInstallSnapshot。需要注意是，即使lab2成功通过了，也应该注意该函数是否只是简单return true。如果是的话，可能就需要重新理解并实现InstallSnapshot和CondInstallSnapshot。我就是因为这一点卡了很久。

## lab3-A

### 客户端

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

### 服务端

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

## 参考

[lab2-D CondInstallSnapshot的理解](https://www.cnblogs.com/sun-lingyu/p/14591757.html)
