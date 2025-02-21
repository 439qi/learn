# 字符集与编码
>
> [字符集与编码](https://www.cnblogs.com/linvanda/p/15906679.html)  
> [Unicode与UCS](https://www.cnblogs.com/malecrab/p/5300503.html)  
> [UTF16是如何编码的](https://zhuanlan.zhihu.com/p/27827951)
> [ASCII, Unicode和UTF8](https://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)
>
## 字符编码模型

### 抽象字符表(Abstract Character Repertoire)

明确了该编码标准可以对哪些字符进行编码

### 编码字符集(Coded Character Set)

> 常简称为字符集

编码每个字符分配一个唯一的数字编码，称为码点(Codepoint)  

如[Unicode](#unicode)

### 字符编码形式(Character Encoding Form)

描述码点如何在计算机中表示  
如[UTF8](#utf8)

分为两步

1. 定义计算机表达字符编码的单位，术语叫编码单元（Code Unit），简称码元  
2. 定义表达规则，即如何用一个或多个码元来表示码点

对于所有码点采用相同数量的码元表示，称为定长编码方式  
对于不同字符采用不同数量的码元表示，称为变长编码方式

### 字符编码方案(Character Encoding Scheme)

规定多字节编码的字节序
> Windows 操作系统使用的小端模式，Mac OS 以及网络数据传输采用的是大端模式  
>有些 CPU 架构可以切换大小端
>
## ASCII

单字节定长字符集/编码  
使用7位二进制数来表示所有的大写和小写字母，数字0到9、标点符号，以及美式英语中使用的特殊控制字符

## EASCII

ASCII仅使用了128个码点，部分字符集/编码使用剩余128个码点进行扩展

### ISO 8859-1/Latin 1

单字节定长字符集/编码  
欧洲地区主要使用的字符集，完全使用了剩余128个码点，包括西欧语言、希腊语、泰语、阿拉伯语、希伯来语对应的文字符号
> 由于Latin1完全使用了单字节所有256个码点，无保留码点，在Latin1编码的系统中传输和存储任意编码的字节流都不会被抛弃，MySQL默认编码为Latin1即是因此
>
## UNICODE

Unicode分为17个**平面**(plane)，每个平面包含$2^{16}=65536$个码点，即从0-0x10FFFF，其中0xD800-0xDFFF保留作[代理机制](#utf16)  
第一个平面称为基本多语言平面(Basic Multilingual Plane, BMP)，其余平面称为辅助平面(Supplementary Planes)

一般写成 16 进制，在前面加上 U+

宽字符串使用wchar_t数组存储

### UTF8

Unicode的变长编码实现
|Unicode编码|UTF8字节流|
|-|-|
|0x0-0x7f|0XXXXXXX|
|0x80-0x7ff|110XXXXX 10XXXXXX|
|0x800-0xffff|1110XXXX 10XXXXXX 10XXXXXX|
|0x10000-0x10ffff|11110XXX 10XXXXXX 10XXXXXX 10XXXXXX|

### UCS-2

Unicode的双字节定长编码实现  
所有字符均使用2个字节来表示，仅覆盖BMP，其中ASCII字符高字节为0，  
为兼容Unicode，0xD800-0xDFFF码点未使用

在Windows 2000之前的Windows，仅支持UCS-2

### UTF16

基于UCS-2  
Unicode的变长编码实现  

对于BMP直接映射，使用两个字节  
对于辅助平面使用[代理机制](#代理机制)映射，将其Unicode码点值减去0x10000映射到0-0xFFFFF，将前10bit加上0xD800作为UTF16前两个字节编码，后10bit加上0xDC00作为UTF16后两个字节编码，共四个字节

从Windows XP开始支持

#### 代理机制

使用代理对，即两个BMP中未使用的码点(0xD800-0xDFFF)来表示一个辅助平面的码点  
代理对的两个码点，分别称为高代理码点和低代理码点  
高代理码点和低代理码点起始值分别为0xD800和0xDC00，两者后10位均为0，合计共20bit，可用于表示辅助平面的所有码点

### UTF32

Unicode的四字节定长编码实现  

## GB2312、GBK、GB18030

### 全角、半角字符

全角字符和半角字符在字符集中是两个不同码点的字符  

双字节中文编码的历史遗留问题  
在当时纯文本的界面中，为了让西文和中日韩的方块字对齐，使得西文字母、数字和标点也占用一个汉字的视觉空间，并使用 2 个字节存储，即为全角字符

### GB2312

字符集/双字节定长编码  
共分为94个区，每个区有94个字符  
编码时两个字节分别对应分区编号和区内编号，并将两个字节的最高位置1  

### GBK

字符集/双字节定长编码  
在GB2312的基础上扩展了部分汉字，兼容GB2312  

### GB18030

字符集/变长多字节编码  
每个字符由1、2或4个字节组成  
单字节0-0x7fF，兼容ASCII  
双字节第一个字节从0x81-0xFE，第二个字节从0x40-0xFE(除0x7F)，兼容GBK  
四字节第一个字节从0x81-0xFE，第二个字节从0x30-0x39，第三个字节从0x81-0xFE，第四个字节从0x30-0x39

## MBCS(Multi Bytes Character Set)

多字节字符集  
用于无法单字节表示的字符集(日文、中文等)  

### DBCS(Double Bytes Character Set)

双字节字符集  
字符使用1个或2个字节表示  
DBCS是对各国的双字节编码方案的一个统称
根据当前活动代码页划分不同子集和对应的编码方式(SHIFT-JS, [GB18030](#gb18030)等)

使用char数组存储  

## 输入法与字符编码
>
> Windows系统为例  

输入法程序向系统注册自身，用户选择指定的输入法后

1. 通过键盘输入时，输入法从Windows系统消息获取ASCII码序列
2. 输入法将ASCII码序列组合，转换成输入法的内部表示  
3. Windows将输入法输出的字符通过系统消息 `WM_IME_CHAR` 传递给要求输入的程序  
    对于消息函数`GetMessageA()`和窗口过程函数`WindowProcA()`为单字节字符即A后缀的情况，Windows传递ANSI即系统当前代码页对应的编码的字符  
    对于消息函数`GetMessageW()`和窗口过程函数`WindowProcW()`为双字节字符即W后缀的情况，Windows传递UTF16编码的字符
