# gRPC 介绍

**参考文档**

[gRPC初探—概念介绍以及如何构建一个简单的gRPC服务](https://www.cnblogs.com/takumicx/p/10059448.html)

---

## 什么是 gRPC

gRPC 是由 Google 开发并开源的 RPC 框架,它具有以下特点：

* **语言中立**：支持C，Java，Go 等多种语言来构建 RPC 服务，这是 gRPC 被广泛的应用在微服务项目中的重要原因，因为不同的微服务可能用不同的语言构建。

* **基于HTTP/2协议**：支持双向流，消息头压缩，单 TCP 的多路复用，服务端推送等，这些特性使得 gRPC 更加适用于移动场景下的客户端和服务端之间的通信。

* **基于 IDL 定义服务**：编写 `.proto` 文件即可生成特定语言的数据结构、服务端接口和客户端。

* **支持 Protocol Buffer 序列化**：Protocol Buffer 是由 Google 开发的一种数据序列化协议 (类似于XML、JSON、Hession)，与平台无关，压缩和传输效率高，语法简单，表达能力强。

* 在服务端实现这个接口，并运行一个 gRPC 服务器来处理客户端调用。在客户端拥有一个 stub（存根）能够像服务端一样的方法。

<img src="https://img2018.cnblogs.com/blog/1422237/201812/1422237-20181203170957572-1172416823.png" title="" alt="" data-align="center">

工作流程：

1. 客户端（gRPC Stub）调用 A 方法，发起 RPC 调用。

2. 对请求信息使用 Protobuf 进行对象序列化压缩（IDL）。

3. 服务端（gRPC Server）接收到请求后，解码请求体，进行业务逻辑处理并返回。

4. 对响应结果使用 Protobuf 进行对象序列化压缩（IDL）。

5. 客户端接受到服务端响应，解码请求体。回调被调用的 A 方法，唤醒正在等待响应（阻塞）的客户端调用并返回响应结果。

&emsp;

## 服务接口定义

```protobuf
service UserManagement {
  rpc CreateNewUser (NewUser) returns (User) {}
  rpc GetUsers (GetUserReq) returns (UserList) {}
}
```
