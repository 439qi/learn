
<!-- 2023.07.25 created-->
<!-- 2023.10.12 format corrected-->
<!-- 2023.12.26 add Unicode -->
# note for windows programming

## 一、开始

### 第一个windows程序

___

```cpp
#include <windows.h>
int WINAPI WinMain (HINSTANCE hInstance, HINSTANCE hPrevInstance,
                   PSTR szCmdLine, int iCmdShow)
{
  MessageBox (NULL, TEXT ("Hello, Windows 98!"), TEXT ("HelloMsg"), 0);
  return 0 ;
}
```

> 把应用程序类型改为windows类型：  
Go to "Linker settings -> System". Change the field "Subsystem" to "Windows"

1. 头文件`<windows.h>`  
   主要的头文件，包含了其他常用头文件  
  
   |     |     |
   | --- | --- |
   |windef.h |基本型态定义　 |
   |winnt.h |支持Unicode的型态定义|
   |winbase.h |Kernel函数|
   |winuser.H |使用者接口函数|
   |wingdi.H |图形设备接口函数|

2. 程序入口

   ```cpp
   int WINAPI WinMain(HINSTANCE hInstance,
                  HINSTANCE hPrevInstance,
                  PSTR szCmdLine,
                  int iCmdShow)
   ```

   1. `WINAPI`定义在`<windef.h>`  

      `#define WINAPI __stdcall`  
      `__stdcall`指定了如何产生机器码以及如何在堆栈中如何存储参数  

   2. `HINSTANCE hInstance`

      句柄,windows中标识该程序的数字
   3. `HINSTANCE hPrevInstance`  

      已废弃，总是为`NULL`
      > 早期windows同一个程序的多个实例instance共享程序和只读内存。该参数用于确认是否有其他实例运行，以便省略一些初始化操作并从其他实例处迁移数据
   4. `PSTR szCmdLine`  

      执行程序的命令列
   5. `int iCmdShow`  

      指定程序最初显示的方式
   6. `MessageBox`
      - 第一个参数为窗口句柄
      - 第二个参数为显示的字符串
      - 第三个参数为标题栏的字符串
      - 第四个参数为定义在`<winuser.h>`中的常量，指定了要显示的按钮

### 编译、链接和执行  

___
编译阶段，从源码生成目标文件.obj ；链接阶段，将目标文件.obj和库文件.lib链接生成可执行文件.exe  

1. `kernel32.lib` `user32.lib` `gdi32.lib`  

   主要windows系统的引用链接库  
   包含了`kernel32.dll` `user32.dll` `gdi32.dll`的名称和引用信息  

___

## 二、Unicode简介

### 字符集简史

Windows 自NT之后，系统内部使用[Unicode字符集](note_encoding.md/#unicode)  
Windows NT系统(Windows 2000之前)仅支持UCS-2编码，在Windows 2000之后，支持UTF16

1. ASCII  
   不能很好满足其他语言的需求
2. 扩展ASCII  
   标准ASCII使用7比特位，相对于计算机1字节8比特位留下了扩展空间  

   IBM引入了代码页code page的概念。最低的128个代码总是相同的；较高的128个代码取决于定义代码页的语言
3. 双字节字符集double-byte character set  
   部分字符由两个字节表示，部分字符（主要是ASCII）由1个字节表示  
   导致附加的程序设计问题，如必须检查每个字节来决定长度
4. Unicode  
   宽字符集：使用两个字节表示一个字符  

### 宽字符和C

___

1. char字符串  
   使用一个字节表示一个字符，并在末尾添加'\0'表示字符串结尾
2. 宽字符  
   `typedef unsigned short wchar_t;`  
   使用两个字节表示一个字符，末尾的0也占用两个字节  
   `wchar_t* p = L"Hello!";`  
   `L`表示该字符串使用宽字符保存  

   对于标准库中的字符串相关函数都有对应的宽字符版本  
   `<string.h>`或`<wchar.h>`

   |||
   |--|--|
   | `strlen` | `wcslen` |
   | `printf` | `wprintf`|

### 宽字符和Windows

___

1. `<tchar.h>`  
   该头文件为标准库中字符串相关函数提供了替代名称  
  
   | |未定义`_UNICODE` |已定义`_UNICODE`|
   |--|--|--|
   |`_tcslen` |`#define _tcslen strlen` |`#define _tcslen wcslen`|
   |`TCHAR` |`typedef char TCHAR` |`typedef wchar_t TCHAR` |
   |`__T(x)`<br>`_T(x)`<br>`_TEXT(x)`|`#define __T(x) x`|`#define __T(x) L##x`|  

2. Windows函数调用  
   Windows API与字符串相关的函数基本都提供了两个版本ASCII版和宽字符版，通过函数名最后的`A`和`W`区分，根据是否定义`UNICODE`将函数替换为对应版本  
3. `sprintf`系列  
   `int sprintf(char* szBuffer, const char* szFormat, ...);`  

    `vsprintf`是`sprintf`的一个变种  
    `int vsprintf(char *szBuffer, const char *szFormat, va_list arg)`  

    windows对于以上两个函数提供了windows版本，`wsprintf`和`wvsprintf`，功能类似但无法处理浮点数  
    随着宽字符发表，又需要区分ASCII版本和宽字符版本，命名十分混乱，如下所示  
对于`sprintf`  

    ||常规|ASCII|宽字符|
    |-|-|-|-|
    |标准版|_stprintf|swprintf|sprintf|
    |最大长度版|_sntprintf|_snwprintf|_snprintf|
    |windows版|wsprintf|wsprintfW|wsprintfA|  

    对于`vsprintf`  

   | |常规|ASCII|宽字符|
   |-|-|-|-|
   |标准版|_vstprintf|vswprintf|vsprintf|
   |最大长度版|_vsntprintf|_vsnwprintf|_vsnprintf|
   |windows版|wvsprintf|wvsprintfW|wvsprintfA|

___

## 三、窗口和消息

### 自己的窗口

___

1. 总体结构  

   注册窗口类->创建窗口->显示窗口->循环处理消息并更新窗口

2. 标识符  
   **不要费力气去记忆Windows程序设计中的数值常数，Windows中使用的每个数值常数在表头文件中均有相应的标识符定义**

   |前缀|含义|
   |-|-|
   |CS |窗口类别样式|
   |CW |建立窗口|
   |DT |绘制文字|
   |IDI|图标ID|
   |IDC|游标ID|
   |MB |消息框|
   |SND|声音|
   |WM |窗口消息|
   |WS |窗口样式|

3. 句柄  
   句柄是一个（通常为32位的）整数，它代表一个对象
4. 注册窗口类  

   窗口类别定义了窗口消息处理程序和依据此类别建立的窗口的其它特征  

   不同窗口可以依照同一种窗口类别建立  

   `WNDCLASS`结构中最重要的两个字段是第二个和最后一个  
   - `lpfnWndProc`  
     窗口消息处理函数的地址
   - `lpszClassName`  
     窗口类别的文字名称  
     在只建立一个窗口的程序中，窗口类别名称通常设定为程序名称  

   其余字段  
   - `wndclass.cbClsExtra = 0;`
   - `wndclass.cbWndExtra = 0;`  
     在窗口类别结构和Windows内部保存的窗口结构中预留一些额外空间  

   - `wndclass.hInstance = hInstance;`  
     程序的执行实体句柄  

   - `wndclass.hIcon = LoadIcon (NULL, IDI_APPLICATION);`  
     设置图标  

   - `wndclass.hCursor = LoadCursor (NULL, IDC_ARROW);`  
     设置游标  

   - `wndclass.hbrBackground = GetStockObject (WHITE_BRUSH);`  
     窗口背景颜色  
     hbr指`handle to a brush`画刷句柄，画刷指定了填充一个区域的着色样式  

   - `wndclass.lpszMenuName = NULL;`  
      指定窗口类别菜单

5. 创建窗口  

   ```c++
   hwnd = CreateWindow (szAppName,    // window class name
    TEXT ( "The Hello Program"), // window caption    
    WS_OVERLAPPEDWINDOW,     // window style
    CW_USEDEFAULT,           // initial x position    
    CW_USEDEFAULT,           // initial y position    
    CW_USEDEFAULT,            // initial x size
    CW_USEDEFAULT,           // initial y size    
    NULL,                        // parent window handle    
    NULL,                     // window menu handle    
    hInstance,                  // program instance handle    
    NULL) ;                      // creation parameters
   ```

   - 标记为`window class name`的参数是`szAppName`  
     这是程序注册的窗口类别名称。建立的窗口通过该参数连接窗口类别  
   - window caption
     窗口标题
   - `WS_OVERLAPPEDWINDOW`  
     定义在`<winuser.h>`中的宏，由几个flag组合而成  
   - 标记了initial的几个参数  
     指定了窗口左上角相对于屏幕左上角的位置和窗口宽高  
     `CW_USEDEFAULT`表明使用预定尺寸  
   - creation parameters
     该参数可用于存取稍后程序中可能用到的数据
6. 显示窗口  
   - `ShowWindow(hwnd, iCmdShow)`;  
     `iCmdShow`指定最初如何显示窗口，  
     一般大小（`SW_SHOWNORMAL`）  
     - 指定一般大小时，窗口的显示区域就会被窗口类别中定义的背景画刷所覆盖  

     最小化（`SW_SHOWMINNOACTIVE`）  
     最大化（`SW_SHOWMAXIMIZED`）  
   - `UpdateWindow(hwnd);`  
     通过发送`WM_PAINT`消息触发重绘
7. 消息循环  
   - `while(GetMessage(&msg, NULL, 0, 0))`  
     Windows用从消息队列中取出的下一个消息来填充消息结构的各个字段  
     对于`WM_QUIT`，`GetMessage`返回0，否则返回非0值  
     第二、第三和第四个参数设定为NULL或者0，表示程序接收它自己建立的所有窗口的所有消息  

     ```c++
     typedef struct tagMSG
     {    
       HWND   hwnd ;    //接收消息的窗口句柄
       UINT   message ; //消息标识符
       WPARAM wParam ;  //消息参数，含义依据消息而定
       LPARAM lParam ;  //同上
       DWORD  time ;    //时间戳
       POINT  pt ;      //鼠标坐标
     } MSG, * PMSG ;
     ```

   - `TranslateMessage(&msg);`  
     将msg结构传给Windows，进行一些键盘转换。详见第六章  
   - `DispatchMessage (&msg);`  
     将msg结构传回windows，windows调用窗口消息处理程序  
8. 窗口消息处理程序  
`LRESULT CALLBACK WndProc(HWND hwnd, UINT message, WPARAM wParam, LPARAM lParam)`  

   自定义窗口消息处理程序不予处理的所有消息应该被传给名为`DefWindowProc`的Windows函数  
   - `WM_CREATE`消息  
     当Windows在WinMain中处理CreateWindow函数时，WndProc接收这个消息  
   - `WM_PAINT`消息  
     当窗口显示区域的一部分显示内容或者全部变为**无效**，以致于必须**更新画面**时，将由这个消息通知程序  
   - `WM_DESTROY`消息  
     Windows正在根据使用者的指示关闭窗口

### 设计难点

___
程序的所有实际动作均在窗口消息处理程序中发生  
Windows程序所作的一切，都是响应发送给窗口消息处理程序的消息  

1. 队列化消息与非队列化消息  
   队列化的消息是由Windows放入程序消息队列中的  
   非队列化的消息在Windows呼叫窗口时直接送给窗口消息处理程序  

___

## 四、输出文字

### 绘制和更新

___
Windows通过发送`WM_PAINT`消息通知窗口消息处理程序，窗口的部分显示区域需要绘制  

1. `WM_PAINT`  
   以下行为会收到`WM_PAINT`消息
   - 在使用者移动窗口或显示窗口时，窗口中先前被隐藏的区域重新可见
　 
   - 使用者改变窗口的大小（如果窗口类别样式有着`CS_HREDRAW`和`CS_VREDRAW`位flag的设定）
　 
   - 程序使用`ScrollWindow`或`ScrollDC`函数滚动显示区域的一部分
　 
   - 程序使用`InvalidateRect`或`InvalidateRgn`函数刻意产生`WM_PAINT`消息  

   以下行为可能会收到`WM_PAINT`消息  
   - Windows擦除覆盖了部分窗口的对话框或消息框。
　 
   - 菜单下拉出来，然后被释放
　 
   - 显示工具提示消息  

   以下行为Windows会自动保存显示区域并恢复
   - 鼠标光标穿越显示区域  
   - 图标拖过显示区域  
2. 有效矩形和无效矩形  
   通常不需要更新整个显示区域，只需要更新一个较小的区域。该区域称为无效区域或更新区域  

   如果在窗口消息处理程序处理`WM_PAINT`消息之前显示区域中的另一个区域变为无效，则Windows计算出一个包围两个区域的新的无效区域（以及一个新的无效矩形），并将这种变化后的信息放在绘制信息结构中。Windows不会将多个`WM_PAINT`消息都放在消息队列中。

### GDI简介

___

1. 设备上下文Device Context
   设备上下文是GDI内部保存的数据结构，与特定的显示设备相关。对于显示器，设备上下文总是与显示器上的特定窗口相关。

   当程序需要绘图时，它必须先取得设备上下文句柄。在取得了该句柄后，Windows用内定的属性值填入内部设备上下文结构

   当程序在显示区域绘图完毕后，它必须释放设备上下文句柄。句柄被程序释放后就不再有效，且不能再被使用。程序必须在处理单个消息处理期间取得和释放句柄。除了呼叫`CreateDC`函数建立的设备上下文之外，程序不能在两个消息之间保存其它设备上下文句柄。
2. `BeginPaint`和`EndPaint`  

   ```c++
   hdc = BeginPaint(hwnd, &ps);
   ...
   EndPaint (hwnd, &ps);
   ```  

   `BeginPaint`函数一般在准备绘制时使用窗口类中指定的画刷导致无效区域的背景被擦除  
  
    如果不调用`BeginPaint`和`EndPaint`（或者`ValidateRect`），则Windows不会使该区域变为有效。相反，Windows将发送另一个`WM_PAINT`消息，且一直发送下去。故若不处理`WM_PAINT`消息，则它必须将`WM_PAINT`消息传递给Windows中`DefWindowProc`（内定窗口消息处理程序）
    - 绘图信息结构  
      Windows为每个窗口保存一个绘图信息结构，定义如下

      ```c++
      typedef struct tagPAINTSTRUCT
      {    
        HDC    hdc;  
        BOOL   fErase;       
        RECT   rcPaint;    
        BOOL   fRestore;  
        BOOL   fIncUpdate;
        BYTE   rgbReserved[32];
      } PAINTSTRUCT ;
      ```

      - `fErase`指定是否需要擦除背景，通常为`false`(`BeginPaint`中已擦除)  

      - `rcPaint`限制了绘图区域的矩形，若需要在该区域之外绘图，在`BeginPaint`之前调用`InvalidateRect(hwnd,NULL,true)`使得整个区域无效并擦除背景  
3. `GetDC`和`ReleaseDC`  

   ```c++
   hdc = GetDC (hwnd);
   ...
   ReleaseDC (hwnd, hdc);
   ```

   `GetDC`不会使无效区域变为有效  
   可以调用`GetDC`和`ReleaseDC`来对键盘消息（如在字处理程序中）和鼠标消息（如在画图程序中）作出反应
4. `TextOut`  
   `TextOut (hdc, x, y, psText, iLength);`  

   `hdc`为设备上下文句柄  
    设备上下文的属性控制了被显示的字符串的特征

   `psText`参数是指向字符串的指针,`iLength`为字符串长度  
   字符串中不能包含ASCII控制字符  

   `x`和`y`指定字符串开始位置  
   第一个字符的左上角坐标即为`(x,y)`
5. 系统字体  
   系统字体是一种点阵字体，字符被定义为图素块  
  
   - `GetSystemMetrics`  

     ```c++
     TEXTMETRIC tm;
     hdc = GetDC (hwnd);
     GetTextMetrics (hdc, &tm);     
     ReleaseDC (hwnd, hdc);
     ```

     调用`GetSystemMetrics`可以获得字体大小  
     Windows启动后，系统字体的大小就不会发生改变，故只需要调用一次`GetTextMetrics`即可
   - `TEXTMETRIC`

     ```c++
     typedef struct tagTEXTMETRIC     
     {       
       LONG tmHeight; 
       LONG tmAscent;  
       LONG tmDescent;    
       LONG tmInternalLeading;   
       LONG tmExternalLeading;     
       LONG tmAveCharWidth;   
       LONG tmMaxCharWidth;     
       其它结构字段 
      }TEXTMETRIC, * PTEXTMETRIC;
     ```

     ![avatar][pic1]
     - `tmAscent`表示两个字符间的间距  
     - `tmHeight`是`tmAscent`和`tmDescent`的和。这两个值表示了基准在线下字符的最大纵向高度

### 滚动条

___

### 更好的滚动

___

## 五、图形基础

### GDI结构

___

1. GDI函数
   - 取得（或建立）和释放（或清除）设备上下文的函数
   - 取得有关设备上下文信息的函数
   - 取得和设定设备上下文参数的函数
   - 使用GDI对象的函数  
2. GDI基本图形  
   - 直线和曲线
   - 填入区域
   - 位图  

### 设备上下文

___

1. 获取设备上下文句柄
2. 取得设备上下文信息  

   包括显示器的大小（单位为图素或者实际长度单位）和色彩显示能力等
3. 设备上下文属性  

   程序取得一个设备内容的句柄时，Windows用默认值设定所有的属性
   - 保存设备内容  
    `wndclass.style = CS_HREDRAW | CS_VREDRAW | CS_OWNDC;`  
     注册窗口类时指定`CS_OWNDC`可以使对设备上下文属性所做的改变在下一次调用时仍有效  
     `CS_OWNDC`风格只影响`GetDC`和`BeginPaint`获得的设备上下文，不影响其它函数（如`GetWindowDC`）获得的设备上下文

   - 恢复设备上下文  

      ```c++
      idSaved = SaveDC (hdc);
      ...
      RestoreDC (hdc, idSaved);
      ```

      另一种调用方式  

      ```c++
      SaveDC (hdc);
      ...
      RestoreDC (hdc, -1);
      ```

4. 分辨率
   **NFY**
5. 色彩  
   Full-Color  
   RGB各8位

   High-Color  
   RGB5,6,5位，绿色多一位是因为人眼对绿色更敏感

### 画点和线

___

1. 点  
   `SetPixel(hdc, x, y, crColor);`  
   `crColor = GetPixel(hdc, x, y);`
2. 直线  
   - `LineTo`  

     `MoveToEx(hdc, xBeg, yBeg, NULL);`  
     `LineTo(hdc, xEnd, yEnd);`  
     `MoveToEx`实际上不会画线，它只是设定了设备内容的「目前位置」属性。然后`LineTo`函数从目前的位置到它所指定的点画一条直线，初始点位默认为(0,0)  
     `GetCurrentPositionEx(hdc, &pt);`  
     返回当前位置，pt为POINT结构体

   - `Polyline(hdc, apt, 5);`  
     apt为POINT型数组，5为数组元素个数  

3. 边界框函数  
   以下函数会填充内部且依据矩形边界框来绘图  
   Windows只画到（而不包含）右坐标和底坐标  

   - `Rectangle(hdc, xLeft, yTop, xRight, yBottom);`  
   - `Ellipse(hdc, xLeft, yTop, xRight, yBottom);`  
   - `RoundRect(hdc, xLeft, yTop, xRight, yBottom, xCornerEllipse, yCornerEllipse);`  
   - `Arc(hdc, xLeft, yTop, xRight, yBottom, xStart, yStart, xEnd, yEnd);`  
   - `Chord(hdc, xLeft, yTop, xRight, yBottom, xStart, yStart, xEnd, yEnd);`  
   - `Pie(hdc, xLeft, yTop, xRight, yBottom, xStart, yStart, xEnd, yEnd);`  

4. 贝塞尔曲线  
   `PolyBezier(hdc, apt, iCount);`  
   前四个点（按照顺序）给出贝塞尔曲线的起点、第一个控制点、第二个控制点和终点。此后的每一条贝塞尔曲线只需给出三个点，因为后一条贝塞尔曲线的起点就是前一条贝塞尔曲线的终点  

   `PolyBezierTo(hdc, apt, iCount);`  
   使用目前点作为第一个起点，第一条以及后续的贝塞尔曲线都只需要给出三个点。当函数传回时，目前点设定为最后一个终点  

5. 画笔  
   画笔决定线的色彩、宽度和画笔样式，其中画笔样式可以是实线、点划线或者虚线  

   `GetStockObject(WHITE_PEN);`  
   获得现有画笔的句柄  
   `SelectObject(hdc, hPen);`  
   令设备上下文使用该画笔，直到选择另外一个画笔或者释放设备上下文句柄为止。返回值为上一个画笔的句柄  

   使用画笔等GDI对象时，应遵守三条规则
     - 最后要删除自己建立的所有GDI对象
     - 当GDI对象正在一个有效的设备内容中使用时，不要删除它
     - 不要删除现有对象

- 画笔的创建、使用和删除  
  `hPen = CreatePen(iPenStyle, iWidth, crColor);`  
  `hPen = CreatePenIndirect(&logpen);`  
  > `logen`为`LOGPEN`结构体，结构体有三个成员：`lopnStyle`（`UINT`）是画笔样式，`lopnWidth`（`POINT`结构）是按逻辑单位度量的画笔宽度，`lopnColor`(`COLORREF`)是画笔颜色。Windows只使用`lopnWidth`结构的`x`值作为画笔宽度，而忽略`y`值。  

  `CreatePen`和`CreatePenIndirect`函数建立与设备上下文没有联系的逻辑画笔。直到呼叫`SelectObject之后`，画笔才与设备上下文发生联系  
  
  `DeleteObject(hPen)`

- 虚线间的空隙  
  空隙的着色取决于设备上下文的两个属性，背景模式和背景颜色。  
  默认背景模式为`OPAQUE`，使用背景色填充空隙  
  使用`TRANSPARENT`，Windows将不会填充空隙  
  
- 绘图方式  
  Windows使用画笔来画线时，它实际上执行画笔图素与目标位置处原来图素之间的某种位布尔运算，称为光栅运算ROP,或二元光栅运算ROP2  
  `SetROP2(hdc, iDrawMode);`  
  `iDrawMode = GetROP2(hdc);`

### 绘制填充区域

___
Windows使用画刷来绘制填充区域  

1. 多边形填充模式  
   `ALTERNATE` `WINDING`
2. 画刷  
   - `hBrush = CreateSolidBrush(crColor);`
   - `hBrush = CreateHatchBrush(iHatchStyle, crColor);`  
   - `CreatePatternBrush`  
   - `CreateDIBPatternBrushPt`  
   - `hBrush = CreateBrushIndirect(&logbrush);`  
     `logbrush`为`LOGBRUSH`结构体

### GDI映像方式

___

|映像方式|逻辑单位|x|y|
|-|-|-|-|
|MM_TEXT|像素|右|下|
|MM_LOMETRIC| 0.1 mm|右|上|
|MM_HIMETRIC|0.01 mm|右|上|
|MM_LOENGLISH|0.01 inch|右|上|
|MM_HIENGLISH|0.001 inch|右|上|
|MM_TWIPS|1/1440 inch|右|上|
|MM_ISOTROPIC|任意(x=y)|可选|可选|
|MM_ANISOTROPIC|任意(x!=y)|可选|可选|  

> 默认方式为`MM_TEXT`

`SetMapMode(hdc, iMapMode);`  
`iMapMode = GetMapMode(hdc);`  

1. 设备坐标和逻辑坐标  
   Windows对所有消息，所有非GDI函数，部分GDI函数，永远使用设备坐标  
2. 设备坐标系  
   Windows将GDI函数中指定的逻辑坐标映像为设备坐标  

   - 屏幕坐标  
     屏幕左上角为原点
   - 全窗口坐标
     整个窗口（包括标题栏）的左上角为原点
   - 显示区域坐标系  
     显示区域左上角为原点  

   `ClientToScreen`和`ScreenToClient`可以将显示区域坐标与屏幕坐标互相转换，`GetWindowRect`函数获得屏幕坐标下的整个窗口的位置和大小  

3. 视端口和窗口

## 六、键盘

### 键盘基础

键盘输入以消息的形式传递给程序的窗口消息处理程序  
Windows用八种不同的消息来传递不同的键盘事件  

1. 忽略键盘  
   - 系统功能相关的按键一般可以忽略
   - 通过快捷键启动菜单项，快捷键一般定义在资源描述文件中，由Windows自动转换  
   - 对话框处于活动状态时，键盘输入一般由Windows处理
2. 焦点  
   在按键时，只有一个窗口处理程序WndProc接收键盘消息，由消息中的hwnd字段指定  

   有输入焦点的窗口是**活动窗口**或**活动窗口的衍生窗口**（活动窗口的直接与间接子窗口）
   - 活动窗口通常是顶层窗口  
   - 活动窗口有子窗口，那么有输入焦点的窗口既可以是活动窗口也可以是其子窗口
   - 所有程序都是最小化的时候输入焦点不在任何窗口中
3. 队列和同步  
   系统消息队列用于初步保存使用者从键盘和鼠标输入的信息

   分为两步，首先在系统消息队列中保存消息，然后将它们放入应用程序的消息队列
   > 原因是需要同步。使用者的输入速度可能比应用程序处理按键的速度快，并且特定的按键可能会使焦点从一个窗口切换到另一个窗口，后来的按键就输入到了另一个窗口。但如果后来的按键已经记下了目标窗口的地址，并放入了应用程序消息队列，那么后来的按键就不能输入到另一个窗口。  

   只有当应用程序处理完前一个输入消息时，Windows才会从系统消息队列中取出下一个消息放入应用程序的消息队列中

4. 按键和字符  
   对产生可显示字符的按键组合，Windows不仅给程序发送按键消息，而且还发送字符消息。有些键不产生字符，Windows只产生按键消息。
   > A键，按下该键是一次按键，释放该键也是一次按键。根据Ctrl、 Shift和Caps Lock键的状态，A键又能产生不同的字符；  
   shift键、功能键、光标移动键和特殊字符键如Insert和Delete，这些键不产生字符

### 按键消息

___
按下一个键时，Windows把`WM_KEYDOWN`或者`WM_SYSKEYDOWN`消息放入有输入焦点的窗口的消息队列；  
释放一个键时，Windows把`WM_KEYUP`或者`WM_SYSKEYUP`消息放入消息队列中  
按住一个键并释放时，会产生一列`WM_KEYDOWN`消息和一个`WM_KEYUP`消息

1. 系统按键和非系统按键  
   程序通常忽略`WM_SYSKEYUP`和`WM_SYSKEYDOWN`消息，并将它们传送到`DefWindowProc`
2. 虚拟键码  
   虚拟键码保存在`WM_KEYDOWN`、`WM_KEYUP`、`WM_SYSKEYDOWN`和`WM_SYSKEYUP`消息的`wParam`参数中  

   Windows程序通常不需要监视Shift、Ctrl或Alt键的状态  

   数字和字母的虚拟键码是ASCII码。Windows程序几乎从不使用这些虚拟键码而是使用ASCII码字符的字符消息
3. `lParam`消息  
   ![avatar][pic2]  
   - 重复计数  
     重复计数是该消息所表示的按键次数，多数情况下为1
   - OEM扫描码  
     OEM扫描码是由硬件（键盘）产生的代码  
     除非需要依赖实际键盘布局的样貌，否则Windows程序可以忽略掉几乎所有的OEM扫描码信息
   - 扩展键flag
     如果按键结果来自IBM增强键盘(102、108键键盘)的附加键之一，那么扩充键flag为1  
     Windows程序通常忽略扩展键旗标
   - 上下文码  
     假如同时压下ALT键，那么该位为1  
     即对`WM_SYSKEYUP`与`WM_SYSKEYDOWN`而言，此位为1；对`WM_KEYUP`与`WM_KEYDOW`消息而言，此位为0  

     例外：  
     - 如果活动窗口最小化了，这时候所有的按键都会产生`WM_SYSKEYUP`和`WM_SYSKEYDOWN`消息，但若未按下Alt，该位为0
     - 对于一些非英文键盘，有些字符是通过Shift、Ctrl或者Alt键与其它键相组合而产生，此时该位为1，但消息并非系统按键消息
   - 先前按键状态  
     如果在此之前键是释放的，则该为0，否则为1
   - 转换状态  
     如果键正被按下，该位为0，如果键被释放，该位为1。
4. 位移状态  
   `iState = GetKeyState(VK_SHIFT);`  
   返回是否按下对应按键。可用于判断是否按下了位移键（Shift、Ctrl和Alt）或开关键（Caps Lock、Num Lock和Scroll Lock）  
   `GetKeyState`并非实时检查键盘状态，而只是检查正在处理的消息的键盘状态  
5. 使用按键消息  
   Windows程序通常为不产生字符的按键使用`WM_KEYDOWN`消息
   多数情况下，只需要为光标移动键（有时也为Insert和Delete键）处理`WM_KEYDOWN`消息。在使用这些键的时候，通过`GetKeyState`来检查Shift键和Ctrl键的状态

### 字符消息

___
对于字符消息，最好交由Windows处理国际键盘间的差异问题  

`TranslateMessage`函数将按键消息转换为字符消息
> 如果消息为`WM_KEYDOWN`或者`WM_SYSKEYDOWN`，并且按键与位移状态相组合产生一个字符，则`TranslateMessage`把字符消息放入消息队列中。此字符消息将是`GetMessage`从消息队列中得到的按键消息之后的下一个消息。  

字符消息的`lParam`与产生该字符消息的按键消息的`lParam`相同  
`WParam`为ANSI或Unicode代码而非虚拟键码
> 如果注册窗口类时调用的是`RegisterClassA`，则为ANSI代码  
如果调用的是`RegisterClassW`，则为Unicode代码

1. 4类字符消息  

   ||字符|死字符|
   |-|-|-|
   |非系统字符|WM_CHAR|WM_DEADCHAR|
   |系统字符|WM_SYSCHAR|WM_SYSDEADCHAR|  

   多数情况下，不需要处理`WM_CHAR`外的其他消息  
2. 消息顺序  
   如果按下Shift键，再按下A键，然后释放A键，再释放Shift键，就会输入大写的A，而窗口消息处理程序会接收到五个消息

   |消息|按键或者代码|
   |-|-|
   |WM_KEYDOWN|VK_SHIFT|
   |WM_KEYDOWN|A的虚拟键码|
   |WM_CHAR|A的字符代码|
   |WM_KEYUP|A的虚拟键码|
   |WM_KEYUP|VK_SHIFT|

3. 控制字符  
   对于ASCII控制字符，作者的经验是在字符消息中处理而非按键消息  
4. 死字符  
   在某些非英语键盘上，有些键本身不产生字符，只用于给字母加上音调，称之为**死键**。按下这些键就会产生死字符

### 键盘消息和字符集

___

1. 字符集和字体

### 插入符号

___

## 七、鼠标

通常认为，键盘便于输入和操作文字数据，而鼠标则便于画图和操作图形对象

### 鼠标基础

___
`fMouse = GetSystemMetrics(SM_MOUSEPRESENT);`  
返回鼠标是否存在  
`cButtons = GetSystemMetrics(SM_CMOUSEBUTTONS);`  
返回鼠标上按键的个数

### 显示区域鼠标消息

___

1. **无论该窗口是否活动或者是否拥有输入焦点，只要鼠标跨越窗口或者在某窗口中按下鼠标按键，那么窗口消息处理程序就会收到鼠标消息**  

2. 消息次数取决于鼠标硬件以及处理鼠标消息的速度  

3. 鼠标在窗口显示区域移动时，窗口信息处理程序会收到`WM_MOUSEMOVE`消息。按下或释放鼠标按键时，会收到如下消息  

   |键|按下|释放|双击|
   |-|-|-|-|
   |左|WM_LBUTTONDOWN|WM_LBUTTONUP|WM_LBUTTONDBCLK|
   |中|WM_MBUTTONDOWN|WM_MBUTTONUP|WM_MBUTTONDBCLK|
   |右|WM_RBUTTONDOWN|WM_RBUTTONUP|WM_RBUTTONDBCLK|

   > **已过时，现在有更多鼠标消息了**  
4. 对以上消息，`lParam`低字节为x坐标，高字节为y坐标（显示区域的相对坐标）；`wParam`指示鼠标按键以及Shift和Ctrl键的状态

   |||
   |-|-|
   |MK_LBUTTON|按下左键|
   |MK_MBUTTON|按下中键|
   |MK_RBUTTON|按下右键|
   |MK_SHIFT|按下Shift键|
   |MK_CONTROL|按下Ctrl键  |

   将`wParam`与以上宏按位与即可得到对应按键状态
5. 双击鼠标按键  
   必须在窗口类中包含`CS_DBLCLKS`才能收到双击消息  
   `wndclass.style = CS_HREDRAW | CS_VREDRAW | CS_DBLCLKS;`  

   双击的第一次最好执行单击功能，否则鼠标处理会变得十分复杂

### 非显示区域鼠标消息

___

1. 鼠标在窗口的非显示区域移动或按键时，会收到非显示区域鼠标消息
2. `wParam`指明移动或者按鼠标按键的非显示区域，`lParam`指明鼠标在屏幕上的绝对坐标
3. 命中测试消息  
   `WM_NCHITTEST`  
   - 该消息优先于所有其他的鼠标消息，故拦截该消息即可禁用鼠标功能
   - `lParam`指明鼠标的绝对坐标，`wParam`参数未使用
   - 该消息通常交由`DefWindowProc`处理，然后Windows依据该消息产生与鼠标位置相关的其他鼠标消息。对于非显示区域鼠标消息，其设置了消息中的`wParam`

     |||
     |-|-|
     |HICLIENT|显示区域|
     |HTNOWHERE|不在窗口中|
     |HTTRANSPARENT|窗口被覆盖|
     |HTERROR|产生警告|

### 程序中的命中测试

手动计算或通过子窗口来处理

|参数|主窗口|子窗口|
|-|-|-|
|窗口类别|"Checker3"|"Checker3_Child"|
|窗口标题|"Checker3..."|NULL|
|窗口样式|WS_OVERLAPPEDWINDOW|WS_CHILDWINDOW \| WS_VISIBLE|
|水平位置|CW_USEDEFAULT|0|
|垂直位置|CW_USEDEFAULT|0|
|宽度|CW_USEDEFAULT|0|
|高度|CW_USEDEFAULT|0|
|父窗口句柄|NULL|hwnd|
|菜单句柄/子ID|NULL|(HMENU) (y << 8 \| x)|
|执行实体句柄|hInstance|(HINSTANCE) GetWindowLong (hwnd, GWL_HINSTANCE)|
|额外参数|NULL|NULL|  

 对于子窗口，菜单句柄的参数代表子ID，即自身相对于父窗口的唯一ID

### 拦截鼠标

### 鼠标滑轮

## 八、定时器  

## 九、子窗口控件

## 十、菜单及其他资源

### 图标、光标、字符串和自定资源

___

1. 将图标添加到程序  
   - 资源描述档
     > 扩展名为.RC的文件，有时也称作资源定义文件

     该文件列出了程序的所有资源和一个让程序引用资源的表头文件（RESOURCE.H）  

     资源编译器RC.EXE将文字资源描述文件转化为扩展名.RES的二进制文件。然后，已编译的资源文件随同.OBJ和.LIB文件一起在LINK步骤中被指定连结  
2. 取得图标句柄  
   `LoadIcon(hInstance, MAKEINTRESOURCE(IDI_ICON));`  

   在Windows文件的某些部分，`LoadIcon`被称为「过时的」，并推荐使用`LoadImage`。`LoadImage`更为灵活，但开销也更大

   标识符也可以是字符串
3. 使用图标  
   - 在注册窗口类时指定图标  
     `wc.hIcon =`  
     `WNDCLASSEX`相对于`WNDCLASS`多了两个字段：`cbsize`指明结构体大小，`hIconSm`指明小图标的句柄
   - 调用`SetClassLong`动态更改图标  
     `SetClassLong(hwnd, GCL_HICON, LoadIcon(hInstance, MAKEINTRESOURCE(IDI_ALTICON))) ;`

4. 使用自定义光标
   - 在注册窗口类时指定  
     `wc.hCursor =`  
   - 调用`SetClassLong`动态更改光标  

   - 调用`SetCursor(hCursor)`动态改变光标
5. 字符串资源  
   `LoadString`  
   字符串资源主要是为了让程序转换成其它语言时更为方便  

6. 其他自定义资源  
   - 通过VS图形化工具添加
   - `hResource = LoadResource(hInstance, FindResource(hInstance, TEXT("BINTYPE"), MAKEINTRESOURCE(IDR_BINTYPE1)));`  
     返回指向资源内存区块的句柄，但`LoadResource`不会立即将资源加载到内存  
   - `LockResource(hResource)`  
     `FreeResource(hResource)`

### 菜单

一个菜单是一列可用的选项，告诉使用者一个应用程序能够执行哪些操作

1. 菜单概念  
   顶层菜单或弹出式菜单项可以被"启用"、"禁用"或"无效化"  
   Windows只为"启用"的菜单项向程序发送`WM_COMMAND`消息
2. 菜单结构  
   菜单中的每一项都有三个特性
   - 菜单中显示什么，可以是字符串或位图
   - `WM_COMMAND`消息中Windows发送给程序的菜单ID
   - 菜单项的属性，包括是否被禁用、无效化或被选中  
3. 定义菜单  
   通过VS可视化工具添加  
4. 在程序中引用菜单  
   - 在窗口类中指定  
     `wndclass.lpszMenuName = MAKEINTRESOURCE(ID_MENU);`  
   - 创建窗口时指定  
     `hMenu = LoadMenu(hInstance, MAKEINTRESOURCE (ID_MENU));`  
     将`hMenu`作为`CreateWindow`的第九个参数

     `CreateWindow`指定的菜单将覆盖窗口类中指定的  
   - 函数调用  
     `SetMenu(hwnd, hMenu);`  
5. 菜单和消息  
   选择一个菜单项时，Windows通常向窗口消息处理程序发送几个不同的消息。大多数情况下， 可以忽略大部分消息  
   - `WM_INITMENU`  
     `wParam`： 主菜单句柄  
     `lParam`： 0  
   - `WM_MENUSELECT`
     > 在菜单项中移动光标或者鼠标

     `LOWORD(wParam)`：被选中项目的菜单ID或者弹出式菜单句柄  
     `HIWORD(wParam)`：选择flag，可以是以下flag的组合`MF_GRAYED`、`MF_DISABLED`、`MF_CHECKED`、`MF_BITMAP`、`MF_POPUP`、`MF_HELP`、`MF_SYSMENU`和`MF_MOUSESELECT`  

     `lParam`: 包含被选中项目的菜单句柄
   - `WM_INITMENUPOPUP`  
     > Windows准备显示一个弹出式菜单

     `wParam`： 弹出式菜单句柄  
     `LOWORD(lParam)`：弹出式菜单索引  
     `HIWORD(lParam)`： 系统菜单为1，其它为0
   - `WM_COMMAND`  
     > 选中了一个被启用的菜单项  

     `WM_COMMAND`消息也可以由子窗口控件产生，两者区别如下

     ||菜单|控件|
     |-|-|-|
     |`LOWORD(wParam)`|菜单ID|控件ID   |
     |`HIWORD(wParam)`|0|通知码  |
     |`lParam`|0|子窗口句柄   |

   - `WM_MENUCHAR`
     > 按下Alt和一个与菜单项不匹配的字符时，或者在显示弹出式菜单时按下一个与弹出式菜单里的项目不匹配的字符键时  

     `LOWORD(wParam)`: 字符代码（ASCII或Unicode）  
     `HIWORD(wParam)`: 选择码。0:不显示弹出式菜单；`MF_POPUP`：显示弹出式菜单；`MF_SYSMENU`:显示系统弹出式菜单  
     `lParam`: 菜单句柄

## 十一、对话框

对话框分为两类："模态的"和"非模态的"  
显示模态对话框时，不能在对话框与同一个程序中的另一个窗口之间进行切换，使用者必须主动结束该对话框  
显示非模态对话框时，允许在对话框与建立对话框的窗口之间进行切换

### 模态对话框  

___

1. 建立About对话框  

2. 对话框及其模板  
   通过VS可视化工具添加
3. 对话框消息处理函数  
   `BOOL CALLBACK AboutDlgProc(HWND hDlg, UINT message,WPARAM wParam, LPARAM lParam)`  
   - 返回值为`BOOL`而非窗口消息处理函数的`LRESULT`  
   - 如果忽略某个消息，返回`FALSE`而非调用`DefWindowProc`
   - 一般不需要处理`WM_PAINT`和`WM_DESTROY`
   - 不会收到`WM_CREAT`，取而代之的是`WM_INITDIALOG`
   - 模态对话框的消息不通过程序的消息队列  
4. 激活对话框  
   `DialogBox(hInstance, TEXT ("AboutBox"), hwnd, AboutDlgProc) ;`
   - `TEXT ("AboutBox")`  
     资源描述文件中定义的对话框名称，也可通过`MAKEINTRESOURCE`将宏定义转为字符串传入

   对话框结束后，调用`EndDialog(hDlg, 0);`移交控制权，第二个参数将作为对话框的传回值
5. 资源描述语法  
   NFY
6. 对话框控件  
   对话框通过`SendMessage`与子窗口控件互相通信  
   - 一种简化的方法  
     `SendDlgItemMessage(hDlg, id, iMsg, wParam, lParam);`
     其等价于`SendMessage(GetDlgItem (hDlg, id), id, wParam, lParam);`

   对于子窗口句柄，通过如下语句获取  
   `hwndCtrl = GetDlgItem(hDlg, id);`  
   hDlg为对话框句柄，id为控件id
7. Tab停留和分组  
   使用Tab键存取的控件需要在其窗口样式中指定`WS_TABSTOP`  

   `hwndCtrl = GetNextDlgTabItem(hDlg, hwndCtrl, bPrevious);`  
   `hwndCtrl = GetNextDlgGroupItem(hDlg, hwndCtrl, bPrevious);`  
   获取前一个或后一个Tab键停留项
8. 自定义对话框控件  
   需要第九章内容 NFY  

### 非模态对话框

___
`hDlgModeless = CreateDialog(hInstance, szTemplate, hwndParent, DialogProc);`

1. 模态与非模态对话框区别  
   - 非模态通常包含标题列和系统菜单按钮
   - 非模态要么在STYLE叙述中包含`WS_VISIBLE`，要么调用`CreateDialog`后调用`ShowWindow`，否则窗口不会显示  
   - 非模态的消息会先经过程序的消息队列  
   - 非模态使用`DestroyWindow`结束对话框  
   NFY  

### 通用对话框  

1.

## 十三、使用打印机

### 打印入门

___

1. 打印和背景处理  
   - 打印流程  

     1. 首先使用`CreateDC`或`PrintDlg`来取得指向打印机设备内容的句柄
        > 使得打印机设备驱动程序动态链接库模块被加载到内存（如果还没有加载内存的话）并自己进行初始化

     2. 然后调用`StartDoc`函数
        > GDI模块会调用打印机设备驱动程序中的Control函数告诉设备驱动程序准备进行打印  

     3. 调用`StartPage`函数通知新的一页  

     4. 输出打印内容
        > GDI模块将打印操作存储在metafile中，打印机驱动程序可能会跳过该步骤

     5. 调用`EndPage`函数结束一页
        > 打印机驱动程序将metafile中的命令翻译成打印机输出数据

     6. 调用`EndDoc`函数结束打印  
        > 只有当没出错时，才调用EndDoc函数。如果其它打印函数中的某一个传回错误代码，那么GDI实际上已经中断了文件的打印  

   - 背景处理  
     从应用程序的角度来看，「打印」只发生在GDI模块将所有打印输出数据储存到磁盘文件中的时候，在这之后（如果打印是由第二个线程来操作的，甚至可以在这之前）应用程序可以自由地进行其它操作。真正的文件打印操作成了后台打印程序的任务，而不是应用程序的任务  

2. 打印机设备上下文  

   ```c++
   EnumPrinters(PRINTER_ENUM_LOCAL, NULL, 4, 
               NULL, 0, &dwNeeded, &dwReturned);
   pinfo4 = malloc(dwNeeded);
   EnumPrinters(PRINTER_ENUM_LOCAL, NULL, 4, 
               (PBYTE) pinfo4, dwNeeded, &dwNeeded, &dwReturned);
   ```

   > 获取可用打印机  

   调用两次，第一次获得所需结构体大小，第二次填入结构体

   `hdc = CreateDC(NULL, szDeviceName, NULL, pInitializationData);`  

   `szDeviceName`指向一个字符串，以告诉Windows打印机设备的名称  
   `pInitializationData`一般设为NULL

### 打印图像和文字

## 动态链接库  

### 动态链接库的基本知识

动态链接库通常并不能直接执行，也不接收消息。它们是一些独立的文件，其中包含能被程序或其它DLL呼叫来完成一定作业的函数

动态链接库的目的之一就是提供能被许多不同的应用程序所使用的函数和资源。有些动态链接库（如字体文件等）被称为「纯资源」。它们只包含数据（通常是资源的形式）而不包含程序代码。

动态链接库的标准扩展名为dll，Windows能自动加载。其他扩展名必须使用`LoadLibrary`或`LoadLibraryEx`加载  

Windows为所有的动态链接库模块提供「引用计数」，当引用计数为零时，Windows将从内存中把链接库删除掉，因为不再需要它了  

1. 其他链接库  
   > VS中添加lib的三种方式
   > 1. 项目设置
   > 2. `#pragma comment(lib,"YOUR.lib")`
   > 3. 将lib添加到项目文件列表中
   - 静态链接库  
     扩展名为.lib的文件。其代码会「**静态链接**」到exe文件中。本质是多个obj文件打包
   - 引用链接库  
     另一种特殊的静态链接库。其本身不包含程序代码，而是为链接程序提供动态链接库的索引信息  

     若没有引用链接库，使用DLL时必须通过`GetProcAddress`先获取函数地址然后才能调用

2. 链接库入口/出口点  
   `int WINAPI DllMain(HINSTANCE hInsDLL, DWORD fdwReason, PVOID pvReserved)；`  

   当系统启动或终止进程或线程时，它将使用进程的第一个线程为每个加载的 DLL 调用入口点函数  

   `fdwReason`可以是以下四种取值  
   1. `DLL_PROCESS_ATTACH`  
      - 启动进程或调用 `LoadLibrary`，DLL 正在加载到当前进程的虚拟地址空间中  
      - pvReserved` 参数指示 DLL 是静态加载还是动态加载
   2. `DLL_PROCESS_DETACH`  
      - DLL 正在从调用进程的虚拟地址空间中卸载  
      - `lpvReserved` 参数指示是否由于 `FreeLibrary` 调用、加载失败或进程终止而卸载 DLL
   3. `DLL_THREAD_ATTACH`  
      - 当前进程正在创建新线程
   4. `DLL_THREAD_DETACH`  
      - 线程正在完全退出  

3. 在DLL中共享内存  
   多个程序能够共享一个动态链接库中相同的**程序代码**。但是，DLL为每个程序所储存的**数据**都不同。若希望多个程序共享数据，需要使用共享内存。  

   - ```c++
     #pragma     data_seg("shared")     
     ...   
     #pragma     data_seg()
     ```  

     `#pragma data_seg("shared")`建立数据段并命名为shared，`#pragma data_seg()`标识数据段结束  

     对于其中的变量必须初始化，否则将被存储在普通的未初始化数据段中而不是shared中  
   - `#pragma comment(linker,"/SECTION:shared,RWS")`  
     指定链接选项，使得该数据段对链接器可见。RWS表示该段可读、写和共享
     > 也可在VS项目属性中设置`/SECTION:shared,RWS`

### 各种DLL讨论

___

1. 纯资源链接库  
   没有输出函数，故没有lib文件生成  
   主调程序需要使用`LoadLibrary`加载DLL，然后使用`LoadBitmap`、`LoadIcon`等函数加载资源
   
## Windows API

NFY

### .NET

### Win32

### COM

引入了 ABI 以及许多新的机制

### Windows Runtime

<https://www.cnblogs.com/hez2010/p/18026102/intro-to-winrt-abi>

#### C++/CX
>
> deprecated
>
#### WRL, Windows Runtime Library
>
> deprecated
>
#### C++/WinRT

对于自定义类型，使用投影类型时必须显式指明原类型的命名空间  
集合绑定，在 xaml 中绑定元素属性时必须加上 `Mode`

[pic1]:data:image/png;base64,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
[pic2]:data:image/png;base64,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
