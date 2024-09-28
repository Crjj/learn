## +++内存管理

#### 1. ELF文件

- 可执行与可链接格式 (Executable and Linkable Format) 是一种用于可执行文件、目标代码、共享库和核心转储 （core dump） 的标准文件格式，操作系统在加载 ELF 文件时会将按照标准依次读取每个段中的内容，并将其加载到内存中，同时为该进程分配栈空间，并将 pc 寄存器指向代码段的起始位置，然后启动进程。

#### 2. 内存分区（用户空间）

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



## 设计原则

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



## 设计模式

[设计模式](https://refactoringguru.cn/design-patterns/behavioral-patterns)

#### 1. 创建型

- 工厂方法模式
- 抽象工厂模式
- 生成器模式
- 原型模式
- 单例模式

#### 2. 结构型模式

- 适配器模式

- 桥接模式

- 组合模式

- 装饰模式

  ```c++
  ```

  

- 外观模式

- 享元模式

- 代理模式

#### 3. 行为模式

- 责任链模式
- 命令模式
- 迭代器模式
- 中介者模式
- 备忘录模式
- 观察者模式
- 状态模式
- 策略模式
- 模板方法模式
- 访问者模式



## 多态

#### 1. 静态多态

- 编译期间的多态，编译器在编译期间完成的，编译器根据函数实参的类型
  - 函数重载：包括普通函数的重载和成员函数的重载
  - 函数模板的使用

#### 2. 动态多态

- 运行时的多态，在程序执行期间(非编译期)判断所引用对象的实际类型，根据其实际类型调用相应的方法
  - 通过基类类型的引用或者指针调用虚函数



## 五种构造函数

#### 1. 无参构造函数（默认）

```c++
class Person {
  public:
  string name;
  int age;
 
  Person() { // 拷贝构造函数
    name = "";
    age = 18;
  }  
}
```

#### 2. 一般构造函数(带参)

```c++
class Person {
  public:
  string name;
  int age;
 
  Person(const string &name, int age) { // 拷贝构造函数
    this.name = name;
    this.age = age;
  }  
}
```

#### 3. 拷贝（复制）构造函数

```c++
class Person {
public:
  string name;
  int age;
 
  Person(const Person& other) { // 拷贝构造函数
    name = other.name;
    age = other.age;
  }
};
```

#### 4. 移动构造函数

```c++
class Person {
public:
  string name;
  int age;
 
  Person(Person&& other) { // 拷贝构造函数
    name = std::move(other.name);
    age = other.age;
  }
};
/*
Person p1("Bob", 25); // 创建 Person 对象
Person p2(std::move(p1)); // 移动构造函数创建 p2   此时p1.name已经转移到p2，p1,age正常
*/
```

#### 5.类型转换构造函数

```c++
//需要注意的一点是，这个其实就是一般的构造函数，但是对于出现这种单参数的构造函数，C++会默认将参数对应的类型转换为该类类型，有时候这种隐私的转换是我们所不想要的，所以需要使用explicit来限制这种转换。
// 例如：下面将根据一个double类型的对象创建了一个Complex对象
Complex(double r)
{
    m_real = r;
    m_imag = 0.0;
}
// 调用  Complex c2= 5.2;  
```

#### 常见考核

[在C++中，为了让某个类只能通过new来创建实例（即如果直接创建对象，编译器将报错），怎样做？_c++如何让某个类只能通过new来创建。-CSDN博客](https://blog.csdn.net/tkp2014/article/details/48846715)

- ##### 让某个类只能通过new来创建实例（堆上创建）

  将析构函数定义为**protected**(因为private时子类无法继承)，编译器在创建类时会检查所有

- #### 让某个类只能在栈上创建实例

  将operator new() 设为私有的

- #### 为什么模板的声明和实现不能分写在.h与.cpp中

  模板类在编译器进行替换，当实例化一个模板时，编译器必须看到模板确切的定义而不仅仅是它的声明，或者使用export关键字

## 强制类型转换

#### 1. static_cast：用于各种隐式转换

#### 2.  const_cast：用来移除变量的const或volatile限定符

> volatile表示变量可以被编译器未知的因素所更改，遇到volatile编译器不再进行优化，从而提供对特殊地址的稳定访问

#### 3. reinterpret_cast：允许将任何指针转换为任何其它指针类型，没有动态类型检查，不安全

#### 4. dynamic_cast：安全的向下（父类到子类）进行类型转换，只能用于含有虚函数的类，只能转指针或引用



## 虚指针(vptr)与虚函数表(vtbl)

[https://www.cnblogs.com/ZY-Dream/p/10016731.html]()

#### 1. 虚指针：任何含有虚函数的类中，不论这个虚函数是继承的还是定义的，都有一个函数指针指向一个虚函数表，虚函数表是一个数组，数组中的每个元素都是一个虚函数指针。

```c++
class A
{
private:
    int data1;
    int data2;
public:
    A(){};
   virtual ~A(){};
};

class B :public A
{

};
int main()
{
    A a;
    B b;
    cout<<sizeof(a)<<endl; //16
    cout<<sizeof(b)<<endl; //16
    return 0;
}
B继承了A，虽然没有显式声明，其依然继承了A的所有的元素, 包含两个int类型的成员变量和一个虚指针，因此内存大小是16

类的大小与它的构造函数、析构函数和其他成员函数无关，只已它的数据成员有关,类的大小也遵守类似class字节对齐的补齐规则。
C++编译器强制给空类插入一个缺省成员，长度为1。如果有自定义的变量，变量将取代这个缺省成员。
静态数据成员和成员函数不占空间

class A
{
    void FuncA();
}
// sizeof(A) = 1
```

#### 2. 虚函数表: 虚指针指向的就是虚函数表，本质是一个数组，存着所有的虚函数指针。

**同一个类的不同实例共用同一份虚函数表，他们都通过一个虚函数表指针指向该虚函数表**

#### 3. 虚继承：当存在虚拟继承时，派生类中会有一个指向虚基类表的指针

```c++
class C : virtual public A, virtual public B
{
  int c;
};

```



## C++特性

https://blog.csdn.net/qq_39071254/article/details/140082686

https://blog.csdn.net/qq_41854911/article/details/119657617

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

- `std::shared_ptr`：只有共享的最后一个引用释放资源销毁

  > 原理：利用一个计数器，当发生使用赋值拷贝构造函数或运算符重载，作为函数返回值，或者作为一个参数传递给另外一个参数，计数+1，当shared_ptr赋新值或者销毁，计数-1.直到计数为0，调用析构函数释放对象
  >
  > std::shared_ptr<T> p = std::make_shared<T>(..); // 初始化方式1
  >
  > std::shared_ptr<T> p(new T(...)); 初始化方式2

- `std::unique_ptr`：独占式的指针，离开 unique_ptr 对象的作用域时，会自动释放资源

  > 原理：由于指针或引用在离开作用域是不会调用析构函数的，但对象在离开作用域会调用析构函数。unique_ptr本质是一个类，将**复制构造函数**和**赋值构造函数**声明为delete就可以实现独占式，只允许移动构造和移动赋值。

- `std::weak_ptr`：shared_ptr一起使用，也叫弱指针，作为资源的观察者，不影响对象的引用计数

  > 当使用std::shared_ptr会引起循环引用时，则可采用弱指针std::weak_ptr解决
  >
  > weak_ptr是为了配合shared_ptr而引入的一种智能指针，它指向一个由shared_ptr管理的对象而不影响所指对象的生命周期，将一个weak_ptr绑定到一个shared_ptr不会改变shared_ptr的引用计数。不论是否有weak_ptr指向，一旦最后一个指向对象的shared_ptr被销毁，对象就会被释放。从这个角度看，weak_ptr更像是shared_ptr的一个助手而不是智能指针
  >

- auto_ptr：已弃用，由unique_ptr代替，std::unique_ptr不仅加入了移动语义的支持，同时也关闭了左值拷贝构造和左值赋值功能

  >auto_ptr支持**operator=**和**拷贝构造**，这使得有些场景下为了确保指针所有者唯一，会转移所有权
  >
  >```cpp
  >void do_somthing(std::auto_ptr<People> people){
  >    // 该函数内不对people变量执行各种隐式/显示的所有权转移和释放
  >    ...
  >}
  >
  >std::auto_ptr<People> people(new People("jony"));
  >do_something(people);
  >...
  >
  >std::cout << people->get_name() << std::endl; 
  >// 此时的people为空
  >```

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

- `std::forward` - 完美转发，指在模板中保持参数的左值或右值属性，

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

> ```
> #include <iostream>
> #include <array>
> using namespace std;
> 
> void dis_1(const int x){
>     //错误，x是只读的变量  ------ 此处的x还是变量
>     array <int,x> myarr{1,2,3,4,5};
>     cout << myarr[1] << endl;
> }
> 
> void dis_2(){
>     const int x = 5; // ---- 常量
>     array <int,x> myarr{1,2,3,4,5};
>     cout << myarr[1] << endl;
> }
> 
> int main()
> {
>    dis_1(5);
>    dis_2();
> }
> ```
>
> 为了解决 const 关键字的双重语义问题，保留了 const 表示“只读”的语义，而将“常量”的语义划分给了新添加的 constexpr 关键字
>
> ```c++
> // const 只读，不意味着不能修改，如下
> int main()
> {
>     int a = 10;
>     const int & con_b = a;
>     cout << con_b << endl;  // 输出10
>     a = 20;
>     cout << con_b << endl;	// 输出20，通过修改a达到修改b的作用
> }
> ```

#### 9. 线程支持

#### 10. 静态断言`stdtic_assert`：用于在编译时进行断言

#### 11. 强类型枚举`enum class`提供了强类型枚举

#### 12. 统一初始化语法：支持花括号`{}`初始化

#### 13. 基本堆栈的内存分配: `std::array`和`std::vector`的改进

#### 14. 模板别名`using`关键字用于定义模板别名

```c++
typedef无法重定义一个模板，如 

// C++98/03
template <typename Val>
struct str_map
{
    typedef std::map<std::string, Val> type;
};

str_map<int>::type map1;

// c++11后    
template <typename Val>
using str_map_t = std::map<std::string, Val>;

str_map_t<int> map1;
```

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

#### 1. 互斥量：用于保护临界区，二进制信号量，只能取0或1两个值
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

### 常见算法考核

- 快排：分而治之，快速排序的主要思想是分治法，将一个大问题分割成小问题，解决小问题后再合并它们的结果。

  1. 从待排序的数组中选择一个元素，称之为枢纽元（pivot）。
  2. 将数组中小于枢纽元的元素移到枢纽元的左边，将大于枢纽元的元素移到枢纽元的右边，这个过程称为分区（partition）。
  3. 递归地对枢纽元左边的子数组和右边的子数组进行排序。
  4. 当所有子数组都有序时，整个数组就自然有序了

- 实现链表逆序，三个(一个临时指针)指针遍历链表，逐个反转指针方向

  ```C++
  ListNode* reverseList(ListNode* head)
  {
      ListNode* prev = nullptr;
      ListNode* curr = head;
      while (curr != nullptr)
      {
          ListNode* nextTemp = curr->next;
          curr->next = prev;
          prev = curr;
          curr = nextTemp;
      }
      return prev;
  }
  ```

- 判断一个链表是否有环

  使用快慢指针（Tortoise and Hare算法）。快指针一次移动两步，慢指针一次一步，如果链表有环，两个指针最终会相遇

- 如何设计一个线程安全的单例模式，C++11之后的`std::call_once`或静态局部变量

  ```c++
  class Singleton
  {
  public:
      static Singleton& getInstance()
      {
          static Singleton instance;
          return instance;
      }
  private:
      Singleton() {}
      Singleton(const Singleton&) = delete;
      Singleton& operator=(const Singleton&) = delete;
  };
  ```

  

## STL

[https://zhuanlan.zhihu.com/p/614287445]()

#### 1. 容器

#### 2. 算法

#### 3. 迭代器

#### 4. 仿函数

#### 4. 适配器

#### 5. 分配器



### UML

[UML一一 类图关系 (泛化、实现、依赖、关联、聚合、组合)_uml类图关系-CSDN博客](https://blog.csdn.net/m0_37989980/article/details/104470064)



## 面试

[C++/QT PC客户端面试题 | Skykey's Home](https://blog.skykey.fun/2022/06/20/C++面试/)
