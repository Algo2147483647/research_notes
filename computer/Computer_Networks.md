# $Computer\ Networks$

[TOC]

# Networks Architecture
* OSI (7 Layers)
  * Application Layer
  * Presentation Layer
  * Session Layer
  * Transport Layer
  * Network Layer
  * Data-link Layer
  * Physical layer

* TCP/IP (4 Layers)
  * Application Layer
  * Transport Layer
  * Network Layer
  * Local Access Layer

* Comprehensive
  * Application Layer
  * Transport Layer
  * Network Layer
  * Data-link Layer
  * Physical layer

## Physical Layer

### Equipment: Hub

## Data-link Layer
* MTU
* MAC Address: (每个设备都有唯一且不变的MAC Address)

### Equipment: Switch

只发给目标 MAC地址指向的那台电脑.

## Network Layer
### Equipment: Router

作为一台独立的拥有 MAC 地址的设备, 并且可以帮助把数据包做一次转发. (Router每一个端口, 都有独立MAC地址)

* 子网划分、子网掩码

### Protocol
  * **IP (Internet Protocol)**

  * **ICMP (Internet Control Message Protocol)**

  * **ARP (Address Resolution Protocol)**

  * **RARP (Reverse Address Resolution Protocol)**

## Transport Layer

### Protocol

- UDP (User Datagram Protocol)
- TCP (Transmission Control Protocol)

#### UDP (User Datagram Protocol)

* UDP datagram header
    |Bit|意义|
    |---|---|
    |0-15|Source port|
    |16-31|Destination port|
    |32-47|Length|
    |48-63|Checksum|
    |||

* Application: 使用UDP的协议
    * DNS
    * TFTP
    * SNMP
    * NFS

#### TCP (Transmission Control Protocol)





## Application Layer
* 数据传输基本单位为报文

### Protocol
  * **FTP (File Transfer Protocol)**

  * **Telnet**

  * **DNS (Domain Name System)**

  * **SMTP (Simple Mail Transfer Protocol)**

  * **POP3 (Post Office Protocol - Version 3)**

  * **HTTP (Hyper Text Transfer Protocol)**


#### HTTP (Hyper Text Transfer Protocol)

* Format
  - GET：请求读取由URL所标志的信息
  - POST：给服务器添加信息（如注释）
  - PUT：在给定的URL下存储一个文档
  - DELETE：删除给定的URL所标志的资源

* HTTP 状态码  

  |Code|Description|  
  |---|---|
  |1**|信息, 服务器收到请求, 需要请求者继续执行操作|
  |100|Continue
  |101|Switching Protocols
  |||
  |2**|成功, 操作被成功接收并处理
  |200|OK|
  |201|Created|
  |202|Accepted|
  |203|Non-Authoritative Information|
  |204|No Content|
  |205|Reset Content|
  |205|Reset Content|
  |||
  |3**|重定向,需进一步操作以完成请求|
  |301|Moved Permanently|
  |||
  |4**|客户端错误,请求包含语法错误或无法完成请求|
  |400|Bad Request|
  |401|Unauthorized|
  |403|Forbidden|
  |404|Not Found|
  |404|Method Not Allowed  |
  |||
  |5**|服务器错误, 服务器在处理请求的过程中发生了错误|
  |500|Internal Server Error|
  |||


  * **HTTP (Hyper Text Transfer Protocol over Secure Socket Layer)**

  * **Cookie / Session**

## *Q: What happens when you click on a URL in your browser?*

<img src="./assets/image-20230809125410705.png" alt="image-20230809125410705" style="zoom:50%;" />

[TCP](./TCP.md)

## 应用

[Remote_Procedure_Call](./Remote_Procedure_Call.md)



[Message_Queue](./Message_Queue.md)
