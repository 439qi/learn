<!-- 2023.08.30 created -->
<!-- 2023.09.07 modified: error corrected -->
<!-- 2023.09.11 modified: ui plug-ins and rendering-plug-ins added -->

# Pscript 架构

---

## Pscript 微型驱动

1. PPD 文件（PostScript Printer Description file）  
   以文本格式描述 PostScript 打印机的特征  
   Pscript 会读取该文件转换成二进制格式的.bpd 文件

   - TrueGray 功能

     > ADTrueGray

     检查文本和矢量图形的 RGB 颜色，对于 R=G=B 即灰色，将其从 RGB 颜色空间转换为打印机的灰色颜色空间

2. NTF 文件  
   以二进制格式描述设备支持的字体
3. INF 文件  
   标识安装微型驱动的信息  
   需要包含数据段和安装段

## Pscript 用户界面

- 通用的 UI 代码

  > ps5ui.dll

  使用 CPSUI 创建属性表页，用以约束用户可自定义的选项

  1. 打印机属性表
     打印机窗口的属性 Printer Properties
  2. 文档属性表的"布局"、"纸张/质量"、"高级"页
     - 打印文档窗口的首选项 Preferences
     - 程序调用`PrinterProterties`或`DocumentProperties`函数

- 自定义 UI 插件
  - `IPrintOemUI`接口  
    由插件实现，打印机接口 DLL 调用  
     必须定义所有接口方法，对于不需要的方法，返回`E_NOTIMPL`表明方法未实现
  - `IPrintOemDriverUI`接口  
    打印机接口 DLL 的父接口，对插件提供特定操作  
     在`IPrintOemUI`的`PublishDriverInterface`方法中可以获取打印机 DLL 的该接口指针
  - `DEVMODE`结构体  
    可以向`DEVMODE`结构体增加私有成员，被增加的成员必须修饰`DMEXTRAHEADER`修饰，且增加成员后必须实现`IPrintOemUI::DevMode`方法

## Pscript 渲染器

- 通用渲染代码

  > Pscript5.dll

  通过打印机图形 DLL 实现，导出 DDI 函数

- 自定义渲染插件
  - `IPrintOemPS`接口  
    由插件实现，打印机图形 DLL 调用  
     必须定义所有接口方法，对于不需要的方法，返回`E_NOTIMPL`表明方法未实现
  - `IPrintOemDriverPS`接口  
    打印机图形 DLL 的父接口，对插件提供特定操作  
     在`IPrintOemPS`的`PublishDriverInterface`方法中可以获取打印机 DLL 的该接口指针
  - `PDEV`结构体  
    `PDEV`完全由驱动程序自定义  
    若自定义`PDEV`，则需要实现`IPrintOemPS::EnablePDEV`、`IPrintOemPS::DisablePDEV`、`IPrintOemPS::ResetPDEV`

## hostfont 注册表项

NFY

## Pscript 支持的转义

详见 [Microsoft learn/Pscript-supported escapes](https://learn.microsoft.com/en-us/windows-hardware/drivers/print/non-com-based-rendering-plug-ins)

# Pscript 渲染插件实现-微软示例代码

---

## 一些微软的宏定义

1. `UNREFERENCED_PARAMETER`

   > 避免编译器对未引用参数的警告

   一般编译公开项目会使用/W4 的警告等级，该等级会对未使用的变量进行警告，但有些项目中为保证 API 的通用性会有冗余参数，该宏定义可避免警告

## `oemps/oemps.h`

`OEMPDEV`的定义，自定义的 `PDEV`

该示例中仅用来保存 Pscript5.dll 的 DDI 函数的指针

## `oemps/intrface.h` `oemps/intrface.cpp`

基于 COM 的自定义 Pscript 渲染插件的接口，

1. `HRESULT __stdcall IOemPS::QueryInterface(const IID& iid, void** ppv)`  
   COM 接口相关，根据 iid 转型成对应的接口
2. `AddRef` 、`Release`  
   COM 接口相关，引用计数
3. `HRESULT __stdcall IOemPS::GetInfo ( DWORD dwMode, PVOID pBuffer, DWORD cbSize, PDWORD pcbNeeded)`  
   根据`dwMode`返回对应 dll 信息，由`pBuffer`带回
4. `HRESULT __stdcall IOemPS::PublishDriverInterface( IUnknown *pIUnknown)`  
   插件通过该函数获取 Pscript5.dll 或 Unidrv.dll 的`IPrintOemDriverUI`, `IPrintCoreUI2`, `IPrintCoreHelperPS`, 或`IPrintCoreHelperUni`接口  
   Pscript5.dll 会依次用其对应接口作为参数调用该函数

   该示例中用于获取了 Pscript5.dll 的`IPrintOemDriverPS`接口  
   该接口提供了`DrvGetSetting`以返回打印机信息和`DrvWriteSpoolBuf`以发送打印机数据到假脱机

5. `HRESULT __stdcall IOemPS::EnableDriver(DWORD dwDriverVersion, DWORD cbSize, PDRVENABLEDATA pded)`  
   初始化驱动  
   该示例中填充了`pded`的驱动版本信息，hook 函数指针数组的指针及其大小

6. `HRESULT __stdcall IOemPS::DisableDriver(VOID)`  
   释放了 `PublishDriverInterface` 中获取的接口
7. `HRESULT __stdcall IOemPS::EnablePDEV( PDEVOBJ pdevobj, __in PWSTR pPrinterName, ULONG cPatterns, HSURF *phsurfPatterns, ULONG cjGdiInfo, GDIINFO *pGdiInfo, ULONG cjDevInfo, DEVINFO *pDevInfo, DRVENABLEDATA *pded, OUT PDEVOEM *pDevOem)`

   由 Pscript5.dll 调用，为 oemps.h 中定义的`OEMPDEV`分配空间和初始化

   **`pded`并不等价先前`EnableDriver`中填充的`pded`，此时`pded`的`pdrvfn`中的函数指针指向的是 Pscript5.dll 中被 hook 的 DDI 函数**

   示例中保存了 Pscript5.dll 对应的 DDI 函数指针到自定义的`PDEV`中

8. `DevMode` `Command`  
   分别路由到 devmode.h/hrOEMDevMode 和 command.h/PSCommand

## `oemps/devmode.h` 、 `oemps/devmode.cpp`

`OEMDEV`的定义以及相关操作，自定义的`DEVMODE`

## `oemps/ddihook.cpp`

hook 函数的具体实现

该示例中什么也没有做只是回调了 Pscript5.dll 中的 DDI 函数（在`oemps/intrface.cpp/IOemPS::DisableDriver`中将其指针保存到了自定义的`OEMPDEV`中）

## `oemps/command.h` 、 `oemps/command.cpp`

1. `HRESULT PSCommand(PDEVOBJ pdevobj, DWORD dwIndex, PVOID pData, DWORD cbSize, IPrintOemDriverPS* pOEMHelp, PDWORD pdwReturn)`  
   Pscript5.dll 会在输出的特定时刻调用该函数，以便插入代码至最终的 PostScript 命令中

   `dwIndex`标明调用的时刻，其取值是一组`PSINJECT_`前缀的宏定义

   该示例中插入了一些注释信息

# Pscript UI 插件实现-微软示例代码

---

## `oemui/intrface.h` `oemui/intrface.cpp`

基于 COM 的自定义 Pscript UI 插件的接口

1. `HRESULT __stdcall IOemPS::QueryInterface(const IID& iid, void** ppv)`  
   COM 接口相关，根据 iid 转型成对应的接口
2. `AddRef` 、`Release`  
   COM 接口相关，引用计数
3. `STDMETHOD(GetInfo) (THIS_ DWORD dwMode, PVOID pBuffer, DWORD cbSize, DWORD pcbNeeded)`  
   基类`IPrintOemUI`的虚函数，根据 dwMode 返回 dll 相关信息

   该示例中可返回 dll 签名，版本

4. DevMode  
   基类`IPrintOemUI`的虚函数，执行对自定义`DEVMODE`操作

   该示例中路由到`oemui/devmode.cpp/hrOEMDevMode`

5. `HRESULT __stdcall IOemUI::CommonUIProp(DWORD dwMode, POEMCUIPPARAM pOemCUIPParam)`

   基类`IPrintOemUI`的虚函数，用于设置树形菜单选项  
   打印机接口 DLL 会调用该函数两次，可根据`pOEMUIParam->pOEMOptItems`是否为 NULL 来判断。第一次应通过设置`pOEMUIParam->cOEMOptItems`返回新添加的可选项数量，之后打印机接口 DLL 为`pOEMUIParam->pOEMOptItems`分配空间；第二次调用用于设置新添加的可选项

   该示例中路由到`oemui/oemui.cpp/hrOEMPropertyPage`

6. `HRESULT __stdcall IOemPS::PublishDriverInterface( IUnknown *pIUnknown)`

   插件通过该函数获取 Pscript5.dll 或 Unidrv.dll 的`IPrintOemDriverUI`, `IPrintCoreUI2`, `IPrintCoreHelperPS`, 或`IPrintCoreHelperUni`接口  
   Pscript5.dll 会依次用其对应接口作为参数调用该函数

   该示例中用于获取了 Pscript5.dll 的`IPrintCoreHelperPS`接口

7. `DocumentPropertySheets` `DevicePropertySheets`  
   文档属性表和设备属性表

   该示例中分别路由到`oemui/oemui.cpp/hrOEMDocumentPropertySheets`和`oemui/oemui.cpp/hrOEMDevicePropertySheets`

8. `HRESULT __stdcall IOemUI::DevQueryPrintEx( POEMUIOBJ poemuiobj, PDEVQUERYPRINT_INFO pDQPInfo, PDEVMODE pPublicDM, PVOID pOEMDM)`  
   基类`IPrintOemUI`的虚函数，用于确认某个打印任务是否可行  
   Pscript5.dll 的`DevQueryPrintEx`函数无法确定是否可行时，会调用所有关联的 UI 插件的`DevQueryPrintEx`，若均返回`S_OK`，则认为可行

   该示例中未实现，仅返回了`E_NOTIMPL`，表明方法未实现

9. `DeviceCapabilities`  
   UI 插件通过该函数指明自定义设备功能

   该示例未实现

10. `UpgradePrinter`  
    UI 插件通过该函数升级 upgrade 注册表中存储的该设备可选项的值

    该示例未实现

11. `PrinterEvent` `DriverEvent` `QueryColorProfile` `FontInstallerDlgProc` `UpdateExternalFonts`  
    该示例未实现

## `oemui/oemui.h` `oemui/oemui.cpp`

定义了`hrOEMPropertyPage` `hrOEMDocumentPropertySheets` `hrOEMDevicePropertySheets`，分别用于设置树形列表、文档属性表和设备属性表

## `oemui/globals.h` `oemui/globals.cpp`

该插件的全局实例句柄声明和定义，用于 oemui.cpp 中加载 UI 的资源

## `oemui/devmode.h` `oemui/devmode.cpp`

`OEMDEV`的定义和操作函数，自定义的`DEVMODE`结构体
