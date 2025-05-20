
### Hint:

在函数中,容器会被衰退为指针,所以不能使用范围for循环!!!
![[Pasted image 20250520170246.png]]


### 一、C++ 范围 for 循环  
范围 for 循环（Range-based for loop）是 C++11 引入的特性，用于简化容器或数组的遍历，自动处理迭代器，无需手动管理索引。  
**基本语法**：  
```cpp  
for (元素类型 变量 : 容器) {  
    // 对变量操作（变量是元素的拷贝）  
}  
```  
若要直接修改容器元素，可通过引用（`&`）避免拷贝：  
```cpp  
for (元素类型& 变量 : 容器) {  
    // 修改变量即修改容器元素  
}  
```  

**案例 1：遍历数组**  
```cpp  
#include <iostream>  
using namespace std;  

int main() {  
    int arr[] = {1, 2, 3, 4, 5};  
    for (int num : arr) {  
        cout << num << " "; // 输出：1 2 3 4 5  
    }  
    return 0;  
}  
```  

**案例 2：遍历并修改 `vector`**  
```cpp  
#include <vector>  
#include <iostream>  
using namespace std;  

int main() {  
    vector<int> vec = {1, 2, 3};  
    for (int& num : vec) { // 通过引用修改元素  
        num *= 2;  
    }  
    for (int num : vec) {  
        cout << num << " "; // 输出：2 4 6  
    }  
    return 0;  
}  
```  

**案例 3：遍历 `map`**  
```cpp  
#include <map>  
#include <iostream>  
using namespace std;  

int main() {  
    map<string, int> m = {{"a", 1}, {"b", 2}};  
    for (const auto& pair : m) { // 常引用避免拷贝，适合只读操作  
        cout << pair.first << ": " << pair.second << endl;  
    }  
    return 0;  
}  
```  

---

### 二、C++ lambda 表达式  
lambda 表达式用于定义匿名函数，可捕获外部变量，常用于 STL 算法或简化代码。  
**基本语法**：  
```cpp  
[capture列表](参数列表) -> 返回类型 {函数体}  
```  
- **`capture`**：捕获外部变量（`= `值捕获，`& `引用捕获，也可混合）。  
- **`parameters`**：参数列表（可省略）。  
- **`-> 返回类型`**：返回类型（可省略，自动推导）。  

**案例 1：简单求和**  
```cpp  
#include <iostream>  
using namespace std;  

int main() {  
    auto add = [](int a, int b) { return a + b; };  
    cout << add(3, 5) << endl; // 输出：8  
    return 0;  
}  
```  

**案例 2：捕获变量**  
```cpp  
#include <iostream>  
using namespace std;  

int main() {  
    int x = 10;  
    auto lambda = [x]() { // 值捕获 x  
        cout << "x (copied): " << x << endl;  
    };  
    x = 20;  
    lambda(); // 输出：x (copied): 10（捕获的是拷贝）  

    auto lambda_ref = [&x]() { // 引用捕获 x  
        cout << "x (referenced): " << x << endl;  
    };  
    x = 30;  
    lambda_ref(); // 输出：x (referenced): 30  
    return 0;  
}  
```  

**案例 3：作为 `sort` 比较函数**  
```cpp  
#include <vector>  
#include <algorithm>  
#include <iostream>  
using namespace std;  

int main() {  
    vector<int> vec = {3, 1, 4};  
    sort(vec.begin(), vec.end(), [](int a, int b) {  
        return a > b; // 降序排序  
    });  
    for (int num : vec) {  
        cout << num << " "; // 输出：4 3 1  
    }  
    return 0;  
}  
```  

**案例 4：结合 STL 算法统计元素**  
```cpp  
#include <vector>  
#include <algorithm>  
#include <iostream>  
using namespace std;  

int main() {  
    vector<int> vec = {1, 2, 3, 4, 4};  
    int count = count_if(vec.begin(), vec.end(), [](int x) {  
        return x > 3; // 统计大于 3 的元素个数  
    });  
    cout << "Count: " << count << endl; // 输出：2  
    return 0;  
}  
```  

**案例 5：可变 `lambda`（`mutable`）**  
```cpp  
#include <iostream>  
using namespace std;  

int main() {  
    int x = 5;  
    auto lambda = [x]() mutable { // mutable 允许修改值捕获的变量  
        x *= 2;  
        cout << "Modified x: " << x << endl;  
    };  
    lambda(); // 输出：Modified x: 10（修改的是拷贝）  
    cout << "Original x: " << x << endl; // 输出：Original x: 5（原始 x 未变）  
    return 0;  
}  
```  

通过以上案例，可清晰理解范围 for 循环的简洁遍历和 lambda 表达式的灵活应用，两者结合能大幅提升代码的简洁性与可读性。 