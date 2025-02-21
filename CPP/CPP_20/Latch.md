---
title: std::latch
created: 2025-02-17
tags:
  - cpp/20
  - TBD
---
> [!cite]- References  
> [C++20 多執行序間的同步點 barrier 與 latch – vimL Blog](https://viml.nchc.org.tw/c20-barrier-and-latch/)

```cpp
#include <latch>
```

## latch  

`std::latch` 用于多线程场景下的同步，允许多个线程在特定的同步点上等待，直到所有线程均到达该同步点  

具体实现使用 `atomic` 变量进行计数( MSVC 实现)  

```cpp
class latch
{
//......
private:
    atomic<ptrdiff_t> _Counter;	
}
```
### 基本使用  
 
子线程中的 `count_down()` 递减计数器  
主线程中的 `wait()` 阻塞直至计数器归0  

```cpp
std::latch is_done(3);
for(int i=0;i<3;i++)
{
std::thread(
  [&is_done](){
    is_done.count_down();
  }
);
}
is_done.wait();
```
另有成员函数 `arrive_and_wait()` 将 `count_down()` 与 `wait()` 合二为一  
