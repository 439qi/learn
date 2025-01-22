# 计算机网络

[互联网协议入门（一）](https://www.ruanyifeng.com/blog/2012/05/internet_protocol_suite_part_i.html)  
[HTTP协议入门](https://www.ruanyifeng.com/blog/2016/08/http.html)  
[TCP协议详解](https://zhuanlan.zhihu.com/p/64155705)  
[HTTP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)  
[What is an IPv6 address? [Fully explained]](https://www.cloudns.net/blog/what-is-an-ipv6-address/)  
[任播研究综述](https://www.jos.org.cn/html/2023/1/6435.htm)

## Layer

### 实体层

规定电气特性  
作用是负责传输0和1的电信号

### 链接层

规定0和1的电信号的分组方式，即多少个电信号为一组，每个信号位的含义  
作用是局域网内的点对点通信  

[**以太网**(Ethernet)协议](#ethernet)  
ARP协议

### 网络层

规定网络地址，区分不同计算机是否属于同一子网络  
作用是允许不同子网的任意计算机之间传输数据  

IPX 协议（对于大型网络如互联网支持较差，随着 TCP/IP 的流行而逐渐式微）  
IP 协议(IPv4, IPv6)

### 传输层

规定端口，表明数据包属于哪一个进程使用  
作用是不同计算机的不同进程间进行通信  

[TCP 协议](#tcp)  
UDP 协议  
QUIC 协议  

### 应用层

[HTTP 协议](#httphyper-text-transfer-protocol)  
[基于 HTTP 的 IPP](note_IPP(NFY).md)  
HTTPS  
DNS  
DHCP  
FTP  
Telnet  
SMTP  
POP3  

## Ethernet

链路层传输协议  

规定一组电信号构成一个**数据包**(packet)，叫做**帧**(Frame)，每一帧分成两个部分：**首部**(Head)和数据(Data)/**负载**(payload)  
数据包大小固定，最初为1518字节，后来增加为1522字节，其中首部22字节，负载1500字节
> 1500字节的设计是基于传输距离标准要求下协议使用的时钟同步  
> 同一个信号包的传输时长随传输距离增加而增加，而时钟同步信号必须在固定时隙中发出，为保证时隙固定必须限制 MTU 大小
>

## IPv6

网络层协议  
当两个对应的 IP 地址互相识别后，才能在之间建立路由从而通信

### IPv6 地址规格

IPv6 地址为 128 二进制位数字，其中每16位为一组，共8组，组间用冒号分隔  

其中，组内可省略高位0，组间全为0的组可省略为双冒号 ::

```ipv6
AF02:0000:0000:0000:0000:0000:0000:0002
AF02:0:0:0:0:0:0:2
AF02::2
```  

### IPv6 地址类型
>
> RFC4291  

- **单播**(unicast)  
  寻址时，发送方与接收方 1:1 关联
  在单播下，网络中的每一个节点都有**唯一**的 IP 地址，
- **选播/任播**(anycast)  
  寻址时，发送方与接收方 1:1(1 of N) 关联  
  Anycast 网络中的节点在公开网络上宣告相同的 IP 地址，发送方请求通过 BGP 协议路由到任一最近的节点

  在 IPv4 中，任播与单播共用地址池  
  在 IPv6 中，任播以特殊格式区分
- **多播/组播**(multicast)  
  寻址中，发送方与接收方之间 1:N 或 N:N 关联  
  在一个单一的传输中，数据报被同时发送给多个收件人  
- **广播**(broadcast)  
  IPv6 不支持  
  广播特性由多播实现

## TCP

基于 Ethernet 和 IP 协议的传输层传输协议  
TCP 是[面向连接](#tcp连接)的，即必须先建立 TCP 连接，传输完毕后关闭连接  

TCP 提供[**可靠交付**](#tcp可靠传输)的服务，即保证传输的数据无差错、不丢失、不重复、按序到达  
TCP 提供**全双工通信**  
TCP 面向字节流，即 TCP 不关心上层协议的数据具体内容，仅视作无结构的字节流

### TCP报文段(Segment)

TCP 报文段分为 TCP 首部和负载两部分  
TCP 报文段负载大小为$$ IP 数据包负载-TCP 报文段首部$$称之为 MSS(Max Segment Size)，理论最大为1480-20=1460字节  
> 若上层协议的数据包为1500字节，那么就必须分为两个 TCP 报文段发送，HTTP/2 协议对此进行了改进，[压缩 HTTP 头部信息](#首部压缩)使其尽可能在一个 TCP 报文段中
>

首部字段如下：

- **源端口**和**目标端口**各2字节
- **序号**seq 4字节，TCP为每个字节编号，值域$[0,2^{32}-1]$，当序号到达上限后重新从0开始
- **确认号**ack 4字节，期望收到对方下一个报文段的序号
- **偏移** 4位，负载相对于TCP报文段的偏移，等价于TCP首部长度
- **保留** 4位
- 8位**控制位**
  - CWR(Congestion Window Reduced)
  - ECE(ECN-Echo)
  - URG(Urgent)  
    为1时表明紧急指针字段有效  
    告知系统该报文段应尽快传输而非按照排队顺序
  - ACK(Acknowledgment)  
    仅用于建立连接，连接建立后始终为1
  - PSH(PUSH)  
    表明希望键入命令后立即得到响应(Telnet???)
  - RST(RESET)  
    表明TCP连接出现严重错误，必须释放连接再重新建立  
    也用于拒绝非法报文段以及拒绝打开连接
  - SYN(Synchronization)  
    建立连接时用于同步序号
  - FIN(Finish)  
    要求释放连接
- **窗口win** 2字节，接收方让发送方设置发送[窗口](#tcp流量控制)的大小
- **校验和** 2字节，校验范围包括首部和负载  
- **紧急指针** 2字节，仅当URG=1时有效
- **可选项**，长度可变，最大为40字节
- **填充**，长度随可选项而变，保证整个报文段4字节对齐

### TCP连接

每一条TCP连接只能点对点/一对一  
TCP连接的端点称作**套接字/插口**(socket)
> 烂翻译套接字  

RFC793 将其定义为$$socket=[IP地址]:[端口号]$$  
TCP 连接由两个 socket 所确定$$connection=\{socket1,socket2\}$$

建立连接分为两个部分：通知对方自己已经准备就绪 和 确认已接受到对方的通知  

对于握手，双方尚未开始发送数据，因而服务端接收到连接请求后可以立即就绪，对客户端连接请求的确认和就绪通知可以合并为一个报文段，共三次握手  
对于挥手，服务端接收到断开连接请求后可能还有数据需要发送，因而先发送对断开连接请求的确认，待剩余数据发送完毕后再发送断开连接就绪的通知，共四次挥手

#### 建立连接/握手

初始情况下，接收方 TCP 进程已创建**传输控制块**(TCB, Transmission Control Block)并处于 `LISTEN` 状态，即监听

1. 第一次握手  
    SYN=1, seq=x  
    发送方TCP进程创建传输控制块，随机**初始序列号**(ISN, Inital Sequence Number)并发送请求连接报文段，该报文段不可携带数据  
    TCP进程进入`SYN-SENT`状态，即同步已发送  

    > 初始序列号的目的在于防止数据乱序，否则短时间内重新建立连接，先前的报文段若延迟到达会导致混乱  
    > 随机的目的在于防止伪造报文段攻击
2. 第二次握手  
    SYN=1, ACK=1, seq=y, ack=x+1  
    若同意建立连接，同样随机初始化序列号发送确认报文段，该报文段不可携带数据  
    TCP进程进入`SYN-RCVD`状态，即同步已接收  

    此次握手可分为两个报文段，先发送ACK=1, ack=x+1，再发送SYN=1, seq=y  
    此时变为四次握手，等价于三次握手
3. 第三次握手  
    ACK=1, seq=x+1, ack=y+1  
    发送方TCP进程再向接收方TCP进程进行确认，该报文段可以携带数据，若不携带则下一个序号seq仍为x+1  
    TCP进程进入`ESTABLISHED`状态  
    接收方接收到报文段后同样进入`ESTABLISHED`状态  
    > 最后的确认报文段用于防止已失效的连接请求报文段延迟到达而产生错误
    >
#### 释放连接/挥手

1. 第一次挥手  
  FIN=1, seq=u  
  其中u=已发送的最后一个字节序号+1  
  发送方发送连接释放报文段，停止发送数据，主动关闭TCP连接  
  TCP进程进入`FIN-WAIT-1`状态，即终止等待1  
2. 第二次挥手  
  ACK=1, seq=v, ack=u+1  
  其中v=已发送的最后一个字节序号+1  
  从发送方到接收方的连接已释放，但发送方仍需接收从接收方的数据，TCP进程此时应通知上层协议进程
  接收方TCP进程进入`CLOSE-WAIT`状态  
  发送方TCP进程收到确认后进入`FIN-WAIT-2`状态
3. 第三次挥手  
  FIN=1, ACK=1, seq=w, ack=u+1  
  当接收方已发送完所有数据后，发送连接释放报文段  
  TCP进程进入`LAST-ACK`状态  
4. 第四次挥手  
  ACK=1, seq=u+1, ack=w+1  
  TCP进程进入`TIME-WAIT`状态，在等待2MSL(Maximum Segment Lifetime)后，进入`CLOSED`状态  
  接收方收到报文段后进入`CLOSED`状态

    > 若最后一个确认报文段超时或丢失，B会重传FIN+ACK报文段，A接收后会重传确认报文段并重新等待2MSL  
    > 等待2MSL用于防止已失效的报文段延迟到达而产生错误，2MSL之后该次连接的所有报文段都将从网络消失
    >

### TCP可靠传输

理想的可靠传输需要以下两点

1. 传输信道不产生差错  
  [确认和超时重传](#确认和超时重传)
2. 接收方总是来得及处理收到的数据  
  [TCP流量控制](#tcp流量控制)  

#### 确认和超时重传  

确认即接收到报文段后，返回一个确认应答消息  
超时重传即未发送的分组设置计时器，超过指定时间没有收到确认即视作发送的报文段已丢失并重新发送

##### 超时重传时间  

TCP记录一个报文段发出的时间以及收到确认的时间，两者之差即为往返时间$RTT$(Round-Trip Time)  

计算加权平均往返时间$RTTs$，其中$α∈[0, 1)$，推荐值为0.125  
$$new\_RTTs=(1-α)·old\_RTTs+α·new\_RTT$$

计算偏差加权平均值$RTTD$，其中$β∈[0, 1)$，推荐值为0.25  
$$new\_RTTD=(1-β)·old\_RTTD+β·|RTTs-new\_RTT|$$

计算超时重传时间$RTO$(Retransmission Timeout)  
$$RTO=RTTs+4·RTTD$$

##### 停止ARQ(Automatic Repear reQuest, ARQ)协议

每发送一个报文段均需要等待对方的确认  
为每一个发送的分组设置超时计时器，一定时间内没有收到确认即视作发送的报文段已丢失并重新发送

##### 连续ARQ协议

利用了滑动窗口协议的思想

- 流水线传输  
  等待ARQ协议信道利用率低，为提高传输效率，可以使用流水线传输，即连续发送多个分组，而非每发送一个就等待对方确认  

- 累计确认  
  即收到n个分组后，对按序到达的最后一个分组发送确认  

  累计确认问题在于无法反映已经接收到的所有分组信息，连续的n个分组，若丢失中间的某个分组，发送方无法得知其之后的分组是否已正常到达

##### 选择性确认(Selective Acknowledgment)

在TCP首部可选项字段增加SACK数据，将丢失的中间分组信息发送给发送方  
SACK必须TCP通信双方均支持

##### 重复选择性确认(Duplicate Selective Acknowledgment)

用于通知发送方哪些数据被重复接收了
NFY

#### TCP流量控制

TCP流量控制利用了**以字节为单位**的滑动窗口  

分为发送窗口和接收窗口

- 发送窗口  
  发送方依据收到的确认报文段和[拥塞窗口](#tcp-拥塞控制)构造窗口，窗口为$[ack, ack+min(win, cwnd))$  
  在没有收到确认的情况下，发送方可以将窗口内的数据连续发送出去，且在收到确认前必须保留用于重传  

  发送窗口维护三个指针p1,p2,p3，其中$[p1, p2)$为已发送但未收到确认的数据，$[p2, p3)$为可以发送但未发送的数据，$[p1, p3)$即为当前发送窗口  

  当p2与p3重合时，发送方停止发送，若一段时间后仍未收到确认则重传
- 接收窗口  
  接收方仅对**按序收到**的数据的最高序号给出确认  
  当窗口左端开始的连续个数据收到后，窗口相应整体右移

- 避免死锁  
  TCP为每一个连接设置一个**持续计时器**(persistence timer)，当收到通信另一方的零窗口通知时即启动，当到期时发送一个零窗口探测报文段，若确认报文段中窗口仍为0，则重置计时器；否则继续通信

### TCP 缓存与传输

TCP维持发送缓存和接收缓存  

- 发送缓存  
  包含已发送并确认的数据，发送窗口和未发送但超出发送窗口大小的数据  
- 接收缓存  
  包含已接收并确认的数据，接收窗口和未收到但超出接收窗口大小的数据

当缓存中数据达到MSS时或上层协议指定PSH操作位或计时器超时，立即将未发送的部分组装成报文段发送出去

#### 糊涂窗口综合征

TCP接收方的接收窗口仅剩很少字节数可用，且这种状态将持续一段时间，导致网络的效率很低  

解决方案是接收方等待直至缓存空间足够时再发送确认报文段向发送方更新窗口大小信息

### TCP 拥塞控制

- 拥塞  
  在计算机网络中的链路容量(即宽带)、交换结点中的缓存和处理机等，都是网络资源。在某段时间，若对网络中某一资源的需求超过了该资源所能提供的可用部分，网络的性能就要变差

拥塞控制目的是防止过多的数据注入到网络中导致拥塞

cwnd: 当前窗口大小  
ssthresh: 慢启动上限，通常为65536  
传输轮次: 拥塞窗口所允许的报文段均发送出去且都收到了确认

- 慢开始(slow start)
  cwnd初始值为MSS，每经过一个传输轮次即加倍，呈指数增长  
  当达到ssthresh时，使用拥塞避免

- 拥塞避免(congestion avoidance)
  cwnd每经过一个传输轮次加1  

- 快重传(fast retransmit)
  由于累计确认机制，当丢失某个中间的报文段时，仅会确认最后一个按序到达的报文段，当连续收到3个相同的确认报文段时，发送方即会在计时器超时之前，立即重传

- 快恢复(fast recovery)
  ssthresh减半
  cwnd设置为ssthresh当前值，并执行拥塞避免

拥塞窗口首先位于慢开始阶段，若达到ssthresh则进入拥塞避免  
当发生拥塞时，若未触发快重传(网络超时)时，则将ssthresh减半，并将cwnd置1  
若触发快重传，则在快重传之后进入快恢复

## UDP

基于 Ethernet 和 IP协议 的传输层传输协议  
UDP 是无连接、不可靠但快速的传输协议，因而可靠性需要由上层应用层协议实现  

### UDP 报文段

- **源端口**和**目标端口**各2字节
- 长度 2字节，标识UDP报文长度，以字节为单位
- 校验和 2字节，可选项，对包括UDP头和数据部分进行校验  
NFY

## QUIC

基于UDP的多路复用传输协议  

设计用于提供快速设置和低延迟的 HTTP 连接，在 HTTP/3 中取代 TCP 作为传输层  

- 集成了TLS握手，无需 TCP 和 TLS 两次握手
- HTTP/2 在单个 TCP 连接上进行 HTTP 事务的多路复用，当 TCP 丢包重传时会影响所有事务；QUIC 在 UDP 上为每条流单独实现丢包检测和重传，仅会影响丢包的流

## HTTP(Hyper Text Transfer Protocol)

基于 TCP/IP 协议的应用层传输协议，不涉及数据包(packet)传输，主要规定客户端和服务器的通信格式

HTTP 是一种无状态协议，不会对发送的请求和通信状态进行持久化处理，因而 HTTP/1.1 引入 cookie 来记录状态

### URI、URL、URN

- HTTP 使用统一资源标识符(Uniform Resource Identifiers, URI)来传输数据和建立连接  

- 统一资源定位符(Uniform Resource Locator, URL)是一种特殊的 URI  
  HTTP URL 格式:  
  `http://host[:<port>][<abs_path>][?param][#anchor]`

- 统一资源命名(Uniform Resource Name, URN)也是一种 URI，其仅命名资源但不指定如何定位资源

### MIME(Multipurpose Internet Mail Extension)

多用途互联网邮件扩展  
浏览器常常通过 MIME 类型而非扩展名来处理 URL

```mime
type/subtype;parameter=value
```

type 标识该资源所属的大致分类，subtype 标识确切分类  
parameter 为可选，提供额外信息  
mime 不区分大小写

### HTTP报文

#### HTTP 请求

规定客户端发送给服务器的数据格式，包括请求行、请求头、空行、请求体，其中空行用于分隔请求头和请求体，即使请求体为空
<table>
    <tr><td>Method</td><td>Request URL</td><td>HTTP Version</td></tr>
    <tr align="center"><td colspan="4">Headers</td></tr>
    <tr align="center"><td colspan="4">Empty Line</td></tr>
    <tr align="center"><td colspan="4">Content</td></tr>
</table>

##### 请求行

- 请求方式

    |Method|Description|
    |-|-|
    |GET| 请求指定的页面信息，请求参数拼接在请求URL之后 |
    |POST| 向指定资源提交数据进行处理请求，请求参数封装在请求体中 |
    |HEAD| 类似与GET，但响应中没有具体内容，用于获取报头|
    |PUT| 请求更新指定页面  |
    |PATCH| PUT方法的补充，对已知资源进行局部更新 |
    |DELETE| 请求删除指定页面 |
    |OPTIONS| 允许客户端查看服务器性能  |
    |CONNECT||
    |TRACE| 回显服务器收到的请求，用于测试  |

- 请求 URL
- HTTP 版本

##### 请求头

描述客户端配置信息

|Index|Header|Description|Example|
|-|-|-|-|
|1| Accept |指定客户端能够接收的内容类型|Accept: text/plain, text/html  |
|2 |Accept-Charset| 浏览器可以接受的字符编码集。| Accept-Charset: iso-8859-5  |
|3 |Accept-Encoding|指定浏览器可以支持的web服务器返回内容压缩编码类型| Accept-Encoding: compress, gzip |
|4| Accept-Language| 浏览器可接受的语言| Accept-Language: en,zh |
|5 |Authorization |HTTP授权的授权证书|Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ== |
|6| Connection| 表示是否需要持久连接。（HTTP 1.1默认进行持久连接）| Connection: close|
|7 |Cookie | HTTP请求发送时，会把保存在该请求域名下的所有cookie值一起发送给web服务器|Cookie: $Version=1; Skin=new;|
|8| Cache-Control|指定请求和响应遵循的缓存机制| Cache-Control: no-cache |
|9 |Content-Length |请求的内容长度|Content-Length: 348  |
|10 |Content-Type|请求的与实体对应的[MIME](#mimemultipurpose-internet-mail-extension)信息| Content-Type: application/x-www-form-urlencoded |
|11| Date |请求发送的日期和时间|Date: Tue, 15 Nov 2010 08:12:31 GMT |
|12 |Host | 指定请求的服务器的 域名:端口号 也可以是 IP:端口号|Host: <www.zcmhi.com> |
|13 |Referer | 任何非首次请求都有这个参数, 该首部用于所有请求.指示了请求来自于哪个页面|Referer: <http://www.example.com/archives/71.html>|
|14 |Origin |只有跨域请求或跳转时才有这个参数, 该首部用于 CORS 请求或者 POST 请求, 指示了请求来自于哪个站点。该字段仅指示服务器名称，并不包含任何路径信息。|Origin: <https://example.com> |
|15 |User-Agent|包含发出请求的用户信息| User-Agent: Mozilla/5.0 (Linux; X11)  |
|16 |Warning|关于消息实体的警告信息| Warn: 199 Miscellaneous warning  |
|17 |Proxy-Authorization |连接到代理的授权证书|Proxy-Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==  |

##### 请求体

发送给服务器的内容

#### HTTP 响应

<table>
    <tr><td>HTTP Version</td><td>Status Code</td><td>Status Message</td></tr>
    <tr align="center"><td colspan="4">Headers</td></tr>
    <tr align="center"><td colspan="4">Content</td></tr>
</table>

##### 状态行

- HTTP 协议版本
- 状态码
- 状态文本

##### 响应头

格式类似于[请求头](#请求头)

##### 响应体

发送给客户端的内容

#### TLS

网景(Netscape)公司在 TCP/IP 协议之上开发了额外的加密层 SSL，并最终标准化为 TLS

### HTTP/1.x连接

#### 短连接
>
> deprecated

HTTP 最早期和1.0使用的连接模型，每发起一个请求创建一个新的TCP连接，并在收到响应后关闭

缺点在于性能损耗十分严重

#### 长连接

长连接会保持一段时间，重复用于发送一系列请求，并在空闲一段时间后关闭

缺点在于空闲状态下也会消耗服务器资源，且重负载时会遭受 DoS 攻击

#### HTTP 流水线

> deprecated  
> 实现过于复杂且受制于队头阻塞问题，已被 HTTP/2 的多路复用算法取代

类似[连续ARQ协议](#连续arq协议)

### HTTP/2

基于 Google 的 SPDY 协议

HTTP/2 是一个彻底的二进制协议，头信息和数据体都是二进制，称之为称为**帧**(frame)

#### HTTP/2 帧

HTTP/2 在 HTTP/1.x 语法和底层协议间增加了新的中间层，将 HTTP/1.1 报文划分为头信息帧和数据帧并嵌入流中

#### 首部压缩

首部信息使用 gzip 或 compress 压缩后在发送  
客户端和服务器同时维护一张首部信息表并生成索引号，之后通信时仅传输索引号

### HTTP/3

基于 QUIC 的 HTTP  

## HTTPS
  