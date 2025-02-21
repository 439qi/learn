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

`std::barrier` 类似于 [`std::latch`](CPP/CPP_20/Latch.md)， 但可以指定指定触发事件，以及可以多次使用  

> [!warning] 两个类的接口并不完全相同  
> `std::barrier` 不支持 `count_down()`，且 `wait()` 函数需要==右值== token 参数  
### 指定触发事件  

`std::barrier` 的构造函数可以接受一个 **`noexcept`** 可调用对象，当所有线程到达同步点时调用该对象  

```cpp
std::barrier is_done(3, 
	[]()noexcept{std::cout << "all reached barrier." << std::endl; });
for (int i = 0; i < 3; i++)
{
	std::thread(
		[&is_done]() {
			is_done.arrive_and_wait();
		}
	).detach();
}
```

### 多次使用  

`arrive_and_wait()` 在到达同步点时等待其余线程同步  
`arrive_and_drop()` 在达到同步点后不等待继续执行，且将内部计数上限减 1  

```cpp
std::barrier is_done(3, 
	[]()noexcept{std::cout << "all reached barrier." << std::endl; });

for (int i = 0; i < 2; i++)
{
	std::thread(
		[&is_done]() {
			is_done.arrive_and_wait();//phase 1
			is_done.arrive_and_wait();//phase 2
			is_done.arrive_and_wait();//phase 3
		}
	).detach();
}

std::thread(
	[&is_done](){
		is_done.arrive_and_drop();//phase 1
	}
).detach();
```

> [!note] `std::barrier` 与 `std::latch`  
> 基于两个类的不同特性  
> - `std::latch` 更适用于检查多个线程的工作是否完成  
> - `std::barrier` 更适用于作为多个线程间的同步点，以协调完成工作  