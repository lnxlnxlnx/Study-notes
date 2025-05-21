
### C++ STL与可调用对象深度解析（含API+多示例）  


---

## **一、STL容器：数据存储的基石**  
STL容器分为**序列式**、**关联式**、**无序关联式**和**容器适配器**四大类，覆盖了从动态数组到哈希表的所有常见数据存储需求。  


### **1.1 序列式容器（顺序存储）**  
**核心API**（以`vector`为例）：  
| 操作                | 描述                                  | 时间复杂度       |  
|---------------------|---------------------------------------|------------------|  
| `push_back(val)`     | 尾部插入元素                          | O(1)（均摊）     |  
| `emplace_back(args)` | 原地构造并插入元素（避免拷贝）        | O(1)（均摊）     |  
| `insert(it, val)`    | 在迭代器`it`位置插入元素              | O(n)             |  
| `erase(it)`          | 删除迭代器`it`位置的元素              | O(n)             |  
| `resize(size)`       | 调整容器大小（可能触发扩容）          | O(n)             |  


#### **示例1：`vector` 的动态扩容与高效插入**  
```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> vec;
    // 预分配空间（避免多次扩容）
    vec.reserve(5);  // 预留5个元素空间，size仍为0

    // emplace_back直接构造int（比push_back更高效）
    vec.emplace_back(1);  // vec: [1]
    vec.push_back(2);     // vec: [1, 2]

    // 在第二个位置插入3（迭代器指向vec.begin()+1）
    vec.insert(vec.begin() + 1, 3);  // vec: [1, 3, 2]

    // 删除第一个元素
    vec.erase(vec.begin());  // vec: [3, 2]

    cout << "当前容量: " << vec.capacity() << endl;  // 输出：5（预留空间未释放）
    cout << "元素: ";
    for (int num : vec) cout << num << " ";  // 输出：3 2
    return 0;
}
```


### **1.2 关联式容器（有序存储）**  
以`map<K,V>`（键值对）和`set<T>`（单值集合）为例，基于红黑树实现，自动排序（默认升序）。  
**核心API**：  
| 操作                | 描述                                  | 时间复杂度       |  
|---------------------|---------------------------------------|------------------|  
| `insert({k, v})`     | 插入键值对（`map`）或元素（`set`）    | O(log n)         |  
| `emplace(k, v)`      | 原地构造并插入（避免临时对象）        | O(log n)         |  
| `find(k)`            | 查找键`k`（返回迭代器）               | O(log n)         |  
| `erase(k)`           | 删除键`k`对应的元素                   | O(log n)         |  


#### **示例2：`map` 的高效插入与查找**  
```cpp
#include <iostream>
#include <map>
#include <string>
using namespace std;

int main() {
    map<string, int> student_scores;

    // emplace直接构造键值对（比insert更高效）
    student_scores.emplace("Alice", 95);
    student_scores.insert({"Bob", 88});  // 等价于emplace，但可能多一次拷贝

    // 查找Alice的分数
    auto it = student_scores.find("Alice");
    if (it != student_scores.end()) {
        cout << "Alice分数: " << it->second << endl;  // 输出：95
    }

    // 删除Bob的记录
    student_scores.erase("Bob");
    cout << "map大小: " << student_scores.size() << endl;  // 输出：1
    return 0;
}
```


### **1.3 无序关联式容器（哈希存储）**  
以`unordered_map<K,V>`和`unordered_set<T>`为例，基于哈希表实现，平均时间复杂度为O(1)（最坏O(n)）。  
**核心API**（与`map`类似，但无自动排序）：  
| 操作                | 描述                                  | 平均时间复杂度   |  
|---------------------|---------------------------------------|------------------|  
| `bucket_count()`     | 返回哈希桶数量                        | O(1)             |  
| `load_factor()`      | 返回负载因子（元素数/桶数）            | O(1)             |  


#### **示例3：`unordered_map` 自定义哈希函数**  
```cpp
#include <iostream>
#include <unordered_map>
#include <string>
using namespace std;

// 自定义哈希函数（针对string）
struct StringHash {
    size_t operator()(const string& s) const {
        size_t hash = 0;
        for (char c : s) hash = hash * 31 + c;  // 简单的多项式哈希
        return hash;
    }
};

int main() {
    unordered_map<string, int, StringHash> umap;
    umap["apple"] = 10;
    umap["banana"] = 20;

    // 遍历所有桶
    for (size_t i = 0; i < umap.bucket_count(); ++i) {
        cout << "桶" << i << ": ";
        for (auto it = umap.begin(i); it != umap.end(i); ++it) {
            cout << "(" << it->first << "," << it->second << ") ";
        }
        cout << endl;
    }
    return 0;
}
```


### **1.4 容器适配器（受限接口）**  
`stack`（栈，LIFO）、`queue`（队列，FIFO）、`priority_queue`（优先队列，最大堆默认）。  
**核心API**（以`priority_queue`为例）：  
| 操作                | 描述                                  | 时间复杂度       |  
|---------------------|---------------------------------------|------------------|  
| `push(val)`          | 插入元素（堆调整）                    | O(log n)         |  
| `top()`              | 获取堆顶元素（最大元素）              | O(1)             |  
| `pop()`              | 删除堆顶元素                          | O(log n)         |  


#### **示例4：`priority_queue` 实现大顶堆**  
```cpp
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

int main() {
    priority_queue<int> pq;  // 默认大顶堆
    pq.push(3);
    pq.push(1);
    pq.push(4);

    cout << "堆顶元素: " << pq.top() << endl;  // 输出：4
    pq.pop();  // 删除4
    cout << "新堆顶: " << pq.top() << endl;     // 输出：3
    return 0;
}
```


---

## **二、STL算法：数据处理的引擎**  
STL算法库（`<algorithm>`）提供了超过100种通用算法，通过迭代器操作容器，覆盖排序、查找、修改、统计等场景。  


### **2.1 排序与分组**  
**核心API**：  
| 算法                | 描述                                  | 时间复杂度       |  
|---------------------|---------------------------------------|------------------|  
| `sort(it_begin, it_end)` | 快速排序（不稳定）                   | O(n log n)       |  
| `stable_sort(...)`   | 稳定排序（保持相等元素的相对顺序）    | O(n log n)       |  
| `nth_element(...)`   | 部分排序（找到第n大元素）             | O(n)             |  


#### **示例5：`sort` 与 `stable_sort` 的区别**  
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

struct Student {
    string name;
    int score;
};

int main() {
    vector<Student> students = {
        {"Alice", 90}, {"Bob", 90}, {"Charlie", 85}
    };

    // 按分数排序（不稳定排序，相同分数顺序可能变化）
    sort(students.begin(), students.end(), [](const Student& a, const Student& b) {
        return a.score > b.score;
    });
    cout << "不稳定排序后: ";
    for (auto& s : students) cout << s.name << " ";  // 可能输出：Alice Bob Charlie 或 Bob Alice Charlie

    // 稳定排序（保持相同分数的原始顺序）
    stable_sort(students.begin(), students.end(), [](const Student& a, const Student& b) {
        return a.score > b.score;
    });
    cout << "\n稳定排序后: ";
    for (auto& s : students) cout << s.name << " ";  // 输出：Alice Bob Charlie（原始顺序保留）
    return 0;
}
```


### **2.2 查找与统计**  
**核心API**：  
| 算法                | 描述                                  | 时间复杂度       |  
|---------------------|---------------------------------------|------------------|  
| `find(it_begin, it_end, val)` | 线性查找元素                        | O(n)             |  
| `find_if(...)`       | 按条件查找（传入可调用对象）          | O(n)             |  
| `binary_search(...)` | 二分查找（要求容器已排序）            | O(log n)         |  


#### **示例6：`find_if` 结合Lambda筛选元素**  
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> nums = {1, 3, 5, 7, 9};

    // 查找第一个大于5的元素
    auto it = find_if(nums.begin(), nums.end(), [](int n) {
        return n > 5;
    });

    if (it != nums.end()) {
        cout << "第一个大于5的元素: " << *it << endl;  // 输出：7
    }
    return 0;
}
```


### **2.3 修改与变换**  
**核心API**：  
| 算法                | 描述                                  | 时间复杂度       |  
|---------------------|---------------------------------------|------------------|  
| `transform(...)`     | 对容器元素应用变换（如平方、转换类型）| O(n)             |  
| `copy(...)`          | 复制元素到另一个容器                  | O(n)             |  
| `replace(...)`       | 替换满足条件的元素                    | O(n)             |  


#### **示例7：`transform` 实现元素平方**  
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> nums = {1, 2, 3};
    vector<int> squares;
    squares.reserve(nums.size());  // 预分配空间

    // 将nums的每个元素平方后存入squares
    transform(nums.begin(), nums.end(), back_inserter(squares),
              [](int n) { return n * n; });

    cout << "平方后: ";
    for (int s : squares) cout << s << " ";  // 输出：1 4 9
    return 0;
}
```


---

## **三、可调用对象：逻辑封装的利器**  
可调用对象是C++中能像函数一样调用的实体，包括普通函数、Lambda、函数对象（仿函数）、`std::function` 和 `std::bind` 绑定对象。  


### **3.1 Lambda表达式**  
**核心特性**：  
- 捕获列表：`[]`（无捕获）、`[=]`（值捕获）、`[&]`（引用捕获）、`[this]`（捕获当前对象指针）。  
- 泛型Lambda（C++14）：参数用`auto`声明。  


#### **示例8：Lambda的捕获与泛型**  
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> nums = {1, 2, 3};
    int multiplier = 10;

    // 引用捕获multiplier，将nums元素放大10倍
    for_each(nums.begin(), nums.end(), [&](int& n) {
        n *= multiplier;  // 引用捕获，修改原multiplier会影响结果
    });

    // 泛型Lambda（支持任意类型相加）
    auto add = [](auto a, auto b) { return a + b; };
    cout << "int相加: " << add(2, 3) << endl;        // 输出：5
    cout << "string相加: " << add("hello", "world") << endl;  // 输出：helloworld
    return 0;
}
```


### **3.2 函数对象（仿函数）**  
通过重载`operator()`实现，可携带状态（如计数器）。  


#### **示例9：仿函数的状态保持**  
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

class Counter {
private:
    int count = 0;

public:
    void operator()(int n) {
        if (n % 2 == 0) count++;  // 统计偶数数量
    }

    int getCount() const { return count; }
};

int main() {
    vector<int> nums = {1, 2, 3, 4, 5};
    Counter counter = for_each(nums.begin(), nums.end(), Counter());

    cout << "偶数数量: " << counter.getCount() << endl;  // 输出：2（2和4）
    return 0;
}
```


### **3.3 `std::function`：类型擦除的容器**  
`std::function` 可以存储任意可调用对象（函数、Lambda、仿函数），统一调用接口。  


#### **示例10：`std::function` 存储多类型可调用对象**  
```cpp
#include <iostream>
#include <functional>
using namespace std;

// 普通函数
int add(int a, int b) { return a + b; }

// 仿函数
struct Multiplier {
    int operator()(int a, int b) { return a * b; }
};

int main() {
    function<int(int, int)> func;

    func = add;  // 存储普通函数
    cout << "add(3,4): " << func(3,4) << endl;  // 输出：7

    func = Multiplier();  // 存储仿函数
    cout << "Multiplier(3,4): " << func(3,4) << endl;  // 输出：12

    func = [](int a, int b) { return a - b; };  // 存储Lambda
    cout << "Lambda(3,4): " << func(3,4) << endl;  // 输出：-1
    return 0;
}
```


### **3.4 `std::bind`：参数绑定与调整**  
`std::bind` 可以绑定可调用对象的参数，调整参数顺序，或绑定默认值。  


#### **示例11：`std::bind` 调整参数顺序**  
```cpp
#include <iostream>
#include <functional>
using namespace std;

void print(int a, int b) {
    cout << "a=" << a << ", b=" << b << endl;
}

int main() {
    // 绑定print的第一个参数为10，第二个参数由调用时提供（placeholders::_1）
    auto bind1 = bind(print, 10, placeholders::_1);
    bind1(20);  // 输出：a=10, b=20

    // 交换参数顺序（第一个参数是调用时的第二个参数）
    auto bind2 = bind(print, placeholders::_2, placeholders::_1);
    bind2(20, 10);  // 输出：a=10, b=20（参数顺序交换）
    return 0;
}
```


---

### **总结**  
- **STL容器**：通过丰富的API（如`vector::emplace_back`、`map::find`）实现高效数据存储，需根据场景选择合适容器（如动态数组用`vector`，快速查找用`unordered_map`）。  
- **STL算法**：通过迭代器与容器解耦（如`sort`、`transform`），结合可调用对象实现灵活逻辑（如Lambda自定义排序）。  
- **可调用对象**：Lambda简化短逻辑封装，仿函数保持状态，`std::function`统一接口，`std::bind`调整参数，共同提升代码灵活性。  

通过大量示例和API说明，可快速掌握STL与可调用对象的核心用法，在实际开发中显著提升效率。