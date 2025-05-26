
### C++ 中 `function` 和 `bind` 的详细讲解


#### **一、`std::function` 知识点与 API**
- **头文件**：`#include <functional>`  
- **模板定义**：  
  ```cpp
  template<class Ret, class... Args>
  class function<Ret(Args...)>;
  ```  
  - `Ret`：返回值类型（如 `int`、`void`）。  
  - `Args...`：参数类型列表（如 `int, double`）。  
- **功能**：封装 **可调用对象**（函数、lambda、成员函数、`bind` 对象等），提供统一调用接口。  
- **调用方式**：通过 `operator()` 调用，如 `f(1, 2.5);`（假设 `f` 是 `function<double(int, double)>` 类型）。


#### **二、`std::bind` 知识点与 API**
- **头文件**：`#include <functional>`  
- **函数定义**：  
  ```cpp
  template<class F, class... Args>
  auto bind(F&& f, Args&&... args) -> /* 推导的可调用类型 */;
  ```  
  - `f`：可调用对象（函数、成员函数、lambda 等）。  
  - `args`：参数列表，可包含 **具体值** 或 **占位符**（`std::placeholders::_1, _2, ...`，表示后续调用时的参数位置）。  
- **功能**：生成新的可调用对象，绑定部分参数（柯里化）或调整参数顺序。


#### **三、`function` 使用示例**

##### **1. 存储普通函数**
```cpp
#include <iostream>
#include <functional>

int add(int a, int b) { return a + b; }

int main() {
    std::function<int(int, int)> f = add; // 封装加法函数
    std::cout << f(3, 4) << std::endl; // 输出 7
    return 0;
}
```

##### **2. 存储 Lambda 表达式**
```cpp
int main() {
    std::function<int(int, int)> f = [](int a, int b) { return a * b; };
    std::cout << f(3, 4) << std::endl; // 输出 12
    return 0;
}
```

##### **3. 存储成员函数（需传递对象实例）**
```cpp
class Math {
public:
    int multiply(int a, int b) { return a * b; }
};

int main() {
    Math obj;
    // 成员函数封装：第一个参数是对象（引用/指针），后续是成员函数参数
    std::function<int(Math&, int, int)> f = &Math::multiply;
    std::cout << f(obj, 3, 4) << std::endl; // 输出 12
    return 0;
}
```

##### **4. 存储 `bind` 对象（结合 `bind` 使用）**
```cpp
int sub(int a, int b) { return a - b; }

int main() {
    auto bound = std::bind(sub, 10, std::placeholders::_1); // 绑定第一个参数为 10，第二个参数为占位符 _1
    std::function<int(int)> f = bound; // 封装 bind 对象
    std::cout << f(3) << std::endl; // 输出 7（10 - 3）
    return 0;
}
```


#### **四、`bind` 使用示例**

##### **1. 绑定固定参数（柯里化）**
```cpp
void print(int a, int b, int c) {
    std::cout << a << ", " << b << ", " << c << std::endl;
}

int main() {
    using namespace std::placeholders; // 启用占位符
    auto bound = std::bind(print, 1, _2, 3); // 绑定第一个参数为 1，第三个为 3，第二个为 _2（后续调用时传入）
    bound(2); // 输出 1, 2, 3（调用时传入 _2 的值 2）  //这里有问题,最好还是动手自己敲一下,跑下程序验证
    return 0;
}
```

##### **2. 调整参数顺序**
```cpp
int div(int a, int b) { return a / b; }

int main() {
    using namespace std::placeholders;
    auto bound = std::bind(div, _2, _1); // 交换参数顺序（原 a/b 变为 b/a）
    std::cout << bound(4, 2) << std::endl; // 输出 0（2/4，整数除法）
    return 0;
}
```

##### **3. 绑定成员函数（需对象实例）**
```cpp
class Printer {
public:
    void printMsg(int num) { std::cout << "Msg: " << num << std::endl; }
};

int main() {
    Printer obj;
    using namespace std::placeholders;
    auto bound = std::bind(&Printer::printMsg, &obj, _1); // 绑定对象指针 &obj，参数 _1
    bound(10); // 输出 Msg: 10
    return 0;
}
```

##### **4. 混合绑定与占位符**
```cpp
int func(int a, int b, int c, int d) { return a + b * c - d; }

int main() {
    using namespace std::placeholders;
    // 绑定 a=1, c=3，b 为 _2，d 为 _1（调用时先传 d，再传 b）
    auto bound = std::bind(func, 1, _2, 3, _1); 
    std::cout << bound(5, 2) << std::endl; // 计算 1 + 2*3 - 5 = 2
    return 0;
}
```


#### **五、常见场景总结**
1. **`function` 作为回调函数**：  
   存储不同类型的可调用对象（如函数、lambda），实现统一回调接口。  
   ```cpp
   void process(std::function<void()> callback) {
       callback(); // 调用任意可调用对象
   }

   int main() {
       process([]() { std::cout << "Lambda callback" << std::endl; });
       return 0;
   }
   ```

2. **`bind` 实现参数适配**：  
   调整函数参数顺序，或固定部分参数，简化函数调用（如适配第三方库的回调接口）。

3. **成员函数与 `function`/`bind` 结合**：  
   封装成员函数时，需先绑定对象实例（通过指针或引用），再处理参数。


#### **六、注意事项**
- **头文件与命名空间**：  
  必须包含 `#include <functional>`，使用占位符时需 `using namespace std::placeholders;` 或直接写 `std::placeholders::_1`。  
- **空 `function` 调用**：  
  若 `function` 未绑定可调用对象，调用时会抛出 `std::bad_function_call` 异常，需检查初始化。  
- **参数传递方式**：  
  `bind` 对 **预先绑定的参数** 按值传递，对 **占位符参数** 按引用传递（若原参数为引用，需注意生命周期）。


### 总结
- **`function`** 是通用的可调用对象容器，统一管理函数、lambda、成员函数等。  
- **`bind`** 是灵活的参数适配器，支持参数绑定、顺序调整，生成新的可调用对象。  
- 两者结合使用，可大幅提升代码的灵活性（如回调函数、事件处理、泛型编程等场景）。

通过上述示例，初学者可逐步掌握：  
1. 如何用 `function` 封装不同类型的可调用对象。  
2. 如何用 `bind` 调整参数、固定值，生成新的调用逻辑。  
3. 常见场景（如成员函数绑定、回调适配）的实际应用。

继续练习不同的可调用对象和参数绑定场景，可加深对这两个工具的理解。