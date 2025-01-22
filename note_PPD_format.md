<!-- 2023.09.04 created -->
<!-- 2023.09.18 modified: error corrected -->
> Adobe/PostScript Printer Description File Format Specification.pdf

# 注释

以`\*%`开始，到换行符结束

# 条目 entry

条目由主关键词、可选项关键词和值组成，只有主关键词是必须的

```
*Default<main keyword>: <optionn>
*<main keyword> <option1>: "PostScript language code"
*<main keyword> <optionn>: "some other PostScript language code"
*?<query keyword>: "PostScript language query code"
```

### 主关键词 main keyword

- 以`*`开始，`*`之前不允许出现空白符
- 大小写敏感
- 包含`*`最长 40 个字符
- 若没有出现某个主关键词，则表示设备不支持其代表的功能
- 以下主关键词必须出现，  
  `*NickName` `*ModelName` `*PCFileName`  
  `*Product` `*PSVersion` `*FileVersion`  
  `*FormatVersion` `*LanguageEncoding` `*LanguageVersion`  
  `*PageSize` `*PageRegion` `*ImageableArea`  
  `*PaperDimension` `*PPD-Adobe`
- 若某个主关键词无法识别，则整个条目将被跳过；`*OpenUI` 和`*CloseUI` 之间的例外
- 主关键词有两个子集
  1. 查询关键词 Query keywords  
     以`*?`开始，没有可选项关键词，其值部分为 PostScript 代码序列，当下载到设备时返回设备当前状态信息
  2. 默认关键词 Default keywords  
     以`*Default`开始，没有可选项关键词，其值部分为可选项关键词的字符串值

### 可选项关键词 option keyword

可选项关键词紧跟在主关键词后，由一个或多个空格分隔

- 只能包含可打印 ASCII 字符
- 不允许出现破折号`-`、冒号`:`、空格 、制表符、换行符
- 可选项关键词可以有扩展名，称为限定符，以`.`分隔

### 值 value

冒号用于分隔关键词和值，冒号和值之间可以出现任意制表符和空格

1. 调用值 InvocationValue

   - 出现在可选项关键词后
   - 以双引号(ASCII 67)`"`开始和结束
   - 双引号之间的任何字符都被视为字面值，包括换行符
   - 双引号之间不允许再出现双引号，且没有转义机制

   调用值将被插入最终生成的输出文件，直接被 PostScript 解释器执行

2. 引用值 QuotedValue
   - 仅在没有可选项关键词的条目出现
   - 以双引号(ASCII 67)`"`开始和结束
   - 双引号之间不允许再出现双引号`"`，和尖括号(ASCII 60)`<` 、(ASCII 62)`>`，且没有转义机制
3. 符号值 SymbolValue
   - 以(ASCII 94)`^`开始，换行符结束
   - 不允许出现空白符
4. 字符串值 StringValue
   - 只能包含可打印 ASCII 字符
   - 不能被双引号(ASCII 67)`"`包围，不能以双引号(ASCII 67)`"`、(ASCII 94)`^`开始
   - 不允许出现斜杠(ASCII 47)`/`，且没有转义机制
5. 空值 NoValue
   - 没有出现可选项关键词，没有出现其他值，仅有主关键词

### PostScript 语言序列 PostScript Language Sequences

通常作为调用值出现，有时作为引用值出现（如二进制数据）  
跨越多行的序列以`*End`结尾

# 结构关键词

### `*OpenUI`、`*CloseUI`

```
*OpenUI *<main keyword>:PickOne|PickMany|Boolean
*CloseUI:*<main keyword>
```

为区别用于提供可选择的 UI 的主关键词和用于提供设备信息的主关键词，`*OpenUI`、`*CloseUI` 用于包围为用户提供可选项的 UI 的主关键词

- `PickOne`和`PickMany`如其字面含义所表示，`Boolean`表示仅有两个选择：`True`、`False`
