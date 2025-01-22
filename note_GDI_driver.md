<!-- 2023.08.25 created -->
<!-- 2023.09.11 modified: spooling added;error corrected -->
<!-- 2023.09.11 modified: add more details about render flow -->
<!-- 2023.09.15 modified: add more details about UI dll -->
# GDI 驱动
>
> 图形驱动程序设计指南

## GDI 打印机驱动程序架构

驱动程序的安装最终结果其实就是对系统的两个永久性更改：

1. 描述驱动程序的系统注册表项，包括驱动程序的卸载顺序以及任何正确的配置资料
2. 驱动程序文件被复制到一个合适的系统目录中

### GDI 与图像驱动程序

1. 表面 Surface  
   DDI 使用浮点数坐标，使用 32 位表示，高 28 位为整数部分，低四位为小数部分

   表面分为 GDI 管理的表面，设备管理的标准格式表面，设备管理的非标准表面

   - GDI 管理的表面  
     由 GDI 创建和管理（`EngCreateBitmap`）
     使用设备无关位图 DIB(Device Independent Bitmap)格式，单平面 plane，每个像素的数据连续存储，每行按 4 字节对齐
   - 设备管理的标准格式表面  
     由设备创建和管理（`DrvCreateDeviceBitmap`）  
     对于不透明的设备管理表面，GDI 无法知晓其位图格式信息，必须实现`DrvBitBlt`和`DrvTextOut`和`DrvStockePath`  
     透明的设备管理表面类似于 GDI 管理的表面  
     可以`EngModifySurface`将不透明表面转换到透明表面
   - 设备管理的非标准表面  
     调用`EngCreateDeviceSurface`创建并返回句柄  
     使用设备相关位图 DDB(Device Dependent Bitmap)  
     ???

1. 位图  
   GDI 使用设备无关位图 DIB(Device Independent Bitmap)管理位图，若设备使用非标准格式帧缓冲区，则驱动程序自身必须实现所有要求的绘图函数  
   设备管理的位图或表面，必须实现 `DrvCopyBits`，和位图之间进行块传输
1. 通信  
   驱动程序对于 GDI 只导出 export`DrvEnableDriver`，其余 DDI 函数在调用`DrvEnableDriver`时通过参数的函数指针数组带回
1. 字体
1. 绘制文本

### GDI 与打印

GDI 与打印相关的功能主要有三个方面

1. 为应用程序提供打印 API 函数

   应用程序只需要调用标准 API 函数就可以完成打印任务而无需关心具体的打印机设备，实现了设备无关性

2. 控制打印机工作

   DDI 是 GDI 为所有打印机驱动程序规定的共同的标准。打印机驱动程序实现相应的 DDI 函数，打印时 GDI 调用相应的 DDI 函数

3. 补充打印机的绘制功能

   若打印驱动程序未实现 GDI API 对应的 DDI 函数，则 GDI 会尝试模拟实现该功能

   > 例如打印机不支持贝塞尔曲线，则 GDI 会将贝塞尔曲线分解成一系列直线

### 打印机接口 DLL

- 打印机接口 DLL 主要职责

  1. 为打印机的配置选项提供用户界面
  2. 实现可被后台处理程序调用的函数，以通知驱动程序打印相关的系统事件
     > 如驱动安装、升级，打印机添加、连接等

- 打印机属性表页
  - 实现`DrvDevicePropertySheets`函数来创建特定于打印机的属性表页  
    打开打印机属性页时也会调用`DrvDocumentPropertySheets`???  

  - 实现`DrvDocumentPropertySheets`函数来创建文档特定的属性表页
    该函数会被多次调用。

    - 当参数`pPSUIInfo`为`NULL`时  
      spooler 直接调用该函数，此时参数`lParam`为`DOCUMENTPROPERTYHEADER`结构体指针，若其`fMode`成员为 0，仅需要返回自定义 DEVMODE 的字节数，否则根据`fMode`执行相应操作  
      其中，当`fMode`为`DM_IN_BUFFER`或`DM_MODIFY`时（在属性页显示之前），需要将`fdmIn`指向的`DEVMODE`的内容复制到自定义的 DEVMODE 结构体中；当为`DM_OUT_BUFFER`或`DM_COPY`时（在属性页显示之后），需要将自定义的 DEVMODE 结构体中的内容拷贝到`fdmOut`指向的`DEVMODE`中

    - 当参数`pPSUIInfo`不为`NULL`时  
      spooler 通过 CPSUI 间接调用该函数，通过参数中`pPSUIInfo`的 `Reason` 成员以及`DOCUMENTPROPERTYHEADER`结构体的`fMode`成员决定要执行的操作
      > 当`Reason`为`PROPSHEETUI_REASON_INIT`时，`lparam`为`DOCUMENTPROPERTYHEADER`的指针，其余情况可从`PROPSHEETUI_INFO`的`lParamInit`成员取得指针

  有两种设计方法

  - CPSUI（Common Property Sheet User Interface, 通用属性表单用户接口）

    对于 CPSUI 方法，两个属性表页函数中调用 CPSUI 的`ComPropSheet`函数(通常使用`CPSFUNC_ADD_PCOMPROPSHEETUI`函数代码)来为 CPSUI 提供属性表页说明和页面事件回调

  - 通过窗口生成函数创建非标准的属性页面

- 打印机的颜色功能  
  系统调用`DrvDeviceCapabilities`，并传递`DC_COLORDEVICE`参数以获取打印机颜色功能。对于彩色设备，返回 1；对于单色和灰阶设备，返回 0

### 打印机图形 DLL

- 打印机图形 DLL 主要职责

  1. 协助 GDI 绘制
  2. 将处理后的数据流传输到假脱机(Spooler)

- 渲染打印作业流程

  1. **打印前**调用`CreateDC`后 GDI 与 DLL 交互

     1. GDI 加载 DLL 并调用`DrvEnableDriver`,该函数返回包含驱动程序实现的所有 DDI 函数的函数指针数组
     2. GDI 调用 DLL 的`DrvEnablePDEV`，驱动程序创建物理设备实例并返回设备特征  
        参数的`DEVMODE`包含了驱动程序 UI 接口和渲染接口共用的数据：驱动程序名称，纸的大小，朝向，dpi 等  

        驱动程序通过多个结构将有关硬件设备功能的信息返回到 GDI
        - 填充`GDIINFO`，`GDIINFO`结构体用于向GDI描述设备的图形能力：设备基本技术是光栅 or 矢量、页面大小和分辨率、调色板和灰度信息、字体和文本能力等。GDI 调用`DrvEnablePDEV`前会用 0 填充该结构
        - 填充`DEVINFO`结构，`DEVINFO`通过图形能力的位标记向图像引擎描述驱动程序和物理设备。其中也存储了逻辑字体定义。GDI 调用`DrvEnablePDEV`前会用 0 填充该结构
        - 分配 PDEV 的空间，初始化并返回指针
          > PDEV 结构体是一个通用术语，由打印机驱动程序和渲染插件自定义
     3. GDI 调用 DLL 的`DrvCompletePDEV`，为设备实例提供 GDI 句柄  
         GDI 通过参数传递一个`PDEV`的句柄
     4. GDI 调用 DLL 的`DrvEnableSurface`,设置要绘制的图面并与物理设备实例关联  
        每个 PDEV 只能启用一个主表面

        4.1. 若图面由 GDI 管理，驱动程序调用`EngCreateBitmap`；若图面由设备管理，驱动程序调用`EngCreateDeviceSurface`

        4.2. 可选）程序程序调用`EngMarkBandingSurface`通知 GDI 使用 Banding

        4.3. 驱动程序调用`EngAssociateSurface`，通知 GDI 将图面与物理设备实例关联以及驱动支持的绘图函数

  2. **打印中**

     > 若 DLL 提供的 DDI 函数执行可能超过 5 秒，应至少每 5 秒调用一次`EngCheckAbort`检查打印作业是否被终止

     1. GDI 调用 DLL 的`DrvStartDoc`
        传入曲面指针，打印的文件名，作业 id
     2. GDI 调用 DLL 的`DrvStartPage`
     3. (可选)调用`ResetDC`后 GDI 与 DLL 交互

        3.1. GDI 为新的 Context 调用 DLL 的`DrvEnablePDEV`  
        3.2. GDI 调用 DLL 的`DrvResetPDEV`， DLL 使用旧的 Context 信息更新新的 Context  
        3.3. GDI 为旧的 Context 调用 DLL 的`DrvDisableSurface`和`DrvDisablePDEV`  
        3.4. GDI 为新的 Context 调用 DLL 的`DrvEnableSurface`  
        3.5. GDI 调用`DrvStartDoc`开始打印

     4. 输出
        - 文本
          - `DrvGetGlyphMode`  
             传入`PDEV`和`FONTOBJ`决定如何缓存字形 glyph 信息：GDI 存储、设备存储、GDI 存储`PATHOBJ`  
            对于连续的相同字体的文本视为一组，对每组执行以下流程
          - `DrvTextOut`

            - `FONTOBJ_pifi`  
              将`FONTOBJ`与`IFIMETRICS`字形矩阵关联
            - `DrvBitBlt`  
              输出背景色

            将单个字符转换为字形(位图 BITMAP 或图形组合 PATHOBJ)输出
        - 图像
          - `DrvStretchBlt`  
            会将图片分数次调用`DrvStretchBlt`，具体划分原则暂时未知
     5. GDI 调用 DLL 的`DrvSendPage`
     6. GDI 调用 DLL 的`DrvEndDoc`指示文档已完全渲染

  3. **打印后**

     1. GDI 调用 DLL 的`DrvDisableSurface`
        > 若调用过`EngCreateBitmap`，则必须在`DrvDisableSurface`中调用`EngDeleteSurface`
     2. 调用`DeleteDC`后 GDI 调用 DLL 的`DrvDisablePDEV`

  4. **卸载打印机 DLL 前**GDI 调用 DLL 的`DrvDisableDriver`

- 设备字体  
  若打印机提供设备字体，DLL 必须定义`DrvTextOut`函数以生成文本输出命令。DLL 还必须定义以下函数：
  - `DrvQueryAdvanceWidths`
  - `DrvQueryFont`
  - `DrvQueryFontData`
  - `DrvQueryFontTree`
- 返回打印机特定信息  
  GDI 有时会调用`DrvQuery`前缀的 DDI 函数，请求打印机图形 DLL 在打印作业之间返回打印机特定的信息

- 用户模式或内核模式
  NOT FINISHED

- 生成 DLL 的规则  
  |用户模式图形 DLL|内核模式图形 DLL|
  |-|-|
  |在源文件中设置 `TARGETTYPE=DYNLINK`|在源文件中设置 `TARGETTYPE=GDI_DRIVER`|  
  |在包含 `winddi.h` 之前，必须在源文件中定义预处理器宏`USERMODE_DRIVER`| 不得定义预处理器宏`USERMODE_DRIVER`|
  |对象模块必须与`umpdddi.lib`和`gdi32.lib`导入库链接|对象模块必须与`win32k.lib`导入库链接。
  |对于`DRVQUERY_USERMODE`， `DrvQueryDriverInfo`函数必须返回`TRUE` |`DrvQueryDriverInfo`函数必须为`DRVQUERY_USERMODE`返回`FALSE`。 (也可以省略函数。)|

### 数据格式

- 增强型元文件 EMF(Enhanced Metafile)  
  由调用 GDI 函数的指令构成。打印处理器对于 EMF 文件，调用相应 GDI 函数绘制可打印图像  
  EMF 比 RAW 类型能更快速发送到打印服务器
- RAW  
  由 PCL 命令构成  
  PostScript 可以被当作 RAW 数据
- TEXT
  由 ANSI 文本构成。打印机处理器调用 GDI 绘制字母，并发送 RAW 格式的结果到假脱机  
  相当于用写字板打开输入文件然后打印文件

### 打印假脱机体系

假脱机技术当主机处理器给外部设备传送数据时为了减少占用主机处理器的时间，把外存当作端口的缓冲存储器，具体的发送工作在后台处理

- 意义

  1. 避免了长时间占用 CPU，使控制权迅速返回给用户以便进行其它操作
  2. 便于多路由输出
  3. 将底层有关端口的操作以独立部件实现

- Windows 打印假脱机组件
  1. Winspool.drv  
     假脱机的客户端接口，假脱机的 Win32 API 由其导出的函数构成。查询打印机、改变打印机设置等
  2. Spoolsv.exe  
     假脱机的 API 服务器。该模块实现一些 API 函数，多数函数调用通过路由的方式通知到打印提供者。
  3. Spoolss.dll  
     假脱机的路由器。基于打印机的名称或每一个函数调用提供的句柄，将函数调用传递到正确的打印提供者
  4. Print Provider  
     核心组件  
     负责将打印作业直接送到本地或远程的打印设备，以及打印队列管理

### 打印处理器

由本地打印提供者 Print Provider 调用

负责转换打印作业的[假脱数据](#数据格式)到[打印监视器](#打印监视器)的格式，以及负责处理应用程序对暂停、重新开始及撤消打印作业等的请求

打印作业的假脱机数据包含在一个假脱机文件中，打印处理器读取这些文件，在数据流上执行转换操作，并将转换的数据写到假脱机，假脱机然后发送数据流到合适的打印监视器

### 打印监视器

详见[打印监视器](note_GDI_print_monitor.md)  
打印监视器负责从打印假脱机传送打印机数据流到一个合适的端口驱动程序  
包含语言监视器和端口监视器

- 语言监视器 language monitors  

  语言监示器是一些用户模式的 DLL，主要用于支持打印机的双向通信、监视打印机状态、获取并处理一些事件

  - 主要功能
    1. 在打印假脱机与双向打印机之间提供一个全双工的通信通道，从而具有提供软件可存取的状态信息的能力
    2. 增加打印机控制信息到数据流，如由打印机作业语言定义的命令等

- 端口监视器  
  由用户模式的一些 DLL 组成。它们负责在用户模式的打印假脱机及访问 I/O 硬件端口的内核模式的端口驱动程序之间提供通讯

## GDI 打印机驱动程序实现

### 具体实现

1. `plotui/cpsui.c/DefCommonUIFunc`

    - 增加新属性页 PROPSHEETPAGE 的定义，定义具体同 windows 编程-属性页
    - 调用`pfnComPropSheet`，注册该属性页

2. `plotter/enable.c/DrvEnablePDEV`

    - 填充`DEVMODE`，`DEVMODE`包含了驱动程序 UI 接口和渲染接口共用的数据：驱动程序名称，纸的大小，朝向，dpi 等
    - 填充`GDIINFO`，`GDIINFO`描述了设备的图形能力：页面大小和分辨率、字体和文本功能等。GDI 调用`DrvEnablePDEV`前会用 0 填充该结构
    - 填充`DEVINFO`结构，`DEVINFO`用于图像引擎描述驱动程序和物理设备。GDI 调用`DrvEnablePDEV`前会用 0 填充该结构
    - 分配 PDEV 的空间，初始化并返回指针
      > PDEV 结构体是一个通用术语，由打印机驱动程序和渲染插件自定义

3. `plotter/enable.c/DrvEnableSurface`

    - 在该函数中计算根据纸张大小（单位 point，定义为 1/10mm）计算逻辑曲面的大小（单位为 dot）  
      $dot = \frac{point}{10*25.4}*dpi$

4. `plotter/page.c/DrvStartPage`  
    该函数中实现纸张的初始化

5. `plotter/page.c/DrvSendPage`  
    执行到函数时需要打印的内容已全部写入曲面 SURFOBJ

### 注意点

1. 单位换算  
   注意某些函数的参数的单位，大小不对应会调用失败，如`EngEraseSurface`  
   纸张大小的单位是 1/10 毫米，屏幕尺寸单位为像素，两者之间换算公式为

   $screenSize = \frac{paparSize}{10*25.4}*dpi$

2. 每行字节数  
   bmp 为加快读取速度，每行的字节数必定是 4 的倍数，不足补 0，计算公式如下:

   $rowBytes = [\frac{BitsPerPixel*width+31}{32}]*4$  
   []表示向下取整

3. RGB to gray  
   RGB 值和灰度的转换，公式如下：  
   $Gray = 0.299*R + 0.587*G + 0.114*B$

4. 运算符优先级  
   `?:`优先级低于`+`，如果在表达式中使用`?:`务必记得加括号
