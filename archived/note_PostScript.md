<!-- 2023.08.26 created -->
<!-- 2023.09.04 modified: 1.change command declaration grammer <cmd><params> -> <params><cmd><rets>  2. add more control commands -->
<!-- 2023.09.05 modified: add more about PostScript file structure and excution environment -->
<!-- 2023.09.06 modified: almost done, a little bit more details required -->
# PostScript

> Adobe/PostScript Language Reference 3rd edition.pdf  

PostScript 是一种用于描述**矢量图形**的页面描述语言

## 目的

---

为了保证电子页面在传播过程中不因具体传播环境的不同而产生种种问题

> 电子页面是指由计算机生成的虚拟页面

## PostScript 文件

---

### PostScript 解释器

解释执行，边读边执行  
PostScript 程序的顺序解释执行包括扫描和程序执行两个过程，扫描由扫描程序完成，执行由 PostScript 解释器完成

### EPS(Encapsulated PostScript)

- 文件独立性  
  eps 格式的文件是一种最终文件形式，不能对文件本身做任何修改
- 封装性  
  执行 eps 文件后，不会对解释器产生任何副作用

### 文本结构约定 DSC(Document Structure Conventions)

关于 PostScript 文件的页面结构和所需资源等额外信息的一套**注释**约定

- 文本管理器 DM(Document Manager)  
  DSC 将被 DM 处理，若没有 DM，将被当作一般注释  
  DM 负责程序执行中的资源管理与异常处理，提高 PS 解释器的执行效率
- DSC 语法  
  以`%%`开始，操作符在前，操作数在后  
  详见`Adobe/PostScript Language Document Structuring Conventions Specification`

### PostScript 文件构成

- 序言部分
  1. 头段 Header Section  
     必须使用 DSC，说明所依赖的系统环境
  2. 默认段 Defaults Section  
     包含在多个页面中重复出现的页面级注释
  3. 过程定义段 Procedures Section  
     包括一系列过程的定义
- 描述部分  
  相当于其它语言中的主程序部分
  1. 文档设置段 Document Setup Section  
     进行设备设置，包括选择物理介质，下载资源，设置图形状态参数
  2. 页面段 Pages Section  
     包括若干个页面内容的描述
  3. 文档尾段 Documen Trailer Section  
     进行一些后期处理 Post Processing

## PostScript 成像模型

---

### 构造路径

路径是一系列描述图像的线 line 和区域 area 的集合

路径构造结束后，当前页面上就存在一条当前路径，指定了当前页的哪些区域可以进行着色操作

当且仅当两个线段连续定义时才认为两个线段是相连的，不是连续定义的线段即使相交也不会被认为相连

### 着色操作

在着色操作过程中，各着色命令都将隐式地使用一些保存在图形状态中的图形参数

- 描边
- 填充  
  根据填充规则判断一个点是否属于路径内点，然后对内点构成的区域进行填充
  1. 非零环绕原则
     默认规则  
     路径为矢量，对任一点，向无穷远处引一条射线，当沿射线方向移动时，路径从左到右穿过射线时，环绕数加 1，从右到左穿过射线时，环绕数减 1，若最终环绕数为 0，则为外部点，否则为内部点
  2. 奇偶规则
     对任一点，向无穷远处引一条射线，计算其与路径的交点，偶数为外部点，奇数为内部点  

    ......

### 裁剪路径

规定可以进行着色操作的区域轮廓，缺省情况下为页面的整个可成像区域

### 图形状态

图形状态存储区就是用于存放图形状态参数的一块内存区

- 与设备无关的参数  
  用于页面上图形的描述控制

  1. 当前变换矩阵 CTM(Current Transformation Matrix)  
     用于将用户空间坐标系转换到设备空间坐标系
  2. 颜色空间 ColorSpace  
     决定了当前颜色空间模型及其特点。初始值为 DeviceGray
  3. 颜色 Color  
     根据当前颜色空间为着色命令提供相应的颜色。初始值为 0
  4. 当前点位置  
     当前点在用户空间中的坐标。没有初始值

  当前路径、当前裁剪路径、、裁剪路径栈、字库、线宽、线端类型、线的连接形状、斜接限制、虚线样式、笔画调整

- 设备相关的参数  
  用于控制还原处理过程细节。在安装时由相关设备决定，尽量不要改动

  色彩还原机制、叠印、黑版生成、底色去除、变换函数、半色调、平直度、光滑度、设备

- 图形状态的保存与恢复

  - 图形状态栈
    `gsave`将当前图形状态的拷贝入栈，`grestore`将栈顶的图形状态出栈
  - VM 的`gstate`对象

    > 需要 level 2

    通过`gstate`创建一个新的`gstate`对象，`currentgstate`将当前图形状态复制到`gstate`对象，`setgstate`恢复到原先图形状态

## 坐标系

---

### 用户空间

PostScript 程序使用的设备无关的独立坐标系

左下角为原点，x 轴向右增长，y 轴向上增长

基本单位为 point，定义为 $$1 point = \frac{1}{72} inch$$

### 设备空间

输出设备在输出页面内容时使用的坐标系，不同设备不尽相同  
通过图形状态的 CTM 由用户空间转换到设备空间

### 其他坐标空间

字符空间、图案空间等，由系统隐式转换到用户空坐标系

### 坐标变换

变换矩阵为右乘矩阵

$$
\begin{bmatrix}
a&b&0\\
c&d&0\\
{t_{x}}&{t_{y}}&1\\
\end{bmatrix}
$$

PostScript 中表示该矩阵时省略第三列，表示为

$$
\begin{bmatrix}
a&b&c&d&{t_{x}}&{t_{y}}\\
\end{bmatrix}
$$

当对用户空间进行变换后，将变换矩阵与 CTM 复合即得到新 CTM  
$${CTM_{new}} = matrix * {CTM_{old}}$$

坐标变换的一般次序是先平移/缩放、再旋转  
坐标变换只影响后续的相关命令，已经完成的路径、图形等不会受影响

### 用户空间的保存与恢复

- `gsave`、`grestore`
- `currentmatrix`、`setmatrix`

## 图像

---

level 1 仅支持灰度图像，且图像源数据只能来自过程  
level 2 支持了彩色和任意颜色空间，图像源数据可以来自文件或字符串
level 3 支持了二值图像蒙版和颜色键蒙版

### 图像表示

图像的源格式可由宽 width、高 height、颜色分量 components、每分量的比特数 bits/component 来描述

图像源数据的字节流使用大端序，且每行字节数为 8 的倍数，不足补 0

对于多分量的图像，其源数据可来自单一数据源，或多数据源，即不同分量来自不同数据源

### 图像坐标系

PostScript 假设图像按行组织，原点在左下角，x 轴向右，y 轴向上  
对于其他坐标系，由一个特殊矩阵将其变换到惯例坐标系。该矩阵使用平移，旋转，反射，缩放将用户空间坐标系(0,0)到(1,1)的矩形变换到到图像坐标系的图像边界

### 图像插值

图像分辨率小于给定分辨率时，直接缩放会产生伪影，需根据插值算法在图像锐利部分生成平滑过渡  
插值默认关闭，需要设置图像字典的`Interpolate`条目

### 蒙版图像

- 模板蒙版 stencil masking  
  1 允许绘制，0 不允许绘制  
  蒙版的颜色分量数始终为 1，每分量的比特数始终为 1
- 显式蒙版 explicit masking  
  显式蒙版不要求与图像分辨率一致，但交集不能为空？？？
- 颜色键蒙版 color key masking  
  通过设定 type 4 的图像字典的`MaskColor`的条目，指定透明色

## Form

对重复出现的图形、图像和文字等内容进行描述后存放在 Form 资源中，以便需要时快速显示

PostScript 解释器会将 Form 的图像输出保存在缓存中，当再次使用该 Form 时，解释器将 Form 的定义替换为保存的输出结果

Form 通过 Form 字典定义

## PostScript 执行环境

PostScript 使用的内存分为四块

### 1. 栈 Stack

PostScript 使用栈来存储 PostScript 对象

对于需要参数的命令，会从栈中取数据

> **注意**：多参数的命令，参数顺序即入栈顺序，与出栈顺序相反

栈又细分为操作数栈、字典栈、执行栈、图形状态栈、裁剪路径栈

### 2. 虚存

用于存放复合对象**值**的一块内存空间

`save` `restore`用于存储局部虚存和恢复局部虚存，常用于：

- 保持页与页之间的描述独立性
- 封装插入的外部图
- 显式回收虚存空间

### 3. 标准输入输出文件

PostScript 程序与用户终端或另一台计算机进行实时数据与信息交换时的通讯信道

标准输入输出文件是两个特殊的文件，它们作为 PostScript 程序的执行环境而始终存在，不需要在程序中人为地创建或关闭它们  
标准输入文件即当前解释执行的 PostScript 程序，标准输出文件是用于存放`print`或`=`的输出信息及错误信息和执行状态信息的一块内存区域

### 4. 图形状态

用于存放图形状态参数的一块内存空间

## PostScript 命名资源

---

资源是一组命名对象的集合，存储在虚拟内存或可随命令被加载进虚拟内存  
资源的键名通常为名字对象和字符串对象，也可使用其他类型作为键值，但不会尝试从外部加载

### 常规资源

`findresource`对于这些资源的返回值通常是其他运算符的操作数

- 字体 Font

### 隐式资源

隐式资源代表 PostScript 解释器的一些内置功能  
`findresource`对于这些资源的返回值是其键值  
不允许对隐式资源使用`defineresource`和`undefineresource`

1. 函数 Function

   > level 3

   函数的输入和输出值必须为数值对象

2. 过滤器 Filter

   > level 2/3

   一种特殊的文件，用于读写数据并对数据做变换

   数据源和数据目标可为文件、过程和字符串

   - 文件  
     关闭过滤器并不会关闭基础文件，除非显式使用`CloseSource`、`CloseTarget`
   - 过程  
     过程用作数据源时，当过滤器需要输入数据时会被调用。过程必须将字符串压入操作数栈  
     过程用作数据目标时，当过滤器处理输出数据时会被调用。调用前会将一个字符串和 boolean 压入操作数栈
   - 字符串

   PostScript 支持三种标准过滤器，每种包括编码过滤器和解码过滤器:

   - ASCII 编解码过滤器

   - 解压缩过滤器
   - 子文件过滤器

## PostScript 语法与命令

---

### 对象 object

1. 数据均被视为对象
   |简单对象 simple objects|复合对象 composite objects|
   |--|--|
   |boolean|array|
   |fontID|dictionary|
   |integer|file|
   |mark|gstate(level 2)|
   |name|packedarray(level 2)|
   |null|save|
   |operator|string|
   |real||

   简单对象为常量，其类型、属性和值绑定在一起且不可变

   复杂对象，其值和对象本身分开存储。复杂对象拷贝时，仅拷贝对象本身，原对象与新对象共享值

2. 属性  
   每个对象至少有一个属性

   - 字面属性和可执行属性不可同时拥有
   - 存取属性  
     只有复合对象才具有存取属性，用于限制对该对象值的操作  
     存取属性有四种：无限制、只读、只执行、空

### 基本语法

- 换行符作为语句的分隔符
- 注释以`%`开始，至换行符结束
- 所有字符均为 ANSI
- `/str`引入具有字面属性的名字对象
- []数组
- ()字符串
- <>十六进制字符串，每两位表示一位 ASCII 字符，若不足两位，末尾补 0
- <~~>based85 编码字符串，需要 level 2 和 level 3
- <<>>字典
  键可以是除 null 外所有 PostScript 对象，值可以是任意 PostScript 对象
- {...}
  定义过程，其等价于{}内的命令集合，相当于其他语言中的函数
- `base#num`  
  base 进制数，num

### 命令

1. 算术命令

   - `(n1,n2)add(ret)`
   - `(n1,n2)sub(ret)`
   - `(n1,n2)mul(ret)`
   - `(n1,n2)div(ret)`
   - `(n1,n2)idiv(ret)`
     整除
   - `(n1,n2)mod(ret)`
   - `(num)neg(ret)`

2. 流程控制

   1. 比较
      若成立则`true`入栈，否则`false`入栈

      - `(a,b)eq(bool)`
        a==b
      - `(a,b)ne(bool)`
        a!=b
      - `(a,b)gt(bool)`
        a\>b
      - `(a,b)lt(bool)`
        a\<b
      - `(a,b)ge(bool)`
        a\>=b
      - `(a,b)le(bool)`
        a\<=b

   2. 条件
      - `(flag，{...})if`
        若`flag`为`true`则执行{}程序块
      - `(flag,{...},{...})ifelse`
      - `(key)where(bool)`  
        在字典栈中查找 key，若找到，将字典对象和`true`入栈，否则`false`入栈
      - `(any)stopped(bool)`  
        执行 any 对象，若 any 正常执行完成，将 false 入栈；若 any 异常中止，将 true 入栈。
        类比于 try-catch
   3. 循环

      - `(begin,step,end,{...})for`  
        闭区间循环  
        每次循环之前将计数器当前值入栈  
        类比

        ```cpp
        for(double i = begin;i<=end;i+=step)
        {
          stack.push(i);
          ...
        }
        ```

      - `({...})loop` `()exit`  
        `exit`用于退出`loop`
      - `(num,{...})repeat`  
        执行{...}num 次

3. 绘制命令

   - `newpath` `closepath`
     开始和结束一段闭合路径
   - `stroke`
     描边当前路径但不会连接起点和终点
     会销毁当前路径
   - `fill`
     连接起点和终点并填充区域
     会销毁当前路径
   - `gsave` `grestore`
     存储状态 与 恢复状态
     包括路径、灰度、线宽、坐标系
   - `showpage`
     立即绘制当前页
   - `([dash_len,space_len],offset)setdash`  
     设置接下来的 line 为虚线，其中每一段由 dash_len 的线段和 space_len 的空白组成
   - `(width)setlinewidth`
   - `(0/1/2)setlinecap`  
     设置线端点形态，0 为平头，1 圆形，2 为投影正方形
   - `(0/1/2)setlinejoin`  
     设置两线段交点处形状，0 斜接，1 圆形斜接，2 平头斜接  
     斜接长度是从线段内边界相交的点到线段外边界相交的点之间的距离，这个长度随着两线段之间的角度减少而增长
   - `(num)setmiterlimit`  
     设置两线段斜接长度最大值
   - `(r,g,b)setrgbcolor`
   - `(gray)setgray`
   - `(x,y)moveto`
   - `(x,y)lineto`
   - `(x,y,r,angle,angle2)arc`  
     arc1，arc2 单位为角度，以第一象限 x 轴正方向为 0°，逆时针绘制
   - `(x,y,r,angle,angle2)arcn`  
     顺时针的 arc  
   - `(x1,y1,x2,y2,r)arcto`  
     绘制圆角
   - `(x1,y1,x2,y2,x3,y3,x4,y4)curveto`  
     (x1,y1)起点，(x4,y4)终点，(x2,y2)、(x3,y3)为控制点绘制贝塞尔曲线
     (x1,y1)可省略，以当前位置作为起点
   - `(x,y)rlineto`  
     由当前点绘制向量(x,y)
   - `(x1,y1,x2,y2,x3,y3,x4,y4)rcurveto`
     ???
   - `clip`  
     根据非零环绕原则计算当前路径和当前裁剪路径的内部区域，将其交集设为新的裁剪路径
   - `eoclip`  
     根据奇偶原则计算当前路径和当前裁剪路径的内部区域，将其交集设为新的裁剪路径
   - `rectclip`  
     将矩形区域与裁剪路径的交集设为新的裁剪路径

4. 字符串
   - `(str)print`  
     将字符串`string`输出到标准输出文件，转义字符会被转义后输出
     不会对页面产生任何影响
   - `(len)string(str)`  
     将一个长度为`len`的空字符串对象压入操作数栈
   - `(file,str)readhexstring(substr,bool)`  
     从`file`读取 16 进制数字写入`str`,若字符串已满返回`true`，若字符串满之前遇到 EOF 返回`false`
5. 字体

   - `(font_name)findfont(font)`  
     加载字体并压入操作数栈，`font_name`必须为`/`开头的具有字面属性的命名对象  
     等价于`/Font findresource`
   - `(scale)scalefont`  
     设置字号,以 point 为单位
   - `(font)setfont`  
     设置字体
   - `definefont`  
     等价于`/Font defineresource`
   - `undefinefont`  
     等价于`/Font undefineresource`
   - `(str)show`  
     从当前点开始绘制`str`
     在没有设置字体之前不会绘制，且不报错  
   - `(str)stringwidth(v_x,v_y)`  
     输出结果为`str`被`show`命令输出后，当前点移动的距离

6. 图像
   - `(width,height,bits,matrix[],data_src)image`  
     `(img_dict)image`  
     灰度图像  
     `bits`表示每像素的位数，`matrix`为从图像空间到用户空间的变换矩阵
   - `(width,height,bits,matrix[],data_src,multi,ncomp)colorimage`  
     `(img_dict)colorimage`  
     `ncomp`为颜色分量数，1，3，4 分别表示`DeviceGray`、`DeviceRGB`、`DeviceCMYK`颜色空间；`multi`为`boolean`，表明数据源是否为多数据源
7. 坐标系命令
   - `(x,y)translate`
     坐标轴平移(x,y)
     相当于移动原点，变换相对于当前原点而非最初的原点
   - `(angle)rotate`
     坐标轴逆时针旋转，单位为度
   - `(scale_x,scale_y)scale`
     x 轴缩放为 scale_x 倍，y 轴缩放为 scale_y 倍
8. 栈命令
   - `pstack`
     按出栈顺序打印栈中所有元素
   - `pop`
     栈顶元素出栈
   - `=`
     操作数栈栈顶元素出栈并输出到标准输出文件
   - `==`
     类似于`=`，但输出结果更接近 PostScript 语法
     > 对于`(str)`，`=`会输出`str`，`==`会输出`(str)`
   - `clear()`
     清空堆栈
   - `dup(elem)`
     复制栈顶元素并再次入栈
   - `(count)copy`
     批量的 dup，复制栈顶 count 个元素
   - `exch`
     交换栈顶数据
   - `(count,seq)roll`
     将栈顶 `count` 个元素循环滚动 `seq`
     > 类比循环队列元素向后移动 `seq`
   - `currentpoint`
     当前位置入栈，先 x 坐标，后 y 坐标
   - `(any ...)cvx`
     将栈顶对象转换为可执行对象
   - `(execObj)exec`
     将栈顶对象压入执行栈；对于操作数，将其入操作数栈，对于函数，立即执行
9. 字典命令

   - `(key，value)def`  
     将键值对存入当前字典（字典栈栈顶的字典），其中`key`必须为具有字面属性的名字对象  
     若已存在相同的键，则替换原先的值
   - `(dict,key)undef`

     > level 2

     删除`dict`中的键为`key`的键值对，若不存在也不会报错

   - `(key,value)store`  
     查找字典栈中的所有字典，替换第一个与键`key`匹配的键值对,若找不到则将键值对存入当前字典
   - `(dict)begin`  
     将字典`dict`压入字典栈
   - `end`
     将字典栈栈顶的字典出栈

   - `({...}1)bind（{...}2）`  
     将`{...}1`中的值为运算符对象 operator object 的可执行名字对象 name object 直接替换为对应的运算符对象，避免多次调用时频繁查找字典栈的开销

     > 如`{add 1 div}`，每次执行均要在字典栈上查找`add`和`div`；执行`{add 1 div}bind`就会查找字典栈并将`add`和`div`替换为对应的运算符

     仅会替换值为运算符对象的名字对象，值为其他类型如过程对象 procedure object 的名字对象并不会被替换

     > 如果一个操作符的名字被重定义了，那么重定义之后的名字也不会被替换

     level 3 的`bind`还会执行惯用语识别 idiom recognition，将特定的过程 procedure 替换为效率更高的过程 procedure。查找 IdiomSet 匹配已绑定的过程并替换

10. 打印机控制
    - `(dict)setpagedevice`  
      设置页面设备字典 page device dictionary 的内容
11. 资源
    - `(catgory,instance)findresource(resource_obj)`  
      如果请求的示例尚未被加载进虚拟内存，则该命令会尝试从外部加载  
      该过程对 PostScript 程序是透明的
