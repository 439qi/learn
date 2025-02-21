---
title: Rust
tags:
  - TBD
  - rust
---
> [!cite]- References
> [Rust官方教程](<https://doc.rust-lang.org/book/>)  
> [Rust语言圣经](https://github.com/sunface/rust-course)  
## Cargo

Rust 内置的构建系统和包管理器  

```shell
# 显示版本
cargo --version
# 创建项目
cargo new <project_name> [--vcs=git]
# 构建项目，默认为debug
cargo build [--release]
# 运行项目生成的可执行文件
cargo run
# 检查项目是否有错误
cargo check
```

Cargo 创建项目时默认使用 [git](note_git.md) 作为版本控制工具，toml 作为配置文件  

Cargo 会生成 `Cargo.toml` 和 `src` 文件夹以及 `src/main.rs`  
其中 `Cargo.toml` 内容大致如下  

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"

[dependencies]
```

> [!note] cargo install [crate] --vers [version]  
## 1 变量和可变性

```rust
let mut var = 5;
```

无 `mut` 修饰的变量为不可变变量，一旦绑定值后就不能改变  

将本身无需改变的变量声明为不可变会避免一些多余的 runtime 检查  

### 1.1 **隐藏**(Shadowing)  

1. 在同一作用域下，使用 `let` 重新声明同名变量会隐藏先前声明的变量，不同于 C++ 会报错  
2. 子作用域声明同名变量会**暂时**隐藏父作用域同名变量，直至子作用域结束  

隐藏本质上创建了一个新变量，因而可以与被隐藏的同名变量有不同的类型  

## 2 常量

```rust
const VALUE: u32=1;
```

常量不同于不可变变量，通过 `const` 关键字声明且必须注明数据类型  
其值必须为编译期可计算值  
> [!summary]
> 常量在整个程序运行期间无法改变或重新绑定  
> 不可变变量无法改变，但可以重新绑定  
> ✔ 如下程序是合法的  
> ```rust
> let a = 123;
> let a = 456;
> ```
> ❌ 但如下程序是非法的  
> ```rust
> const a:i32=123;
> let a = 456;
> ```
## 3 数据类型

Rust 为**静态类型**(statically typed)语言，变量类型必须在编译期可知  
Rust 数据类型分为两类：**标量**(scalar)和**复合**(compound)  

### 3.1 标量

标量代表一个单独的值  
Rust 有4种基本标量类型  

1. 整型  

    |长度|有符号|无符号|
    |-|-|-|
    |8bit|i8|u8|
    |16bit|i16|u8|
    |32bit|i32|u8|
    |64bit|i64|u64|
    |128bit|8128|u128|
    |arch|isize|usize|  

    其中 isize 和 usize 依赖于设备，在32位架构上占用32位，在64位上占用64位  

    对于右移运算，整数补0，负数补1  

    - 字面值  
        字面值允许添加后缀以指明类型，形如 `57u8`  
        也允许以 `_` 为分隔符增加可读性，形如 `1_000`  

        字面值默认为十进制，对于其他进制，允许如下表示方法  

        |进制|例子|
        |-|-|
        |Hex|0xff|
        |Octal|0o77|
        |Binary|0b1111|
        |Byte(u8)|b'A'|
2. 浮点型  
    浮点型分为 f32 和 f64，默认为 f64  
    浮点数采用 IEEE-754 标准  

    对于浮点数的非法操作返回特殊结果 `NaN`，`NaN` 无法与自身比较，可通过 `is_nan()` 方法判断  
    `NaN` 不满足自反性，因而Rust中浮点数是[偏序集合]，用于比较实现的是 `std::cmp::PartialEq` trait，不同于其他类型比较实现的 `std::cmp::Eq`  
    NFY: 偏序集合

3. 布尔型  
    可选值为 `true`、`false`  
    非布尔值不会隐式转换为布尔值  
4. 字符型  
    char 类型大小为4字节，使用 UTF32 编码  

### 3.2 复合类型

复合类型将多个值组合成一个类型  
原生的复合类型有**元组**(tuple)和**数组**(array)  

#### 3.2.1 元组

元组长度不可变，其元素类型不必相同

```rust
let tup: (i32,u8,f64)=(500,1,6.4);
```

使用模式匹配将元组分为不同变量，称为**解构**(destructuring)  

```rust
let (x,y,z)=tup;
```

元组索引访问通过 `.` 而非 `[]`  

```rust
let first=tup.0;
```

#### 3.2.2 数组

数组长度不可变，其元素类型必须相同  
数组索引访问通过 `[]`  

```rust
let arr: [i32; 3]=[1,2,3];
let arr2=[3;5];//初始化为5个3
```

`[3;5]` 语法糖的底层是通过拷多次贝完成的，对于非基本类型元素的数组，若该类型未实现 `Copy` trait，则无法使用该语法，应使用 `std::array::from_fn`  

```rust
use std::array::from_fn;
let arr3: [String; 8] = from_fn( |_i| String::from("abc") );
```

#### 3.2.3 结构体

将多个不同类型的值组合到一起，结构体各部分数据的名字和类型称为**字段**(field)  
Rust 不允许只将某个字段标记为可变  
结构体中字段为引用时需要使用[生命周期(lifetimes)](#10-生命周期)  

```rust
struct User
{
    name: String,
    age: i32,
}
fn main()
{
    
    let usr=User{
        name: "asdads".to_string(),
        age: 12
    };
}
```

- 字段初始化简写语法(field init shorthand)  
    当用于初始化的参数与字段名相同时，可以使用字段初始化简写语法  

    ```rust
    let name="aaa".to_string();
    let age=112;
    let usr=User{
        name,
        age,
    };
    ```

- 结构体更新语法(struct update syntax)  
    当从其余结构体对象创建新的对象时，可以仅列出值不同的字段  

    ```rust
    let usr2=Usr{
        age: 12,
        ..usr1
    }
    //println!("{}", usr1);
    println!("{}", usr1.age);
    ```

    对于堆字段通过移动进行拷贝，被移动的结构体会无效，但其余栈字段仍可单独访问  

##### 3.2.3.1 元组结构体(tuple struct)

元组结构体仅有结构体名称而没有具体的字段名  

```rust
struct Color(i32,i32,i32);
```

##### 3.2.3.2 方法Method

结构体内部的方法，第一个参数总是为 `&self` ，也可为 `&mut self` 或 `self`  
一个 `impl` 块内可定义多个方法，也可分散到多个 `impl` 块  

```rust
impl Usr
{
    fn func(&self)
    {
        println!("{}", self.name);
    }
}
```

> 自动引用和解引用(automatic referencing and dereferencing)  
> 当调用引用的方法时，会自动解引用为 `(*self).func()`  
>
- 关联函数  

    定义在 `impl` 块中但不以 `self` 为第一个参数的函数称为**关联函数**(associated function)  
    通过结构体名和 `::` 语法来调用关联函数  
    > 类比静态类成员函数  
    >

#### 3.2.4 枚举
>
> Rust 中的枚举更类似于标记联合体  

```rust
enum Color
{
    red, blue   //枚举类型
}
let c = Color::red;//枚举值
```

Rust枚举类型的枚举值可以与一种数据类型相关联，此时Rust会自动定义与该枚举值同名的函数，其以关联的数据类型为参数，返回枚举类型实例

```rust
enum Color
{
    red(u8,u8,u8), 
    white(u8), 
}
let r = Color::red(0xff, 0, 0);
let w = Color::white(0xff);
```  

#### 3.2.5 空值  

Rust 不使用 `null`，而是通过枚举类型 `Option::None` 来表达空值  
`Option` 变量要么有值 `Some(T)`，要么为空 `None`，并且显式地要求解构以确保用户使用前对空值进行了处理  

### 3.3 类型转换

NFY

## 4 函数

函数通过关键字 `fn` 声明，若有参数必须显式指定类型  

```rust
fn func(value: i32)->i32
{
    return 1;
}
```

- 语句和表达式  
    Rust 中**语句**(statement)指执行操作但不返回值的指令  
    **表达式**(expression)计算并产生一个值  

    表达式结尾没有 `;`，若加上 `;` 则构成了语句  

可以指定返回类型而不显式 `return`，通过结尾**表达式**隐式返回值  

```rust
fn func()->i32
{
    1
}
```

- 无返回值和不返回  
    若函数不显式指定返回值，则隐式返回空元组 `()`  
    永不返回的函数称为**发散函数**(diverse function)，其返回值标注为`!`

## 5 控制流

### 5.1 if 表达式

```rust
let num = 3;
if num<5
{
    //do sth
}else
{
    //do sth else
}
```

`if` 条件必须为布尔值  

`if` 为表达式，其可以有返回值，且每个分支的返回值必须为相同类型  

```rust
let num=if true {1} else{0};
```

### 5.2 loop循环

```rust
loop{
    //do sth
}
```

`break` 跳出当前循环，`continue` 跳过当前循环剩余代码，进行下一次循环  

- break可返回值

    ```rust
    let ret=loop{
        //do sth
        break 1;
    };
    ```

- 循环标签  
    多层循环 `break` 和 `continue` 默认作用域最内层循环  
    通过指定循环标签可以作用于指定循环  

    ```rust
    'label1: loop
    {
        'label2 loop
        {
            break 'label1;
        }
    }
    ```

### 5.3 while循环

```rust
let mut num=8;
while num!=0
{
    num-=1;
}
```

### 5.4 for遍历

```rust
let arr=[1,2,3,4];
for num in arr
{
    println!("value: {num}");_
}
//反向遍历
for num in (1..4).rev()
{
    println!("value: {num}");_    
}
```

相对于 while 遍历集合元素，for 消除了可能由于超出数组的结尾或遍历长度不够而缺少一些元素而导致的 bug  

底层是隐式地通过[迭代器](note_Rust.md#14.%20迭代器)实现的  
因而对于未实现 `Copy` trait 的类型元素，通过所有权遍历之后，该集合便不可再使用  

|syntax| equivalent syntax|ownership|
|- |- |-|
|for i in col | for i in col.into_iter()| 转移所有权|
|for i in &col| for i in col.iter() | 不可变引用|
|for i in &mut col| for i in col.iter_mut()| 可变引用   |

### 5.5 match

将一个值与一系列模式比较，根据匹配的模式执行相应代码  
匹配的表达式的结果值将作为 `match` 表达式的返回值  
`match` 可以用于从模式中取出绑定的数据，详见[枚举值](#324-枚举)  

分支部分格式为 `模式=>表达式，`  

```rust
match val
{
    1|2 =>{} //单分支多模式
    3..=4 =>{} //范围匹配
    5 =>{}
    _ => {}
}
```

`match` 分支必须为可穷尽的，要么列出**该类型**的所有可能值，要么使用变量或 `_` 关键字作为通配符  

- 匹配守卫  

    ```rust
    let x = Some(5);
    let y = 10;
    //...
    match x{
        Some(n) if n<y =>{};
    }
    ```

- matches!  
    将表达式与模式进行匹配，返回布尔值  

    ```rust
    enum pat
    {
        pat1,
        pat2,
    }
    fn main()
    {
        let a = pat::pat1;
        let ret: bool = matches!(a, pat::pat1);
    }
    ```

- if let  
    `match` **匹配**语法糖  

    ```rust
    match val
    {
        Some(value)=>{//do sth
        }
        _=>(),
    }
    //等价于
    if let Some(value) = val
    {
        //do sth
    }//else{}, 可省略
    ```
    
## 6 模式匹配

模式分为**可辨驳**(refutable)的与**不可辩驳**(irrefutable)的  

### 6.1 可辨驳性

可辨驳模式，可能无法匹配某些可能的值，即匹配可能失败  
以下场景允许可辨驳模式，但会对不可辩驳模式警告  

- match???
- if let  
- while let  

    ```rust
    let mut stk = Vec::new();
    while let Some(top) = stk.pop()
    {
        //do sth
    }
    ```

不可辩驳模式，能匹配任何可能传递的值，即确保匹配一定成功  
以下场景**仅**允许不可辩驳匹配  

- let  
- 函数与闭包参数  
- for  

    ```rust
    let arr = vec![1,2,3];
    for (index, val) in arr.iter().enumerate(){
        //do sth
    }
    ```

### 6.2 模式

在模式中，不允许使用 `&` 关键字，转而使用 `ref`  

`|`可用于匹配多个模式  

- 通配符模式(wildcard pattern)  
    `_`  

- 字面值模式(literal pattern)  

    字面值模式实现精确匹配，允许使用基本标量类型的字面值  
    > 浮点数类型的字面值模式将来会被废弃  

- 标识符模式(identifier pattern)  

    将匹配的值绑定给某个变量  
    该变量会隐藏作用域内的同名变量，其生命周期视上下文而定(如let和match中的变量作用域)  
    若该值的类型未实现 Copy trait，则会发生所有权转移  
    若将引用类型的值绑定到非引用模式，该模式的类型会自动转为引用  
- 范围模式(range pattern)  

    对于闭区间使用 `a..=b`; 对于开区间使用`a...b`，比range表达式多一个`.`  
- 引用模式(reference pattern)  

    引用模式必须为不可辩驳的  
- 结构体模式(struct pattern)  

### 6.3 解构

模式可用于解构/解包(destructuring)结构体、枚举、元组、数组和引用  
<!-- 
- 解构结构体

    ```rust
    struct Point{
        x: i32, y:i32,
    }
    fn main(){
        let p = Point{x:0, y:0};
        let Point{x: a, y:b} = p;
        //若模式变量名与字段名一致，可简写为
        let Point{x,y} = p;
    }
    ```

- 解构枚举

    ```rust
    enum Msg{
        Quit,
        Move {x: i32, y:i32}, //匿名结构体
        Color(i32, i32, i32)
    }
    //...
    match msg{
        Msg::Quit=>{},
        Msg::Move{x,y}=>{},
        Msg::Color(r,g,b)=>{},
    }
    ```

- 解构数组

    ```rust
    let arr =[1,2,3];
    let [x,y..] = arr;
    ```

    其中 `..` 忽略部分值，但每个模式中仅可出现一次 -->  

## 7. 所有权

不同于 C++ 手动管理内存，或 Java 的垃圾回收机制，Rust 通过**所有权**(ownership)管理内存  

1. Rust 中每一个值都有**所有者**(owner)  
2. 值在任一时刻只有一个所有者  
3. 当所有者离开作用域时，值将被丢弃  

### 7.1 内存分配

将堆上内存赋给另一个变量时执行**移动**(move)语义，即浅拷贝后使原先变量无效  
当持有堆中数据值的变量离开作用域时，其值将通过 `drop()` 被清理掉，除非数据被移动为另一个变量所有  
> 类似 C++ RAII  

- 移动  
    对于堆内存，默认复制方式为**移动**(move)，以避免二次释放问题  

    ```rust
    let str1=String::from("hello");
    let str2=str1;  //此时str1无效，不可再使用
    ```

    当离开作用域时，`str1` 已无效，仅为 `str2` 调用 `drop()` 函数  
- 深拷贝  
    Rust 堆上内存的深拷贝必须显式使用 `clone()` 函数  
- 直接复制  
    对于编译时已知大小的类型被整个存储在栈上，栈内存直接复制  

特别地，对于实现了 `Copy` trait 的类型，其复制方式将为深拷贝  
> 若一个类型或其部分实现了 `Drop` trait，则编译器禁止实现 `Copy` trait  

### 7.2 所有权与函数

当向函数传递堆变量或通过函数返回堆变量时，其堆内存会被移动到形参  
仅使用值而不转移所有权，使用**引用**(reference)  

### 7.3 引用与借用

```rust
let s1= String::from("hello");
let _s2 = &s1;
```

> 不同于 C++ 引用，Rust引用初始化时被引用的对象必须加 `&`  

引用使用值但不获取其所有权，创建引用的行为称为**借用**(borrowing)  
Rust语法保证引用总是有效的  

不同于变量作用域，引用作用域结束位置是其最后一次使用的位置而非 `}`  

#### 7.3.1 可变引用

引用默认为不可变，若要修改引用的对象需显式使用**可变引用**(mutable reference)  

```rust
let s1=String::from("hello");
let _s2=&mut s1;
```

可变引用与不可变引用不能同时借用  
同时只能存在一个可变引用  
该限制用以防止**数据竞争**(data race)
> 数据竞争指如下情况  
>
> - 两个或更多指针同时访问同一数据  
> - 至少有一个指针被用来写入数据  
> - 没有同步数据访问的机制  

#### 7.3.2 切片Slice

切片属于引用的一种，引用的对象为集合中一段连续的元素序列  

```rust
let str = String::from("hello world");
let hello=&s[0..5]
```

若索引从0开始，则0可省略；若引用包含集合的末尾元素，则 `..` 后可省略
字符串字面值数据类型为字符串切片 `&str`

## 8. 模块系统

Rust 管理代码的组织，包括哪些内容可以被公开，哪些内容作为私有部分，以及程序每个作用域中的名字的功能统称为**模块系统**(module system)  

### 8.1 包 package/项目 project

**包**(package)是提供一系列功能的一个或多个 crate  
其中包含一个 Cargo.toml 文件描述如何构建 crate  
package 中至少包含一个 crate，其中至多一个库 crate，以及任意个二进制 crate  

每个 package 的 module tree 与文件结构没有必然联系，必须显式指明  
初始情况下，module tree 中仅有 `main.rs` 或 `lib.rs`  

### 8.2 crate

crate 是 Rust 编译时的最小代码单位，crate 可以包含定义在其他文件的模块  
crate 有两种形式：可被编译为可执行程序的二进制项、以及库  
**crate 根节点**(crate root)指模块树的根节点，对于二进制 crate 为 `main.rs`，对于库 crate 为 `lib.rs`  

### 8.3 模块  

> Rust2018 之后不推荐使用 mod.rs 文件  
>
#### 8.3.1 声明模块

在 crate 根节点文件中声明新模块  

```rust
mod new_module_inline{
    mod new_sub_module_inline{
        fn fnc(){}
    }
    //modules...
}
mod new_module;
```

编译器会尝试在如下位置查找声明的模块  

- 当模块名后为大括号时，编译器在当前文件中查找模块  
- src/new_module.rs  
- src/new_module/mod.rs  

#### 8.3.2 声明子模块

在 crate 根节点之外的文件中声明的模块为子模块  

```rust
//new_module.rs
mod new_sub_module;
```

编译器会尝试在如下位置查找声明的模块

- 当模块名后为大括号时，编译器在当前文件中查找模块  
- src/new_module/new_sub_module.rs  
- src/new_module/new_sub_module/mod.rs  

### 8.4 路径

#### 8.4.1 pub

函数、方法、结构体、枚举、模块和常量默认对其父模块私有，父模块的项对子模块是公开的  
声明时使用 `pub` 关键字使子模块的项对父模块公开  
对于公有结构体，其字段仍为私有，需要单独声明；对于公有枚举，其枚举值也为公有  

当模块或子模块属于模块树的一部分时，之间可以互相引用代码  
其中 `crate` 为根节点，`super` 为父节点  

```rust
//new_sub_module.rs
let x: crate::new_module::New_type;//绝对路径
let y: super::new_module::New_type;//相对路径
```

#### 8.4.2 use

`use` 关键字将路径引入作用域  

```rust
use crate::new_module;
let z: new_module::New_type;

use crate::new_module::New_type;
let z1: New_type;

use crate::new_module::{New_type, New_type2};//引用多个项
use crate::new_module::*;//引用所有项
```

对于函数，通常指定到其父节点  
对于其他项，通常指定到完整路径，除非两个项名称相同必须通过父节点区分  

`as` 关键字为引用的路径指定本地别名  

```rust
use crate::new_module as NModule;
```  

通过 `use` 引用的路径，在当前作用域之外仍是私有的，使用 `pub use` 可以使其公开，称为**重导出**(re-exporting)  

## 9. 集合

### 9.1 Vector

- 新建 Vector  

    ```rust
    let v: Vec<i32> = Vec::new();//空Vector
    //let v = Vec<i32>::new();！！！报错
    let v = vec![1,2,3];//使用vec!宏
    ```

- 插入元素  

    ```rust
    let mut v = Vec::new();
    v.push(1);
    ```

    当向 Vector 中插入元素后，原先对 Vec 元素的引用会失效，因为新元素插入可能导致 Vector 重新分配空间并拷贝  
- 访问元素

    ```rust
    let a = v[0];
    let b: Option<&i32>= v.get(0);
    ```

    通过 `[]` 访问越界元素时会导致panic  
    通过 `get()` 访问越界元素时返回 `None`  

### 9.2 字符串

Rust 标准库的 `String` 是通过 `Vec<u8>` 实现的字节集合而非 `char` 集合，通常使用的是其切片 `&str`  
`String` 使用 UTF-8 编码  

- 新建 String  

    ```rust
    let stri1 = String::new();
    let mut stri2 = "string".to_string();
    let stri3 = String.from("string");
    ```

- 更新字符串  

    ```rust
    stri2.push_str("ssss");
    stri2.push('s');
    ```

    `push_str()` 参数类型为 `&str`，因而不会移动参数的所有权  
- 拼接字符串  

    ```rust
    let stri4 = stri2+&stri3;
    ```  

    `+` 运算符调用函数如下

    ```rust
    fn add(self, s: &str)->String;
    ```

    如上代码 `stri2` 所有权会被移动而被无效，`stri3` 仍可使用  
    `stri3` 由 `&String` 被强制转换为 `&str`  

- 字符串索引

    `String` 使用 UTF-8 编码，因而不允许通过索引访问数据  
    允许字符串切片  
- 遍历字符串  

    ```rust
    for c in "ssss".chars(){
        //do sth
    }
    for b in "avas".bytes(){
        //do sth
    }
    ```

### 9.3 哈希表

不同于 `Vec` 和 `String`，`HashMap` 不包含在 prelude 中  

- 新建 HashMap  

    ```rust
    use std::collections::HashMap;

    let mut map = HashMap::new();

    ```

- 插入元素  

    ```rust
    map.insert(1,10);
    ```

- 访问元素  

    ```rust
    let val = map.get(&1).copied().unwrap_or(0);
    ```  

    `copied()` 返回 `Option<i32>` 而非 `Option<&i32>`  
    `unwrap_or(0)` 在没有对应键时返回0  
- 更新哈希表  
    当使用相同键插入时，原先的值将被覆盖  

    `entry().or_insert()` 若键值对已经存在则就返回这个值的可变引用，若不存在则连同值一块插入  

    ```rust
    map.entry(&1).or_insert(0);
    ```

## 10. 泛型

通过泛型为函数签名、结构体、枚举、方法创建定义，使其可用于多种不同的具体数据类型  

```rust
fn find_max<T>(list: &[T])->&T{
    //return max in the array
}

struct Point<T>{
    x:T,
    y:T,
}
impl<T> Point<T>{
    fn fnc(&self, &T){
        //do sth
    }
}

enum Option<T>{
    Some(T),
    None
}
```

## 11. trait

`trait` 以一种抽象的方式定义共同行为  
> 类似于其他语言中**接口**(interface)的概念  
>
- 定义 trait  

    ```rust
    pub trait MyTrait{
        fn fnc_must_impl(&self);
    }
    ```

    可以为 trait 方法定义默认实现  

    ```rust
    pub trait MyTrait2{
        fn fnc_can_impl(&self){
            //do sth
        }
    }
    ```

- 实现 trait  

    ```rust
    pub struct MyStruct{
        pub data: i32;
    }

    impl MyTrait for MyStruct{
        fn fnc_must_impl(&self){
            //do sth
        }
    }
    ```

    若 trait 方法已有默认实现，则只需 `impl MyTrait2 for MyStruct{}`  
    > 不同于 C++，Rust 若重载了 trait 方法，则无法从重载方法中调用默认方法  

  - **相干性**(coherence)/**孤儿规则**(orphan rule)  
    不能为外部类型实现外部 trait，即类型和 trait 必须有一个属于当前 crate  

- trait 作为参数  
    在函数体中对于该参数只能调用指定 trait 的方法  

    ```rust
    //trait_bound语法
    pub fn fnc<T: MyTrait>(item: &T){
        //do sth
    }
    //语法糖
    pub fn fnc2(item: &impl MyTrait){
        //do sth
    }
    ```

    `+` 指定多个 trait  

    ```rust
    //trait_bound语法
    pub fn fnc<T: MyTrait+MyTrait2>(item: &T){
        //do sth
    }
    //语法糖
    pub fn fnc2(item: &impl MyTrait+MyTrait2){
        //do sth
    }
    ```

    当有多个 trait 参数时可通过 `where` 简化语法  

    ```rust
    pub fn fnc<T: MyTrait, U: MyTrait+MyTrait2>(t: &T, u: &U){
        //do sth
    }
    //where 语法
    pub fn fnc2<T, U>(t: &t, u: &U)
    where 
        T: MyTrait, 
        U: MyTrait+MyTrait2,{
            
        }
    ```

- 返回 trait 类型  

    ```rust
    pub fn fnc()->impl MyTrait{
        //return MyTrait
    }
    ```

- 使用泛型有条件的实现方法或 trait  

    ```rust
    impl <T: MyTrait> Point<T>{
        fn fnc_need_impl(&self){
            //do sth
        }
    }
    impl<T: MyTrait> MyTrait2 for T{
        //implement MyTrait2 only for MyTrait
    }
    ```

## 12. 宏

Rust 中的宏会展开为 AST (abstract syntax tree， 抽象语法树)，而非直接替换成源码  

```rust
macro_rules! macro_name
{
    ()=>(
        //do sth
    )
}
```

### 12.1 模式与指示符

宏的参数使用 `$` 为前缀，通过指示符(designator)注明类型  

```rust
maco_rules! func{
    ($func_def: ident)=>
    (
        fn $func_def(){
            //do sth
        }
    )
}
```

可用指示符如下所示：

- block
- expr 表达式
- ident 变量名或函数名
- item
- literal 字面常量
- pat 模式
- path
- stmt 语句
- tt 标记树
- ty 类型
- vis 可见性描述符  

### 12.2 宏重载

```rust
macro_rules! func{
    ($left: expr; and $right: expr)=>(
        //do sth
    );//; 不同重载必须以 ; 结尾
    ($left: expr; or $right: expr)=>(
        //do sth
    );
}
```

## 10. 生命周期

Rust 中的所有**引用**都有一个**生命周期**(lifetime)，表明该引用所指向变量的作用域范围  
通常情况下，生命周期都是隐式的，编译器会根据一些简单的规则自动推断  
复杂情况下，编译器无法准确的判断引用的生命周期，则需要手动标注  

- 生命周期标注基本语法  

    ```rust
    &'a i32 //显式生命周期引用
    &'a mut i32//显式生命周期可变引用
    ```

生命周期标注描述了多个引用的生命周期之间的关系，但并不影响生命周期，不会改变引用的生命周期长度  

### 10.1 函数生命周期标注

从函数返回引用时，返回的类型的生命周期至少要与其中一个参数的生命周期相匹配  
> 若返回的引用不指向参数，其只能指向函数内部创建的值，即为非法的悬垂引用  

其中 `<'a>` 称为泛型生命周期  
此处标注指示编译器返回值的生命周期小于等于相同标注的参数  

```rust
fn longer<'a, 'b>(x: &'a str, y: &'b str) -> &'a str {
    //do sth;
}
```

下方代码表明 `'b` 的生命周期大于等于 `'a`  

```rust
fn longer<'a, 'b: 'a>(x: &'a str, y: &'b str)->&'astr{
    //do sth;
}
```

### 10.2 结构体生命周期标注

结构体中的引用字段必须添加生命周期标注  

此处标注指示编译器引用字段的生命周期大于等于结构体  

```rust
struct MyStruct<'a>{
    data: &'a str;
}
```

### 10.3 生命周期省略

对于生命周期可预测并且遵循明确的模式的场景，其被编码进 Rust 引用分析并由编译器自动推断而无需手动标注  

- 输入生命周期  
    函数或方法参数的生命周期  
- 返回生命周期  
    返回值的生命周期  

编译器采用三条规则判断  

1. 编译器为每一个引用参数分配不同的生命周期参数  
2. 如果只有一个输入生命周期参数，它会被赋予所有输出生命周期参数  
3. 如果有多个输入生命周期参数且其中一个为 `&self` / `&mut self`，所有输出生命周期参数被赋予 `self` 的生命周期  

### 10.4 静态生命周期  

静态生命周期存活于整个程序期间  
字符串字面值均拥有 `'static` 生命周期  

```rust
let s:&'static str = "adad";
```

## 11. 错误处理

### 11.1 Panic! 处理不可恢复的错误

对于不可恢复的错误，可以调用 `panic!()` 使程序执行**展开**(unwinding)  

> 展开: 回溯栈并清理数据

默认情况下 `panic!()` 会展开，可通过在 `Cargo.toml` 的 `[profile]` 中切换为直接**终止**(abort)

```toml
[profile.release]
panic = 'abort'
```

### 11.2 Result 处理可恢复的错误

`Result` 定义如下  

```rust
enum Result<T, E>{
    Ok(T),
    Err(E),
}
```

通过 `match` 返回值类型以处理错误  

```rust
let file = match std::fs::File::open("filename"){
    Ok(file)=>file,
    Error(error)=>match error.kind() {
        //do sth
    },
}
```

`Result` 的 `unwrap()` 方法类似于上方代码，若为 `Ok` 则返回值，否则 `panic!()`  
`expect()` 方法类似于 `unwrap()` 但可指定 `panic!()` 参数  

```rust
let file = std::fs::File::open("filename").unwrap();
let file2 = std::fs::File::open("filename").expect("error info");
```

也可将错误**传播**(propagating)给调用者  

```rust
fn fnc()->Result<String, io::Error>{
    let file = match std::fs::File::open("filename"){
        Ok(file)=>/*do sth*/,
        Err(e)=> return Err(e),
    };
    //other process
}
```

如下代码为语法糖，要求函数返回值必须为 `Result` 或 `Option`  

```rust
fn fnc()->Result<String, io::Error>{
    let file = std::fs::File::open("filename")?;
    //other process
}
```

## 12. 自动化测试

Rust 中的测试函数用于验证非测试代码是否按照期望的方式运行  

测试函数体通常执行如下三种操作:  

1. 设置任何所需的数据或状态  
2. 运行需要测试的代码  
3. 断言其结果是我们所期望的  

### 12.1 测试函数

Rust 中的测试函数为带有 `#[test]` 属性注解的函数  
当测试函数中出现 panic 时表示测试失败  

```rust
#[cfg(test)]
mod tests{
    #[test]
    fn test_fnc(){
        //do sth
    }
}
```

### 12.2 检查结果

- `assert!()`  
    其接收布尔型参数，为 `true` 不会执行任何操作，当为 `false` 时会调用 `panic!()`  
    可选参数为格式字符串，用于输出错误信息  
- `assert_eq!()`、`assert_ne!()`  
    接受两个参数并比较是否相等/不等  
    相比于 `assert!()`，这两个宏会输出两个参数的具体值  
    可选参数为格式字符串，用于输出错误信息  
- `#[should_panic]`  
    带有该属性注解的测试函数在没有panic时会失败  
    可选参数为错误信息字符串  

    ```rust
    #[test]
    #[should_panic(expected = "this is an error info string" )]
    fn fnc(){
        //do sth
    }
    ```

- [Result](#112-result-处理可恢复的错误)  
    使用 `Result` 测试时，不能使用 `#[should_panic]` 以及 `?` 语法糖  

    ```rust
    fn fnc()->Result<(), String>{
        //do sth
        if a==b{
            Ok(())
        }else{
            Err("This is an error info string".to_string())
        }
    }
    ```

### 12.3 执行测试

当通过 `cargo test` 运行测试时，Rust 默认使用多线程并行执行多个测试函数  

- 若测试函数互相依赖或共享资源而无法并行执行，可以指定线程数为1  

    ```shell
    cargo test -- --test-threads=1
    ```

- 当测试通过时，Rust 会截获所有输出到标准输出的内容，仅在失败时不会截获  
    可以指定其在任何情况下都输出被截获的内容  

    ```shell
    cargo test -- --show-output
    ```

- 仅执行特定测试  
    仅会执行函数名带有 `<pattern>` 的测试函数  

    ```shell
    cargo test <pattern>
    ```

- 忽略特定测试  

    ```rust
    #[test]
    #[ignore]
    fn fnc(){
        //do sth
    }
    ```

    通过 `cargo test` 测试时，默认忽略带有 `#[ignore]` 属性的测试函数  

    若仅执行带有 `#[ignore]` 属性的测试函数  

    ```shell
    cargo test -- --ignored
    ```  

    若执行所有测试函数  

    ```shell
    cargo test -- --include-ignored
    ```

### 12.4 测试的组织结构

NFY

## 13. 闭包

Rust 闭包的实现类似于 C++，通过匿名结构体实现，结构体中存储捕获的变量  

### 13.1 语法

```rust
let my_closure = |parameter: i32|->i32{
    //do sth;
};
```

闭包通常很短，并只关联于小范围的上下文而非任意情境，编译器能根据**函数体内容**自动推断参数类型和返回值类型，因而闭包不严格要求注明参数类型和返回值类型  

### 13.2 捕获环境

指闭包将捕获的变量如何存储在匿名结构体中  
闭包会通过函数体内容自动推断捕获哪些变量  

闭包通过三种方式捕获其环境，按优先级为  

1. 不可变引用  
2. 可变引用  
3. 所有权转移  

使用可变引用捕获时，闭包本身必须被声明为 `mut`  
`move` 关键字强制闭包通过所有权捕获变量  

```rust
let my_closure = move ||{
    //do sth
    };
```

### 13.3 Fn trait

编译器会根据函数及闭包在调用时的行为自动实现 Fn trait  

Fn trait 仅与闭包体中如何使用变量有关，与如何捕获变量无关，`move` 关键字不会影响 Fn trait 的实现  

任何**函数**均实现了以下三种 trait  

1. FnOnce  
    适用于能被至少调用一次的闭包，所有闭包都至少实现了这个 trait  
    若闭包通过上下文变量的所有权访问该变量，则编译器只为闭包实现 `FnOnce`  

    ```rust
    pub trait FnOnce<Args> {
        type Output;
        extern "rust-call" fn call_once(self, args: Args) -> Self::Output;
    }
    ```

2. FnMut  
    适用于可被多次调用的闭包  
    若闭包仅通过引用而非所有权访问上下文变量，则编译器为其实现 `FnMut`

    ```rust
    pub trait FnMut<Args>: FnOnce<Args> {
        extern "rust-call" fn call_mut(&mut self, args: Args) -> Self::Output;
    }
    ```

3. Fn  
    适用于可被调用多次而不改变环境的闭包，常用于多并发场景  
    若闭包仅通过不可变引用访问上下文变量，则编译器为其实现 `Fn`  
    实现 `Fn` 的闭包必定实现了 `FnOnce` 和 `FnMut`  

    ```rust
    pub trait Fn<Args>: FnMut<Args> {
        extern "rust-call" fn call(&self, args: Args) -> Self::Output;
    }
    ```

## 14. 迭代器

**迭代器**(iterator)负责遍历序列中的每一项和决定序列何时结束的逻辑
Rust 的迭代器是惰性求值的  

`iter()` 方法返回不可变引用的迭代器  
`iter_mut()` 方法返回可变引用的迭代器  
`into_iter` 方法获取序列元素所有权并返回拥有所有权的迭代器

### 14.1 next()

迭代器都实现了 `Iterator` trait  
其中 `next()` 方法要求必须实现  
`next()` 方法一次返回迭代器中的一个项，当迭代结束时，返回 `None`  

```rust
pub trait Iterator{
    type Item;
    fn next(&mut self)->Option<Self::Item>{
        //default implementation
    }
    //other methods
}
```  

`next()` 方法会改变迭代器中记录序列位置的状态，称为**消费**(consume)或使用了迭代器  
`Iterator` 还有一些提供了默认实现的方法，其中部分的默认实现调用了 `next()` 方法，称为**消费适配器**(consuming adaptors)  

### 14.2 迭代器适配器

`Iterator` trait 定义了将迭代器转换为不同类型迭代器的方法，称为**迭代器适配器**(iterator adaptors)  

## 标准库

### 智能指针

Rust 智能指针通过结构体实现，并实现了 `Deref` 和 `Drop` trait

#### Box\<T\>

```rust
let b = Box::new(5);
```

#### Deref trait

#### Drop trait

```rust
pub trait Drop
{
    fn drop(&mut self);
}
```

等价于析构函数，在对象离开作用域时自动调用  

### 线程

```rust
use std::thread;
```

Rust 标准库使用1:1的线程实现，每一个语言级线程对应着一个系统级线程  

#### 线程创建

- 调用 `std::thread::spawn()` 并传递一个闭包以创建线程  

```rust
std::thread::spawn(||{
    //do sth;
    });
    ```

- `join` 等待线程结束

    ```rust
    let handle = thread::spawn(||{
        thread::sleep(Duration::from_millis(1));
    });
    handle.join().unwrap();
    ```

- `move` 闭包

#### 线程通信

信道通信类似于单所有权，一旦一个值被传送到信道中，发送者将无法再使用  
共享内存类似于多所有权，索格现场可以同时访问相同的内存位置，但这会增加额外的复杂性  

##### 消息传递

Rust 线程通过**信道**(channel)来进行**消息传递**(message passing)，以实现线程间的通信  

- 创建多生产者，单消费者信道  

    ```rust
    let (tx, rx) = std::sync::mpsc::channel;
    thread::spawn(move||{
        tx.send(1).unwrap();
    });

    let recv_val = rx.recv().unwrap();
    ```

    其中，`tx` 为**发送者**(transmitter)，`rx` 为**接收者**(receiver)  

    `send()` 方法返回值为 `Result<T, E>` 类型，若接收端已被丢弃，则发送失败  
    `send()` 方法会获取参数所有权至接收者，以避免数据竞争问题  

    `recv()` 方法会阻塞接收者所在线程直至从信道中接受一个值  
    `try_recv()` 方法会立即返回，若有消息则返回 `Ok`，否则 `Err`  

    ```rust
    let tx1 = tx.clone();
    ```

    通过克隆发送者以创建多个生产者  

##### 共享内存并发

Rust 通过**互斥锁**(mutex)来进行共享内存通信  

```rust
let m = std::sync::Mutex::new(5);
{
    let mut num = m.lock().unwrap();
    *num = 6;
}
```

`lock()` 方法会阻塞当前线程直至拥有锁  
其返回值为 `MutexGuard` 的智能指针，当离开作用域时会调用 `Drop()` 释放锁  
