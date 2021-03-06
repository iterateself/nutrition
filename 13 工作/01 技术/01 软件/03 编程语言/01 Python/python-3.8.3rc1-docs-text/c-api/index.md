Python/C API 参考手册
*********************

本手册描述了希望编写扩展模块并将 Python 解释器嵌入其应用程序中的 C 和
C++ 程序员可用的 API。同时可以参阅 扩展和嵌入 Python 解释器 ，其中描述
了扩展编写的一般原则，但没有详细描述 API 函数。

* 概述

  * 代码标准

  * 包含文件

  * 有用的宏

  * 对象、类型和引用计数

  * 异常

  * 嵌入Python

  * 调试构建

* 稳定的应用程序二进制接口

* The Very High Level Layer

* 引用计数

* 异常处理

  * Printing and clearing

  * 抛出异常

  * Issuing warnings

  * Querying the error indicator

  * Signal Handling

  * Exception Classes

  * Exception Objects

  * Unicode Exception Objects

  * Recursion Control

  * 标准异常

  * 标准警告类别

* 工具

  * 操作系统实用程序

  * 系统功能

  * 过程控制

  * 导入模块

  * 数据 marshal 操作支持

  * 语句解释及变量编译

  * 字符串转换与格式化

  * 反射

  * 编解码器注册与支持功能

* 抽象对象层

  * 对象协议

  * 数字协议

  * 序列协议

  * 映射协议

  * 迭代器协议

  * 缓冲协议

  * 旧缓冲协议

* 具体的对象层

  * 基本对象

  * 数值对象

  * 序列对象

  * 容器对象

  * 函数对象

  * 其他对象

* Initialization, Finalization, and Threads

  * 在Python初始化之前

  * 全局配置变量

  * Initializing and finalizing the interpreter

  * Process-wide parameters

  * Thread State and the Global Interpreter Lock

  * Sub-interpreter support

  * 异步通知

  * 分析和跟踪

  * 高级调试器支持

  * Thread Local Storage Support

* Python初始化配置

  * PyWideStringList

  * PyStatus

  * PyPreConfig

  * Preinitialization with PyPreConfig

  * PyConfig

  * Initialization with PyConfig

  * Isolated Configuration

  * Python Configuration

  * 路径配置

  * Py_RunMain()

  * Multi-Phase Initialization Private Provisional API

* 内存管理

  * 概述

  * 原始内存接口

  * 内存接口

  * 对象分配器

  * 默认内存分配器

  * Customize Memory Allocators

  * The pymalloc allocator

  * tracemalloc C API

  * 示例

* 对象实现支持

  * 在堆中分配对象

  * Common Object Structures

  * Type 对象

  * Number Object Structures

  * Mapping Object Structures

  * Sequence Object Structures

  * Buffer Object Structures

  * Async Object Structures

  * Slot Type typedefs

  * 例子

  * 使对象类型支持循环垃圾回收

* API 和 ABI 版本管理
