
> [!cite]- References  
> <https://www.cnblogs.com/wwwbdabc/p/10861680.html>  
> <https://www.nowcoder.com/discuss/353159042002526208>  
> <https://cloud.tencent.com/developer/article/1795692>  
> <https://blog.kitlau.dev/posts/what-is-asynchronous-is-it-multi-threading-or-async-await/>  
> [Writing custom C++20 coroutine systems](<https://www.chiark.greenend.org.uk/~sgtatham/quasiblog/coroutines-c++20/>)  
> [如何理解 C++11 的六种 memory order？ - 知乎用户的回答 - 知乎](https://www.zhihu.com/question/24301047/answer/1193956492)  


**它本身与多线程无关，是限制的单一线程当中指令执行顺序**。但是（合理的）指令执行顺序的重排在单线程环境下不会造成逻辑错误而在多线程环境下会，所以这个问题的**目的是为了解决多线程环境下出现的问题**。