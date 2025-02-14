---
title: CPP Concept
tags:
  - cpp/20
  - TBD
---
```cpp
#include <concept>
```

concept 是一种泛型约束的手段，用于简化 SFINAE，通过**编译期**布尔常量表达式的逻辑组合，以限定模板参数的类型或模板常量的范围  
concept 本身也可以参与表达式

concept 支持如下使用方式  

1. requires 语句

    ```cpp
    template <typename T>
    requires std::is_integral<T>
    void F(T a){  //do sth
    };
    ```

2. 后缀 requires 语句

    ```cpp
    template <typename T>
    void F(T a) requires std::is_integral<T>{//do sth
    }
    ```

3. 受约束的模板参数

    ```cpp
    template <std::is_integral T>
    void F(T a){//do sth
    }
    ```

4. 简化的函数模板

    ```cpp
    template <typename T>
    void F(std::is_integral<T> auto a){//do sth
    }
    ```

5. 定义 concept 后使用

    ```cpp
    template <typename T>
    concept limit= std::is_integral<T>;
    
    template <limit Y >
    void F(Y a){}
    ```