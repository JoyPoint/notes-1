# 在CLI环境下使用LLDB调试程序

我们一般调试程序都是使用IDE来调试，但是有时候我们没法用到IDE，比如在Linux服务器上，或者IDE上压根没有集成这种语言的调试器，这时候我们就需要用到CLI环境上启动调试器来调试啦。最近因为要写C和CPP的代码，而在Mac上用不惯Xcode和Clion(还是习惯用Vim=_+)，所以只能在开个iTerm用`lldb`来调了。

## 开始调试程序

1. 直接用`lldb`来启动调试的程序

```sh
lldb app           # app是编译出来的可执行文件
```

2. 调试带参数的程序

```sh
lldb -- app 1 2 3  # 等于./app 1 2 3
```

3. 调试正在运行的进程

```sh
lldb               # 先启动LLDB
```

进入以后，可以使用`lldb`内部命令挂到运行中的进程上去：

```sh
# 然后你找一个进程的pid出来，我找的是QQ音乐
# 你可以ps aux | grep casa （casa是我的用户名） 在这个列表里面挑一个程序的pid
# 输入process attach --pid 你找到的pid，来把调试器挂到这个进程上去调试
(lldb) process attach --pid 9939        # 简写命令: attach -p 9939
                                        #       或  pro att -p 9939
# 你也可以告诉lldb你要挂在那个进程名下
(lldb) process attach --name Safari     # 简写命令: attach -n Safari
                                                或  pro att -n Safari
```

## 查看代码

如果构建的程序是在`Debug`模式下构建的，那么就可以使用`lldb`来查看程序的代码。

1. 用`list`命令看代码，简写为`l`，一直`list`的话，就会往下显示代码。如果想从M行开始看的话，直接`list M`就可以了。
2. 要看其它文件的代码，只需要`list file`就可以了，之后按`list`的话，会自动从该文件往下看。
3. 要看某个函数的代码，只需要`list method`既可。

## 下断点

调试必须要做的就是打断点啦，不然就会眼花花的看着程序跑过去咯。CLI下打断点是要打命令的，这点是比IDE麻烦，不过也麻烦不了哪里去。

1. 根据文件名和行号下断点。

```sh
(lldb) breakpoint set --file DebugDemo.c --line 10
Breakpoint 1: where = DebugDemo.run`main + 92 at DebugDemo.c:10, address = 0x0000000100000a7c
```

2. 根据函数名下断点

```sh
# C函数
(lldb) breakpoint set --name main

# C++类方法
(lldb) breakpoint set --method foo

# Objective-C选择器
breakpoint set --selector alignLeftEdges:
```

3. 根据某个函数调用语句下断点(Objective-C比较有用)

```sh
# lldb有一个最小子串匹配算法，会知道应该在哪个函数那里下断点
breakpoint set -n "-[SKTGraphicView alignLeftEdges:]"
```

4. 给断点设置别名

```sh
# 比如下面的这条命令
(lldb) breakpoint set --file DebugDemo.c --line 10

# 你就可以写这样的别名
(lldb) command alias bfl breakpoint set -f %1 -l %2

# 使用的时候就像这样就好了
(lldb) bfl DebugDemo.c 10
```

5. 查看断点列表、启用/禁用断点、删除断点

```sh
# 查看断点列表
(lldb) breakpoint list

Current breakpoints:
1: file = 'DebugDemo.c', line = 10, locations = 1
1.1: where = DebugDemo.run`main + 92 at DebugDemo.c:10, address = DebugDemo.run[0x0000000100000a7c], unresolved, hit count = 0
2: name = 'main', locations = 1
2.1: where = DebugDemo.run`main + 68 at DebugDemo.c:8, address = DebugDemo.run[0x0000000100000a64], unresolved, hit count = 0

# 禁用断点
# 根据上面查看断点列表的时候的序号来操作断点
(lldb) breakpoint disable 2
1 breakpoints disabled.
(lldb) breakpoint list
Current breakpoints:
1: file = 'DebugDemo.c', line = 10, locations = 1
1.1: where = DebugDemo.run`main + 92 at DebugDemo.c:10, address = DebugDemo.run[0x0000000100000a7c], unresolved, hit count = 0
2: name = 'main', locations = 1 Options: disabled 
2.1: where = DebugDemo.run`main + 68 at DebugDemo.c:8, address = DebugDemo.run[0x0000000100000a64], unresolved, hit count = 0

# 启用断点
(lldb) breakpoint enable 2
1 breakpoints enabled.
(lldb) breakpoint list
Current breakpoints:
1: file = 'DebugDemo.c', line = 10, locations = 1
1.1: where = DebugDemo.run`main + 92 at DebugDemo.c:10, address = DebugDemo.run[0x0000000100000a7c], unresolved, hit count = 0
2: name = 'main', locations = 1
2.1: where = DebugDemo.run`main + 68 at DebugDemo.c:8, address = DebugDemo.run[0x0000000100000a64], unresolved, hit count = 0

# 删除断点
(lldb) breakpoint delete 1
1 breakpoints deleted; 0 breakpoint locations disabled.
(lldb) breakpoint list
Current breakpoints:
2: name = 'main', locations = 1
2.1: where = DebugDemo.run`main + 68 at DebugDemo.c:8, address = DebugDemo.run[0x0000000100000a64], unresolved, hit count = 0
```

## 启动调试程序

- 启动
设置好断点我们就可以开始调试程序了，启动程序很简单，直接一条`run`命令既可。

```sh
# run命令就是启动程序
(lldb) run
Process 11500 launched: '/Users/casa/Playground/algorithm/leetcode/DebugDemo/DebugDemo.run' (x86_64)
Process 11500 stopped   # 这里执行到断点了
* thread #1: tid = 0x1af357, 0x0000000100000a64 DebugDemo.run`main + 68 at DebugDemo.c:8, queue = 'com.apple.main-thread', stop reason = breakpoint 2.1
    frame #0: 0x0000000100000a64 DebugDemo.run`main + 68 at DebugDemo.c:8
   5    void yell(void);
   6    
   7    int main () {
-> 8        size_t result_array[2] = {0, 0};  # 这一行前面的箭头表示调试器在这里停住了
   9    
   10       FILE *file_handler = fopen("TestData", "r");
   11       json_error_t error;
        }
```

- 下一步、步入、步出、继续执行

```sh
# 下一步 (next 或 n)
(lldb) next
Process 11500 stopped
* thread #1: tid = 0x1af357, 0x0000000100000a7c DebugDemo.run`main + 92 at DebugDemo.c:10, queue = 'com.apple.main-thread', stop reason = step over
    frame #0: 0x0000000100000a7c DebugDemo.run`main + 92 at DebugDemo.c:10
   7    int main () {
   8        size_t result_array[2] = {0, 0};
   9    
-> 10       FILE *file_handler = fopen("TestData", "r");
   11       json_error_t error;
   12   
   13       json_t *root = json_loadf(file_handler, JSON_DECODE_ANY, &error);

#---------------------------------------------------------------------------

# 步入(step 或 s)
Process 11668 stopped
* thread #1: tid = 0x1b4e9d, 0x0000000100000c06 DebugDemo.run`main + 486 at DebugDemo.c:29, queue = 'com.apple.main-thread', stop reason = breakpoint 3.1
    frame #0: 0x0000000100000c06 DebugDemo.run`main + 486 at DebugDemo.c:29
   26               printf("input array is %"JSON_INTEGER_FORMAT"\n", json_integer_value(item));
   27           
                }
   28   
-> 29           yell();
   30   
   31           json_array_foreach(input_array, index, item) {
   32

(lldb) step
Process 11668 stopped
* thread #1: tid = 0x1b4e9d, 0x0000000100000e1f DebugDemo.run`yell + 15 at DebugDemo.c:57, queue = 'com.apple.main-thread', stop reason = step in
    frame #0: 0x0000000100000e1f DebugDemo.run`yell + 15 at DebugDemo.c:57
   54   
   55   void yell()
   56   {
-> 57       printf("here i am yelling\n");
   58   
        }

#---------------------------------------------------------------------------

# 步出(finish)
(lldb) finish
here i am yelling
Process 11668 stopped
* thread #1: tid = 0x1b4e9d, 0x0000000100000c0b DebugDemo.run`main + 491 at DebugDemo.c:31, queue = 'com.apple.main-thread', stop reason = step out
    frame #0: 0x0000000100000c0b DebugDemo.run`main + 491 at DebugDemo.c:31
   28   
   29           yell();
   30   
-> 31           json_array_foreach(input_array, index, item) {
   32   
   33               result_array[0] = index;
   34               json_int_t value = json_integer_value(item);

#---------------------------------------------------------------------------

# 继续执行到下一个断点停, 后面没有断点的话就跑完了（continue 或 c）
(lldb) continue
Process 11668 resuming
result is 2, 3
Process 11668 exited with status = 0 (0x00000000)

#---------------------------------------------------------------------------
                                                                                                                                             }
                }
```

- 查看变量、跳帧查看变量

```sh
# 使用po或p，po一般用来输出指针指向的那个对象，p一般用来输出基础变量。普通数组两者都可用
Process 11717 stopped
* thread #1: tid = 0x1b7afc, 0x0000000100000a7c DebugDemo.run`main + 92 at DebugDemo.c:10, queue = 'com.apple.main-thread', stop reason = step over
    frame #0: 0x0000000100000a7c DebugDemo.run`main + 92 at DebugDemo.c:10
   7    int main () {
   8        size_t result_array[2] = {0, 0};
   9    
-> 10       FILE *file_handler = fopen("TestData", "r");
   11       json_error_t error;
   12   
   13       json_t *root = json_loadf(file_handler, JSON_DECODE_ANY, &error);

(lldb) po result_array
([0] = 0, [1] = 0)

(lldb) p result_array
(size_t [2]) $2 = ([0] = 0, [1] = 0)

#---------------------------------------------------------------------------

# 查看所有帧(bt)
(lldb) bt
* thread #1: tid = 0x1bb7dc, 0x0000000100000e1f DebugDemo.run`yell + 15 at DebugDemo.c:57, queue = 'com.apple.main-thread', stop reason = step in
  * frame #0: 0x0000000100000e1f DebugDemo.run`yell + 15 at DebugDemo.c:57
    frame #1: 0x0000000100000c0b DebugDemo.run`main + 491 at DebugDemo.c:29
    frame #2: 0x00007fff99d175c9 libdyld.dylib`start + 1

#---------------------------------------------------------------------------

# 跳帧（frame select）
(lldb) frame select 1
frame #1: 0x0000000100000c0b DebugDemo.run`main + 491 at DebugDemo.c:29
   26               printf("input array is %"JSON_INTEGER_FORMAT"\n", json_integer_value(item));
   27           }
   28   
-> 29           yell();
   30   
   31           json_array_foreach(input_array, index, item) {
   32

#---------------------------------------------------------------------------

# 查看当前帧中所有变量的值（frame variable）
(lldb) frame variable

(size_t [2]) result_array = ([0] = 0, [1] = 0)
(FILE *) file_handler = 0x00007fff7cfdd070
(json_error_t) error = (line = 0, column = 0, position = 0, source = "", text = "?)
(json_t *) root = 0x0000000000000000
                                                                                                                                                                                                          
#--------------------------------------------------------------------------
```

以上就是`lldb`的基本用法，详细文档可以看下官方文档：http://lldb.llvm.org/tutorial.html
