# 移动语义  

## 值种类 value categories  

任意**表达式**通过两个独立的性质可以划分为三种基本值种类  

### 性质  

1. 拥有身份(has identity)  
    > 身份指地址，名字或别名，即该对象在该表达式以外是否可被访问

    是否指代非临时对象  
2. 可被移动(can be moved from)  
    是否可被右值引用类型匹配  

### 基本值类型

#### 左值 lvalue  

拥有身份且不可被移动  
eg. 形参

#### 将亡值 xvalue

拥有身份且可被移动  

- eg.  
    右值引用类型的返回值  
    prvalue实体化产生的临时对象  
    被 return 的局部变量

#### 纯右值 prvalue  

不拥有身份且可被移动  

- eg.  
    除字符串字面值之外的字面值、表达式的非引用返回值等  
    特定情况下 prvalue 会实体化并产生一个临时对象(temporary object)，种类为 xvalue  
    > C++17 规定了 guaranteed copy elision/RVO，纯右值的返回值用作初始化时直接在要初始化的变量上构造或移动构造  
    >
    > ```cpp
    > T func()
    > {return T();}
    > //...
    > T t = func();
    > ```
    >
    > 如下返回值为 xvalue,不会触发 RVO，但会被 NRVO 优化掉  
    >
    > ```cpp
    > T func2()
    > { T t;
    > ...
    > return t;
    > }
    > ```

### 复合值类型

依据拥有身份或是否可被移动，可划分两种复合值类型

- 泛左值 glvalue = 将亡值 + 左值
- 右值 rvalue = 将亡值 + 纯右值

### 值种类对应的构造方法  
  
|值种类|被初始化的值种类|构造方法|
|-|-|-|
|prvalue|gvalue|直接构建|
|xvalue|lvalue|移动构造函数|
|lvalue|lvalue|拷贝构造函数|

## std::move()  

`move()` 作用于编译期，在运行期无作用  
其原理是将移除传入参数的所有引用性质并强制转换为对应类型的右值引用并返回

```cpp
_EXPORT_STD template <class _Ty>
_NODISCARD _MSVC_INTRINSIC constexpr remove_reference_t<_Ty>&& move(_Ty&& _Arg) noexcept {
    return static_cast<remove_reference_t<_Ty>&&>(_Arg);
}
```

## 模板相关

### 万能引用

- 左值引用：可以指向左值，不能指向右值

  > 例外：const左值引用可以指向右值  

- 右值引用：可以指向右值，不能指向左值  
  常用于延长临时对象的生命周期  
  > 只能延长prvalue，若prvalue被实体化为xvalue则不会延长其生命周期

  NOT FINISHED  
  右值引用能指向纯右值的本质是通过一个临时变量将右值提升为左值，然后通过`std::move`转换为右值  

    ```cpp
    int &&ref=5;
    //等价于
    int temp = 5;
    int &&ref = std::move(temp);
    ```

- 万能引用：形如`T&&`可以绑定到被推导类型的左值或右值的引用  
  万能引用只能用于被推导类型，即函数模板、auto等  
  被const修饰时是右值引用  
  如下代码是右值引用

  ```cpp
  template<typename T>
  void func(std::vector<T> &&x)
  ```

  万能引用遵循引用折叠规则
  - 绑定右值引用的右值引用折叠为右值引用  
  - 其余组合折叠为左值引用，即只要有左值引用参与最终就会折叠为左值引用  

### 完美转发

将万能引用作为实参调用其他函数时，由于万能引用本身是左值，会丢失被绑定对象的值种类，通过`std::forward`可以转换为其原本的值种类  

- `std::forward<T>(u)`  
  T推导为左值引用类型时，将u转换为T类型的左值  
  否则，将u转换为T类型的右值
