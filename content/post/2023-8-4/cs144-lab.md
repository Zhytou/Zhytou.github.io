---
title: "CS144-Minnow Lab总结"
date: 2023-08-04T22:18:06+08:00
draft: false
---

> 课程地址[CS144: Introduction to Computer Networking](https://cs144.github.io/)

[![wakatime](https://wakatime.com/badge/user/a7b329b7-d489-40d2-9239-8be7cf83b65e/project/b892df03-90fb-40ca-a3a3-f14a57f9a941.svg)](https://wakatime.com/badge/user/a7b329b7-d489-40d2-9239-8be7cf83b65e/project/b892df03-90fb-40ca-a3a3-f14a57f9a941)

个人感觉，虽然整个CS144的Lab难度不算特别大（认认真真做一周时间应该是够了），但还是非常有收获的。反正我做完就感觉死去的计网记忆在攻击我:(。难度排序的话，应该是 TCP Sender > TCP receiver ≈Network Interface > IP router > Webget & ByteStream。

## 0 Networking Warmup

实验0的前几个部分主要以使用命令进行发Http请求或邮件并观察响应为主，只有最后两个部分需要写代码。下面介绍一下我的环境以及配置方法。

**WSL Install**：

首先，在确保虚拟化以及WSL功能正确开启后，前往微软商店下载Windows Subsystem For Linux。我使用的发行版是Ubuntu 22.04.2 LTS。更详细的下载方法，可以查看我的另一篇博客[搭建自己的生产环境]中的WSL部分。

接着，在WSL下载cmake、make、git和clang-format。其中，cmake和make是用于项目构建和编译的；git则是用于版本管理；clang-format则是一个格式化工具。它都会在后续使用到。具体下载方法如下：

```bash
sudo apt install cmake make git clang-format
```

**Remote Repository**：

首先，在github中创建一个名为`minnow`的空仓库，作为我们的远程备份仓库。注意，为了遵守课程规定，我们需要将该仓库设为私有。

其次，克隆课程项目，并将其推送到自己的远程仓库上，完成仓库的初始化。具体方法如下：

```bash
# clone the bare git proj
git clone --bare https://github.com/cs144/minnow

# cd the bare git directory and push everything to your own repo
cd minnow.git; git remote add mine git@github.com:Zhytou/minnow.git
git push --all mine
git push --tags mine

# remove the bare git directory and see your own repo
rm -rf minnow.git
git clone git@github.com:Zhytou/minnow.git
```

**Compile & Run**：

CS144课程实验使用cmake构架和编译，方法如下：

```bash
cd minnow
# 编译
cmake -S . -B build
# 运行
cmake --build build --target check_Webget
```

### Writing webget

这部分要求我们在`apps/webget.cc`里面实现`void get_URL( const string& host, const string& path )`函数，最终完成抓取网页并写入到标准输出的功能。

**Socket**：

这个任务在理解了套接字的通信流程之后就会变得非常简单。一个常见的客户端使用套接字向服务器发送信息的API调用顺序如下：

- 首先，服务器创建一个套接字，并使用bind将其绑定到某IP和端口，接着使用listen使其监听该端口的连接请求。
- 接着，客户端创建一个套接字，并使用connect尝试连接服务器。成功后，使用send向其发送请求。
- 此时，服务器介绍到客户端请求，使用accept得到一个新套接字，先使用recv读取请求，接着使用send发送回复。
- 接着，客户端使用recv接收到回复，使用close关闭连接。
- 最后，服务器使用close关闭连接。

此外，还需要注意的就是在写Http请求时，换行是`\r\n`。

```c++
void get_URL( const string& host, const string& path )
{
  TCPSocket sock;
  Address addr( host, "http" );
  sock.connect( addr );
  sock.write( "GET " + path + " HTTP/1.1\r\n" );
  sock.write( "Host: " + host + " \r\n" );
  sock.write( "Connection: close \r\n\r\n" );
  string buf;
  do {
    sock.read( buf );
    if ( buf.empty() ) {
      break;
    } else {
      cout << buf;
    }
  } while ( 1 );
}
```

**Test Results**：

```bash
zhytou@LAPTOP-Q0M4I2VQ:~/minnow/build$ make check_webget
Test project /home/zhytou/minnow/build
    Start 1: compile with bug-checkers
1/2 Test #1: compile with bug-checkers ........   Passed    0.17 sec
    Start 2: t_webget
2/2 Test #2: t_webget .........................   Passed    1.21 sec

100% tests passed, 0 tests failed out of 2

Total Test time (real) =   1.38 sec
Built target check_webget
```

### An in-memory reliable byte stream

这个部分要求我们在`src/byte_stream.hh`和`src/byte_stream.cc`文件中，完成`ByteStream`以及其派生类接口，实现可以读写的字节流。

**ByteStream Data Structure**：

我一开始想到使用队列作为底层数据结构来实现这个类，但后面考虑到其派生类`Reader`中的`peek`接口要求返回string_view类型，于是便使用字符串作为底层结构实现。

因为，如果使用队列作为实际存储数据的结构，在实现`peek`时必然会使用一个临时字符串存储队列中数据，并使用这个字符串构造`string_view`返回。这样一定会报错，因为，`string_view`一种只读的字符串视图，它并没有这段内存的所有权，所以在返回后临时字符串被销毁，那么就会有`string_view`悬空引用和访问无效内存的问题。

```c++
class ByteStream
{
protected:
  uint64_t capacity_;
  std::string buffer_;
  bool is_closed_;
  bool has_error_;
  uint64_t bytes_pushed_;
  uint64_t bytes_popped_;

public:
  explicit ByteStream( uint64_t capacity );
  // Helper functions (provided) to access the ByteStream's Reader and Writer interfaces
  Reader& reader();
  const Reader& reader() const;
  Writer& writer();
  const Writer& writer() const;
};
```

**Test Results**：

```bash
zhytou@LAPTOP-Q0M4I2VQ:~/minnow/build$ make check0
Test project /home/zhytou/minnow/build
      Start  1: compile with bug-checkers
 1/10 Test  #1: compile with bug-checkers ........   Passed    0.17 sec
      Start  2: t_webget
 2/10 Test  #2: t_webget .........................   Passed    1.21 sec
      Start  3: byte_stream_basics
 3/10 Test  #3: byte_stream_basics ...............   Passed    0.01 sec
      Start  4: byte_stream_capacity
 4/10 Test  #4: byte_stream_capacity .............   Passed    0.01 sec
      Start  5: byte_stream_one_write
 5/10 Test  #5: byte_stream_one_write ............   Passed    0.01 sec
      Start  6: byte_stream_two_writes
 6/10 Test  #6: byte_stream_two_writes ...........   Passed    0.01 sec
      Start  7: byte_stream_many_writes
 7/10 Test  #7: byte_stream_many_writes ..........   Passed    0.03 sec
      Start  8: byte_stream_stress_test
 8/10 Test  #8: byte_stream_stress_test ..........   Passed    0.01 sec
      Start 37: compile with optimization
 9/10 Test #37: compile with optimization ........   Passed    3.98 sec
      Start 38: byte_stream_speed_test
             ByteStream throughput: 6.56 Gbit/s
10/10 Test #38: byte_stream_speed_test ...........   Passed    0.07 sec

100% tests passed, 0 tests failed out of 10

Total Test time (real) =   5.49 sec
Built target check0
```

## 1 Stitching Substrings Into A Byte Stream

实验1要求实验一个名为`Reassembler`的整合器类。它能够将乱序传入且可能重叠的字符串排序并输出到给定的字节流中，其结构如下图：

![](https://zhytou.top/post/2023-8-4/)

其中，最重要的一点是`ByteStream`中未读取的部分加上`Reassembler`中无序的部分大小不能超过整个capacity。

**Reassembler Data Structure**：

`Reassembler`中最关键的一点就是如何临时存储可能重叠的字符串。我选择的是使用`map<uint64_t, char>`来存储，有一点滑动窗口的思想。可能更常规的做法是用`map<uint64_t, string>`来存储，但我考虑到重叠的字符串可能会导致维护这个字典的逻辑变得比较复杂，索性直接使用索引到字符的映射，让维护的逻辑变得非常简单。但这样做会导致效率严重下降（test13有可能超时）。`Reassembler`的定义如下：

```c++
class Reassembler
{
private:
  // Bytes stored temorarily in ther Reassembler whose indexes are in the available capacity
  std::map<uint64_t, char> sliding_window_;
  // The index of next byte expected to write in the outbounded stream
  uint64_t expected_index_;
  // The index of last byte expected to write in the outbounded stream
  uint64_t expected_last_index_;

public:
  Reassembler();
  void insert( uint64_t first_index, std::string data, bool is_last_substring, Writer& output );

  // The number of bytes stored in the Reassembler itself
  uint64_t bytes_pending() const;
  // The index of next byte epected
  uint64_t index_expected() const;
  // The index of last byte expected
  uint64_t last_index_expected() const;
};
```

**Putting Substrings In Sequence**：

按前面的思路，`Reassembler::insert`函数的框架就应该包括：

- 处理传入的字符串
  - 紧随expected_index_的直接写入输出字节流
  - 超出容量的直接丢弃
  - 其余的存入sliding_window_
- 检测暂存的字符串是否能够写入
- 判断整合是否结束

```c++
void Reassembler::insert( uint64_t first_index, string data, bool is_last_substring, Writer& output )
{
  if ( first_index <= expected_index_ && first_index + data.size() > expected_index_ ) {
    // if the part of substr lies just behind the expected index, push it to the output
    // be careful to update the expected index with the actual written size
  } else if ( first_index + data.size() > expected_index_
              && first_index < expected_index_ + output.available_capacity() ) {
    // if part of the substr lies in the availabe capacity, add it to the sliding window
    // be careful that the size of sliding window cannot exceed the available capacity of output stream
    for ( uint64_t i = 0; i < data.size() && first_index + i < expected_index_ + output.available_capacity();
          i++ ) {
      // ...
    }
  }

  // erase elements which are already written and check if there are any elements which can be written
  string old_data;
  for ( auto itr = sliding_window_.begin();
        itr != sliding_window_.end() && itr->first <= expected_index_ + old_data.size();
        itr = sliding_window_.erase( itr ) ) {
    // ...
  }

  // set the expected last index if the substr is the last one
  if ( is_last_substring ) {
    expected_last_index_ = first_index + data.size();
  }
  // when the last expected byte is inserted and the sliding window is empty, close the output
  if ( expected_index_ == expected_last_index_ && sliding_window_.empty() ) {
    output.close();
  }

  return;
}
```

**Test Results**：

```bash
zhytou@LAPTOP-Q0M4I2VQ:~/minnow/build$ make check1
Test project /home/zhytou/minnow/build
      Start  1: compile with bug-checkers
 1/17 Test  #1: compile with bug-checkers ........   Passed    0.22 sec
      Start  3: byte_stream_basics
 2/17 Test  #3: byte_stream_basics ...............   Passed    0.01 sec
      Start  4: byte_stream_capacity
 3/17 Test  #4: byte_stream_capacity .............   Passed    0.01 sec
      Start  5: byte_stream_one_write
 4/17 Test  #5: byte_stream_one_write ............   Passed    0.01 sec
      Start  6: byte_stream_two_writes
 5/17 Test  #6: byte_stream_two_writes ...........   Passed    0.01 sec
      Start  7: byte_stream_many_writes
 6/17 Test  #7: byte_stream_many_writes ..........   Passed    0.03 sec
      Start  8: byte_stream_stress_test
 7/17 Test  #8: byte_stream_stress_test ..........   Passed    0.01 sec
      Start  9: reassembler_single
 8/17 Test  #9: reassembler_single ...............   Passed    0.01 sec
      Start 10: reassembler_cap
 9/17 Test #10: reassembler_cap ..................   Passed    0.01 sec
      Start 11: reassembler_seq
10/17 Test #11: reassembler_seq ..................   Passed    0.01 sec
      Start 12: reassembler_dup
11/17 Test #12: reassembler_dup ..................   Passed    0.02 sec
      Start 13: reassembler_holes
12/17 Test #13: reassembler_holes ................   Passed    0.01 sec
      Start 14: reassembler_overlapping
13/17 Test #14: reassembler_overlapping ..........   Passed    0.01 sec
      Start 15: reassembler_win
14/17 Test #15: reassembler_win ..................   Passed    5.64 sec
      Start 37: compile with optimization
15/17 Test #37: compile with optimization ........   Passed    0.08 sec
      Start 38: byte_stream_speed_test
             ByteStream throughput: 6.30 Gbit/s
16/17 Test #38: byte_stream_speed_test ...........   Passed    0.07 sec
      Start 39: reassembler_speed_test
             Reassembler throughput: 14.16 Gbit/s
17/17 Test #39: reassembler_speed_test ...........   Passed    0.11 sec

100% tests passed, 0 tests failed out of 17

Total Test time (real) =   6.26 sec
Built target check1
```

## 2 The TCP Receiver

**Test Results**：

```bash
zhytou@LAPTOP-Q0M4I2VQ:~/minnow/build$ make check2
Test project /home/zhytou/minnow/build
      Start  1: compile with bug-checkers
 1/29 Test  #1: compile with bug-checkers ........   Passed    0.21 sec
      Start  3: byte_stream_basics
 2/29 Test  #3: byte_stream_basics ...............   Passed    0.01 sec
      Start  4: byte_stream_capacity
 3/29 Test  #4: byte_stream_capacity .............   Passed    0.01 sec
      Start  5: byte_stream_one_write
 4/29 Test  #5: byte_stream_one_write ............   Passed    0.01 sec
      Start  6: byte_stream_two_writes
 5/29 Test  #6: byte_stream_two_writes ...........   Passed    0.01 sec
      Start  7: byte_stream_many_writes
 6/29 Test  #7: byte_stream_many_writes ..........   Passed    0.03 sec
      Start  8: byte_stream_stress_test
 7/29 Test  #8: byte_stream_stress_test ..........   Passed    0.01 sec
      Start  9: reassembler_single
 8/29 Test  #9: reassembler_single ...............   Passed    0.01 sec
      Start 10: reassembler_cap
 9/29 Test #10: reassembler_cap ..................   Passed    0.01 sec
      Start 11: reassembler_seq
10/29 Test #11: reassembler_seq ..................   Passed    0.01 sec
      Start 12: reassembler_dup
11/29 Test #12: reassembler_dup ..................   Passed    0.02 sec
      Start 13: reassembler_holes
12/29 Test #13: reassembler_holes ................   Passed    0.01 sec
      Start 14: reassembler_overlapping
13/29 Test #14: reassembler_overlapping ..........   Passed    0.01 sec
      Start 15: reassembler_win
14/29 Test #15: reassembler_win ..................   Passed    5.89 sec
      Start 16: wrapping_integers_cmp
15/29 Test #16: wrapping_integers_cmp ............   Passed    0.01 sec
      Start 17: wrapping_integers_wrap
16/29 Test #17: wrapping_integers_wrap ...........   Passed    0.01 sec
      Start 18: wrapping_integers_unwrap
17/29 Test #18: wrapping_integers_unwrap .........   Passed    0.00 sec
      Start 19: wrapping_integers_roundtrip
18/29 Test #19: wrapping_integers_roundtrip ......   Passed    0.29 sec
      Start 20: wrapping_integers_extra
19/29 Test #20: wrapping_integers_extra ..........   Passed    0.05 sec
      Start 21: recv_connect
20/29 Test #21: recv_connect .....................   Passed    0.01 sec
      Start 22: recv_transmit
21/29 Test #22: recv_transmit ....................   Passed    0.17 sec
      Start 23: recv_window
22/29 Test #23: recv_window ......................   Passed    0.01 sec
      Start 24: recv_reorder
23/29 Test #24: recv_reorder .....................   Passed    0.01 sec
      Start 25: recv_reorder_more
24/29 Test #25: recv_reorder_more ................   Passed    9.54 sec
      Start 26: recv_close
25/29 Test #26: recv_close .......................   Passed    0.01 sec
      Start 27: recv_special
26/29 Test #27: recv_special .....................   Passed    0.01 sec
      Start 37: compile with optimization
27/29 Test #37: compile with optimization ........   Passed    0.07 sec
      Start 38: byte_stream_speed_test
             ByteStream throughput: 6.51 Gbit/s
28/29 Test #38: byte_stream_speed_test ...........   Passed    0.07 sec
      Start 39: reassembler_speed_test
             Reassembler throughput: 16.55 Gbit/s
29/29 Test #39: reassembler_speed_test ...........   Passed    0.10 sec

100% tests passed, 0 tests failed out of 29

Total Test time (real) =  16.62 sec
Built target check2
```

## 3 The TCP Sender

```bash
zhytou@LAPTOP-Q0M4I2VQ:~/minnow/build$ make check3
Test project /home/zhytou/minnow/build
      Start  1: compile with bug-checkers
 1/36 Test  #1: compile with bug-checkers ........   Passed    0.18 sec
      Start  3: byte_stream_basics
 2/36 Test  #3: byte_stream_basics ...............   Passed    0.01 sec
      Start  4: byte_stream_capacity
 3/36 Test  #4: byte_stream_capacity .............   Passed    0.01 sec
      Start  5: byte_stream_one_write
 4/36 Test  #5: byte_stream_one_write ............   Passed    0.01 sec
      Start  6: byte_stream_two_writes
 5/36 Test  #6: byte_stream_two_writes ...........   Passed    0.01 sec
      Start  7: byte_stream_many_writes
 6/36 Test  #7: byte_stream_many_writes ..........   Passed    0.03 sec
      Start  8: byte_stream_stress_test
 7/36 Test  #8: byte_stream_stress_test ..........   Passed    0.01 sec
      Start  9: reassembler_single
 8/36 Test  #9: reassembler_single ...............   Passed    0.01 sec
      Start 10: reassembler_cap
 9/36 Test #10: reassembler_cap ..................   Passed    0.01 sec
      Start 11: reassembler_seq
10/36 Test #11: reassembler_seq ..................   Passed    0.01 sec
      Start 12: reassembler_dup
11/36 Test #12: reassembler_dup ..................   Passed    0.02 sec
      Start 13: reassembler_holes
12/36 Test #13: reassembler_holes ................   Passed    0.01 sec
      Start 14: reassembler_overlapping
13/36 Test #14: reassembler_overlapping ..........   Passed    0.01 sec
      Start 15: reassembler_win
14/36 Test #15: reassembler_win ..................   Passed    5.61 sec
      Start 16: wrapping_integers_cmp
15/36 Test #16: wrapping_integers_cmp ............   Passed    0.01 sec
      Start 17: wrapping_integers_wrap
16/36 Test #17: wrapping_integers_wrap ...........   Passed    0.00 sec
      Start 18: wrapping_integers_unwrap
17/36 Test #18: wrapping_integers_unwrap .........   Passed    0.00 sec
      Start 19: wrapping_integers_roundtrip
18/36 Test #19: wrapping_integers_roundtrip ......   Passed    0.28 sec
      Start 20: wrapping_integers_extra
19/36 Test #20: wrapping_integers_extra ..........   Passed    0.05 sec
      Start 21: recv_connect
20/36 Test #21: recv_connect .....................   Passed    0.01 sec
      Start 22: recv_transmit
21/36 Test #22: recv_transmit ....................   Passed    0.16 sec
      Start 23: recv_window
22/36 Test #23: recv_window ......................   Passed    0.01 sec
      Start 24: recv_reorder
23/36 Test #24: recv_reorder .....................   Passed    0.01 sec
      Start 25: recv_reorder_more
24/36 Test #25: recv_reorder_more ................   Passed    9.01 sec
      Start 26: recv_close
25/36 Test #26: recv_close .......................   Passed    0.01 sec
      Start 27: recv_special
26/36 Test #27: recv_special .....................   Passed    0.01 sec
      Start 28: send_connect
27/36 Test #28: send_connect .....................   Passed    0.01 sec
      Start 29: send_transmit
28/36 Test #29: send_transmit ....................   Passed    0.30 sec
      Start 30: send_retx
29/36 Test #30: send_retx ........................   Passed    0.01 sec
      Start 31: send_window
30/36 Test #31: send_window ......................   Passed    0.06 sec
      Start 32: send_ack
31/36 Test #32: send_ack .........................   Passed    0.01 sec
      Start 33: send_close
32/36 Test #33: send_close .......................   Passed    0.02 sec
      Start 34: send_extra
33/36 Test #34: send_extra .......................   Passed    0.03 sec
      Start 37: compile with optimization
34/36 Test #37: compile with optimization ........   Passed    0.07 sec
      Start 38: byte_stream_speed_test
             ByteStream throughput: 5.47 Gbit/s
35/36 Test #38: byte_stream_speed_test ...........   Passed    0.07 sec
      Start 39: reassembler_speed_test
             Reassembler throughput: 17.48 Gbit/s
36/36 Test #39: reassembler_speed_test ...........   Passed    0.11 sec

100% tests passed, 0 tests failed out of 36

Total Test time (real) =  16.17 sec
Built target check3
```

## 4 The Network Interface

**Test Results**：

```bash
zhytou@LAPTOP-Q0M4I2VQ:~/minnow/build$ make check4
Test project /home/zhytou/minnow/build
    Start  1: compile with bug-checkers
1/2 Test  #1: compile with bug-checkers ........   Passed    0.17 sec
    Start 35: net_interface
2/2 Test #35: net_interface ....................   Passed    0.02 sec

100% tests passed, 0 tests failed out of 2

Total Test time (real) =   0.20 sec
Built target check4
```

## 5 The IP Router

**Test Results**：

```bash
zhytou@LAPTOP-Q0M4I2VQ:~/minnow/build$ make check4
Test project /home/zhytou/minnow/build
    Start  1: compile with bug-checkers
1/2 Test  #1: compile with bug-checkers ........   Passed    0.17 sec
    Start 35: net_interface
2/2 Test #35: net_interface ....................   Passed    0.02 sec

100% tests passed, 0 tests failed out of 2

Total Test time (real) =   0.20 sec
Built target check4
```
