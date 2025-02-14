---
title: Static Reflection
aliases:
  - 静态反射
tags:
  - cpp/26
  - proposal
---
> [!cite]- References  
> [Java 反射特性 · Thinking in Java](https://java.jverson.com/jvm/java-reflection.html)
> [C++26 静态反射提案解析 · ykiko's blog](https://ykiko.me/zh-cn/articles/661692275/)


> [!cite]  [Wikipedia](https://zh.wikipedia.org/wiki/%E5%8F%8D%E5%B0%84%E5%BC%8F%E7%BC%96%E7%A8%8B)  
> 在计算机学中，反射式编程（英语：reflective programming）或反射（英语：reflection），是指计算机程序在运行时（runtime）可以访问、检测和修改它本身状态或行为的一种能力。用比喻来说，反射就是程序在运行的时候能够“观察”并且修改自己的行为。

## 其他语言中的反射  
Java、Python、Go 等语言中的反射可以在编译期不明确对象的成员变量和可用方法，而在运行期根据一个 String（全限定类名） 来得到想要的实体对象，然后调用它的方法，访问它的属性  
 #TBD 
```
need some sample codes here
```

## C++ 静态反射  
以上语言的反射因为在运行期进行类型查询，其不可避免地牺牲了运行效率  
这种机制不符合 C++ 的零成本抽象原则  
> [!cite] Bjarne Stroustrup  
> What you don't use, you don't pay for. And further: What you do use, you could't hand code any better.

C++26 的反射提案中的方案是在编译期完成反射的，因而称之为静态反射  

 #TBD 编译期的值操作  
 #TBD 编译期的类型操作-模板  
 #TBD 从类型映射到类型名  

静态反射在语言层面实现了类型 Type 和值 Value 的双向映射  
```cpp
constexpr std::meta::info val = ^int; // type to value
using T = typename [:val:]; //value to type
typename [:val:] a = 3; //value to type, equal to `int a = 3;`
```