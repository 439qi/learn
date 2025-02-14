---
title: CPP Standard
aliases:
  - C++ 标准
  - C++ Standard
  - CPP 标准
tags:
  - cpp
---
## C++11  

- decltype
- declval
- enum class
- 用户自定义字面量 `operator"" _`
- trailing return type  
- std::atomic
- [std::once_flag, std::call_once](note_cpp_thread.md/#stdonce_flag-stdcall_once)
- [引用限定符](note_cpp.md/#引用限定符)

## C++14

- std::make_index_sequence

## C++17

- [guaranteed copy elision](note_cpp_move.md/#纯右值-prvalue)
- string_view
- [类模板参数自动推导 Class template argument deduction, CTAD](note_cpp.md/#类模板参数自动推导class-template-argument-deduction-ctad)
- if constexpr
- std::mem_fun deprecated
- inline variables
- lambda cpature this by value
- structured bindings
- [overload pattern](note_cpp.md/#重载模式overload-pattern)

## C++20

- [concept](note_cpp.md/#concept)
- requires
- [std::ranges, std::ranges::views/std::views](CPP_20/Range_View.md)
- [module](CPP_20/Module.md)
- [coroutine](note_async.md/#cpp-coroutines)
- consteval  
- new delete at compile time
- constinit  
- lambda 支持模板
- operator<=>
- std::span
- std::mem_fn
- std::jthread
- std::latch, std::barrier
- function defined inside class/struct/union definition is not implicit inline if attached to a named module

## C++23

- std::expected
- [显式this](note_cpp.md/#显式-this-指针-deducing-this)
- auto(x) decay copy
- std::unreachable

## C++26  
- [static reflection](CPP_26/Static_Reflection.md)  
## Future

- transaction memory
