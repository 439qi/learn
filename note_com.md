# COM

## 作用

### 1. 问题

- 当 dll 发生改动时，基于原 dll 编写的程序可能会无法运行

  程序的二进制结构在编译时已经确定，当 dll 中发生了改变时，程序访问原先的地址时就会出问题。

  > 如 dll 改动了一个类的定义，那么其成员相对于首地址的偏移就会发生改变，但程序仍按原先的偏移去访问该成员

- 不同编译器的不兼容问题

  C 的模型较为简单，但 C++ 的许多高级特性，如重载、构造函数、虚构函数、虚基类等，C++ 规范只约束了行为，没有约束编译器的实现细节。同一个特性，不同编译器采用不同的实现方法，对象的布局也会不同。

  >

- 其他等等

### 2. COM 解决方案

- 只暴露接口（C++中的抽象类），不暴露具体的实现

  > COM 基于一个前提，各编译器实现虚函数的数据结构一致

  dll 改动时必须保证接口一致

- 通过编程规范，将编译器有差异的部分限制在模块内部，对象的创建与释放，基类与父类的转型均由实现该对象的模块实现

  对于每一个 COM 类，提供一个工厂类，通过工厂间接创建与删除对象  
  对于每一个 COM 类，必须提供转型函数

## COM 实现

### 1.IUnknown 接口

所有的 COM 类必须继承自 IUnknown 接口

```cpp
struct IUnknown
{
    virtual HRESULT STDMETHODCALLTYPE QueryInterface(
        /* [in] */ REFIID riid,
        /* [iid_is][out] */ _COM_Outptr_ void __RPC_FAR *__RPC_FAR *ppvObject) = 0;

    virtual ULONG STDMETHODCALLTYPE AddRef( void) = 0;

    virtual ULONG STDMETHODCALLTYPE Release( void) = 0;
}
```

- `AddRef`  
  引用计数加一
- `Release`  
  引用计数减一
- `QueryInterface`  
  转型接口  
  对于每一个 COM 类都有一个唯一的 GUID 值，QueryInterface 的 rrid 参数即为要转换为的类的 GUID，转换后的对象由第二个参数带回

## 2. 引用计数

## 3. QueryInterface

## 4. marshaling

## 5. 聚合(aggregation)
