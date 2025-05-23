
# Make、CMake、QMake 使用指南（新手友好版）

## 一、Make：最基础的构建工具
### 1. 核心概念
- **Makefile**：定义编译规则（目标、依赖、命令），如 `app: main.c` 表示生成 `app` 依赖 `main.c`，命令为 `gcc main.c -o app`。
- **Make**：执行 Makefile，自动检测文件变化，仅重新编译修改部分（提高效率）。

### 2. 简单示例（单文件 C）
```makefile
app: main.c
    gcc main.c -o app  # 命令前必须 tab 缩进

clean:
    rm -f app
```
- **使用**：`make` 编译，`make clean` 清理。

### 3. 多文件编译（`main.c` + `func.c`）
```makefile
app: main.o func.o
    gcc $^ -o $@  # $^ 表示所有依赖，$@ 表示目标

%.o: %.c  # 通配符规则，生成所有 .o 文件
    gcc -c $< -o $@  # $< 表示第一个依赖

clean:
    rm -f app *.o
```
- **优势**：通配符简化规则，适合多文件项目。

### 4. 与 Windows 对比
- **Windows**：用 `nmake`（VS 自带，语法类似但需适配 Windows）。
- **Linux**：`make`（GNU Make，开源，语法通用）。

### 5. 常见问题
- **缩进错误**：命令行必须 tab 缩进，否则报错。
- **依赖缺失**：修改头文件后，需手动 `touch` 源文件或添加头文件依赖到 Makefile。


## 二、CMake：跨平台构建神器
### 1. 核心概念
- **CMakeLists.txt**：用 CMake 语法描述项目（比 Makefile 更简洁，跨平台）。
- **CMake**：生成对应平台的构建文件（如 Linux Makefile、Windows VS 项目）。

### 2. 简单示例（单文件 C++）
```cmake
cmake_minimum_required(VERSION 3.10)
project(MyApp)
add_executable(MyApp main.cpp)  # 生成可执行文件
```
- **使用步骤**：
  ```bash
  mkdir build && cd build
  cmake ..  # 生成构建文件（如 Makefile）
  make      # 编译
  ```

### 3. 多文件与库（带头文件）
```cmake
project(MyLibApp)
include_directories(include)  # 头文件目录
add_library(myfunc STATIC src/func.cpp)  # 静态库
add_executable(MyApp main.cpp)
target_link_libraries(MyApp myfunc)  # 链接库
```
- **跨平台**：Windows 下用 `cmake -G "Visual Studio 16 2019" ..` 生成 VS 项目。

### 4. 常用命令
- `add_executable(name srcs)`：生成可执行文件。
- `add_library(name [STATIC/SHARED] srcs)`：生成库。
- `target_include_directories(name PUBLIC dir)`：添加头文件路径。
- `target_link_libraries(name libs)`：链接库。

### 5. 与 Make 对比
- **Make**：手动写 Makefile，适合小项目。
- **CMake**：自动生成 Makefile（或其他构建文件），跨平台，适合大项目。


## 三、QMake：Qt 专属工具
### 1. 核心概念
- **.pro 文件**：描述 Qt 项目，自动处理 `moc`（元对象编译）、`uic`（界面编译）、`rcc`（资源编译）。
- **QMake**：生成 Makefile（或 Qt Creator 项目），专为 Qt 设计。

### 2. 简单 Qt 项目（控制台）
```pro
TARGET = MyQtApp
SOURCES += main.cpp
QT += core  # 包含 Qt Core 模块
CONFIG += console  # 控制台程序
```
- **使用**：`qmake MyQtApp.pro && make`。

### 3. Qt GUI 项目（带 .ui 界面）
```pro
TARGET = MyQtGuiApp
SOURCES += main.cpp mainwindow.cpp
HEADERS += mainwindow.h
FORMS += mainwindow.ui  # 界面文件
QT += widgets  # GUI 模块
```
- **自动处理**：编译时 `uic` 生成界面代码，`moc` 处理 `Q_OBJECT` 宏。

### 4. 与 CMake 对比（Qt 场景）
- **QMake**：`.pro` 文件简洁，自动处理 Qt 工具，适合纯 Qt 项目。
- **CMake**：支持 Qt 项目（需 `find_package(Qt5)`），适合混合非 Qt 代码或复杂构建。

### 5. Qt Creator 集成
- 直接打开 `.pro` 文件，Qt Creator 自动调用 QMake 构建，无需手动命令。


## 四、总结与选择
| 工具   | 适用场景                     | 学习成本 | 跨平台 |
|--------|------------------------------|----------|--------|
| Make   | 小项目、熟悉构建规则         | 低       | 一般（Linux 优先） |
| CMake  | 跨平台项目、大项目           | 中       | 高     |
| QMake  | Qt 项目（纯 Qt 或 GUI）      | 低（Qt 开发者） | 高     |

### 实战路线
1. **Make**：写简单 Makefile，理解目标、依赖、命令（如编译多文件 C 程序）。
2. **CMake**：用 `CMakeLists.txt` 生成跨平台构建（如 Linux Makefile + Windows VS 项目）。
3. **QMake**：在 Qt 项目中使用（配合 Qt Creator，自动生成 `.pro` 文件）。

### 常见命令速查
| 工具   | 核心文件       | 生成命令               | 编译命令  |
|--------|----------------|------------------------|-----------|
| Make   | Makefile       | 手动编写               | `make`    |
| CMake  | CMakeLists.txt | `cmake ..`             | `make`    |
| QMake  | .pro           | `qmake project.pro`    | `make`    |

通过实践（如用 Make 编译 C 程序，用 CMake 构建跨平台库，用 QMake 开发 Qt 界面），快速掌握三者的使用场景和核心功能。遇到问题时，善用 `man make`、`cmake --help` 或 Qt 文档，逐步提升构建工具的使用能力。