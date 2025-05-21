
以下是为你整理的 **C++面向对象核心知识点复习题集**（难度进阶，覆盖全面，含详细解析与扩展），适合冲刺高阶水平：


---

### **一、类与对象（基础+进阶）**
#### 题目1（概念辨析）  
简述以下概念的区别与联系：  
- 类的成员函数（Member Function）与静态成员函数（Static Member Function）  
- 对象的内存布局（Memory Layout）与类的内存布局  
- 构造函数（Constructor）与转换构造函数（Conversion Constructor）  


#### 题目2（代码分析）  
分析以下代码的编译/运行结果，并解释原因：  
```cpp  
#include <iostream>
using namespace std;

class A {
public:
    A() { cout << "A构造" << endl; }
    ~A() { cout << "A析构" << endl; }
};

class B : public A {
public:
    B() { cout << "B构造" << endl; }
    ~B() { cout << "B析构" << endl; }
};

int main() {
    A* ptr = new B();
    delete ptr;  // 输出什么？
    return 0;
}
```  


#### 题目3（编程题）  
设计一个类 `Date`，要求：  
- 包含年、月、日三个私有成员变量（`int`类型）；  
- 提供构造函数（支持默认构造、参数构造、拷贝构造）；  
- 重载 `<<` 运算符以输出日期（格式：`YYYY-MM-DD`）；  
- 实现 `isLeapYear()` 成员函数判断是否为闰年（扩展：考虑闰秒等特殊情况）；  
- 实现 `operator++()`（前置）和 `operator++(int)`（后置）运算符，支持日期自增1天（需处理跨月、跨年逻辑）。  


---

### **二、继承与多态（核心难点）**
#### 题目4（概念辨析）  
解释以下概念的底层机制与设计目的：  
- 虚函数（Virtual Function）与纯虚函数（Pure Virtual Function）  
- 虚函数表（VTable）与虚表指针（VPTR）  
- 多继承（Multiple Inheritance）的二义性（Ambiguity）与虚继承（Virtual Inheritance）  


#### 题目5（代码分析）  
分析以下代码的输出结果，并画出对象的内存布局（假设32位系统，无内存对齐优化）：  
```cpp  
#include <iostream>
using namespace std;

class Base {
public:
    virtual void func1() { cout << "Base::func1" << endl; }
    virtual void func2() { cout << "Base::func2" << endl; }
    int x = 10;
};

class Derived : public Base {
public:
    void func1() override { cout << "Derived::func1" << endl; }
    virtual void func3() { cout << "Derived::func3" << endl; }
    int y = 20;
};

int main() {
    Derived d;
    Base* b = &d;
    cout << sizeof(Base) << endl;    // 输出？
    cout << sizeof(Derived) << endl;  // 输出？
    b->func1();  // 输出？
    return 0;
}
```  


#### 题目6（编程题）  
设计一个图形库，包含以下类：  
- 基类 `Shape`（纯虚类），包含纯虚函数 `double area()`（计算面积）和 `void printInfo()`（打印图形信息）；  
- 派生类 `Circle`（圆，半径 `r`）、`Rectangle`（矩形，长 `a`，宽 `b`）、`Triangle`（三角形，三边 `a,b,c`）；  
- 要求 `Triangle` 类在构造时检查三边是否能构成三角形（否则抛出异常）；  
- 扩展：实现一个 `ShapeManager` 类，支持动态添加 `Shape` 对象，并计算所有图形的总面积（使用多态实现）。  


---

### **三、模板与泛型编程（高阶）**
#### 题目7（概念辨析）  
解释以下概念的区别与联系：  
- 函数模板（Function Template）与类模板（Class Template）  
- 模板特化（Template Specialization）与模板偏特化（Partial Specialization）  
- 类型萃取（Type Traits）与 `constexpr` 表达式  


#### 题目8（代码分析）  
分析以下模板代码的编译结果，并解释原因：  
```cpp  
#include <iostream>
using namespace std;

template <typename T>
class Container {
public:
    void print() { cout << "通用模板" << endl; }
};

template <>
class Container<int> {  // 全特化
public:
    void print() { cout << "int特化模板" << endl; }
};

template <typename T>
class Container<T*> {  // 偏特化（指针类型）
public:
    void print() { cout << "指针偏特化模板" << endl; }
};

int main() {
    Container<double> c1;   // 调用哪个print？
    Container<int> c2;      // 调用哪个print？
    Container<float*> c3;   // 调用哪个print？
    return 0;
}
```  


#### 题目9（编程题）  
实现一个通用的 `Stack` 类模板，要求：  
- 支持 `push`（入栈）、`pop`（出栈）、`top`（获取栈顶）操作；  
- 使用动态数组（`new`/`delete`）实现底层存储；  
- 扩展1：添加 `operator==` 运算符重载，比较两个 `Stack` 是否相等（元素顺序和值均相同）；  
- 扩展2：使用 `std::allocator` 替换动态数组，实现自定义内存分配（关联STL容器的内存管理机制）。  


---

### **四、内存管理与智能指针（工程重点）**
#### 题目10（概念辨析）  
解释以下概念的底层原理与应用场景：  
- 堆内存（Heap）与栈内存（Stack）的区别  
- 内存泄漏（Memory Leak）与野指针（Dangling Pointer）  
- `unique_ptr`、`shared_ptr`、`weak_ptr` 的设计目的与协作关系  


#### 题目11（代码分析）  
分析以下代码的内存问题，并给出修复方案：  
```cpp  
#include <memory>
using namespace std;

class A {
public:
    A() { cout << "A构造" << endl; }
    ~A() { cout << "A析构" << endl; }
};

class B {
public:
    B() { a = new A(); }
    ~B() { delete a; }  // 是否存在问题？
private:
    A* a;
};

int main() {
    B b1;
    B b2 = b1;  // 调用默认拷贝构造函数，是否安全？
    return 0;
}
```  


#### 题目12（编程题）  
设计一个线程安全的 `Singleton`（单例模式），要求：  
- 全局仅一个实例，支持懒汉式（Lazy Initialization）或饿汉式（Eager Initialization）；  
- 使用 `shared_ptr` 管理实例生命周期（自动释放）；  
- 扩展：结合 `mutex` 实现线程安全（避免多线程下的重复构造）；  
- 扩展：禁止通过拷贝构造、赋值运算符复制实例。  


---

### **五、STL与设计模式（综合应用）**
#### 题目13（概念辨析）  
简述以下STL组件的底层数据结构与适用场景：  
- `vector`、`list`、`deque` 的区别  
- `map`（红黑树）与 `unordered_map`（哈希表）的性能对比  
- 迭代器（Iterator）的分类（输入/输出/前向/双向/随机访问）  


#### 题目14（代码分析）  
分析以下代码的运行结果，并解释STL算法的行为：  
```cpp  
#include <vector>
#include <algorithm>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {3, 1, 4, 1, 5, 9};
    sort(v.begin(), v.end());  // 排序后v的内容？
    auto it = unique(v.begin(), v.end());  // it指向哪里？
    cout << v.size() << endl;   // 输出？
    v.erase(it, v.end());      // 最终v的内容？
    return 0;
}
```  


#### 题目15（设计题）  
设计一个图书管理系统，要求：  
- 使用 `class Book` 表示图书（属性：ISBN、书名、作者、价格）；  
- 使用 `std::vector<Book>` 存储所有图书；  
- 实现以下功能：  
  - 按书名/作者/ISBN查询图书（支持模糊搜索）；  
  - 按价格排序（升序/降序）；  
  - 统计库存中价格高于某阈值的图书数量；  
- 扩展：使用策略模式（Strategy Pattern）实现不同的排序策略（价格/书名）。  


---

### **答案与解析（选摘）**
#### 题目2解析：  
- 输出结果：  
  ```  
  A构造  
  B构造  
  A析构  
  ```  
- 原因：基类 `A` 的析构函数未声明为 `virtual`，导致 `delete ptr` 时仅调用基类析构函数，派生类 `B` 的析构函数未被调用，造成内存泄漏。  
- 扩展：**虚析构函数的必要性**：当通过基类指针删除派生类对象时，必须将基类的析构函数声明为 `virtual`，否则无法触发动态绑定，派生类资源无法释放。  


#### 题目5解析：  
- `sizeof(Base)`：`8` 字节（32位系统中，虚表指针 `vptr` 占4字节，`int x` 占4字节）；  
- `sizeof(Derived)`：`12` 字节（继承基类的 `vptr` 和 `x`，新增 `int y` 占4字节，`Derived` 的虚表包含 `func1`（覆盖基类）、`func2`（继承基类）、`func3`（新增））；  
- `b->func1()` 输出 `Derived::func1`：基类指针指向派生类对象时，通过虚函数表动态绑定到派生类的 `func1` 实现。  


#### 题目11解析：  
- 问题1：`B` 的默认拷贝构造函数会复制指针 `a`，导致 `b1` 和 `b2` 的 `a` 指向同一块内存，析构时重复 `delete`，引发未定义行为（崩溃）。  
- 问题2：`B` 的析构函数虽然释放了 `a`，但未处理拷贝构造和赋值运算符的深拷贝问题。  
- 修复方案：  
  ```cpp  
  class B {
  public:
      B() : a(new A()) {}
      ~B() { delete a; }
      // 深拷贝构造函数
      B(const B& other) : a(new A(*other.a)) {}
      // 深拷贝赋值运算符
      B& operator=(const B& other) {
          if (this != &other) {
              delete a;
              a = new A(*other.a);
          }
          return *this;
      }
  private:
      A* a;
  };
  ```  


#### 题目14解析：  
- `sort` 后 `v`：`{1, 1, 3, 4, 5, 9}`；  
- `unique` 会将相邻重复元素移动到末尾，返回指向第一个重复元素的迭代器（指向第二个 `1`）；  
- `v.size()` 仍为 `6`（`unique` 不改变容器大小，仅覆盖重复元素）；  
- `erase` 后 `v`：`{1, 3, 4, 5, 9}`（删除重复的 `1`）。  


---

### **复习建议**  
1. **先独立完成题目**，再对照答案验证，重点关注代码分析题的底层机制（如虚函数表、内存布局）；  
2. **动手编写代码**：编程题需实际运行并测试边界条件（如 `Date` 类的2月29日、`Stack` 的空栈弹出）；  
3. **扩展关联**：结合《C++ Primer》《深度探索C++对象模型》《STL源码剖析》理解底层原理；  
4. **模拟面试**：对概念辨析题进行口头复述，确保能清晰解释（如“为什么多继承需要虚继承？”）。  

通过这套题集，可系统覆盖面向对象C++的核心知识点，提升代码分析、问题排查和工程设计能力。


以下是15道C++面向对象复习题的**详细答案+知识点标注**，覆盖核心概念、代码分析、编程设计，帮你系统巩固知识点：


---

### **一、类与对象（基础+进阶）**

#### **题目1（概念辨析）**  
**问题**：简述以下概念的区别与联系：  
- 类的成员函数（Member Function）与静态成员函数（Static Member Function）  
- 对象的内存布局（Memory Layout）与类的内存布局  
- 构造函数（Constructor）与转换构造函数（Conversion Constructor）  


**答案**：  
1. **成员函数 vs 静态成员函数**  
   - **区别**：  
     - 成员函数属于对象，可访问类的所有成员（包括 `this` 指针）；静态成员函数属于类本身，无 `this` 指针，只能访问静态成员（静态变量/静态函数）。  
     - 调用方式：成员函数通过对象调用（`obj.func()`）；静态函数通过类名调用（`Class::static_func()`）或对象调用（`obj.static_func()`）。  
   - **联系**：均为类的成员，定义在类内部，可被 `public`/`private` 修饰。  

2. **对象的内存布局 vs 类的内存布局**  
   - **类的内存布局**：描述类的成员变量和虚函数表（`vtable`）的结构（仅类型信息，无实际内存）。  
   - **对象的内存布局**：类的实例化对象在内存中的实际存储（包含成员变量的具体值、虚表指针 `vptr`）。  
   - **联系**：对象的内存布局由类的内存布局决定（如基类成员在前，派生类成员在后）。  

3. **构造函数 vs 转换构造函数**  
   - **构造函数**：用于初始化对象，支持重载（默认构造、参数构造、拷贝构造等）。  
   - **转换构造函数**：单参数的构造函数（或除第一个参数外其他有默认值的构造函数），用于将其他类型转换为类类型（如 `A(int x)` 可将 `int` 转为 `A`）。  
   - **联系**：==转换构造函数是构造函数的一种特殊形式==，可通过 `explicit` 关键字禁止隐式转换。  


**知识点**：类成员函数、静态成员、对象内存布局、构造函数重载与转换构造。  


#### **题目2（代码分析）**  
**问题**：分析以下代码的编译/运行结果，并解释原因：  
```cpp  
#include <iostream>
using namespace std;

class A {
public:
    A() { cout << "A构造" << endl; }
    ~A() { cout << "A析构" << endl; }  // 未声明为 virtual
};

class B : public A {
public:
    B() { cout << "B构造" << endl; }
    ~B() { cout << "B析构" << endl; }
};

int main() {
    A* ptr = new B();  // 基类指针指向派生类对象
    delete ptr;  // 输出什么？
    return 0;
}
```  


**答案**：  
- **运行结果**：  
  ```  
  A构造  
  B构造  
  A析构  
  ```  
- **原因**：  
  基类 `A` 的析构函数未声明为 `virtual`，导致 `delete ptr` 时仅调用基类 `A` 的析构函数，派生类 `B` 的析构函数未被调用。这会导致 `B` 对象中可能存在的资源（如动态分配的内存）未被释放，引发内存泄漏。  


**知识点**：虚析构函数的必要性、继承的构造/析构顺序（基类构造→派生类构造；派生类析构→基类析构）。  


#### **题目3（编程题）**  
**问题**：设计一个类 `Date`，要求包含年、月、日私有成员，支持构造函数、`<<` 重载、闰年判断、日期自增。  


**答案**：  
```cpp  
#include <iostream>
#include <stdexcept>
using namespace std;

class Date {
private:
    int year, month, day;
    // 辅助函数：获取某月的天数（考虑闰年2月）
    int getDaysInMonth(int y, int m) const {
        static const int days[] = {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
        if (m == 2 && isLeapYear(y)) return 29;
        return days[m];
    }

public:
    // 默认构造：2000-01-01
    Date() : year(2000), month(1), day(1) {}
    // 参数构造：检查日期合法性
    Date(int y, int m, int d) {
        if (m < 1 || m > 12) throw invalid_argument("月份无效");
        int max_day = getDaysInMonth(y, m);
        if (d < 1 || d > max_day) throw invalid_argument("日期无效");
        year = y;
        month = m;
        day = d;
    }
    // 拷贝构造
    Date(const Date& other) : year(other.year), month(other.month), day(other.day) {}

    // 闰年判断（扩展：考虑格里高利历规则，如整百年需被400整除）
    bool isLeapYear(int y) const {
        return (y % 4 == 0 && y % 100 != 0) || (y % 400 == 0);
    }

    // 前置++：日期自增1天
    Date& operator++() {
        day++;
        if (day > getDaysInMonth(year, month)) {
            day = 1;
            month++;
            if (month > 12) {
                month = 1;
                year++;
            }
        }
        return *this;
    }

    // 后置++：返回自增前的副本
    Date operator++(int) {
        Date temp = *this;
        ++(*this);
        return temp;
    }

    // 友元函数：重载<<输出
    friend ostream& operator<<(ostream& os, const Date& d) {
        os << d.year << "-" << (d.month < 10 ? "0" : "") << d.month 
           << "-" << (d.day < 10 ? "0" : "") << d.day;
        return os;
    }
};

// 测试
int main() {
    try {
        Date d1(2023, 2, 28);
        cout << "d1: " << d1 << endl;  // 2023-02-28
        Date d2 = d1++;
        cout << "d2: " << d2 << ", d1++后: " << d1 << endl;  // d2: 2023-02-28, d1++后: 2023-03-01
        ++d1;
        cout << "++d1后: " << d1 << endl;  // 2023-03-02
        Date d3(2020, 2, 29);  // 闰年2月29日合法
        cout << "d3: " << d3 << endl;  // 2020-02-29
    } catch (const exception& e) {
        cout << "错误: " << e.what() << endl;
    }
    return 0;
}
```  


**知识点**：构造函数重载与异常处理、运算符重载（`++` 前置/后置）、友元函数、日期逻辑（跨月/跨年）。  


---

### **二、继承与多态（核心难点）**

#### **题目4（概念辨析）**  
**问题**：解释以下概念的底层机制与设计目的：  
- 虚函数（Virtual Function）与纯虚函数（Pure Virtual Function）  
- 虚函数表（VTable）与虚表指针（VPTR）  
- 多继承（Multiple Inheritance）的二义性与虚继承（Virtual Inheritance）  


**答案**：  
1. **虚函数 vs 纯虚函数**  
   - **虚函数**：在基类中声明为 `virtual` 的函数，派生类可重写（`override`），实现动态绑定（运行时多态）。  
   - **纯虚函数**：基类中声明为 `virtual 函数=0` 的函数，无具体实现，强制派生类重写（抽象类）。  
   - **设计目的**：虚函数支持多态行为；纯虚函数定义==接口规范==（抽象类不能实例化）。  

2. **虚函数表（VTable）与虚表指针（VPTR）**  
   - **VTable**：类的静态成员，存储虚函数的函数指针数组（每个类有一个 `vtable`）。  
   - **VPTR**：对象的成员，指向所属类的 `vtable`（每个对象有一个 `vptr`，通常位于对象内存起始位置）。  
   - **底层机制**：当通过基类指针调用虚函数时，通过 `vptr` 找到 `vtable`，再调用派生类重写的函数（动态绑定）。  
   - 意思就是说这个之所以这么神奇是因为底层c++帮你做了很多事:添加进去了很多别的指令,而不是单单提供一个编程环境

3. **多继承的二义性 vs 虚继承**  
   - **二义性**：多继承时，若两个基类有同名成员，派生类访问该成员时无法确定来源（如 `class D : public A, public B`，`A` 和 `B` 均有 `func()`，`D` 的对象调用 `func()` 会报错）。  
   - **虚继承**：通过 `virtual` 关键字声明基类（如 `class B : virtual public A`），使多个派生类共享==基类的一份实例==，避免重复存储（解决菱形继承问题）。  


**知识点**：动态多态、虚函数表、抽象类、多继承二义性与虚继承。  


#### **题目5（代码分析）**  
**问题**：分析以下代码的输出结果，并画出对象的内存布局（32位系统，无内存对齐优化）：  
```cpp  
#include <iostream>
using namespace std;

class Base {
public:
    virtual void func1() { cout << "Base::func1" << endl; }
    virtual void func2() { cout << "Base::func2" << endl; }
    int x = 10;
};

class Derived : public Base {
public:
    void func1() override { cout << "Derived::func1" << endl; }
    virtual void func3() { cout << "Derived::func3" << endl; }
    int y = 20;
};

int main() {
    Derived d;
    Base* b = &d;
    cout << sizeof(Base) << endl;    // 输出？
    cout << sizeof(Derived) << endl;  // 输出？
    b->func1();  // 输出？
    return 0;
}
```  


**答案**：  
- **输出结果**：  
  ```  
  8        // sizeof(Base)  
  12       // sizeof(Derived)  
  Derived::func1  
  ```  
- **内存布局分析**（32位系统）：  
  - `Base` 类：包含 `vptr`（4字节，指向 `Base` 的虚表）和 `int x`（4字节），总大小 8 字节。  
  - `Derived` 类：继承 `Base` 的 `vptr` 和 `x`，新增 `int y`（4字节），总大小 12 字节。`Derived` 的虚表覆盖了 `func1`（指向 `Derived::func1`），并新增 `func3`（位于虚表末尾）。  
- **`b->func1()` 调用**：基类指针 `b` 指向 `Derived` 对象，通过 `vptr` 找到 `Derived` 的虚表，调用 `Derived::func1`（动态绑定）。  


**知识点**：虚函数表结构、对象内存布局、动态多态的实现。  


#### **题目6（编程题）**  
**问题**：设计图形库，包含 `Shape` 基类、`Circle`/`Rectangle`/`Triangle` 派生类，支持面积计算、异常检查，以及 `ShapeManager` 管理。  


**答案**：  
```cpp  
#include <iostream>
#include <vector>
#include <cmath>
#include <stdexcept>
using namespace std;

// 基类：抽象类
class Shape {
public:
    virtual double area() const = 0;  // 纯虚函数（面积）
    virtual void printInfo() const = 0;  // 纯虚函数（打印信息）
    virtual ~Shape() = default;  // 虚析构函数
};

// 圆
class Circle : public Shape {
private:
    double r;
public:
    Circle(double radius) : r(radius) {
        if (r <= 0) throw invalid_argument("半径必须大于0");
    }
    double area() const override { return M_PI * r * r; }
    void printInfo() const override {
        cout << "圆（半径=" << r << "），面积=" << area() << endl;
    }
};

// 矩形
class Rectangle : public Shape {
private:
    double a, b;
public:
    Rectangle(double length, double width) : a(length), b(width) {
        if (a <= 0 || b <= 0) throw invalid_argument("长宽必须大于0");
    }
    double area() const override { return a * b; }
    void printInfo() const override {
        cout << "矩形（长=" << a << "，宽=" << b << "），面积=" << area() << endl;
    }
};

// 三角形
class Triangle : public Shape {
private:
    double a, b, c;
public:
    Triangle(double x, double y, double z) : a(x), b(y), c(z) {
        // 检查三边是否能构成三角形（两边之和大于第三边）
        if (a + b <= c || a + c <= b || b + c <= a) {
            throw invalid_argument("三边无法构成三角形");
        }
    }
    double area() const override {
        double p = (a + b + c) / 2;
        return sqrt(p * (p - a) * (p - b) * (p - c));  // 海伦公式
    }
    void printInfo() const override {
        cout << "三角形（边=" << a << "," << b << "," << c << "），面积=" << area() << endl;
    }
};

// 图形管理器
class ShapeManager {
private:
    vector<Shape*> shapes;  // 存储基类指针（多态）
public:
    void addShape(Shape* s) { shapes.push_back(s); }
    //这里可以优化一下,把totalArea定义为一个变量,防止多次查询总面积
    double totalArea() const {
        double sum = 0;
        for (const auto& s : shapes) sum += s->area();
        return sum;
    }
    void printAll() const {
        for (const auto& s : shapes) s->printInfo();
    }
    ~ShapeManager() {  // 释放所有图形
        for (auto& s : shapes) delete s;
    }
};

// 测试
int main() {
    try {
        ShapeManager manager;
        manager.addShape(new Circle(2));       // 面积≈12.566
        manager.addShape(new Rectangle(3, 4));  // 面积=12
        manager.addShape(new Triangle(3, 4, 5)); // 面积=6
        manager.printAll();
        cout << "总面积: " << manager.totalArea() << endl;  // 输出≈30.566
    } catch (const exception& e) {
        cout << "错误: " << e.what() << endl;
    }
    return 0;
}
```  


**知识点**：抽象类与纯虚函数、多态的实现（基类指针调用派生类方法）、异常处理、容器管理动态对象。  


---

### **三、模板与泛型编程（高阶）**

#### **题目7（概念辨析）**  
**问题**：解释以下概念的区别与联系：  
- 函数模板（Function Template）与类模板（Class Template）  
- 模板特化（Template Specialization）与模板偏特化（Partial Specialization）  
- 类型萃取（Type Traits）与 `constexpr` 表达式  


**答案**：  
1. **函数模板 vs 类模板**  
   - **函数模板**：生成函数的蓝图（如 `template <typename T> T add(T a, T b)`），==编译器根据实参推导类型==。  
   - **类模板**：生成类的蓝图（如 `template <typename T> class Vector`），==需显式指定类型==（如 `Vector<int>`）。  
   - **联系**：均通过模板实例化生成具体类型的代码（泛型编程）。  

2. **模板特化 vs 偏特化**  
   - **模板特化**：为特定类型提供模板的定制实现（全特化，如 `template <> class Container<int>`）。  
   - **模板偏特化**：为模板参数的子集提供实现（如 `template <typename T> class Container<T*>` 针对指针类型）。  
   - **联系**：均用于优化特定类型的性能或行为，特化优先级高于通用模板。  

3. **类型萃取 vs `constexpr`**  
   - **类型萃取**：通过 `std::is_pointer`、`std::is_integral` 等模板类，在编译期获取类型信息（如判断是否为指针类型）。  
   - **`constexpr` 表达式**：在编译期计算的表达式（如 `constexpr int square(int x) { return x*x; }`），用于优化运行时性能。  
   - **联系**：均用于编译期元编程，类型萃取侧重类型判断，`constexpr` 侧重数值计算。  


**知识点**：模板实例化、特化与偏特化、编译期元编程。  


#### **题目8（代码分析）**  
**问题**：分析以下模板代码的编译结果，并解释原因：  
```cpp  
#include <iostream>
using namespace std;

template <typename T>
class Container {
public:
    void print() { cout << "通用模板" << endl; }
};

template <>  // 全特化
class Container<int> {
public:
    void print() { cout << "int特化模板" << endl; }
};

template <typename T>  // 偏特化（指针类型）
class Container<T*> {
public:
    void print() { cout << "指针偏特化模板" << endl; }
};

int main() {
    Container<double> c1;   // 调用哪个print？
    Container<int> c2;      // 调用哪个print？
    Container<float*> c3;   // 调用哪个print？
    return 0;
}
```  


**答案**：  
- **输出结果**：  
  ```  
  通用模板  
  int特化模板  
  指针偏特化模板  
  ```  
- **原因**：  
  - `c1`：`double` 非 `int` 或指针类型，匹配通用模板 `Container<T>`。  
  - `c2`：`int` 匹配全特化模板 `Container<int>`。  
  - `c3`：`float*` 是指针类型，匹配偏特化模板 `Container<T*>`（偏特化优先级高于通用模板）。  


**知识点**：模板特化的匹配规则（全特化 > 偏特化 > 通用模板）。  


#### **题目9（编程题）**  
**问题**：实现通用 `Stack` 类模板，支持 `push`、`pop`、`top`，扩展 `operator==` 和 `std::allocator`。  


**答案**：  
```cpp  
#include <iostream>
#include <stdexcept>
#include <memory>  // 用于std::allocator
using namespace std;

template <typename T, typename Alloc = std::allocator<T>>
class Stack {
private:
    T* data;
    size_t capacity;
    size_t size;
    Alloc alloc;  // 内存分配器

    void reallocate() {  // 扩容
        size_t new_cap = capacity == 0 ? 1 : capacity * 2;
        T* new_data = alloc.allocate(new_cap);
        for (size_t i = 0; i < size; ++i) {
            alloc.construct(new_data + i, data[i]);  // 构造新元素
            alloc.destroy(data + i);                 // 析构旧元素
        }
        alloc.deallocate(data, capacity);  // 释放旧内存
        data = new_data;
        capacity = new_cap;
    }

public:
    Stack() : data(nullptr), capacity(0), size(0) {}

    ~Stack() {
        for (size_t i = 0; i < size; ++i) alloc.destroy(data + i);
        alloc.deallocate(data, capacity);
    }

    void push(const T& val) {
        if (size == capacity) reallocate();
        alloc.construct(data + size, val);  // 用分配器构造元素
        size++;
    }

    void pop() {
        if (size == 0) throw out_of_range("栈为空");
        size--;
        alloc.destroy(data + size);  // 析构顶部元素
    }

    const T& top() const {
        if (size == 0) throw out_of_range("栈为空");
        return data[size - 1];
    }

    bool operator==(const Stack& other) const {
        if (size != other.size) return false;
        for (size_t i = 0; i < size; ++i) {
            if (data[i] != other.data[i]) return false;
        }
        return true;
    }
};

// 测试
int main() {
    try {
        Stack<int> s1;
        s1.push(1);
        s1.push(2);
        cout << "s1 top: " << s1.top() << endl;  // 输出2

        Stack<int> s2;
        s2.push(1);
        s2.push(2);
        cout << "s1 == s2: " << boolalpha << (s1 == s2) << endl;  // 输出true

        s2.pop();
        cout << "s2 top: " << s2.top() << endl;  // 输出1
    } catch (const exception& e) {
        cout << "错误: " << e.what() << endl;
    }
    return 0;
}
```  


**知识点**：类模板、内存管理（`allocator`）、运算符重载、异常处理。  


---

### **四、内存管理与智能指针（工程重点）**

#### **题目10（概念辨析）**  
**问题**：解释以下概念的底层原理与应用场景：  
- 堆内存（Heap）与栈内存（Stack）的区别  
- 内存泄漏（Memory Leak）与野指针（Dangling Pointer）  
- `unique_ptr`、`shared_ptr`、`weak_ptr` 的设计目的与协作关系  


**答案**：  
1. **堆 vs 栈**  
   - **栈**：由编译器自动管理（如函数参数、局部变量），大小固定（通常几MB），先进后出（LIFO）。  
   - **堆**：由程序员手动分配（`new`/`malloc`），大小动态（受限于内存），需手动释放（`delete`/`free`）。  
   - **应用场景**：栈用于短期小对象；堆用于长期或大对象。  

2. **内存泄漏 vs 野指针**  
   - **内存泄漏**：分配的堆内存未释放（如 `delete` 遗漏），导致内存占用持续增加（常见于循环中）。  
   - **野指针**：指针指向已释放的内存（如 `delete ptr` 后继续使用 `ptr`），引发未定义行为（崩溃或数据错误）。  

3. **智能指针**  
   - **`unique_ptr`**：独占所有权（不可拷贝，仅可移动），管理单个对象（如 `unique_ptr<int> p(new int(5))`）。  
   - **`shared_ptr`**：共享所有权（引用计数管理），多个指针指向同一对象（计数为0时自动释放）。  
   - **`weak_ptr`**：配合 `shared_ptr`，解决循环引用（不增加引用计数，需通过 `lock()` 获取 `shared_ptr`）。  


**知识点**：内存管理、智能指针的所有权模型、循环引用解决方案。  


#### **题目11（代码分析）**  
**问题**：分析以下代码的内存问题，并给出修复方案：  
```cpp  
#include <memory>
using namespace std;

class A {
public:
    A() { cout << "A构造" << endl; }
    ~A() { cout << "A析构" << endl; }
};

class B {
public:
    B() { a = new A(); }
    ~B() { delete a; }  // 是否存在问题？
private:
    A* a;
};

int main() {
    B b1;
    B b2 = b1;  // 调用默认拷贝构造函数，是否安全？
    return 0;
}
```  


**答案**：  
- **问题分析**：  
  1. `B` 的默认拷贝构造函数会复制指针 `a`，导致 `b1` 和 `b2` 的 `a` 指向同一块内存。析构时 `b1` 和 `b2` 都会调用 `delete a`，引发重复释放（未定义行为，可能崩溃）。  
  2. `B` 的析构函数虽然释放了 `a`，但未处理拷贝构造和赋值运算符的深拷贝问题。  

- **修复方案**（深拷贝+禁止拷贝）：  
  ```cpp  
  class B {
  public:
      B() : a(new A()) {}
      ~B() { delete a; }

      // 禁止拷贝（C++11起可用delete）
      B(const B&) = delete;
      B& operator=(const B&) = delete;

      // 若需要拷贝，实现深拷贝（不推荐）
      // B(const B& other) : a(new A(*other.a)) {}
      // B& operator=(const B& other) {
      //     if (this != &other) {
      //         delete a;
      //         a = new A(*other.a);
      //     }
      //     return *this;
      // }
  private:
      A* a;
  };
  ```  


**知识点**：拷贝构造函数、深拷贝与浅拷贝、内存泄漏与重复释放。  


#### **题目12（编程题）**  
**问题**：设计线程安全的 `Singleton` 单例模式，使用 `shared_ptr` 管理生命周期，禁止拷贝。  


**答案**：  
```cpp  
#include <iostream>
#include <memory>
#include <mutex>
using namespace std;

class Singleton {
private:
    static shared_ptr<Singleton> instance;  // 单例实例
    static mutex mtx;                      // 互斥锁（线程安全）

    Singleton() { cout << "Singleton构造" << endl; }  // 私有构造函数
    ~Singleton() { cout << "Singleton析构" << endl; }
    Singleton(const Singleton&) = delete;            // 禁止拷贝
    Singleton& operator=(const Singleton&) = delete;

public:
    static shared_ptr<Singleton> getInstance() {
        if (!instance) {  // 双重检查锁定（DCLP）
            lock_guard<mutex> lock(mtx);
            if (!instance) {
                instance = shared_ptr<Singleton>(new Singleton());
            }
        }
        return instance;
    }
};

// 静态成员初始化
shared_ptr<Singleton> Singleton::instance = nullptr;
mutex Singleton::mtx;

// 测试
int main() {
    auto s1 = Singleton::getInstance();  // 构造Singleton
    auto s2 = Singleton::getInstance();  // 返回同一实例
    cout << "s1和s2是否相同: " << (s1 == s2) << endl;  // 输出1（true）
    return 0;
}
```  


**知识点**：单例模式、线程安全（互斥锁）、智能指针管理生命周期、禁止拷贝。  


---

### **五、STL与设计模式（综合应用）**

#### **题目13（概念辨析）**  
**问题**：简述以下STL组件的底层数据结构与适用场景：  
- `vector`、`list`、`deque` 的区别  
- `map`（红黑树）与 `unordered_map`（哈希表）的性能对比  
- 迭代器的分类（输入/输出/前向/双向/随机访问）  


**答案**：  
1. **`vector`/`list`/`deque`**  
   - **`vector`**：动态数组（连续内存），支持随机访问（`O(1)`），尾部插入/删除快（`O(1)`），中间/头部操作慢（`O(n)`）。适用：需要频繁随机访问、尾部修改的场景。  
   - **`list`**：双向链表（非连续内存），支持任意位置插入/删除（`O(1)`），不支持随机访问（`O(n)`）。适用：需要频繁插入/删除，无需随机访问的场景。  
   - **`deque`**：双端队列（分段连续内存），头尾插入/删除快（`O(1)`），随机访问（`O(1)`，略慢于 `vector`）。适用：需要双端操作的场景（如队列）。  

2. **`map` vs `unordered_map`**  
   - **`map`**：红黑树（平衡二叉搜索树），插入、删除、查找均为 `O(logn)`，元素有序。  
   - **`unordered_map`**：哈希表（数组+链表/红黑树），平均查找 `O(1)`，最坏 `O(n)`（哈希冲突），元素无序。  
   - **性能对比**：`unordered_map` 查找更快（平均），但插入/删除受哈希函数影响；`map` 性能稳定（无哈希冲突），适合需要有序的场景。  

3. **迭代器分类**  
   - **输入迭代器**：只读，单向移动（如 `istream_iterator`）。  
   - **输出迭代器**：只写，单向移动（如 `ostream_iterator`）。  
   - **前向迭代器**：读写，单向移动（如 `forward_list::iterator`）。  
   - **双向迭代器**：读写，双向移动（如 `list::iterator`、`map::iterator`）。  
   - **随机访问迭代器**：读写，支持 `+=`、`-=` 等操作（如 `vector::iterator`）。  


**知识点**：STL容器的底层结构、迭代器类型、性能对比。  


#### **题目14（代码分析）**  
**问题**：分析以下代码的运行结果，并解释STL算法的行为：  
```cpp  
#include <vector>
#include <algorithm>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {3, 1, 4, 1, 5, 9};
    sort(v.begin(), v.end());  // 排序后v的内容？
    auto it = unique(v.begin(), v.end());  // it指向哪里？
    cout << v.size() << endl;   // 输出？
    v.erase(it, v.end());      // 最终v的内容？
    return 0;
}
```  


**答案**：  
- **运行结果**：  
  ```  
  排序后v: {1, 1, 3, 4, 5, 9}  
  it指向第二个1（索引1）  
  v.size(): 6  
  erase后v: {1, 3, 4, 5, 9}  
  ```  
- **原因**：  
  - `sort`：将 `v` 升序排序为 `{1, 1, 3, 4, 5, 9}`。  
  - `unique`：将相邻重复元素移动到容器末尾，返回指向第一个重复元素的迭代器（即第二个 `1` 的位置）。  
  - `v.size()`：`unique` 不改变容器大小，仅覆盖重复元素（原大小仍为6）。  
  - `erase`：删除 `it` 到末尾的元素（即删除第二个 `1` 及之后的冗余位置），最终 `v` 大小为5。  


**知识点**：`sort`、`unique`、`erase` 算法的行为，容器大小与有效元素的区别。  


#### **题目15（设计题）**  
**问题**：设计图书管理系统，支持查询、排序、统计，使用策略模式实现不同排序策略。  


**答案**：  
```cpp  
#include <vector>
#include <algorithm>
#include <iostream>
#include <string>
using namespace std;

// 图书类
class Book {
public:
    string isbn, title, author;
    double price;

    Book(string i, string t, string a, double p) 
        : isbn(i), title(t), author(a), price(p) {}
};

// 排序策略接口（策略模式）
class SortStrategy {
public:
    virtual bool compare(const Book& a, const Book& b) const = 0;
    virtual ~SortStrategy() = default;
};

// 按价格升序排序
class PriceAscSort : public SortStrategy {
public:
    bool compare(const Book& a, const Book& b) const override {
        return a.price < b.price;
    }
};

// 按书名升序排序
class TitleAscSort : public SortStrategy {
public:
    bool compare(const Book& a, const Book& b) const override {
        return a.title < b.title;
    }
};

// 图书管理器
class BookManager {
private:
    vector<Book> books;
    SortStrategy* currentStrategy = nullptr;  // 当前排序策略

public:
    void addBook(const Book& b) { books.push_back(b); }

    // 设置排序策略
    void setSortStrategy(SortStrategy* strategy) {
        currentStrategy = strategy;
    }

    // 排序（使用当前策略）
    void sortBooks() {
        if (currentStrategy) {
            sort(books.begin(), books.end(), 
                [this](const Book& a, const Book& b) {
                    return currentStrategy->compare(a, b);
                });
        }
    }

    // 按书名模糊查询
    vector<Book> searchByTitle(const string& keyword) {
        vector<Book> res;
        for (const auto& b : books) {
            if (b.title.find(keyword) != string::npos) {
                res.push_back(b);
            }
        }
        return res;
    }

    // 统计价格高于阈值的图书数量
    int countAbovePrice(double threshold) {
        return count_if(books.begin(), books.end(),
            [threshold](const Book& b) { return b.price > threshold; });
    }
};

// 测试
int main() {
    BookManager manager;
    manager.addBook(Book("001", "C++ Primer", "Lippman", 99.9));
    manager.addBook(Book("002", "Effective C++", "Meyers", 89.9));
    manager.addBook(Book("003", "C++ Programming", "Stroustrup", 129.9));

    // 按价格排序
    manager.setSortStrategy(new PriceAscSort());
    manager.sortBooks();
    cout << "按价格升序排序后：" << endl;
    for (const auto& b : manager.searchByTitle("C++")) {  // 模糊查询含"C++"的图书
        cout << b.title << " 价格: " << b.price << endl;
    }

    // 统计价格高于100的图书
    cout << "价格>100的图书数量: " << manager.countAbovePrice(100) << endl;  // 输出1
    return 0;
}
```  


**知识点**：策略模式（`SortStrategy` 接口）、STL算法（`sort`、`count_if`）、模糊查询（`string::find`）。  


---

### **复习重点总结**  
- **高频考点**：虚函数表与动态多态、智能指针（`shared_ptr`/`unique_ptr`）、模板特化、STL容器与算法。  
- **易错题**：虚析构函数的必要性（题目2）、多继承二义性（题目4）、模板特化匹配（题目8）。  
- **工程能力**：异常处理（题目3/6）、内存管理（题目9/11）、设计模式（题目15）。  

通过以上题目与答案，可系统覆盖C++面向对象的核心知识，建议结合代码调试和《C++ Primer》《深度探索C++对象模型》深化理解。