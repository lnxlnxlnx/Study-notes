



### C++ STL与可调用对象：高效编程的两大支柱  

#### **一、STL（标准模板库）**  
STL是C++中极具价值的工具集，包含容器、算法和迭代器，实现了数据结构和算法的高度抽象与复用。  

##### 1.1 容器（Containers）  
- **序列式容器**：`vector`（动态数组）、`list`（双向链表）、`deque`（双端队列），按插入顺序存储元素。  
- **关联式容器**：`set`（有序去重集合）、`map`（键值对映射），基于红黑树实现，自动排序。  

**示例1：`vector` 基本操作**  
```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> nums = {1, 3, 5};
    nums.push_back(7);  // 尾部插入
    nums.insert(nums.begin() + 1, 2);  // 指定位置插入

    cout << "vector元素: ";
    for (int num : nums) {  // 范围for遍历
        cout << num << " ";  // 输出：1 2 3 5 7
    }
    return 0;
}
```

**示例2：`map` 键值对操作**  
```cpp
#include <iostream>
#include <map>
#include <string>
using namespace std;

int main() {
    map<string, int> scores;
    scores["Alice"] = 90;  // 插入/更新
    scores["Bob"] = 85;

    if (scores.count("Alice")) {  // 检查键是否存在
        cout << "Alice的分数: " << scores["Alice"] << endl;  // 输出：90
    }
    return 0;
}
```

##### 1.2 算法（Algorithms）  
STL算法库（`<algorithm>`）提供通用操作，如排序、查找、变换，通过迭代器操作容器。  

**示例3：`sort` 排序与自定义比较**  
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

bool compare(int a, int b) {  // 降序比较
    return a > b;
}

int main() {
    vector<int> nums = {3, 1, 4, 1, 5};
    sort(nums.begin(), nums.end(), compare);  // 自定义排序

    cout << "排序后: ";
    for (int num : nums) {
        cout << num << " ";  // 输出：5 4 3 1 1
    }
    return 0;
}
```

**示例4：`find` 查找元素**  
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> nums = {2, 4, 6, 8};
    auto it = find(nums.begin(), nums.end(), 6);  // 查找6

    if (it != nums.end()) {
        cout << "找到元素6，位置: " << distance(nums.begin(), it) << endl;  // 输出：位置2
    }
    return 0;
}
```

##### 1.3 迭代器（Iterators）  
迭代器是容器与算法之间的接口，分为输入、输出、前向、双向、随机访问迭代器。  

**示例5：迭代器遍历`list`**  
```cpp
#include <iostream>
#include <list>
using namespace std;

int main() {
    list<int> lst = {10, 20, 30};
    for (auto it = lst.begin(); it != lst.end(); ++it) {
        cout << *it << " ";  // 输出：10 20 30
    }
    return 0;
}
```

---

#### **二、可调用对象（Callable Objects）**  
可调用对象是能像函数一样执行的实体，包括普通函数、Lambda、函数对象（仿函数）、`std::function` 和 `std::bind` 绑定的对象。  

##### 2.1 普通函数与函数指针  
**示例6：普通函数作为可调用对象**  
```cpp
#include <iostream>
using namespace std;

int add(int a, int b) {  // 普通函数
    return a + b;
}

int main() {
    int (*ptr)(int, int) = add;  // 函数指针
    cout << "调用函数指针: " << ptr(3, 5) << endl;  // 输出：8
    return 0;
}
```

##### 2.2 Lambda 表达式  
Lambda 是匿名函数对象，可捕获外部变量，适合简短的回调或算法自定义。  

**示例7：Lambda 捕获变量与排序**  
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> nums = {3, 1, 4};
    int threshold = 2;

    // Lambda捕获threshold，筛选元素
    auto filtered = find_if(nums.begin(), nums.end(), [threshold](int n) {
        return n > threshold;  // 查找第一个大于2的元素
    });

    if (filtered != nums.end()) {
        cout << "找到元素: " << *filtered << endl;  // 输出：3
    }
    return 0;
}
```

##### 2.3 函数对象（仿函数）  
通过定义类并重载 `operator()`，创建可调用的对象。  

**示例8：仿函数实现自定义比较**  
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

class GreaterThan {  // 仿函数
public:
    bool operator()(int a, int b) const {
        return a > b;
    }
};

int main() {
    vector<int> nums = {5, 2, 8};
    sort(nums.begin(), nums.end(), GreaterThan());  // 使用仿函数排序

    cout << "排序后: ";
    for (int num : nums) {
        cout << num << " ";  // 输出：8 5 2
    }
    return 0;
}
```

##### 2.4 `std::function` 与 `std::bind`  
- `std::function`：封装可调用对象，实现类型擦除。  
- `std::bind`：绑定可调用对象的参数，调整调用签名。  

**示例9：`std::function` 存储可调用对象**  
```cpp
#include <iostream>
#include <functional>
using namespace std;

int multiply(int a, int b) {
    return a * b;
}

int main() {
    function<int(int, int)> func = multiply;  // 存储函数
    cout << "调用function: " << func(4, 5) << endl;  // 输出：20
    return 0;
}
```

**示例10：`std::bind` 绑定参数**  
```cpp
#include <iostream>
#include <functional>
using namespace std;

void printInfo(string name, int age) {
    cout << "姓名: " << name << ", 年龄: " << age << endl;
}

int main() {
    auto bound = bind(printInfo, "Alice", placeholders::_1);  // 绑定name为"Alice"
    bound(30);  // 输出：姓名: Alice, 年龄: 30
    return 0;
}
```

---

### **总结**  
- **STL**：通过容器存储数据，算法处理数据，迭代器连接两者，实现高效的数据管理与操作。  
- **可调用对象**：普通函数、Lambda、仿函数提供灵活的逻辑定义方式，`std::function` 和 `std::bind` 增强了可调用对象的通用性与适配性。在实际编程中，STL 提升代码复用性，可调用对象简化逻辑封装，两者结合能大幅提高开发效率与代码质量。 