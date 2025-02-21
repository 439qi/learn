# Windows 具名管道

<link rel="stylesheet" type="text/css" href="res/simple.css">

## 原理

详见 [Unix FIFO](../Note/note_Unix.md/#fifo具名管道named-pipe)

## Server

服务端通过 `CreateNamedPipeW()` 创建具名管道，并调用 `ConnectNamedPipe()` 连接到管道  
之后通过 `ReadFile()` 和 `WriteFile()` 进行通信

### Open Modes

#### Access Mode

访问模式等价于设定服务器管道句柄的读写权限  

|access mode| equivalent generic access right|client access right|
|-|-|-|
| PIPE_ACCESS_INBOUND | GENERIC_READ|GENERIC_WRITE|
| PIPE_ACCESS_OUTBOUND | GENERIC_WRITE|GENERIC_READ|
| PIPE_ACCESS_DUPLEX | GENERIC_READ \| GENERIC_WRITE| GENERIC_READ \| GENERIC_WRITE|

客户端创建句柄时指定的读写权限与服务端访问模式存在对应关系

#### Overlapped Mode

是否为 `ReadFile()`、`WriteFile()`、`TransactNamedPipe()` 和 `ConnectNamedPipe()` 启用异步 I/O  
在异步模式下，这些函数会立即返回而非阻塞直至操作完成  

- 若返回时操作已完成，则返回值指示操作是否成功；若失败，则 `GetLastError()` 返回任意非 ERROR_IO_PENDING 错误  
- 若返回时操作尚未完成，则返回值为0，且 `GetLastError()` 返回 ERROR_IO_PENDING；此时调用线程必须同步等待操作完成  
    > 或使用函数的 Ex 版本

启用 Overlapped 模式必须指定 FILE_FLAG_OVERLAPPED，并在调用以上四个函数时传递一个指向合法 `OVERLAPPED` 结构体的指针  
每次请求应当使用不同的 `OVERLAPPED` 结构体对象，而非复用先前的对象，因为先前的请求结果可能尚未返回  

- `OVERLAPPED` 结构体

    ```cpp
    typedef struct _OVERLAPPED {
        ULONG_PTR Internal;
        ULONG_PTR InternalHigh;
        union {
            struct 
            {
                DWORD Offset;
                DWORD OffsetHigh;
            } DUMMYSTRUCTNAME;

            PVOID Pointer;
        } DUMMYUNIONNAME;

        HANDLE    hEvent;
    } OVERLAPPED, *LPOVERLAPPED;
    ```

    `Internal` 和 `InternalHigh` 保留为系统使用，分别为 I/O 请求的状态码和传输的字节数  
    `Offset` 和 `OffsetHigh` 仅用于支持文件指针偏移的设备，其余情况下必须为0  
    `Pointor` 保留为系统使用  
    `hEvent` 必须包含指向 **手动重置**(manual-reset) 事件对象的句柄，该对象通过 `CreateEvent()` 创建，用于进行同步

- ReadFileEx(), WriteFileEx()  
    Ex 版本的函数不使用同步对象，而是通过传递回调函数  
    在操作完成后，这些回调函数会被调用

#### Write-Through Mode

Write-Through 模式**仅**用于[字节型](#type-mode)且服务端与客户端位于不同计算机的管道  

该模式仅能在创建句柄时指定，之后无法更改  
服务端和客户端可以指定不同 Write-Through 模式  
该模式下，函数在确认数据已写入到远程计算机的缓冲池后才会返回，  且本地计算机会立即发送而非缓冲发送数据  

## Client

客户端通过 `CreateFile()` 或 `CallNamedPipe()` 连接到已存在的管道  

- `CreateFile()` 在没有可用管道实例时返回 INVALID_HANDLE_VALUE 并且 `GetLastError()` 返回 ERROR_PIPE_BUSY，此时可调用 `WaitNamedPipe()` 等待可用管道  

- `CallNamedPipe()` 连接并发送一次消息后即关闭连接，等价于依此调用 `CreateFile()`, `TransactNamedPipe()`, `CloseHandle()` 各一次  

之后通过 `ReadFile()` 和 `WriteFile()` 进行通信

### Modes

#### Type Mode

类型模式指定数据如何被写入到管道，默认为字节流  
对于同一个管道的所有实例，类型模式必须完全一致  
|||
|-|-|
|PIPE_TYPE_BYTE |数据以字节流的形式写入管道，系统不区分多次写入的字节|
|PIPE_TYPE_MESSAGE|数据以消息流的形式写入管道，系统将每次写入操作的字节视作一个消息单元，该模式下总是缓冲发送数据|

#### Read Mode

读取模式指定如何从管道中读取数据，默认为字节读取模式  
对于字节型的管道，只能使用字节读取模式  
对于消息型的管道，可以使用字节读取或消息读取模式，且服务器端和客户端读取模式不要求一致  

对于客户端，创建的句柄初始总是 PIPE_READMODE_BYTE，可以调用 `SetNamedPipeHandleState()` 设置读取模式
|||
|-|-|
|PIPE_READMODE_BYTE |以字节流的形式读取数据，单次读取直至管道中所有字节或指定字节数|
|PIPE_READMODE_MESSAGE|以消息流的形式读取数据，单次读取一个完整的消息<br>如果指定的字节数小于消息大小，会尽可能多地读取并返回 ERROR_MORE_DATA 错误|

#### Wait Mode

该模式决定管道为空或缓冲区不足时，函数 `ReadFile()`，`WriteFile()` 和 `ConnectNamedPipe()` 的行为  

对于 `ReadFile()`，`WriteFile()` 返回的句柄，初始总是阻塞等待模式  
`ConnectNamedPipe()` 调用时可以指定等待模式

在阻塞等待模式下，函数会无限等待管道另一端的进程完成一次操作；非阻塞等待模式下，函数会立即返回
|函数|PIPE_WAIT | PIPE_NOWAIT |
|-|-|-|
|`ReadFile()`|管道为空时，等待另一端写入完成|管道为空时，立即返回 ERROR_NO_DATA|
|`WriteFile()`|管道缓冲区不足时，等待另一端读出数据直至有足够缓冲区|管道缓冲区不足时，对于消息型管道，立即返回非0值<br>对于字节型管道，写入尽可能多的字节后返回|
|`ConnectNamedPipe()` |当没有客户端已连接或等待连接时，函数阻塞等待连接|当没有客户端已连接或等待连接时，函数立即返回 ERROR_PIPE_LISTENING|

> 微软小贴士 : )  
> 该模式用于兼容 Microsoft LAN Manager 2.0，不应被用于异步 I/O，异步 I/O 使用 [Overlapped Mode](#overlapped-mode)

## 函数

### FlushFileBuffers()  

当服务端和客户端完成使用管道实例后，服务器应当调用该函数  

该函数会阻塞直至客户端已读取管道中所有数据
  
### GetNamedPipeInfo()  

该函数返回管道类型，I/O 缓冲区大小以及管道的最大实例数  

### GetNamedPipeHandleState(), SetNamedPipeHandleState()  

分别用于获取和设置管道的读写模式和等待模式
