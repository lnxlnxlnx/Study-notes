
以下是完整还原 **7.1-7.9全章节** 的Markdown版本，包含所有控件、对比、示例及扩展内容，确保无遗漏：


# Qt控件开发实战指南：从基础到进阶（全7.1-7.9章节）

## 7.1 按钮家族（QPushButton/QCheckBox/QRadioButton）
### 📌 知识点：Qt vs 纯C++开发对比
- **纯C++（如Win32）**  
  ✅ 需调用`CreateWindow("BUTTON", ...)`手动创建按钮  
  ✅ 通过`WM_COMMAND`消息循环捕获点击事件（`LOWORD(wParam) == BN_CLICKED`）  
  ✅ 手动计算控件坐标（`SetWindowPos(x, y, width, height, ...)`）  
- **Qt**  
  ✅ 一行创建：`QPushButton* btn = new QPushButton("点击我", this);`  
  ✅ 信号槽解耦：`connect(btn, &QPushButton::clicked, [=](){...});`  
  ✅ 内置状态管理：`setCheckable(true)`实现切换按钮  

### 🛠️ API速查表
| 类          | 核心方法                          | 关键信号                     |
|-------------|-----------------------------------|------------------------------|
| QPushButton | `setText()` `setIcon(QIcon)`      | `clicked(bool checked)`       |
| QCheckBox   | `isChecked()` `setTristate(true)` | `stateChanged(int state)`     |
| QRadioButton| `setChecked(true)`                | `toggled(bool checked)`       |

### 📝 实战代码：按钮组实战
```cpp
#include <QApplication>
#include <QWidget>
#include <QVBoxLayout>
#include <QPushButton>
#include <QCheckBox>
#include <QRadioButton>
#include <QButtonGroup> // 单选框分组必备

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);
    QWidget window;
    window.setWindowTitle("按钮组演示");

    // 1. 普通按钮（带点击计数）
    QPushButton* btnCounter = new QPushButton("点击次数：0");
    int count = 0;
    connect(btnCounter, &QPushButton::clicked, [&](){
        count++;
        btnCounter->setText(QString("点击次数：%1").arg(count));
    });

    // 2. 复选框（三态支持）
    QCheckBox* checkBox = new QCheckBox("允许通知");
    checkBox->setTristate(true); // 支持未选/半选/全选
    checkBox->setCheckState(Qt::PartiallyChecked); // 初始半选

    // 3. 单选框（分组管理）
    QRadioButton* radioRed = new QRadioButton("红色");
    QRadioButton* radioBlue = new QRadioButton("蓝色");
    QButtonGroup* colorGroup = new QButtonGroup(&window);
    colorGroup->addButton(radioRed, 1); // 关联ID
    colorGroup->addButton(radioBlue, 2);
    radioRed->setChecked(true); // 默认选中

    // 布局
    QVBoxLayout* layout = new QVBoxLayout(&window);
    layout->addWidget(btnCounter);
    layout->addWidget(checkBox);
    layout->addWidget(radioRed);
    layout->addWidget(radioBlue);
    window.show();
    return app.exec();
}
```

### ❗ 易混淆点
1. **单选框互斥范围**  
   ❌ 不同父容器中的单选框不会自动互斥  
   ✅ 必须通过`QButtonGroup`或同一父容器管理  

2. **信号槽参数匹配**  
   ❌ `QCheckBox`的`stateChanged`信号参数是`int`（0/1/2），而非`bool`  
   ✅ 用`state == Qt::Checked`判断全选状态  


## 7.2 输入控件（QLineEdit/QTextEdit/QSpinBox）
### 📌 核心差异：数据处理对比
| 功能                | 纯C++（Win32）                     | Qt                          |
|---------------------|------------------------------------|-----------------------------|
| 单行输入            | 处理`WM_CHAR`拼接`std::string`     | `text()`直接返回`QString`   |
| 输入验证            | 手动编写正则表达式校验             | `QIntValidator`等内置验证器 |
| 密码显示            | 手动替换显示字符（如`*`）          | `setEchoMode(QLineEdit::Password)` |
| 多行编辑            | 自绘`HWND`+滚动条实现             | `QTextEdit`原生支持          |

### 🛠️ 关键API
#### QLineEdit
- `setInputMask("+99 9999-9999")`：手机号格式掩码  
- `setValidator(new QRegExpValidator("^[a-z]+$", this))`：限制小写字母  
- 信号：`textChanged(const QString&)`实时监听输入  

#### QTextEdit
- `setHtml("<b>加粗</b><br>换行")`：富文本编辑  
- `toPlainText()`：获取纯文本内容  
- `textChanged()`信号：内容变更触发  

#### QSpinBox
- `setRange(1, 100)`：数值范围  
- `setValue(50)`：设置初始值  
- `valueChanged(int)`信号：数值变化触发  

### 📝 实战：用户注册表单
```cpp
#include <QApplication>
#include <QWidget>
#include <QFormLayout>
#include <QLineEdit>
#include <QSpinBox>
#include <QTextEdit>
#include <QPushButton>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);
    QWidget window;
    window.setWindowTitle("注册表单");

    // 用户名（6-16位字母/数字）
    QLineEdit* usernameEdit = new QLineEdit;
    usernameEdit->setValidator(new QRegExpValidator(QRegExp("^[a-zA-Z0-9_]{6,16}$"), this));

    // 年龄（1-150）
    QSpinBox* ageSpin = new QSpinBox;
    ageSpin->setRange(1, 150);
    ageSpin->setValue(18);

    // 简介（多行文本）
    QTextEdit* introEdit = new QTextEdit;
    introEdit->setPlaceholderText("请输入个人简介...");

    // 提交按钮
    QPushButton* submitBtn = new QPushButton("提交");
    connect(submitBtn, &QPushButton::clicked, [=](){
        qDebug() << "用户名：" << usernameEdit->text();
        qDebug() << "年龄：" << ageSpin->value();
        qDebug() << "简介：" << introEdit->toPlainText();
    });

    // 表单布局
    QFormLayout* layout = new QFormLayout(&window);
    layout->addRow("用户名：", usernameEdit);
    layout->addRow("年龄：", ageSpin);
    layout->addRow("个人简介：", introEdit);
    layout->addRow(submitBtn);
    window.show();
    return app.exec();
}
```

### ❗ 易混淆点
- **`QLineEdit` vs `QTextEdit`**  
  ✅ `QLineEdit`：单行，轻量（适合输入框）  
  ✅ `QTextEdit`：多行，支持富文本（适合文本域）  

- **验证器触发时机**  
  ❌ 纯C++：提交时统一校验  
  ✅ Qt：输入时实时拦截非法字符（如输入非数字时自动过滤）  


## 7.3 显示控件（QLabel/QTextBrowser）
### 📌 核心优势：渲染能力对比
| 功能                | 纯C++（GDI）                | Qt                          |
|---------------------|-----------------------------|-----------------------------|
| 文本渲染            | `TextOut`逐字符绘制         | 自动解析HTML标签（如`<b>`）  |
| 图片显示            | `StretchBlt`手动缩放        | `setPixmap()`自动适配尺寸    |
| GIF动画             | 手动更新`HDC`帧缓存         | `QMovie`直接播放GIF          |
| 超链接支持          | 自绘下划线+鼠标消息监听     | `linkActivated`信号自动触发  |

### 🛠️ 关键API
#### QLabel
- `setPixmap(QPixmap(":/icon.png"))`：显示图片  
- `setAlignment(Qt::AlignCenter)`：居中对齐  
- `setWordWrap(true)`：自动换行  
- `setOpenExternalLinks(true)`：点击链接打开浏览器  

#### QTextBrowser
- `setSource(QUrl("https://qt.io"))`：加载网页  
- `setHtml("<h1>标题</h1><p>内容</p>")`：渲染HTML  
- `backward()`/`forward()`：浏览历史导航  

### 📝 实战：图文混排界面
```cpp
#include <QApplication>
#include <QWidget>
#include <QLabel>
#include <QMovie>
#include <QTextBrowser>
#include <QVBoxLayout>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);
    QWidget window;
    window.setWindowTitle("图文演示");

    // 1. GIF动画标签
    QLabel* gifLabel = new QLabel;
    QMovie* movie = new QMovie(":/loading.gif");
    gifLabel->setMovie(movie);
    movie->start(); // 自动循环播放

    // 2. 富文本标签（带超链接）
    QLabel* textLabel = new QLabel;
    textLabel->setTextFormat(Qt::RichText);
    textLabel->setText(
        "<h2>欢迎使用Qt</h2>"
        "<p>官网：<a href='https://qt.io'>点击访问Qt官网</a></p>"
        "<img src=':/qt-logo.png' width='128' height='128'>"
    );
    textLabel->setOpenExternalLinks(true); // 允许打开外部链接

    // 3. 文本浏览器（显示本地HTML）
    QTextBrowser* browser = new QTextBrowser;
    browser->setSource(QUrl::fromLocalFile("help.html"));

    // 布局
    QVBoxLayout* layout = new QVBoxLayout(&window);
    layout->addWidget(gifLabel);
    layout->addWidget(textLabel);
    layout->addWidget(browser);
    window.show();
    return app.exec();
}
```

### ❗ 易混淆点
- **文本格式设置**  
  ❌ 忘记调用`setTextFormat(Qt::RichText)`会导致HTML标签被转义为纯文本  
  ✅ 推荐直接使用`textLabel->setRichText(text);`  

- **`QTextBrowser`加载网络资源**  
  ❌ Qt6需确保项目包含`QT += webenginewidgets`模块  
  ✅ 加载本地文件用`file:///path/to/file.html`协议  


## 7.4 浏览器控件（QWebEngineView）
### 📌 降维打击：集成成本对比
| 功能                | 纯C++（CEF）                | Qt                          |
|---------------------|-----------------------------|-----------------------------|
| 内核集成            | 编译复杂二进制包（>1GB）    | 一行代码`#include <QWebEngineView>` |
| 网页加载            | 手动初始化`CefInitialize`   | `load(QUrl("https://qt.io"))` |
| JS交互              | 实现`CefV8Handler`接口       | `page()->runJavaScript()`直接调用 |
| 内存管理            | 手动管理`CefBrowser`生命周期 | Qt自动管理（父对象机制）     |

### 🛠️ 关键API
- `load(QUrl("https://qt.io"))`：加载URL  
- `setHtml(QString htmlCode)`：直接渲染HTML代码  
- `page()->runJavaScript("alert('Hello Qt!')");`：执行JS代码  
- 信号：`loadFinished(bool ok)`加载完成回调  

### 📝 实战：简易浏览器
```cpp
#include <QApplication>
#include <QWebEngineView>
#include <QLineEdit>
#include <QToolBar>
#include <QPushButton>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);
    QWidget window;
    window.setWindowTitle("Qt浏览器");

    // 地址栏
    QLineEdit* urlEdit = new QLineEdit("https://qt.io");
    QPushButton* goBtn = new QPushButton("前往");

    // 浏览器内核
    QWebEngineView* webView = new QWebEngineView;
    connect(goBtn, &QPushButton::clicked, [=]() {
        QUrl url = QUrl::fromUserInput(urlEdit->text());
        if (url.isValid()) webView->load(url);
    });
    connect(webView, &QWebEngineView::urlChanged, [=](const QUrl &url) {
        urlEdit->setText(url.toString()); // 同步地址栏
    });

    // 工具栏
    QToolBar* toolbar = new QToolBar;
    toolbar->addWidget(urlEdit);
    toolbar->addWidget(goBtn);

    // 主布局
    QVBoxLayout* layout = new QVBoxLayout(&window);
    layout->addWidget(toolbar);
    layout->addWidget(webView);
    window.show();
    return app.exec();
}
```

### ❗ 易混淆点
- **模块依赖（Qt6）**  
  ❌ 必须在`.pro`文件中添加`QT += webenginewidgets`，否则编译报错  
  ✅ Qt5对应模块为`QT += webkitwidgets`（注意版本差异）  

- **JS与C++交互**  
  ✅ 使用`QWebChannel`实现双向通信，避免直接操作跨进程对象  
  ❌ 不要在JS回调中直接修改Qt UI控件（需通过信号槽跨线程）  


## 7.5 布局管理（QVBox/QHBox/QGrid/QForm）
### 📌 灵魂对比：界面适配
| 场景                | 纯C++（手动布局）           | Qt                          |
|---------------------|-----------------------------|-----------------------------|
| 窗口缩放            | 重写`WM_SIZE`计算坐标       | 布局自动拉伸控件            |
| 响应式设计          | 手动适配多分辨率            | `setSizePolicy(Expanding)`自动适应 |
| 复杂嵌套布局        | 多层`SetWindowPos`嵌套调用   | 嵌套布局+拉伸因子轻松实现   |
| 表单对齐            | 手动计算标签与控件间距       | `QFormLayout`自动对齐        |

### 🛠️ 核心API
| 布局类        | 关键方法                          | 典型用法                     |
|---------------|-----------------------------------|------------------------------|
| QVBoxLayout   | `addWidget(widget, stretch)`      | 垂直排列，`stretch`控制拉伸比例 |
| QHBoxLayout   | `addStretch(int factor)`          | 添加可拉伸空白（因子越大占比越高） |
| QGridLayout   | `addWidget(widget, row, col, rowspan, colspan)` | 网格布局（跨行列） |
| QFormLayout   | `addRow("标签", widget)`          | 快速生成标签-控件对           |

### 📝 实战：复杂网格布局
```cpp
#include <QApplication>
#include <QWidget>
#include <QGridLayout>
#include <QLabel>
#include <QLineEdit>
#include <QPushButton>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);
    QWidget window;
    window.setWindowTitle("网格布局");

    // 标签与输入框
    QString labels[] = {"姓名：", "年龄：", "邮箱：", "备注："};
    QLineEdit* edits[4];
    QGridLayout* grid = new QGridLayout(&window);

    for (int i = 0; i < 4; ++i) {
        // 添加标签（第i行，第0列）
        grid->addWidget(new QLabel(labels[i]), i, 0);
        // 添加输入框（第i行，第1列）
        edits[i] = new QLineEdit;
        grid->addWidget(edits[i], i, 1);
        // 备注框跨2行（第3行开始，跨2行）
        if (i == 3) grid->setRowSpan(i, 1, 2);
    }

    // 提交按钮（第5行，跨2列，居中）
    QPushButton* submitBtn = new QPushButton("提交");
    grid->addWidget(submitBtn, 5, 0, 1, 2); // 行5，列0，占1行2列
    grid->setAlignment(submitBtn, Qt::AlignCenter); // 水平居中

    window.show();
    return app.exec();
}
```

### ❗ 易混淆点
- **拉伸因子（Stretch）作用范围**  
  ✅ 同层级控件才会按`stretch`比例分配空间  
  ❌ 不同布局的拉伸因子无法跨层级比较（如父布局的拉伸不影响子布局内部）  

- **绝对定位冲突**  
  ❌ 对已加入布局的控件调用`setGeometry()`会导致布局失效  
  ✅ 布局管理器会自动覆盖手动坐标设置，需通过布局参数控制位置  


## 7.6 空间间隔（QSpacerItem/QSizePolicy）
### 📌 核心概念：空白管理
- **纯C++**：通过`SetWindowPos`设置控件间距，窗口缩放时需重新计算  
- **Qt**：  
  ✅ `QSpacerItem`：固定尺寸空白（如20px间隔）  
  ✅ `addStretch()`：可拉伸空白（随窗口缩放自动扩展）  
  ✅ `QSizePolicy`：控制控件自身的拉伸策略（如`Expanding`允许无限拉伸）  

### 🛠️ 关键API
- `QSpacerItem(int width, int height)`：创建固定大小间隔  
- `addStretch(int factor=1)`：添加可拉伸间隔（factor越大，占比越高）  
- `setSizePolicy(QSizePolicy::Expanding)`：设置控件可拉伸  

### 📝 实战：间隔布局演示
```cpp
#include <QApplication>
#include <QWidget>
#include <QHBoxLayout>
#include <QPushButton>
#include <QSpacerItem>
#include <QSizePolicy>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);
    QWidget window;
    window.setWindowTitle("间隔演示");

    QHBoxLayout* layout = new QHBoxLayout(&window);

    // 左侧按钮（固定大小）
    QPushButton* btnLeft = new QPushButton("左按钮");
    btnLeft->setSizePolicy(QSizePolicy::Fixed, QSizePolicy::Fixed); // 禁止拉伸

    // 中间可拉伸间隔（因子2）
    layout->addWidget(btnLeft);
    layout->addStretch(2); // 占比2倍于右侧拉伸

    // 右侧按钮（可拉伸）
    QPushButton* btnRight = new QPushButton("右按钮");
    btnRight->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Fixed); // 水平可拉伸
    layout->addWidget(btnRight);

    // 底部固定间隔（20px高度）
    layout->setContentsMargins(20, 20, 20, 20); // 边距
    layout->addItem(new QSpacerItem(0, 20, QSizePolicy::Minimum, QSizePolicy::Fixed));

    window.show();
    return app.exec();
}
```

### ❗ 易混淆点
- **`QSpacerItem` vs `addStretch`**  
  ✅ `addStretch()`是快捷方式，等价于`addItem(new QSpacerItem(0, 0, QSizePolicy::Expanding, ...))`  
  ✅ `QSpacerItem`可精确控制宽高和拉伸策略  

- **`QSizePolicy`优先级**  
  ❌ 控件自身的`sizePolicy`会覆盖布局的拉伸设置  
  ✅ 若需强制拉伸，需同时设置布局拉伸因子和控件`sizePolicy`为`Expanding`  


## 7.7 容器控件（QGroupBox/QFrame/QStackedWidget）
### 📌 核心功能：界面分组与切换
| 控件            | 功能描述                          | 纯C++对比                     |
|-----------------|-----------------------------------|------------------------------|
| QGroupBox       | 带标题的分组框（可勾选禁用子控件）| 需手动绘制边框和标题         |
| QFrame          | 带边框的容器（支持阴影/浮雕效果） | 自绘`WM_PAINT`实现边框样式   |
| QStackedWidget  | 多页面栈式切换（无标签栏）        | 手动管理`HWND`显示/隐藏      |

### 🛠️ 关键API
- `QGroupBox::setCheckable(true)`：勾选时启用/禁用子控件  
- `QFrame::setFrameStyle(QFrame::Box | QFrame::Sunken)`：设置凹陷边框  
- `QStackedWidget::setCurrentIndex(int)`：切换页面  
- `QStackedWidget::addWidget(widget)`：添加页面  

### 📝 实战：设置面板（分组框+栈式切换）
```cpp
#include <QApplication>
#include <QWidget>
#include <QGroupBox>
#include <QRadioButton>
#include <QStackedWidget>
#include <QVBoxLayout>
#include <QHBoxLayout>

// 外观设置页面
class AppearancePage : public QWidget {
public:
    AppearancePage(QWidget* parent=nullptr) : QWidget(parent) {
        QVBoxLayout* layout = new QVBoxLayout(this);
        layout->addWidget(new QRadioButton("浅色主题"));
        layout->addWidget(new QRadioButton("深色主题"));
    }
};

// 通知设置页面
class NotificationPage : public QWidget {
public:
    NotificationPage(QWidget* parent=nullptr) : QWidget(parent) {
        QVBoxLayout* layout = new QVBoxLayout(this);
        layout->addWidget(new QCheckBox("邮件通知"));
        layout->addWidget(new QCheckBox("推送通知"));
    }
};

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);
    QWidget window;
    window.setWindowTitle("设置中心");

    // 左侧导航分组框
    QGroupBox* navGroup = new QGroupBox("导航");
    QRadioButton* btnAppearance = new QRadioButton("外观");
    QRadioButton* btnNotification = new QRadioButton("通知");
    QVBoxLayout* navLayout = new QVBoxLayout(navGroup);
    navLayout->addWidget(btnAppearance);
    navLayout->addWidget(btnNotification);
    btnAppearance->setChecked(true); // 默认选中

    // 右侧栈式窗口
    QStackedWidget* stack = new QStackedWidget;
    stack->addWidget(new AppearancePage);
    stack->addWidget(new NotificationPage);

    // 导航与页面联动
    connect(btnAppearance, &QRadioButton::clicked, [=](){ stack->setCurrentIndex(0); });
    connect(btnNotification, &QRadioButton::clicked, [=](){ stack->setCurrentIndex(1); });

    // 主布局（水平分割）
    QHBoxLayout* mainLayout = new QHBoxLayout(&window);
    mainLayout->addWidget(navGroup, 1); // 导航占1份宽度
    mainLayout->addWidget(stack, 3);    // 内容占3份宽度
    window.show();
    return app.exec();
}
```

### ❗ 易混淆点
- **`QGroupBox`的勾选逻辑**  
  ✅ 勾选状态会通过`setEnabled(checked)`影响所有子控件  
  ❌ 子控件需显式调用`setParent(groupBox)`才能被分组管理  

- **`QStackedWidget`切换性能**  
  ✅ 所有页面同时存在（仅显示/隐藏），适合少量页面  
  ❌ 大量页面建议用`QScrollArea`+动态创建销毁控件  


## 7.8 模型视图（QListView/QTreeView + QStandardItemModel）
### 📌 核心思想：数据与视图解耦
| 架构          | 纯C++实现                     | Qt实现                        |
|---------------|------------------------------|-------------------------------|
| 数据存储      | 手动维护`std::vector`+索引    | `QStandardItemModel`统一管理   |
| 界面渲染      | 自绘`WM_PAINT`+`DrawItem`     | `QListView`自动渲染模型数据    |
| 数据更新      | 手动调用`InvalidateRect`重绘  | 模型变更自动通知视图更新      |

### 🛠️ 关键API
- `QStandardItemModel::appendRow(new QStandardItem("条目"))`：添加数据  
- `QListView::setModel(model)`：关联模型  
- `QModelIndex index = model->index(row, column);`：获取模型索引  
- 信号：`QListView::clicked(const QModelIndex&)`点击条目触发  

### 📝 实战：可编辑列表（模型-视图）
```cpp
#include <QApplication>
#include <QListView>
#include <QStandardItemModel>
#include <QVBoxLayout>
#include <QPushButton>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);
    QWidget window;
    window.setWindowTitle("模型视图演示");

    // 1. 创建数据模型
    QStandardItemModel* model = new QStandardItemModel;
    model->setHorizontalHeaderItem(0, new QStandardItem("水果列表")); // 表头

    // 2. 添加初始数据
    QStringList fruits = {"苹果", "香蕉", "橙子"};
    for (const QString& fruit : fruits) {
        QStandardItem* item = new QStandardItem(fruit);
        item->setEditable(true); // 允许编辑
        model->appendRow(item);
    }

    // 3. 列表视图
    QListView* listView = new QListView;
    listView->setModel(model);
    listView->setEditTriggers(QAbstractItemView::DoubleClicked); // 双击编辑

    // 4. 添加按钮（动态添加条目）
    QPushButton* addBtn = new QPushButton("添加水果");
    int count = 3;
    connect(addBtn, &QPushButton::clicked, [&](){
        count++;
        QStandardItem* item = new QStandardItem(QString("水果%1").arg(count));
        model->appendRow(item);
    });

    // 布局
    QVBoxLayout* layout = new QVBoxLayout(&window);
    layout->addWidget(listView);
    layout->addWidget(addBtn);
    window.show();
    return app.exec();
}
```

### ❗ 易混淆点
- **模型索引（QModelIndex）**  
  ❌ 索引不能直接保存（跨模型/销毁后失效）  
  ✅ 通过`QPersistentModelIndex`持久化索引  

- **模型视图 vs 项控件**  
  ✅ 模型视图：数据与视图分离，适合复杂数据（如数据库）  
  ✅ 项控件（如`QListWidget`）：数据内嵌在视图中，适合简单场景  


## 7.9 项控件（QListWidget/QTreeWidget）
### 📌 核心对比：快速开发 vs 灵活性
| 特性          | 项控件（如`QListWidget`）       | 模型视图（如`QListView`）      |
|---------------|---------------------------------|--------------------------------|
| 数据存储      | `QListWidgetItem`内嵌在视图中   | 独立模型（`QStandardItemModel`）|
| 开发效率      | 简单场景快速上手（一行添加项）  | 需学习模型-视图架构           |
| 复杂场景      | 多层级数据需手动管理（如`setChild`）| 天然支持树形/表格结构（`QTreeView`）|
| 性能          | 适用于小数据量（<1000项）       | 大数据量优化更好（延迟加载）   |

### 🛠️ 关键API
#### QListWidget
- `addItem(new QListWidgetItem("条目"))`：添加单行项  
- `item(int row)`：获取项  
- `setSelectionMode(QAbstractItemView::MultiSelection)`：允许多选  
- 信号：`itemClicked(QListWidgetItem*)`点击项触发  

#### QTreeWidget
- `addTopLevelItem(new QTreeWidgetItem({"父节点"}))`：添加根节点  
- `child(int row)`：获取子节点  
- `setColumnCount(2)`：设置多列  
- 信号：`itemExpanded(QTreeWidgetItem*)`节点展开触发  

### 📝 实战：带勾选的待办事项（QListWidget）
```cpp
#include <QApplication>
#include <QListWidget>
#include <QVBoxLayout>
#include <QLineEdit>
#include <QPushButton>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);
    QWidget window;
    window.setWindowTitle("待办事项");

    // 列表控件
    QListWidget* todoList = new QListWidget;
    todoList->setSelectionMode(QAbstractItemView::ExtendedSelection);
    todoList->setItemDelegate(new QStyledItemDelegate); // 启用默认委托

    // 输入框与按钮
    QLineEdit* inputField = new QLineEdit;
    QPushButton* addBtn = new QPushButton("添加任务");
    connect(addBtn, &QPushButton::clicked, [=](){
        QString text = inputField->text().trimmed();
        if (!text.isEmpty()) {
            QListWidgetItem* item = new QListWidgetItem(text);
            item->setCheckState(Qt::Unchecked); // 可勾选
            todoList->addItem(item);
            inputField->clear();
        }
    });

    // 处理勾选状态
    connect(todoList, &QListWidget::itemChanged, [=](QListWidgetItem* item) {
        QString status = item->checkState() == Qt::Checked ? "✅ 完成" : "❌ 未完成";
        qDebug() << item->text() << " - " << status;
    });

    // 布局
    QVBoxLayout* layout = new QVBoxLayout(&window);
    layout->addWidget(inputField);
    layout->addWidget(addBtn);
    layout->addWidget(todoList);
    window.show();
    return app.exec();
}
```

### ❗ 易混淆点
- **项控件的局限性**  
  ❌ 多列数据需手动设置`setData(Qt::UserRole, value)`  
  ✅ 复杂数据建议改用模型视图（如`QTableView`+`QSqlTableModel`）  

- **勾选状态触发**  
  ❌ `QListWidgetItem`默认`flags()`不包含`Qt::ItemIsUserCheckable`  
  ✅ 需显式调用`item->setFlags(item->flags() | Qt::ItemIsUserCheckable);`  


## 扩展：Qt核心特性对比表
| 特性                | 纯C++实现难度 | Qt实现方式                 | 典型场景                     |
|---------------------|---------------|----------------------------|------------------------------|
| 内存管理            | ★★★★★         | 父子对象自动销毁（`QObject`）| 避免`delete`遗漏导致内存泄漏 |
| 事件驱动            | ★★★★☆         | 信号槽机制（`connect`）     | 按钮点击、窗口关闭等响应     |
| 资源管理            | ★★★☆☆         | 资源文件（`.qrc`）          | 打包图片/字体到可执行文件中   |
| 国际化              | ★★★★☆         | `tr()`函数+`linguist`工具   | 多语言界面切换               |
| 样式定制            | ★★★★★         | QSS样式表（类似CSS）        | 一键换肤、动态修改界面风格   |


## 学习路线建议
1. **基础阶段（1-2周）**  
   - 掌握按钮、输入框、标签等基础控件用法  
   - 熟练使用`QVBoxLayout`/`QHBoxLayout`实现简单布局  
   - 理解信号槽机制（用Lambda表达式简化代码）  

2. **进阶阶段（3-4周）**  
   - 深入模型视图架构（`QListView`+`QStandardItemModel`）  
   - 学习复杂布局（`QGridLayout`+`QFormLayout`）  
   - 尝试`QWebEngineView`集成网页功能  

3. **实战阶段**  
   - 开发小型项目：如「带搜索的联系人管理系统」（组合使用`QListWidget`+`QLineEdit`+布局）  
   - 对比纯C++与Qt的代码量差异，体会框架优势  

通过以上系统学习，你将逐步掌握Qt在界面开发中的高效模式，用更少的代码实现更强大的功能！