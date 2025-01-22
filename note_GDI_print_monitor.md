# GDI Print Monitor

打印监视器分为语言监视器和端口监视器

## 语言监视器

语言监视器属于打印监视器的一部分，包含一些用户模式的 DLL  
主要用于支持打印机的双向通信、监视打印机状态、获取并处理一些事件

### 主要功能

1. 在打印假脱机与双向打印机之间提供一个全双工的通信通道，从而具有提供软件可存取的状态信息的能力
2. 增加打印机控制信息到数据流，如由打印机作业语言定义的命令等

### 安装

语言监视器非必须，只有被特定打印机的.inf文件的LanguageMonitor条目中指出才会与打印机关联  

LanguageMonitor条目中需要指明语言监视器的显示名称及其dll文件名

```inf
LanguageMonitor="XXX Language Monitor,[filename].dll"
```

对于自定义安装程序， 可以调用spooler的`AddMonitor`函数显式安装

## 端口监视器

端口监视器属于打印监视器的一部分，包含一些用户模式的 DLL  
主要用于提供spooler和内核的端口驱动之间的通信路径，以及管理和配置服务器的打印机端口

对于Windows 2000及以上版本，端口监视器分为端口监视器UI dll和端口监视器服务器dll，UI dll通过Spooler的`XcvData`函数与服务器dll通信

- UI dll  
  实现UI相关功能，位于客户端的System32目录中
- 服务器dll  
  实现端口通信功能，在打印服务器上运行

## 语言监视器与端口监视器的关系

若打印机与语言监视器关联，语言监视器从打印处理器处接收数据流，做相应处理后发送给端口监视器  
> 更准确地说，打印处理器处理完数据后发送给spooler，spooler再发送给打印监视器的语言监视器

若打印机没有与语言监视器关联，则数据流会直接发送给端口监视器  

因而打印监视器所定义的函数，绝大部分对于语言监视器和端口监视器是相同的  
打印监视器要么调用语言监视器所实现的版本并由其处理后间接调用端口监视器所实现的版本，要么直接调用端口监视器所实现的版本

### 需要实现的函数

#### `DllMain()`  

  ```cpp
  BOOL WINAPI DllMain(
  _In_ HINSTANCE hinstDLL,
  _In_ DWORD     fdwReason,
  _In_ LPVOID    lpvReserved
  );
  ```

  dll入口点函数  
  当启动和终止进程或线程时，对于每个加载的dll调用其入口点函数  
  或者当调用`LoadLibrary`和`FreeLibrary`动态加载和卸载dll时也会调用  

  参数的`fdwReason`表明该函数被调用的原因  
  |可选值|含义|
  |-|-|
  |DLL_PROCESS_ATTACH<br>1|该dll被静态或动态加载至某个进程的虚拟地址空间；<br>此时`lpvReserved`表明该dll是静态还是动态加载|
  |DLL_PROCESS_DETACH<br>0|该dll被从某个进程的虚拟地址空间卸载；<br>此时`lpvReserved`表明是dll加载失败还是因调用`FreeLibrary`或进程终止而计数归零|
  |DLL_THREAD_ATTACH<br>2|当前进程正在创建一个新线程，在新线程的上下文调用所有关联的dll入口点函数|
  |DLL_THREAD_DETACH<br>3|某个线程正在退出|

#### `InitializePrintMonitor2()`

```cpp
LPMONITOR2 InitializePrintMonitor2(
[in]  PMONITORINIT pMonitorInit,
[out] PHANDLE      phMonitor
);
```

该函数用于打印监视器初始化，只有在加载dll时才会调用该函数  
语言监视器和端口监视器服务器dll必须**导出**该函数  

- 参数`phMonitor`用于给spooler返回监视器句柄
- 返回值为监视器dll实现的函数指针数组

#### `OpenPortEx()`

```cpp
BOOL WINAPI OpenPortEx(
_In_  HANDLE           hMonitor,
_In_  HANDLE           hMonitorPort,
_In_  LPWSTR           pPortName,
_In_  LPWSTR           pPrinterName,
_Out_ PHANDLE          pHandle,
_In_  struct _MONITOR2 *pMonitor
)
```

打开打印机端口  
语言监视器必须支持该函数  
当打印队列连接到端口时，spooler会调用该函数

#### `StartDocPort()`  

```cpp
typedef BOOL ( WINAPI *pfnStartDocPort)(
  _In_ HANDLE hPort,
  _In_ LPWSTR pPrinterName,
  _In_ DWORD  JobId,
  _In_ DWORD  Level,
  _In_ LPBYTE pDocInfo
);
```

在指定端口上启动打印作业  
语言监视器和端口监视器服务器dll必须支持该函数  
语言监视器通常转发给端口监视器  

端口监视器服务器dll通常调用`CreateFile()`函数建立与内核模式的端口驱动的连接

#### `WritePort()`

```cpp
BOOL WritePort(
_In_  HANDLE  hPort,
_In_  LPBYTE  pBuffer,
_In_  DWORD   cbBuf,
_Out_ LPDWORD pcbWritten
);
```

向打印机端口写数据  
语言监视器和端口监视器服务器dll必须支持该函数  

通常一次打印任务中会将数据分段多次调用该函数  

该函数必须确保在特定时间内返回以防止阻塞Spooler

语言监视器通常向数据流中写入一些打印机语言相关的命令后转发给端口监视器  

端口监视器服务器dll通常调用`WriteFile`函数将数据流发送给内核模式的端口驱动

#### `ReadPort()`

```cpp
BOOL ReadPort(
  _In_  HANDLE  hPort,
  _Out_ LPBYTE  pBuffer,
  _In_  DWORD   cbBuf,
  _Out_ LPDWORD pcbRead
);
```

语言监视器和端口监视器需要支持该函数

#### `SendRecvBidiDataFromPort()`

```cpp
DWORD SendRecvBidiDataFromPort(
 _In_ HANDLE                   hPort,
 _In_ DWORD                    dwAccessBit,
 _In_ LPCWSTR                  pAction,
 _In_ PBIDI_REQUEST_CONTAINER  pReqData,
 _Out_  PBIDI_RESPONSE_CONTAINER *ppResData
);
```

提供应用程序与打印机或打印机服务器之间的双向通信  

- Access Right 为32位的位标记，用于表明对象的访问权限，当线程不具备对象执行特定操作所需的权限时，系统会拒绝执行操作  
`dwAccessBit`为用户调用Spooler的`OpenPrinter`函数时传入的参数`_PRINTER_DEFAULTS`结构体的成员`DesiredAccess`

  | 可选值| 含义|
  |-|-|
  |PRINTER_ACCESS_ADMINISTER| 执行管理性的任务
  |PRINTER_ACCESS_USE|  执行基础打印操作
  |PRINTER_ACCESS_MANAGE_LIMITED| 执行管理性的任务<br>Windows 8.1及之后版本可用
  |PRINTER_ALL_ACCESS| 除了同步之外的所有操作|

- pAction表明请求的动作
  |常量|可选值|说明|  
  |-|-|-|  
  |BIDI_ACTION_ENUM_SCHEMA|L"EnumSchema"|枚举schema
  |BIDI_ACTION_GET|L"Get"|获取指定schema的值
  |BIDI_ACTION_GET_ALL|L"GetAll"|获取指定schema所有子节点的值
  |BIDI_ACTION_SET| L"Set"|设置schema的值

- pReqData包含bidi请求列表

  ```cpp
  struct _BIDI_REQUEST_CONTAINER {
  DWORD             Version;  //bidi api schema版本，当前为1
  DWORD             Flags;  // 保留给系统使用，必须为0
  DWORD             Count; //aData数组元素个数，
  BIDI_REQUEST_DATA aData[1]; //bidi request数组
  };
  
  struct _BIDI_REQUEST_DATA {
  DWORD     dwReqNumber;//request索引，用于与response对应
  LPWSTR    pSchema;  //schema字符串
  BIDI_DATA data;
  };
  
  struct _BIDI_DATA {
  DWORD dwBidiType;//BIDI_TYPE枚举类型，表明联合体类型
  union {
    BOOL             bData; //BIDI_BOOL
    LONG             iData; //BIDI_INT
    LPWSTR           sData;//BIDI_TEXT/BIDI_STRING
    FLOAT            fData;//BIDI_FLOAT
    BINARY_CONTAINER biData;//BIDI_BLOB
  } u;
  };
  ```

#### GetPrinterDataFromPort()

```cpp
typedef BOOL ( WINAPI *pfnGetPrinterDataFromPort)(
  _In_  HANDLE  hPort,
  _In_  DWORD   ControlID,
  _In_  LPWSTR  pValueName,
  _In_  LPWSTR  lpInBuffer,
  _In_  DWORD   cbInBuffer,
  _Out_ LPWSTR  lpOutBuffer,
  _In_  DWORD   cbOutBuffer,
  _Out_ LPDWORD lpcbReturned
);
```

获取打印机状态信息并返回

- `ControlID`  
  设备I/O控制代码  
  为0时请求的信息由`pValueName`提供，非0时由`lpInBuffer`提供

## WinDDK 示例代码

### 语言监视器实现 —— Pjlmon.dll
