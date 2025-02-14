---
title: Range View
tags:
  - cpp/20
  - TBD
---
> [!cite]- References  
> [\[Hannes Hauswedell\] A beginner's guide to C++ Ranges and Views](<https://hannes.hauswedell.net/post/2019/11/30/range_intro/>)   
> [\[程鑫\] STL库的ranges](https://www.cnblogs.com/chengxin1985/p/18226526)
> [\[克莉丝\] C++ 用户定义 Ranges 算子 - 克莉丝随笔](https://xr1s.me/2023/08/13/cxx-user-defined-range-adaptor/))

```cpp
#include <ranges>
```

## 范围 Range  
范围是对于可迭代对象或物品集合的抽象，用以分类不同的对象以及可在对象上进行的操作  
只要一个对象具有期望的行为（如特定的成员变量或方法，或者让作为参数能让某段代码编译通过），就可以在代码中当作符合某种约定的对象来处理，而无需关心对象的确切类型  

### 范围实现  
范围通过 [concept](Concept.md) 定义  
在 `<xutility>` 中，`std::ranges::range` 定义了基本的范围  

```cpp
template< class T >
concept range = requires( T& t ) {
  ranges::begin(t); // equality-preserving for forward iterators
  ranges::end(t);
};
```

下表是部分用于其他的范围概念  

| concept                          | description   |
| -------------------------------- | ------------- |
| std::ranges::input_range         | 可从头至尾迭代至少一次   |
| std::ranges::forward_range       | 可从头至尾迭代多次     |
| std::ranges::bidirectional_range | 可反向迭代         |
| std::ranges::random_access_range | 可在常数时间内随机访问元素 |
| std::ranges::contiguous_range    | 元素在内存中连续存储    |

## 视图 View  
视图是一种特殊的范围，其定义依赖于另一个范围，并通过一些算法或操作在其上进行变换  
不同于标准库的容器(`std::vector`, `std::map` 等)，视图本身不拥有数据( `owning_view` 除外)  
> 参照 `std::string_view`，视图本身只存储对范围的引用和在其上进行的操作  
### 惰性求值 laze-evaluation  

视图在创建时不会对数据做任何操作，只有当访问视图元素时才会执行操作  
### 组合性  

视图不能直接创建，而是通过适配器间接创建  

适配器接收特定的范围，并返回对应的视图对象  
1. `filter`：创建一个仅包含符合特定条件的元素的视图。
2. `transform`：对每个元素应用给定的转换函数，并生成一个新的视图。
3. `take`：创建一个包含指定数量元素的视图。
4. `drop`：创建一个去除指定数量元素后的视图。
5. `split`：将范围分割成指定大小的子范围序列。
6. `reverse`：反转范围中的元素顺序。
7. `join`：将范围的范围中的子范围连接成单个范围。
8. `elements`：从范围中的元组中选择指定索引的元素，并将其表示为一个范围。

适配器隐藏了特定的视图类型，并且允许适配器之间通过管道运算符 `|` 链式组合  

```cpp
auto rev_view = std::vector{1,2,3} | std::views::reverse |std::views::drop(2);
```
## 范围算法  

引入范围之后，`<algorithm>` 中的很多算法都实现了范围版本，其不再接收迭代器 `begin` 和 `end()` 为参数，而是一个单独的范围参数  