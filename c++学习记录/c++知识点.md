## 内存管理

#### 1. ELF文件

- 可执行与可链接格式 (Executable and Linkable Format) 是一种用于可执行文件、目标代码、共享库和核心转储 （core dump） 的标准文件格式，操作系统在加载 ELF 文件时会将按照标准依次读取每个段中的内容，并将其加载到内存中，同时为该进程分配栈空间，并将 pc 寄存器指向代码段的起始位置，然后启动进程。

#### 2. 内存分区

- 栈：主要存放函数的局部变量、函数参数、返回地址等，由操作系统自动申请释放，如果进程结束后还没有释放，操作系统会自动回收
- 堆：动态申请的内存空间，由malloc或者new函数申请，free或者delete进行释放，如果进程结束后还没有释放，操作系统会自动回收
- 全局区/静态存储区：主要为.bss和.data段，存放全局变量和静态变量，程序运行结束操作系统自动释放，在 `C` 中，未初始化的放在 `.bss` 段中，初始化的放在 `.data` 段中，`C++` 中不再区分了
- 常量存储区：.rodata段，存放常量，不允许修改，程序运行结束自动释放
- 代码区：.text段，存放代码，不允许修改，但可以执行，编译后的二进制文件存放在这里

```c++
#include <iostream>
using namespace std;
/*
说明：C++ 中不再区分初始化和未初始化的全局变量、静态变量的存储区，如果非要区分下述程序标注在了括号中
*/
int g_var = 0; // g_var 在全局区（.data 段）
char *gp_var;  // gp_var 在全局区（.bss 段）

int main()
{
    int var;                    // var 在栈区
    char *p_var;                // p_var 在栈区
    char arr[] = "abc";         // arr 为数组变量，存储在栈区；"abc"为字符串常量，存储在常量区
    char *p_var1 = "123456";    // p_var1 在栈区；"123456"为字符串常量，存储在常量区
    static int s_var = 0;       // s_var 为静态变量，存在静态存储区（.data 段）
    p_var = (char *)malloc(10); // 分配得来的 10 个字节的区域在堆区
    free(p_var);
    return 0;
}
```



## 编译与链接

#### 编译处理过程

编译器读取源文件 cpp，并将其翻译为可执行文件「ELF」，ELF 文件可以经过操作系统进行加载执行。常见的编译过程分为四个过程：编译预处理、编译、汇编、链接

**hello.c -> 预处理器 -> hello.i -> 编译器 -> hello.s ->汇编器-> hello.o -> 链接器 -> hello（可执行二进制文件）**

- 编译预处理：主要处理源代码中的预处理指令，引入头文件（#include），去除注释、处理所有条件编译指令（#ifdef、#ifndef,、#elif、#endif），宏替换（#define），添加行号，保留所有编译器指令
- 编译：针对预处理后的文件进行词法解析、语法分析、符号汇总、汇编代码生成，即将cpp文件翻译成.s的汇编代码
- 汇编：将汇编代码.s翻译成机器指令.o文件，一个.cpp文件只会生成一个.o文件
- 链接：将多个.o文件进行链接，生成一个可被系统加载执行的ELF程序文件
  - 静态链接：linux(.a)、windows(.lib)
  - 动态链接：linux(.so)、windows(.dll)



## 设计原则&23种设计模式(未完)

#### 1.  单一职责原则（SRP）：一个类应该只负责一项职责，降低代码复杂度

   ```c++
   class ReportGenerator {
   public:
       void generateReport() {
           // 生成报告
       }
   };
   
   class ReportPrinter {
   public:
       void printReport() {
           // 打印报告
       }
   };
   // ReportGenerator 负责生成报告，ReportPrinter 负责打印报告，遵循了单一职责原则
   ```

#### 2. 开闭原则（OCP）：对扩展开放，对修改关闭，通过扩展而不修改现有代码来实现新的功能

   ~~~c++
   class Shape {
   public:
       virtual void draw() = 0;
   };
   
   class Circle : public Shape {
   public:
       void draw() override {
           // 画圆
       }
   };
   
   class Rectangle : public Shape {
   public:
       void draw() override {
           // 画矩形
       }
   };
   // 通过继承 Shape 类来扩展新的形状类，而不需要修改现有的 Shape 类   
   ~~~

#### 3. 里氏替换原则（LSP）：任何基类出现的地方，子类一定可以出现

   ```c++
   class Bird {
   public:
       virtual void fly() {
           // 飞行逻辑
       }
   };
   
   class Sparrow : public Bird {
   public:
       void fly() override {
           // 麻雀飞行逻辑
       }
   };
   // Sparrow可以替代Bird，且系统功能不受影响
   ```

#### 4. 接口隔离原则（ISP）: 使用多个专门的接口，而不是单一的总接口，减少类之间的依赖

   ```c++
   class Printer {
   public:
       virtual void print() = 0;
   };
   
   class Scanner {
   public:
       virtual void scan() = 0;
   };
   
   class AllInOnePrinter : public Printer, public Scanner {
   public:
       void print() override {
           // 打印逻辑
       }
   
       void scan() override {
           // 扫描逻辑
       }
   };
   // Printer 和 Scanner 接口分离，避免了单一接口的臃肿
   ```

#### 5. 依赖倒置原则（DIP）：针对接口编程，不要对实现编程，通过依赖抽象层来解耦高层和低层模块
   - 高层模块不应该依赖低层模块，二者都应该依赖对象
   - 抽象不应依赖细节，细节应该依赖抽象

   ```c++
   class Database {
   public:
       virtual void connect() = 0;
   };
   
   class MySQLDatabase : public Database {
   public:
       void connect() override {
           // MySQL 连接逻辑
       }
   };
   
   class Application {
   private:
       Database& db;
   public:
       Application(Database& db) : db(db) {}
       
       void run() {
           db.connect();
           // 业务逻辑
       }
   };
   // Application 依赖于 Database 抽象接口，而不是具体实现，遵循了依赖倒置原则
   ```

#### 6. 合成复用原则（CARP）：优先使用对象组合而不是继承来达到复用目的（面向对象设计中主要有两种方式复用已有的设计和实现，即合成复用原则和继承）

   ```c++
   class Engine {
   public:
       void start() {
           // 启动引擎
       }
   };
   
   class Car {
   private:
       Engine engine;
   public:
       void drive() {
           engine.start();
           // 驾驶逻辑
       }
   };
   // Car 通过组合 Engine 对象来实现驱动功能，而不是通过继承
   ```



## C++特性

> https://blog.csdn.net/qq_39071254/article/details/140082686

### C++11

#### 1. 自动推导 auto、decltype、编译期间完成

- auto：根据=号右侧的值进行推导，必须初始化
- decltype:  根据=号在侧表达式进行推导，与右侧无关， decltype会保留原始表达示的任何类型
- 对cv限定符（const：只读， volatile：数据可变、易变）处理：如果表达式的类型是指针或者引用，auto 将保留 cv 限定符，否则丢弃
- 对引用的处理：decltype 会保留引用类型，而 auto 会抛弃引用类型

#### 2. 范围for循环：简化遍历容器的方式

```c++
std::vector<int> vec {1, 2, 3};
for(auto &v: vec) {
    std::cout << v << " ";
}
```

#### 3. Lambda表达式：支持匿名函数

```c++
auto add = [](int a, int b) { return a+b; }
std::cout << add(1, 2);
```

#### 4. 智能指针

- `std::shared_ptr`
- `std::unique_ptr`
- `std::weak_ptr`

#### 5. 右值引用：优化对象的移动操作，`&&`表示右值引用

```c++
void foo(int&& x) {
    std::cout << x;
}
foo(10);  // 10 是右值
```

#### 6. 移动语义与完美转发

> https://zhuanlan.zhihu.com/p/398817111

- `std::move` - 移动语义，将一个左值强制转化为右值引用，继而可以通过右值引用使用该值

   ```c++
   对于内置类型（如 int），移动和复制没有区别，因此这里的移动操作不会带来任何优化，仅仅是转换为右值引用,
   std::move() 通常在处理复杂对象时使用，例如动态分配的内存、文件资源等。在这些情况下，移动语义允许你避免深拷贝，而是“窃取”资源，从而提高效率
   // std::move() 将左值转换为右值引用，这使得你能够调用特定的移动构造函数或移动赋值操作符，从而避免拷贝
   ```

- `std::forward` - 完美转发，指在模板中保持参数的左值或右值属性

  ```cpp
  引用折叠：
  - 左值-左值 T& &
  - 左值-右值 T& &&
  - 右值-左值 T&& &
  - 右值-右值 T&& &&
  // 在模板中T&&为万能引用（配合引用折叠），可接左值或者右值作为参数   
  假设模板定义如下：
  template<typename T>
  void PrintType(T&& param){ ... }
   
  当T为int&类型时， param被推导为int& &&将引起编译器错误，所以有以下引用折叠规则：
  如果任一引用为左值引用，则结果为左值引用。否则（即两个都是右值引用），结果为右值引用。
  // param最终被折叠成int &类型。
       
  假设模板定义如下：
  template<typename T>
  void PrintType(T&& param) {
     actualRun(param); // 根据C++规则，当你传递一个有名字的右值引用时，它实际上会被视为左值。
  }
  在PrintType函数内部，param被传递给actualRun函数。但是，注意到param虽然被推导为右值引用（X&&），但当它被传递给actualRun时，param是一个左值，因为它在这个上下文中已经有了名字。
  因此，如果actualRun有多个重载版本，比如一个接受左值引用，一个接受右值引用，那么在这种情况下，会调用接受左值引用的版本，而不是你期望的右值引用版本
      
  解决方案：完美转发
  template<typename T>
  void PrintType(T&& param) {
      actualRun(std::forward<T>(param));
  }
  std::forward<T>(param) 会根据param的类型来决定是否转发为左值或右值
  ```

#### 7. `nullptr` ：用于表示空指针，替代`NULL`

#### 8. `constexpr`常量表达：用于定义常量表达式函数

#### 9. 线程支持

#### 10. 静态断言`stdtic_assert`：用于在编译时进行断言

#### 11. 强类型枚举`enum class`提供了强类型枚举

#### 12. 统一初始化语法：支持花括号`{}`初始化

#### 13. 基本堆栈的内存分配: `std::array`和`std::vector`的改进

#### 14. 模板别名`using`关键字用于定义模板别名

#### 15. 变长模板，支持模板参数包

###  

### C++14

#### 1. 泛型Lambda表达式：Lambda表达式支持模板参数

```c++
auto lambda = [](auto x, auto y) { return x + y};
std::cout << lambda(1, 2);  // 输出3
```

#### 2. 返回类型推导：函数返回类型可以通过`auto`关键字推导

```c++
auto add(int a, int b) {
    return a + b;
}
```

#### 3. 二进制字面量：可以直接使用二进制表示法 

```c++
int bin = 0b1010;  // bin = 10
```

#### 4. 数字分隔符：用单引号`'`分隔数据

```c++
int million = 1'000'000; // 方便读写
```

#### 5. `constexpr`改进，允许更复杂的`constexpr`表达式

#### 6. 为`std::make_unique`提供便捷创建方法



###  C++17

#### 1. 结构化绑定：允许解构变量赋值

> auto [a, b] = pair
>
> std::tuple<int,double, std::string> t(1, 2.3, "hello");
>
> auto [i,d,s] = t;

#### 2. 折叠表达式：用于简化变长模板参数的处理

#### 3. `std::optional`：用于表示可空值

#### 4. `std::variant`: 支持类型安全的联合体

#### 5. `std::any`: 可以保存任意类型的值

#### 6. `if`和`switch`语句中的初始化：允许在条件语句中进行变量初始化

#### 7. `std::string_view`：提供更高效的字符串视图

#### 8. `constexpr if`: 编译时条件判断

#### 9. 文件系统库`<filesystem>`提供文件系统操作支持



## 线程同步

#### 1. 互斥量：用于保护临界区，二进制信号量，只熊取0或1两个值
   - std::mutex: 通过lock与unlock获取和释放锁

#### 2.  条件变量：用于线程间的同步，需要与互斥量配合使用

   - 使用std::condition_vaiable和std::condition_variant_any实现线程间的同步

   - 通过wait()、wait_for()、和wait_until()方法来等待条件变量的满足

   - 通过notify_one()、notify_all()方法来唤醒等待的线程

#### 3. 原子操作

   - 使用std::atomic原子类型来执行不可中断的原子操作

   - 通过load()、store()、exchange()和compare_exchange_*()等方法进行原子操作

#### 4. 信号量

   > 用于协调并发访问，可以取任意非负整数值, 主要场景用于限制并发访问，生产者-消费者模型，实现读写锁

   - 使用 `std::counting_semaphore` 和 `std::binary_semaphore` 来实现信号量机制
   - 通过acquire()和release()方法执行P（阻塞）操作和V（唤醒）操作

#### 5. 读写锁

- 使用 `std::shared_mutex` 或 `std::shared_timed_mutex` 来实现读写锁。

- 通过 `lock()`、`unlock()`、`lock_shared()` 和 `unlock_shared()` 方法来获取和释放读写锁



## 数据结构

#### 1. 数组

#### 2. 链表

- 单向链接
- 双向链接

#### 3. 栈

#### 4. 队列

- 普通队列
- 双端队列
- 优先队列

#### 5. 哈希表

#### 6. 树

- 二叉树
- 二叉搜索树
- 平衡树

#### 7. 图

#### 8. 集合
