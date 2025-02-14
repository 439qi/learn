---
title: Quick UDP Connection
aliases:
  - QUIC
---
## QUIC

基于 [UDP](UDP.md) 的多路复用传输协议  

设计用于提供快速设置和低延迟的 HTTP 连接，在 HTTP/3 中取代 TCP 作为传输层  

- 集成了 TLS 握手，无需 TCP 和 TLS 两次握手
- HTTP/2 在单个 TCP 连接上进行 HTTP 事务的多路复用，当 TCP 丢包重传时会影响所有事务；QUIC 在 UDP 上为每条流单独实现丢包检测和重传，仅会影响丢包的流  
