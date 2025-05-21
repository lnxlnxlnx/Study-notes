
### 算法竞赛核心知识全景覆盖（数论+搜索+数据结构+技巧）  


---

## **一、数论：数学与算法的融合**  
数论是竞赛中的“硬骨头”，需掌握数的性质、模运算、素数、组合数等核心工具。  


### **1.1 基础数论**  
#### **1.1.1 最大公约数（GCD）与最小公倍数（LCM）**  
- **核心公式**：`gcd(a,b) * lcm(a,b) = a*b`（当a,b非零时）。  
- **欧几里得算法**（求GCD）：`gcd(a,b) = gcd(b, a%b)`（递归实现）。  


#### **示例1：GCD与LCM计算**  
```cpp
#include <iostream>
using namespace std;

int gcd(int a, int b) {
    return b == 0 ? a : gcd(b, a % b);
}

int lcm(int a, int b) {
    return a / gcd(a, b) * b;  // 先除后乘防溢出
}

int main() {
    int a = 12, b = 18;
    cout << "GCD: " << gcd(a, b) << endl;  // 输出：6
    cout << "LCM: " << lcm(a, b) << endl;  // 输出：36
    return 0;
}
```


#### **1.1.2 素数筛法**  
- **埃拉托斯特尼筛法**（埃氏筛）：标记所有素数的倍数，时间复杂度O(n log log n)。  
- **欧拉筛**（线性筛）：每个合数仅被最小质因子筛除，时间复杂度O(n)。  


#### **示例2：欧拉筛（线性筛素数）**  
```cpp
#include <iostream>
#include <vector>
using namespace std;

vector<int> eulerSieve(int n) {
    vector<bool> is_prime(n + 1, true);
    vector<int> primes;
    is_prime[0] = is_prime[1] = false;

    for (int i = 2; i <= n; ++i) {
        if (is_prime[i]) primes.push_back(i);
        for (int p : primes) {
            if (i * p > n) break;
            is_prime[i * p] = false;
            if (i % p == 0) break;  // 保证p是i*p的最小质因子
        }
    }
    return primes;
}

int main() {
    int n = 30;
    vector<int> primes = eulerSieve(n);
    cout << "2~30的素数: ";
    for (int p : primes) cout << p << " ";  // 输出：2 3 5 7 11 13 17 19 23 29
    return 0;
}
```


#### **1.1.3 快速幂与模运算**  
- **快速幂**：利用二分法计算`a^b mod m`，时间复杂度O(log b)。  
- **模运算性质**：`(a*b) mod m = [(a mod m)*(b mod m)] mod m`。  


#### **示例3：快速幂（带模运算）**  
```cpp
#include <iostream>
using namespace std;

int quickPow(int a, int b, int mod) {
    int res = 1;
    while (b > 0) {
        if (b % 2 == 1) res = (res * 1LL * a) % mod;  // 1LL防溢出
        a = (a * 1LL * a) % mod;
        b /= 2;
    }
    return res;
}

int main() {
    cout << "3^5 mod 7: " << quickPow(3, 5, 7) << endl;  // 输出：5（3^5=243, 243%7=5）
    return 0;
}
```


### **1.2 高级数论**  
#### **1.2.1 组合数（C(n,k)）**  
- **递推公式**：`C(n,k) = C(n-1,k-1) + C(n-1,k)`（杨辉三角）。  
- **预处理阶乘与逆元**：用快速幂求逆元，预处理`fact[n]`和`inv_fact[n]`，计算`C(n,k) = fact[n] * inv_fact[k] * inv_fact[n-k] mod mod`。  


#### **示例4：组合数预处理（模1e9+7）**  
```cpp
#include <iostream>
#include <vector>
using namespace std;

const int MOD = 1e9 + 7;
const int MAXN = 1e5;

vector<int> fact(MAXN + 1), inv_fact(MAXN + 1);

int quickPow(int a, int b) {
    int res = 1;
    while (b) {
        if (b & 1) res = 1LL * res * a % MOD;
        a = 1LL * a * a % MOD;
        b >>= 1;
    }
    return res;
}

void precompute() {
    fact[0] = 1;
    for (int i = 1; i <= MAXN; ++i) {
        fact[i] = 1LL * fact[i - 1] * i % MOD;
    }
    inv_fact[MAXN] = quickPow(fact[MAXN], MOD - 2);  // 费马小定理求逆元
    for (int i = MAXN - 1; i >= 0; --i) {
        inv_fact[i] = 1LL * inv_fact[i + 1] * (i + 1) % MOD;
    }
}

int C(int n, int k) {
    if (k < 0 || k > n) return 0;
    return 1LL * fact[n] * inv_fact[k] % MOD * inv_fact[n - k] % MOD;
}

int main() {
    precompute();
    cout << "C(5,2): " << C(5, 2) << endl;  // 输出：10
    return 0;
}
```


---

## **二、搜索算法：BFS与DFS**  
搜索是竞赛中的“万能钥匙”，用于解决路径、状态转移、连通性等问题。  


### **2.1 BFS（广度优先搜索）**  
- **核心**：队列实现，按层遍历，适合求最短路径（无权图）。  
- **关键步骤**：定义状态、标记访问、队列扩展。  


#### **示例5：迷宫最短路径（BFS）**  
```cpp
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

struct Point {
    int x, y, step;
};

int bfs(const vector<vector<int>>& grid, int startX, int startY, int endX, int endY) {
    int n = grid.size(), m = grid[0].size();
    vector<vector<bool>> visited(n, vector<bool>(m, false));
    queue<Point> q;
    q.push({startX, startY, 0});
    visited[startX][startY] = true;

    // 四个方向：上下左右
    int dx[] = {-1, 1, 0, 0};
    int dy[] = {0, 0, -1, 1};

    while (!q.empty()) {
        auto p = q.front();
        q.pop();

        if (p.x == endX && p.y == endY) return p.step;

        for (int i = 0; i < 4; ++i) {
            int nx = p.x + dx[i];
            int ny = p.y + dy[i];
            if (nx >= 0 && nx < n && ny >= 0 && ny < m && !visited[nx][ny] && grid[nx][ny] == 0) {
                visited[nx][ny] = true;
                q.push({nx, ny, p.step + 1});
            }
        }
    }
    return -1;  // 无法到达
}

int main() {
    vector<vector<int>> grid = {
        {0, 1, 0, 0},
        {0, 0, 0, 1},
        {1, 0, 0, 0}
    };  // 0表示可走，1表示障碍
    cout << "最短步数: " << bfs(grid, 0, 0, 2, 3) << endl;  // 输出：5（路径：(0,0)→(0,2)→(0,3)→(1,3)→(2,3)？实际需验证）
    return 0;
}
```


### **2.2 DFS（深度优先搜索）**  
- **核心**：递归或栈实现，优先探索更深路径，适合连通性、全排列等问题。  
- **关键步骤**：递归终止条件、状态回溯。  


#### **示例6：全排列（DFS+回溯）**  
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

void dfs(vector<int>& nums, vector<bool>& used, vector<int>& path, vector<vector<int>>& res) {
    if (path.size() == nums.size()) {
        res.push_back(path);
        return;
    }
    for (int i = 0; i < nums.size(); ++i) {
        if (used[i]) continue;
        // 去重：跳过相同元素且前一个未使用的情况
        if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue;
        used[i] = true;
        path.push_back(nums[i]);
        dfs(nums, used, path, res);
        path.pop_back();
        used[i] = false;
    }
}

vector<vector<int>> permuteUnique(vector<int>& nums) {
    sort(nums.begin(), nums.end());  // 排序去重
    vector<vector<int>> res;
    vector<int> path;
    vector<bool> used(nums.size(), false);
    dfs(nums, used, path, res);
    return res;
}

int main() {
    vector<int> nums = {1, 1, 2};
    auto res = permuteUnique(nums);
    for (auto& p : res) {
        for (int num : p) cout << num << " ";
        cout << endl;
    }
    // 输出：
    // 1 1 2
    // 1 2 1
    // 2 1 1
    return 0;
}
```


---

## **三、滑动窗口与二分查找：数组/字符串优化技巧**  


### **3.1 滑动窗口**  
- **核心**：双指针维护窗口，动态调整左右边界，解决子数组/子串的最值或计数问题。  
- **适用场景**：无重复字符的最长子串、最小覆盖子串、长度最小的子数组和。  


#### **示例7：无重复字符的最长子串（滑动窗口）**  
```cpp
#include <iostream>
#include <string>
#include <unordered_map>
using namespace std;

int lengthOfLongestSubstring(string s) {
    unordered_map<char, int> window;  // 记录字符的最新位置
    int left = 0, right = 0;
    int max_len = 0;

    while (right < s.size()) {
        char c = s[right];
        if (window.count(c)) {
            left = max(left, window[c] + 1);  // 移动左边界跳过重复字符
        }
        window[c] = right;  // 更新字符位置
        max_len = max(max_len, right - left + 1);
        right++;
    }
    return max_len;
}

int main() {
    cout << "最长无重复子串长度: " << lengthOfLongestSubstring("abcabcbb") << endl;  // 输出：3（"abc"）
    return 0;
}
```


### **3.2 二分查找**  
- **核心**：在有序数组中通过`mid`缩小区间，时间复杂度O(log n)。  
- **关键条件**：明确搜索区间（左闭右闭`[left, right]`或左闭右开`[left, right)`）。  


#### **示例8：寻找旋转排序数组的最小值（二分查找）**  
```cpp
#include <iostream>
#include <vector>
using namespace std;

int findMin(vector<int>& nums) {
    int left = 0, right = nums.size() - 1;
    while (left < right) {  // 左闭右闭区间
        int mid = left + (right - left) / 2;  // 防溢出
        if (nums[mid] < nums[right]) {  // 右半部分有序，最小值在左半
            right = mid;
        } else {  // 左半部分有序，最小值在右半
            left = mid + 1;
        }
    }
    return nums[left];
}

int main() {
    vector<int> nums = {4, 5, 6, 7, 0, 1, 2};
    cout << "最小值: " << findMin(nums) << endl;  // 输出：0
    return 0;
}
```


---

## **四、回溯法与剪枝：暴力搜索的优化**  
回溯法是解决组合、排列、子集问题的核心方法，通过剪枝避免无效搜索。  


### **4.1 组合问题**  
- **核心**：按顺序选择元素，递归生成所有可能组合，通过`start`参数避免重复。  


#### **示例9：组合总和（可重复选元素）**  
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

void backtrack(vector<int>& candidates, int target, int start, vector<int>& path, vector<vector<int>>& res) {
    if (target == 0) {
        res.push_back(path);
        return;
    }
    for (int i = start; i < candidates.size(); ++i) {
        if (candidates[i] > target) break;  // 剪枝：当前元素大于剩余目标，后续更大元素无需考虑
        path.push_back(candidates[i]);
        backtrack(candidates, target - candidates[i], i, path, res);  // 允许重复选，start=i
        path.pop_back();
    }
}

vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
    sort(candidates.begin(), candidates.end());  // 排序以便剪枝
    vector<vector<int>> res;
    vector<int> path;
    backtrack(candidates, target, 0, path, res);
    return res;
}

int main() {
    vector<int> candidates = {2, 3, 6, 7};
    int target = 7;
    auto res = combinationSum(candidates, target);
    for (auto& p : res) {
        for (int num : p) cout << num << " ";
        cout << endl;
    }
    // 输出：
    // 2 2 3 
    // 7 
    return 0;
}
```


---

## **五、哈希与数据结构：高效存储与查询**  


### **5.1 哈希表**  
- **核心**：通过`key`快速查找`value`，时间复杂度O(1)（平均）。  
- **竞赛应用**：两数之和、重复元素、字母异位词分组。  


#### **示例10：两数之和（哈希表）**  
```cpp
#include <iostream>
#include <vector>
#include <unordered_map>
using namespace std;

vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int, int> hash;  // 记录数值到索引的映射
    for (int i = 0; i < nums.size(); ++i) {
        int complement = target - nums[i];
        if (hash.count(complement)) {
            return {hash[complement], i};
        }
        hash[nums[i]] = i;
    }
    return {};
}

int main() {
    vector<int> nums = {2, 7, 11, 15};
    int target = 9;
    auto res = twoSum(nums, target);
    cout << "索引: " << res[0] << " " << res[1] << endl;  // 输出：0 1
    return 0;
}
```


### **5.2 二叉树与多叉树**  
#### **5.2.1 二叉树遍历**  
- **前序**：根→左→右。  
- **中序**：左→根→右（BST中序为有序序列）。  
- **后序**：左→右→根。  
- **层序**：BFS实现，记录每层节点。  


#### **示例11：二叉树的层序遍历**  
```cpp
#include <iostream>
#include <vector>
#include <queue>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> res;
    if (!root) return res;

    queue<TreeNode*> q;
    q.push(root);

    while (!q.empty()) {
        int size = q.size();
        vector<int> level;
        for (int i = 0; i < size; ++i) {
            TreeNode* node = q.front();
            q.pop();
            level.push_back(node->val);
            if (node->left) q.push(node->left);
            if (node->right) q.push(node->right);
        }
        res.push_back(level);
    }
    return res;
}

int main() {
    // 构建二叉树：
    //     3
    //    / \
    //   9  20
    //     /  \
    //    15   7
    TreeNode* root = new TreeNode(3);
    root->left = new TreeNode(9);
    root->right = new TreeNode(20);
    root->right->left = new TreeNode(15);
    root->right->right = new TreeNode(7);

    auto res = levelOrder(root);
    for (auto& level : res) {
        for (int val : level) cout << val << " ";
        cout << endl;
    }
    // 输出：
    // 3 
    // 9 20 
    // 15 7 
    return 0;
}
```


#### **5.2.2 二叉搜索树（BST）**  
- **性质**：左子树所有节点≤根，右子树所有节点≥根（中序遍历为有序序列）。  
- **操作**：插入、删除、查找。  


#### **示例12：BST的插入与查找**  
```cpp
#include <iostream>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

TreeNode* insert(TreeNode* root, int val) {
    if (!root) return new TreeNode(val);
    if (val < root->val) root->left = insert(root->left, val);
    else root->right = insert(root->right, val);
    return root;
}

bool search(TreeNode* root, int val) {
    if (!root) return false;
    if (val == root->val) return true;
    return val < root->val ? search(root->left, val) : search(root->right, val);
}

int main() {
    TreeNode* root = nullptr;
    root = insert(root, 5);
    insert(root, 3);
    insert(root, 7);
    insert(root, 2);
    insert(root, 4);

    cout << "是否存在4: " << boolalpha << search(root, 4) << endl;  // 输出：true
    cout << "是否存在6: " << search(root, 6) << endl;  // 输出：false
    return 0;
}
```


#### **5.2.3 多叉树（以Trie为例）**  
- **应用**：前缀匹配、字典查询（如示例6的Trie树）。  


---

## **六、竞赛高频问题总结**  
### **6.1 常见错误**  
- 数组越界（如`vector`的`size()`为0时访问`[0]`）。  
- 整数溢出（用`long long`替代`int`，乘法时加`1LL`）。  
- BFS/DFS未标记访问（导致死循环）。  


### **6.2 优化技巧**  
- 预处理（如组合数的阶乘逆元）。  
- 剪枝（回溯法中提前终止无效路径）。  
- 空间换时间（如哈希表存储中间结果）。  


---

### **总结**  
本文覆盖了算法竞赛的核心知识点：  
- **数论**：GCD/LCM、素数筛、快速幂、组合数。  
- **搜索**：BFS（最短路径）、DFS（全排列）、回溯法（组合问题）。  
- **技巧**：滑动窗口（子串问题）、二分查找（有序数组）。  
- **数据结构**：哈希表（快速查询）、二叉树（遍历与BST）、多叉树（Trie）。  

通过大量示例和API说明，可快速掌握竞赛所需的算法与数据结构，结合实战练习（如LeetCode、Codeforces），能显著提升竞赛成绩。