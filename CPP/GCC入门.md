# GCC入门

大家都知道`C\C++`编译流程都是：预处理 -> 编译 -> 汇编 -> 链接。而这些工作在Linux下都是使用GCC完成的，所以做`C++`开发，必须要懂得怎么使用GCC。

## 编译参数

- `-c`, 对源文件进行编译，不链接，生成目标文件。
- `-o`, 输出最终目标文件。
- `-g`, 给编译文件加入调试信息，以便使用GDB调试。