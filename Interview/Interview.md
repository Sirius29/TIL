### Interview

#### STAR法则
- Situation: 事情发生的背景，项目介绍
- Task: 明确自己的任务、职责
- Action: 为了完成任务，采取的行动、策略和方法
- Result: 最终取得的成果，最好可以通过数据进行量化
- Think：做这件事的反思、收获

#### CPP
- 结构化绑定
``` CPP
for (auto &[key, value] : map) {
    // ...
}
```
- Copy Elision
CPP17 强制复制省略，主要包括**返回值优化**和**右值拷贝优化**。在 17 之前返回临时对象会检查是否存在拷贝构造或移动构造来进行优化，17 之后则直接进行初始化，彻底进行复制消除。
- 左值、右值
拥有身份且不可被移动的表达式被称作 **左值** (lvalue) 表达式，指持久存在的对象或类型为左值引用类型的返还值。
拥有身份且可被移动的表达式被称作 **将亡值** (xvalue) 表达式，一般是指类型为右值引用类型的返还值。
不拥有身份且可被移动的表达式被称作 **纯右值** (prvalue) 表达式，也就是指纯粹的临时值（即使指代的对象是持久存在的）。
左值(lvalue)：表达式结束后依然存在的持久对象。指持久存在（有变量名）的对象或返还值类型为左值引用的返还值，是不可移动的。
右值(rvalue)：表达式结束后就不再存在的临时对象。包含了将亡值、纯右值，是可移动（可被右值引用类型匹配）的值。
范左值：包含了将亡值、左值。
右值引用类型负责匹配右值，左值引用则负责匹配左值。
- 移动语意、万能引用、完美转发
万能引用：发生类型推导（例如模板、auto）的时候，使用T&&类型表示为万能引用，否则表示右值引用。
完美转发 - std::forward<T>：使用万能引用时，即使可以同时匹配左值、右值，但需要转发参数给其他函数时，会丢失引用性质（形参是个左值，从而无法判断到底匹配的是个左值还是右值）。完美转发函数 std:forward<T> 以在模板函数内给另一个函数传递参数时，将参数类型保持原本状态传入（如果形参推导出是右值引用则作为右值传入，如果是左值引用则作为左值传入）。
完美转发的本质是是类型转换，利用引用折叠规则，将右值引用类型的值返还（返还值为右值）。接受左值引用类型时，将左值引用类型的值返还（返还值为左值）。
移动语意 - std::move<T>()：本质是类型转换，先移除形参的所有引用性质得到无引用性质的类型，保证不会发生引用折叠，然后转换为右值返回。
- 模板元编程 - 编译期常量，不需要 static 或 enum
如 std::is_same
``` CPP
// 两个不同类型，返回false
template <class _Tp, class _Up> struct _LIBCPP_TEMPLATE_VIS is_same           : public false_type {};
// 位相同类型写一个特化版本，如下：
template <class _Tp>            struct _LIBCPP_TEMPLATE_VIS is_same<_Tp, _Tp> : public true_type {};

typedef _LIBCPP_BOOL_CONSTANT(true)  true_type;
typedef _LIBCPP_BOOL_CONSTANT(false) false_type;

template <bool __b>
using bool_constant = integral_constant<bool, __b>;  
#define _LIBCPP_BOOL_CONSTANT(__b) bool_constant<(__b)>  // 这里定义了上面的宏

template <class _Tp, _Tp __v>
struct _LIBCPP_TEMPLATE_VIS integral_constant
{
  static _LIBCPP_CONSTEXPR const _Tp      value = __v;
  typedef _Tp               value_type;
  typedef integral_constant type;
  _LIBCPP_INLINE_VISIBILITY
  _LIBCPP_CONSTEXPR operator value_type() const _NOEXCEPT {return value;}
#if _LIBCPP_STD_VER > 11
  _LIBCPP_INLINE_VISIBILITY
  constexpr value_type operator ()() const _NOEXCEPT {return value;}
#endif
};
```
- 虚函数
类的继承中，每个类中都会生成一堆指向虚函数的指针，放在虚表中。对于每一个类对象，都会有存在一个虚指针，指向虚表。
- RTTI原理：虚表中存在一个额外的指针，指向typeinfo。[RTTI](https://zhuanlan.zhihu.com/p/150579874)
- dynamic_cast：通过运行期类型判断，将基类安全的向子类转换
- 虚基类、虚继承、菱形继承
当出现菱形继承时，子类会存在多个基类的数据，这时通过虚继承基类只保留一份基类数据。
- override、final
override可以提供编译期检查，保证子类重写的函数和基类虚函数有相同的函数签名
final可以禁止子类继承，提供编译优化，当编译器知道该类不会被继承，调用函数时可以不通过虚表。

#### 体系结构
- 虚拟内存
- cache结构
[Cache](Cache.md)
- 伪共享
伪共享即存在两个线程运行在 CPU0 和 CPU1 上，分别读取同一个 cache line 中不同的两个数据。thread A 读取并修改了变量 a，此时 CPU0 更新 cache line 并置位 dirty。thread B 在读取变量 b 时，需要 CPU0 更新数据到主存，再从主存读取到 CPU1 的 cache line 中，这样 thread B 读取到的数据就是脏数据。
如果两个线程交替更新这两个数据，这就是典型的 cache 颠簸导致性能下降，这就伪共享问题。
避免伪共享的方法就是将这两个数据进行内存对齐，放入不同的两个 cache line。
[伪共享](https://zhuanlan.zhihu.com/p/124974025)
- 为什么申请 DMA buffer 需要内存对齐
如果 DMA buffer 没有内存对齐。那么 DMA 传输时，需要跨越多个 cache line，这时去操作其中未对齐的无关数据或 cache line 发生替换，会将读取到无关数据。
[Cache和DMA一致性](https://zhuanlan.zhihu.com/p/109919756)

#### Python
- 闭包
在函数中可以（嵌套）定义另一个函数时，如果内部的函数引用了外部的函数的变量，则可能产生闭包。闭包可以用来在一个函数与一组“私有”变量之间创建关联关系。在给定函数被多次调用的过程中，这些私有变量能够保持其持久性。闭包的作用：1、可以读取函数内部的变量；2、让这些变量的值始终保持在内存中。

#### 其他
- 随机数生成
rand5() 随机获取 1-5，写一个函数可以随机获取 1-7
``` CPP
int rand7() {
    int ret = INT_MAX;
    do {
        ret = (rand5() - 1) * 5 + rand5();
    } while (ret > 21) 
    return ret % 7 + 1;
```
- 未知芯片如何知道缓存大小
创建一个连续内存块，进行连续、大量的随机内存访问，当内存大小可以被 cache 容纳时，访问速度会显著的块，当内存大小大于 cache 容量时，访问速度会显著的下降。
``` CPP
#include <iostream>
#include <chrono>
#include <vector>
#include <random>

using namespace std;

int main()
{
    vector<int> sizes;
    for (int i = 1; i < 18; ++i)
    {
        sizes.push_back(1 << i);
    }

    random_device rd;
    mt19937 gen(rd());

    for (auto size : sizes)
    {
        size_t mm = size << 10;
        uniform_int_distribution<> dis(0, mm - 1);
        vector<char> memory(mm);
        fill(memory.begin(), memory.end(), 1);

        int dummy = 0;
        auto start = chrono::high_resolution_clock::now();
        for (int i = 0; i < (1 << 23); ++i)
        {
            dummy += memory[dis(gen)];
        }
        auto end = chrono::high_resolution_clock::now();
        cout << size << " KB, cost " << chrono::duration<double, milli>(end - start).count() << " " << dummy << "\n";
    }

    return 0;
}
```
