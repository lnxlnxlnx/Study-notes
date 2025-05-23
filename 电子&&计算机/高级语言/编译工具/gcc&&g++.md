
### GCC和G++的使用总结

#### 一、基本概念
- **GCC**（GNU Compiler Collection）：GNU编译器套件，支持C、C++、Fortran等多语言，是Linux开发的核心工具。  
- **G++**：GCC的C++专用编译器，默认处理C++代码（如`.cpp`文件），自动启用C++特性（如名称修饰、异常处理）。

#### 二、编译流程（4步，GCC/G++通用）
1. **预处理（`-E`）**：处理`#include`、`#define`，生成`.i`（C）或`.ii`（C++）文件。  
   示例：`gcc -E main.c -o main.i`  
2. **编译（`-S`）**：将预处理文件转为汇编代码（`.s`）。  
   示例：`g++ -S main.cpp`（生成`main.s`）  
3. **汇编（`-c`）**：将汇编代码转为机器码（目标文件`.o`）。  
   示例：`gcc -c main.s -o main.o`  
4. **链接（无参数，或`-o`指定输出）**：链接目标文件和库，生成可执行程序。  
   示例：`g++ main.o -o app`

#### 三、核心命令参数
| **参数**       | **功能**                                                                 | **示例**                                      |
|----------------|--------------------------------------------------------------------------|---------------------------------------------|
| `-o <output>`  | 指定输出文件（可用于各阶段，如预处理、编译、链接）。                    | `g++ main.cpp -o app`（直接编译链接C++代码） |
| `-c`           | 仅编译（生成`.o`，不链接），用于多文件编译。                            | `gcc -c func.c`（生成`func.o`）              |
| `-I<dir>`      | 添加头文件搜索目录（优先搜索`dir`中的头文件）。                        | `g++ -Iinclude main.cpp -o app`              |
| `-L<dir>`      | 添加库文件搜索目录（优先搜索`dir`中的库）。                            | `gcc -Llib -lm main.c -o app`（链接数学库）  |
| `-l<lib>`      | 链接指定库（如`-lpthread`链接线程库，`-lstdc++`链接C++标准库）。       | `g++ -lpthread main.cpp -o app`              |
| `-Wall`        | 开启所有警告（帮助发现代码问题）。                                      | `gcc -Wall main.c -o app`                    |
| `-g`           | 生成调试信息（供GDB使用，调试时必备）。                                | `g++ -g main.cpp -o debug_app`               |
| `-O<level>`    | 优化级别（`-O0`无优化，`-O2`常用，`-O3`最高优化）。                   | `gcc -O2 main.c -o fast_app`                 |
| `-std=cXX`/`-std=c++XX` | 指定语言标准（如`c11`、`c++17`）。                              | `g++ -std=c++20 main.cpp -o app`             |

#### 四、GCC与G++的区别
1. **默认语言**：  
   - GCC默认编译C代码（`.c`），启用C特性。  
   - G++默认编译C++代码（`.cpp`），启用C++特性（如`cout`无需额外链接）。  
2. **库链接**：  
   - G++自动链接C++标准库（`libstdc++`），GCC编译C++代码需手动添加`-lstdc++`。  
   示例：  
   ```bash
   # GCC编译C++（需手动链接）
   gcc main.cpp -o app -lstdc++  
   # G++编译（自动链接）
   g++ main.cpp -o app  
   ```
3. **特性支持**：  
   G++对C++新特性（如Lambda、模板、C++20）支持更完善，是C++开发的首选。

#### 五、实战示例
##### 1. 单文件C编译（GCC）
```c
// main.c
#include <stdio.h>
int main() {
    printf("Hello, GCC!\n");
    return 0;
}
```
编译：`gcc -o hello main.c`，运行：`./hello`（输出`Hello, GCC!`）。

##### 2. 单文件C++编译（G++）
```cpp
// main.cpp
#include <iostream>
using namespace std;
int main() {
    cout << "Hello, G++!" << endl;
    return 0;
}
```
编译：`g++ -o hello main.cpp`，运行：`./hello`（输出`Hello, G++!`）。

##### 3. 多文件C++编译（G++）
```cpp
// func.cpp
#include <iostream>
void print() { cout << "Function called." << endl; }
```
```cpp
// main.cpp
void print(); // 声明
int main() { print(); return 0; }
```
编译：`g++ func.cpp main.cpp -o app`，运行：`./app`（输出`Function called.`）。

##### 4. 包含自定义头文件（G++）
```cpp
// include/func.h
#ifndef FUNC_H
#define FUNC_H
void print();
#endif
```
```cpp
// src/func.cpp（实现print）
#include "func.h"
#include <iostream>
void print() { cout << "Header included." << endl; }
```
```cpp
// src/main.cpp
#include "func.h"
int main() { print(); return 0; }
```
编译：`g++ -Isrc/include src/main.cpp src/func.cpp -o app`（`-I`指定头文件目录）。

#### 六、扩展技巧
1. **查看编译细节**：`-v`参数显示预处理、编译、链接的详细命令。  
   示例：`g++ -v main.cpp -o app`。  
2. **生成依赖文件**：`-MM`生成Makefile依赖（如`main.d`），用于自动化构建。  
   示例：`g++ -MM main.cpp > main.d`。  
3. **静态/动态链接**：  
   - 静态链接（`-static`）：`g++ -static main.cpp -o app_static`（体积大，无需依赖库）。  
   - 动态链接（默认）：依赖系统库，体积小。  
4. **交叉编译**：针对其他架构（如ARM），使用交叉编译器（如`arm-linux-gnueabi-g++`）。  
   示例：`arm-linux-gnueabi-g++ main.cpp -o app_arm`。

#### 七、与Windows编译器对比（以MSVC为例）
- **命令风格**：GCC/G++用`-参数`，MSVC用`/参数`（如`/Fo` vs `-o`）。  
- **流程控制**：GCC/G++命令行透明，可分步控制；MSVC多通过IDE（如VS）操作。  
- **标准支持**：GCC/G++对C/C++最新标准（如C++20）支持更及时，且开源免费；MSVC对Windows特性（如WinAPI）支持更好。

#### 八、常见问题解决
1. **“未定义引用”**：检查是否遗漏源文件或库（如`-l`参数）。  
2. **头文件找不到**：确认`-I`路径正确，或头文件拼写错误。  
3. **C++代码用GCC编译报错**：改用G++，或添加`-lstdc++`（不推荐，直接用G++更简便）。

### 总结
GCC和G++是Linux开发的基石，通过掌握命令参数（如`-o`、`-I`、`-g`）和编译流程，可高效编译C/C++代码。对比Windows编译器，其命令行操作更灵活，适合自动化构建（如Makefile、CMake）。在实际开发中，根据语言选择编译器（C用GCC，C++用G++），合理利用优化和调试选项，提升开发效率。通过多文件编译、库链接等实战练习，可快速上手Linux下的编译工作。