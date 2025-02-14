---
title: CPP Module
tags:
  - cpp/20
  - TBD
---
> [!cite]- References  
> [Understanding C++ Modules: Part 1: Hello Modules, and Module Units](<https://vector-of-bool.github.io/2019/03/10/modules-1.html>)  

## 混合使用  

对于在旧式的 `#include` CPP 项目中引用模块  

### 标准库  

`#include` 包含的标准库和 `import std` 的标准库会导致标准库存在两处不同定义，违反了 ODR  

- 解决方案1  
    将模块中的标准库替换为形如 `import<iostream>` 的形式  
    > 比较奇怪的是如果使用 `#include<>` 的形式，在 `<xstring>` 中会有一个`std::_Pocca()` 函数重载决议失败  
    > MSVS 2022 17.9.2
    >
### 预编译头  

无法兼容预编译头，选择不使用预编译头  