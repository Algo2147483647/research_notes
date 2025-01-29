# Remote Procedure Call

[TOC]

## Concept

RPC (*Remote Procedure Call*) is a communication protocol that allows a program or process to invoke procedures or functions in a remote system or network over a network connection. It enables distributed systems to interact and communicate as if they were local components, abstracting the complexities of network communication.

## Architecture

<img src="./assets/RPC_Architecture.jpeg" alt="Image" style="zoom: 50%;" />

- **RPC Server**: The program or process that receives the RPC request, performs the requested operation, and sends back the response.
- **Registry**
  - ZooKeeper provides a distributed coordination service that can be used as a service registry, among other use cases. It allows services to register themselves as znodes and provides efficient data synchronization and coordination.
- **RPC Client**: The program or process that initiates the RPC request.
- **Interface Definition Language (IDL)**: An IDL specifies the remote procedures or functions available for invocation. It defines the data types, parameters, and return types of the remote procedures, ensuring that the client and server agree on the method signatures.
- **Stub**: On the client side, a stub is responsible for marshaling the parameters, making the remote procedure call, and unmarshaling the response.
- **Skeleton**: On the server side, a skeleton receives the incoming RPC request, unmarshals the parameters, invokes the corresponding procedure or method, and marshals the response.

## Core Functions

### Service Addressing

我们怎么告诉远程机器我们要调用Multiply，而不是Add或者FooBar呢？在本地调用中，函数体是直接通过函数指针来指定的，我们调用Multiply，编译器就自动帮我们调用它相应的函数指针。但是在远程调用中，函数指针是不行的，因为两个进程的地址空间是完全不一样的。所以，在RPC中，所有的函数都必须有自己的一个ID。这个ID在所有进程中都是唯一确定的。客户端在做远程过程调用时，必须附上这个ID。然后我们还需要在客户端和服务端分别维护一个 {函数 <--> Call ID} 的对应表。两者的表不一定需要完全相同，但相同的函数对应的Call ID必须相同。当客户端需要进行远程调用时，它就查一下这个表，找出相应的Call ID，然后把它传给服务端，服务端也通过查表，来确定客户端需要调用的函数，然后执行相应函数的代码。

- 在远程过程调用中所有的函数都必须有一个ID，这个 ID 在整套系统中是唯一确定的。
- 服务消费者在做远程过程调用时，发送的消息体中必须携带这个 ID。
- 服务消费者和服务提供者分别维护一个函数和 ID 的对应表。
- 当服务消费者需要进行远程调用时，它就查一下这个表，找出对应的 ID，然后把它传给服务端，服务端也通过查表，来确定客户端需要调用的函数，然后执行相应函数的代码。
- 服务寻址的实现方式有很多种，比较常见的是：**服务注册中心**。要调用服务，首先你需要一个服务注册中心去查询对方服务都有哪些实例，然后根据负载均衡策略择优选一。
- 像 Dubbo 框架的服务注册中心是可以配置的，官方推荐使用 Zookeeper。
- 服务提供者需要以某种形式提供服务调用相关的信息，包括但不限于服务接口定义、数据结构、或者中间态的服务定义文件。例如Facebook的 Thrift 框架的IDL文件，Web service的 WSDL 文件；服务的消费者需要通过一定的场景获取远程服务调用相关的信息。
- 远程代理对象：服务消费者用的服务实际是远程服务的本地代理，说白了就是通过动态代理来实现的。

### Serialization & Deserialization

只有二进制数据才能在网络中传输。那一个客户端调用远程服务的一个方法，像方法入参这些必然需要转换成二进制才能进行传输，这种将对象转换成二进制流的过程就叫做**序列化编码**。服务端接收到二进制流不能识别，势必要将二进制流转换成对象，这个逆过程就叫做**反序列化解码**。

### Network transmission

由此可见 TCP 的性能确实很好，因此市面上大部分 RPC 框架都使用 TCP 协议，但也有少部分框架使用其他协议，比如 gRPC 就基于 HTTP2 来实现的。数据编解码和网络传输可以有多种组合方式，比如常见的有：HTTP+JSON, Dubbo 协议+TCP 等。

[Thrift](./thrift.md)
