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

**WSL安装**：

首先，在确保虚拟化以及WSL功能正确开启后，前往微软商店下载Windows Subsystem For Linux。我使用的发行版是Ubuntu 22.04.2 LTS。更详细的下载方法，可以查看我的另一篇博客[搭建自己的生产环境]中的WSL部分。

接着，在WSL下载cmake、make、git和clang-format。其中，cmake和make是用于项目构建和编译的；git则是用于版本管理；clang-format则是一个格式化工具。它都会在后续使用到。具体下载方法如下：

```bash
sudo apt install cmake make git clang-format
```

**远程仓库**：

首先，在github中创建一个名为`minnow`的空仓库，作为我们的远程备份仓库。注意，为了遵守课程规定，我们需要将该仓库设为私有。

其次，克隆课程项目，并将其推送到自己的远程仓库上，完成仓库的初始化。具体方法如下：

``` bash
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

**编译运行**：

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

**Socket编程**：

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

**测试结果**：

### An in-memory reliable byte stream

这个部分要求我们在`src/byte_stream.hh`和`src/byte_stream.cc`文件中，完成`ByteStream`以及其派生类接口，实现可以读写的字节流。

**string_view Reader::peek()**：

我一开始想到使用队列作为底层数据结构来实现这个类，但后面考虑到其派生类`Reader`中的`peek`接口要求返回string_view类型，于是便使用字符串作为底层结构实现。

因为，如果使用队列作为实际存储数据的结构，在实现`peek`时必然会使用一个临时字符串存储队列中数据，并使用这个字符串构造`string_view`返回。这样一定会报错，因为，`string_view`一种只读的字符串视图，它并没有这段内存的所有权，所以在返回后临时字符串被销毁，那么就会有`string_view`悬空引用和访问无效内存的问题。

## 1 Stitching Substrings Into A Byte Stream

一开始使用`map<uint64_t, char>`来表示滑动窗口，但后面发现这样虽然逻辑上没有问题，但是针对test13特别容易超时。于是后面还是将滑动窗口的数据结构修改成`map<uint64_t, string>`，维护待写入字符串，避免一个一个位置处理。

## 2 The TCP Receiver

## 3 The TCP Sender

## 4 The Network Interface

## 5 The IP Router
