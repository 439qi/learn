---
title: 计算机网络
tags:
  - TBD
---
> [!cite]- References  
> [互联网协议入门（一）](https://www.ruanyifeng.com/blog/2012/05/internet_protocol_suite_part_i.html)  
> [What is an IPv6 address? [Fully explained]](https://www.cloudns.net/blog/what-is-an-ipv6-address/)  
## Layer

```mermaid
block-beta

Link["Ethernet Head"]
Network["IP Head"]
Transport["TCP/UDP Head"]
Application["Application Data"]:4

style Link fill:#4f81bc,color:white,stroke-width:0
style Network fill:#c0504e,color:white,stroke-width:0
style Transport fill:#92d14f,color:white,stroke-width:0
style Application fill:#ffc000,color:white,stroke-width:0

```
### 实体层 

规定电气特性  
作用是负责传输0和1的电信号  

### 链路层  

规定0和1的电信号的分组方式，即多少个电信号为一组，每个信号位的含义  
作用是局域网内的点对点通信  

[**以太网协议**](Ethernet.md)  
ARP协议  

### 网络层  

规定网络地址，区分不同计算机是否属于同一子网络  
作用是允许不同子网的任意计算机之间传输数据  

> [!question] confirmation required  
> 以太网协议通过在本网络内广播的方式发送数据包，无法做到由不同子网络之间的通信  
> 网络层用于解决以上问题，通过引入网络地址以确定计算机所属的子网络，对于同一子网络采取广播发送，否则通过路由发送  

IPX 协议（对于大型网络如互联网支持较差，随着 TCP/IP 的流行而逐渐式微）  
IP 协议(IPv4, IPv6)

### 传输层  

规定端口，表明数据包属于哪一个进程使用  
作用是不同计算机的不同进程间进行通信  

[TCP 协议](TCP.md)  
[UDP 协议](UDP.md)  
[QUIC 协议](QUIC.md)  

### 应用层  

[HTTP 协议](HTTP.md)  
[基于 HTTP 的 IPP](note_IPP(NFY).md)  
HTTPS  
DNS  
DHCP  
FTP  
Telnet  
SMTP  
POP3  

