### 具体实现

1. `plotui/cpsui.c/DefCommonUIFunc`
   > 增加新属性页PROPSHEETPAGE的定义  

   - 定义具体同windows编程-属性页  
   - 调用`pfnComPropSheet`，注册该属性页
2. `plotter/enable.c/DrvEnablePDEV`
   - 在该函数中设定设备参数，如版本信息，dpi，物理曲面的分辨率等；  
   - 定义调色板
3. `plotter/enable.c/DrvEnableSurface`
   - 在该函数中计算根据纸张大小（单位point，定义为1/10mm）计算逻辑曲面的大小（单位为dot）  
   - 将`EngCreateDeviceSurface`替换为`EngCreateBitmap`，曲面由GDI维护

4. `plotter/page.c/DrvStartPage`  
   在该函数中将曲面设为白色，默认黑色会与默认文字颜色冲突

5. `plotter/page.c/DrvSendPage`  
   执行到函数时需要打印的内容已全部写入曲面SURFOBJ，故在该函数中读取曲面数据写入bmp文件  
6. `plotter/mybitmap.h`  `plotter/mybitmap.c`  
   自定义文件，仿C++类，由`mybitmap`结构体维护位图信息、转换灰度图以及写入到文件

### 注意点

1. 单位换算  
   注意某些函数的参数的单位，大小不对应会调用失败，如`EngEraseSurface`  
   纸张大小的单位是1/10毫米，屏幕尺寸单位为像素，两者之间换算公式为

   $screenSize = \frac{paparSize}{10*25.4}*dpi$

2. 每行字节数  
   bmp为加快读取速度，每行的字节数必定是4的倍数，不足补0，计算公式如下:  

   $rowBytes = [\frac{BitsPerPixel*width+31}{32}]*4$  
   []表示向下取整
3. RGB to gray  
   RGB值和灰度的转换，公式如下：  
   $Gray = 0.299*R + 0.587*G + 0.114*B$

4. 运算符优先级  
   `?:`优先级低于`+`，如果在表达式中使用`?:`务必记得加括号
