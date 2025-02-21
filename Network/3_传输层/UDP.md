---
title: User Datagram Protocol
aliases:
  - UDP
  - 用户数据报协议
tags:
  - TBD
---
## UDP

基于 Ethernet 和 IP协议 的传输层传输协议  
UDP 是无连接、不可靠但快速的传输协议，因而可靠性需要由上层应用层协议实现  

### UDP 报文段

- **源端口**和**目标端口**各2字节
- 长度 2字节，标识UDP报文长度，以字节为单位
- 校验和 2字节，可选项，对包括UDP头和数据部分进行校验  
NFY
