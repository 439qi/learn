# 互联网打印协议(Internet Printing Protocol)
>
> [IPP/1.1: Model and Semantics](https://datatracker.ietf.org/doc/html/rfc8011)  

基于[HTTP/TCP](note_network.md)协议的打印协议

## IPP的简化打印模型

IPP模型将分布式打印中的重要组件抽象并封装为如下对象  
> IPP模型仅抽象了一些重要组件，没有显式定义实际系统实现中的其余组件

每种打印对象与一系列操作集合相关联

1. 打印机(printer)  
    封装了与物理输出设备相关的功能，以及假脱机(spooling)，调度，多设备管理这类与打印服务器相关的功能  
    打印机对象可被被注册为目录中的条目的形式，这种形式主要存储打印机的静态信息
2. 作业(job)
3. 文档(document)
4. (subscription)
