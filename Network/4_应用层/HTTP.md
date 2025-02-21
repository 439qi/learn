---
title: Hyper Text Transfer Protocol
aliases:
  - HTTP
  - 超文本传输协议
---
> [!cite]- References  
> [HTTP协议入门](https://www.ruanyifeng.com/blog/2016/08/http.html)  
> [HTTP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)  
## HTTP

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

| Method | Description|
| - | - |
| GET | 请求指定的页面信息，请求参数拼接在请求URL之后 |
| POST | 向指定资源提交数据进行处理请求，请求参数封装在请求体中 |
| HEAD | 类似与GET，但响应中没有具体内容，用于获取报头|
| PUT | 请求更新指定页面  |
| PATCH | PUT方法的补充，对已知资源进行局部更新 |
| DELETE | 请求删除指定页面 |
| OPTIONS | 允许客户端查看服务器性能  |
| CONNECT | |
| TRACE | 回显服务器收到的请求，用于测试  |

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
  