---
title: std::barrier
created: 2025-02-17
tags:
  - cpp/20
  - sync
  - TBD
---
> [!cite]- References  
> [std::barrier - cppreference.com](https://en.cppreference.com/w/cpp/thread/barrier)  
> [C++20 多執行序間的同步點 barrier 與 latch – vimL Blog](https://viml.nchc.org.tw/c20-barrier-and-latch/)  
> [关于 std:barrier - hzSomthing](https://hedzr.com/c++/algorithm/cxx20-about-std-barrier/)

```cpp
#include <barrier>
```
## barrier  

`std::barrier` 用于多线程场景下的同步，允许多个线程在特定的同步点上等待，直到所有线程均到达该同步点  

具体实现使用 `atomic` 变量进行计数( MSVC 实现)  

```cpp
class barrier
{
//......
private:
    struct _Counter_t {
        constexpr explicit _Counter_t(ptrdiff_t _Initial) : _Current(_Initial), _Total(_Initial) {}
        atomic<ptrdiff_t> _Current;
        atomic<ptrdiff_t> _Total;
    };
    _Compressed_pair<_Completion_function, _Counter_t> _Val;
}
```
