# Cpp 多线程

多线程的两个主要用途  

1. 保护共享数据(Protecting shared data from concurrent access)
2. 同步并行操作(Synchronizing concurrent operations)

## std::thread

```cpp
//introduced in Cpp 11
#include<thread>
using std::thread;
```

### thread 方法

- `thread::get_id()`  
  获取线程ID
- `thread::join()`
  主线程将阻塞直至子线程执行完毕
- `thread::detach()`  
  主线程与子线程分离，即主线程不阻塞
- `this_thread::sleep_for()`
  当前线程休眠指定时间  
  其中时间为 `std::chrono` 类型

### thread 使用

构造函数接收可调用对象及其参数并**立即**创建线程运行该函数  

 可接受的对象包括  

- 普通函数  
- 成员函数  

    ```cpp
    std::thread(&CLASS::METHOD, &class_obj, param).detach();
    ```

- 函数对象  
- lambda  

线程对象创建后必须调用 `join()` 或 `detach()`，否则析构时抛出异常  
线程对象不可复制，只能移动

## std::future, std::promise

```cpp
#include <future>
using std::future;
using std::promise;
```

### 方法

### future/promise 使用

```cpp
template <typename T>
std::promise<T> pro;
std::future<T> fu= pro.get_future();
//one random thread
pro.set_value(1);
//another thread holding responding future
fu.get();
```

### std::promise

`promise` 内部持有对应的 `std::future`  
`promise` 由生产者持有，在某个特定时刻通过`promise::set_value()` 设置值  

### std::future

`future` 无法拷贝，只可移动或绑定  
`future` 由消费者持有，通过 `future::get()` 获取生产者设置的值  
`future::get()` 将阻塞当前线程直至对应的 promise 已设置值  

#### 获取 future 状态

- `future::valid()` 指示 `future` 是否合法  
    只有 `std::promise::get_future()`, `std::packaged_task::get_future()`, `std::async()` 返回的 `future` 且尚未调用 `future::get()` 和 `future::share()` 时为 `true`  

- `future::wait()` 将阻塞至结果可用  
  std::promise::get_future(), std::packaged_task::get_future() or std::async()  

- `future::wait_for()` 接受一个 `std::chrono::duration` 作为参数，在指定时间段之后或结果可用时返回，结果为以下三者之一
    |constant|explanation|
    |-|-|
    |`future_status::deferred`| 持有 `promise` 的对象仍未开始执行|
    |`future_status::timeout` |执行已超时|
    | `future_status::ready`| 结果已就绪  |  

- `future::wait_until()` 类似 `wait_for()`，不过其参数为指定的时间点  

### shared_future()

对于需要多个线程共享 `future` 的情况，通过 `future::share()` 返回 `std::shared_future`

## std::packaged_task

封装可调用对象和 `promise`

### std::packaged_task 使用

接受一个可调用对象，并返回一个 `future`  
该可调用对象的返回值将成为 `future` 获取的值

```cpp
std::packaged_tsk<int()> tsk([](){return 1;});
std::thread(&tsk);
int ret = tsk.get_future().get();
```

## std::async

```cpp
#include<future>
using std::async;
```

对 `std::thread`、`std::future/promise`、`std::packaged_task` 的封装  
在**任务**的层次上进行异步操作，将线程的底层细节交由标准库  

### std::async() 使用

接收启动策略，可调用对象和参数，并返回一个 `future` 对象

```cpp
std::future<bool> fu = std::async(std::launch::async, []()->bool{return true;});
```

可接受的启动策略

- std::launch::async  
    立即在另一线程上执行指定任务  
- std::launch::deferred  
    任务推迟到对应 `future::get()` 调用时执行  
    若没有调用 `future::get()`，则任务也不会执行  

默认启动策略为两者或运算，即交由 `async` 自行决定  

```cpp
//如下两种方式等价
std::async(func_ptr);
std::async(std::launch::async|std::launch::deferred, func_ptr);
```

其中指定的函数的返回值将被设置为 `future` 中的值

### 潜在问题

- 特定情况下退化为同步操作  
如果 `std::async()` 返回的 `future` 没有被移动、或绑定到引用上，则这个临时 `future` 对象的析构函数将一直阻塞直到异步操作完成，退化为同步操作  

  ```cpp
  std::async(std::launch::async, func_1);
  func_2();//直到上一行的func_1()返回，才会执行到func_2()
  ```

  > 本质上因为 `std::async()` 返回的 `future` 的最后一个实例(如果调用了 `future::share()` )的析构函数将会阻塞  
  > `future` 的析构函数当且仅当如上情况时才会阻塞
- 无法创建分离的线程  
  同上，由于 `future` 析构函数的原因，`std::async` 创建的线程的生命周期必然短于 `future` 对象(因为该线程返回值对应 `future` 获取的值)  

- 破坏异常现场  
  `std::async()` 将会捕获所有异常并保存到 `future` 中，直到通过 `future` 取值时再抛出  
  由于异常不携带栈信息，导致无法获取异常实际发生时的上下文  

## std::atomic

`std::atomic` 是模板类，一个模板类型为 T 的原子对象中封装了一个类型为 T 的值

```cpp
template<class T> struct atomic;
```

原子类型对象的主要特点就是从不同线程访问不会导致**数据竞争**(data race)  
因此从不同线程访问某个原子对象是**良性**(well-defined)行为

### atomic 基本方法

- store()  
  修改被封装的值
- operator T()  
  在类型转换时被调用，确保类型转换线程安全  
- exchange()  
  读取并修改被封装的值，将封装的值修改为指定值并返回原先值  
- compare_exchange_weak()  
  比较被封装的值与参数1是否**二进制相等**，若相等则将其修改为参数2，否则将参数1修改为被封装的值  

### atomic 特化方法

atomic 针对整形和指针的特化版本，新增了算术运算和逻辑运算操作

## std::once_flag, std::call_once
