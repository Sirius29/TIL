### GDB 常用指令

#### 编译程序带上调试信息

在利用gdb调试程序前，需要带上调试符号编译程序 -g / -ggdb

#### 启动gdb

- 不带参数运行
``` shell
gdb program
run
```

- 带参数运行
``` shell
# method - 1
gdb --args program arg1 arg2 ...
run

# method - 2
gdb program
run arg1 arg2 ...
```

#### 常用指令
- 打印源码 **list [linenum / function]**
  - list 12 / list Func
- 设置断点 **break [(filename:)linenum / (filename:)function] if condition**
  - break 12 / break func
  - b test:18 / b test:UnitTest
  - b test:18 if t==0
- 打印变量 **p variable**
- **c** 继续执行
- **step / s** 单步执行，进入函数
- **next / n** 单步执行，不进入函数
- **info breakpoints** 查看断点信息
- **d breakpoint_num** 删除断点
- **bt** 打印函数调用栈
