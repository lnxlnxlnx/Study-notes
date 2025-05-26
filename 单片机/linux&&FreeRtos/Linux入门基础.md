
# 从0到1：Linux系统编程核心概念保姆级讲解（多线程/进程/网络/文件全覆盖）


你说“啥也不会”？不存在的！我把这些概念拆成**生活化例子+实战代码+避坑指南**，帮你一步步打通任督二脉。咱们从最基础的“进程/线程”开始，到“线程池/生产者消费者”，再到“TCP网络/内存映射”，全程用**人话**讲，保证你能跟上~


---


## 一、进程 vs 线程：先搞清楚“谁是谁”
### 1.1 用“开公司”理解进程和线程
- **进程**：相当于“一间独立的办公室”。每个进程有自己的**内存空间**（就像办公室有独立的文件柜）、**资源**（比如打印机、电脑），进程之间互相隔离（不能随便翻对方的文件柜）。  
  你运行的 `./a.out`、`chrome` 都是进程。

- **线程**：相当于“办公室里的员工”。同一个进程里的线程**共享内存和资源**（共用文件柜、打印机），但有自己的**执行路径**（每个员工干不同的活）。  
  比如微信进程里，有“收消息线程”“发文件线程”“界面刷新线程”。

### 1.2 为什么需要线程？
- **进程的缺点**：创建进程要复制内存/资源，开销大（类似开新办公室要重新买家具）；进程间通信麻烦（得通过“快递”传文件）。  
- **线程的优点**：线程共享进程资源，创建/切换开销小（员工在办公室内换任务，不用搬家具）；线程间通信方便（直接共享内存）。

### 1.3 关键API对比（应用层开发）
| **操作**         | **进程**                          | **线程**                          |
|------------------|-----------------------------------|-----------------------------------|
| 创建             | `fork()`（复制当前进程）          | `pthread_create()`（在进程内创建线程）|
| 结束             | `exit()`（进程退出）              | `pthread_exit()`（线程退出）       |
| 资源回收         | `waitpid()`（等待子进程结束）     | `pthread_join()`（等待线程结束）   |
| 通信             | 管道/消息队列/共享内存（跨进程）  | 共享变量（同进程内线程）          |


---


## 二、线程实战：从创建到同步（附避坑指南）
### 2.1 线程创建：用 `pthread_create` 生个“员工”
#### 知识点说明
线程函数必须是 `void* (*)(void*)` 类型（参数和返回值都是 `void*`）。创建线程时需要传递**线程属性**（一般用默认）、**线程函数**、**参数**。

#### 代码例子（创建2个线程打印信息）
``` c
#include <stdio.h>
#include <pthread.h>  // 线程相关函数

// 线程函数（员工的工作）
void* thread_func(void* arg) {
    int thread_id = *(int*)arg;  // 从参数中获取线程ID（注意类型转换）
    for (int i = 0; i < 3; i++) {
        printf("线程%d: 我在工作第%d次\n", thread_id, i);
        sleep(1);  // 模拟工作耗时
    }
    return NULL;  // 线程结束返回NULL
}

int main() {
    pthread_t tid1, tid2;  // 线程ID（类似员工工号）
    int id1 = 1, id2 = 2;  // 传给线程的参数（线程ID）

    // 创建线程1（参数是&id1）
    if (pthread_create(&tid1, NULL, thread_func, &id1) != 0) {
        perror("创建线程1失败");
        return -1;
    }

    // 创建线程2（参数是&id2）
    if (pthread_create(&tid2, NULL, thread_func, &id2) != 0) {
        perror("创建线程2失败");
        return -1;
    }

    // 等待线程结束（必须！否则主线程退出会杀死所有子线程）
    pthread_join(tid1, NULL);  // 等待线程1结束
    pthread_join(tid2, NULL);  // 等待线程2结束

    printf("主线程：所有线程已完成工作\n");
    return 0;
}
```

#### 编译&运行
```bash
gcc thread_demo.c -o thread_demo -lpthread  # 必须加-lpthread链接线程库
./thread_demo
```

#### 输出结果（可能乱序，因为线程并行执行）
```
线程1: 我在工作第0次
线程2: 我在工作第0次
线程1: 我在工作第1次
线程2: 我在工作第1次
线程1: 我在工作第2次
线程2: 我在工作第2次
主线程：所有线程已完成工作
```

#### 避坑指南
- **线程参数的坑**：传递局部变量地址时，确保线程函数执行期间变量未被销毁（比如别传栈上的临时变量地址）。  
  反例：`int id = 1; pthread_create(..., &id);` 若主线程快速修改 `id`，线程可能读到错误值。  
  正例：用 `malloc` 分配堆内存传参，或在线程函数内立即复制参数值。

- **忘记 `pthread_join`**：主线程退出时会终止所有子线程！必须用 `pthread_join` 等待子线程结束（或用 `pthread_detach` 设置线程自动回收）。


### 2.2 线程同步：互斥锁（解决“抢厕所”问题）
#### 知识点说明
多个线程共享资源（比如共享变量）时，可能出现“同时修改”的混乱（比如两个线程同时给 `count++`，结果只加1次）。互斥锁（`pthread_mutex_t`）能保证**同一时间只有一个线程访问共享资源**（类似“厕所一次只能进一人”）。

#### 代码例子（用互斥锁保护共享计数器）
```c
#include <stdio.h>
#include <pthread.h>

int count = 0;  // 共享计数器（线程都想修改它）
pthread_mutex_t mutex;  // 互斥锁（厕所门的锁）

void* increment(void* arg) {
    for (int i = 0; i < 100000; i++) {
        pthread_mutex_lock(&mutex);  // 锁门（占用厕所）
        count++;  // 关键操作（只有一个线程能执行）
        pthread_mutex_unlock(&mutex);  // 开门（释放厕所）
    }
    return NULL;
}

int main() {
    pthread_t tid1, tid2;

    pthread_mutex_init(&mutex, NULL);  // 初始化锁（装门锁）

    // 创建两个线程，各加10万次
    pthread_create(&tid1, NULL, increment, NULL);
    pthread_create(&tid2, NULL, increment, NULL);

    pthread_join(tid1, NULL);
    pthread_join(tid2, NULL);

    printf("最终count = %d（预期200000）\n", count);  // 输出应该是200000
    pthread_mutex_destroy(&mutex);  // 销毁锁（拆门锁）
    return 0;
}
```

#### 不加锁的后果
如果注释掉 `pthread_mutex_lock` 和 `pthread_mutex_unlock`，两个线程会同时修改 `count`，导致最终结果**远小于200000**（因为 `count++` 实际是 `读-改-写` 三步，可能被打断）。

#### 避坑指南
- **锁的粒度**：锁的范围要尽可能小（只保护关键代码）。比如别把整个循环都锁起来，否则线程无法并行执行。  
  反例：  
  ```c
  pthread_mutex_lock(&mutex);
  for (int i=0; i<100000; i++) { count++; }  // 锁范围太大，另一个线程完全等待
  pthread_mutex_unlock(&mutex);
  ```

- **死锁**：避免“线程A等锁1，线程B等锁2，同时不释放自己的锁”。  
  解决方法：所有线程按**相同顺序**加锁（比如先锁1再锁2，别有的线程先锁2再锁1）。


### 2.3 线程同步：条件变量（解决“等外卖”问题）
#### 知识点说明
互斥锁能“阻止同时访问”，但无法“通知其他线程”。条件变量（`pthread_cond_t`）用来**让线程等待某个条件满足**（比如“外卖到了”），避免无效的“忙等待”（一直循环检查条件）。

#### 生产者-消费者模型（经典例子）
- **生产者**：往“队列”里放数据（比如做外卖）。  
- **消费者**：从“队列”里取数据（比如吃外卖）。  
- **问题**：队列空时消费者要等待，队列满时生产者要等待。

#### 代码例子（条件变量实现生产者-消费者）
```c
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>
#include <unistd.h>

#define QUEUE_MAX 5  // 队列最大容量
//注意这里是循环队列,所以生产者结束生产进行休息(挂起)的条件是queue.size==QUEUE_MAX而不是front或rear

// 队列结构体
typedef struct {
    int data[QUEUE_MAX];
    int front;  // 队头
    int rear;   // 队尾
    int size;   // 当前元素个数
} Queue;

Queue queue;  // 共享队列
pthread_mutex_t mutex;       // 保护队列的互斥锁
pthread_cond_t cond_producer;  // 生产者等待条件（队列满时等）
pthread_cond_t cond_consumer;  // 消费者等待条件（队列空时等）

// 生产者线程函数（做外卖）
void* producer(void* arg) {
    for (int i = 0; i < 10; i++) {  // 生产10次
        pthread_mutex_lock(&mutex);

        // 队列满时等待消费者取数据
        while (queue.size == QUEUE_MAX) {
            printf("生产者：队列满了（%d/%d），等待消费者...\n", queue.size, QUEUE_MAX);
            pthread_cond_wait(&cond_producer, &mutex);  // 释放锁并等待（被唤醒时重新加锁）
        }

        // 生产数据（放到队尾）
        queue.data[queue.rear] = i;
        queue.rear = (queue.rear + 1) % QUEUE_MAX;
        queue.size++;
        printf("生产者：生产%d，队列大小%d\n", i, queue.size);

        pthread_mutex_unlock(&mutex);
        pthread_cond_signal(&cond_consumer);  // 通知消费者：有数据了！
        sleep(1);  // 模拟生产耗时
    }
    return NULL;
}

// 消费者线程函数（吃外卖）
void* consumer(void* arg) {
    for (int i = 0; i < 10; i++) {  // 消费10次
        pthread_mutex_lock(&mutex);

        // 队列空时等待生产者放数据
        while (queue.size == 0) {
            printf("消费者：队列空了（%d/%d），等待生产者...\n", queue.size, QUEUE_MAX);
            pthread_cond_wait(&cond_consumer, &mutex);  // 释放锁并等待
        }

        // 消费数据（从队头取）
        int data = queue.data[queue.front];
        queue.front = (queue.front + 1) % QUEUE_MAX;
        queue.size--;
        printf("消费者：消费%d，队列大小%d\n", data, queue.size);

        pthread_mutex_unlock(&mutex);
        pthread_cond_signal(&cond_producer);  // 通知生产者：可以继续放了！
        sleep(2);  // 模拟消费耗时（比生产慢）
    }
    return NULL;
}

int main() {
    // 初始化队列
    queue.front = 0;
    queue.rear = 0;
    queue.size = 0;

    // 初始化锁和条件变量
    pthread_mutex_init(&mutex, NULL);
    pthread_cond_init(&cond_producer, NULL);
    pthread_cond_init(&cond_consumer, NULL);

    // 创建生产者和消费者线程
    pthread_t prod_tid, cons_tid;
    pthread_create(&prod_tid, NULL, producer, NULL);
    pthread_create(&cons_tid, NULL, consumer, NULL);

    // 等待线程结束
    pthread_join(prod_tid, NULL);
    pthread_join(cons_tid, NULL);

    // 销毁资源
    pthread_cond_destroy(&cond_producer);
    pthread_cond_destroy(&cond_consumer);
    pthread_mutex_destroy(&mutex);
    return 0;
}
```

#### 输出结果（部分）
```
生产者：生产0，队列大小1
生产者：生产1，队列大小2
生产者：生产2，队列大小3
生产者：生产3，队列大小4
生产者：生产4，队列大小5
生产者：队列满了（5/5），等待消费者...
消费者：消费0，队列大小4
消费者：消费1，队列大小3
消费者：消费2，队列大小2
消费者：消费3，队列大小1
消费者：消费4，队列大小0
消费者：队列空了（0/5），等待生产者...
生产者：生产5，队列大小1
...
```

#### 关键逻辑
- **条件变量+互斥锁**：`pthread_cond_wait` 会先释放锁（让其他线程能修改队列），然后阻塞；被唤醒时重新加锁，继续检查条件（防止虚假唤醒）。  
- **用 `while` 检查条件**：不能用 `if`，因为可能被“虚假唤醒”（条件未满足但被唤醒）。


### 2.4 线程池：避免“线程爆炸”（附通用实现）
#### 知识点说明
如果频繁创建/销毁线程（比如Web服务器处理大量短连接），会导致**资源浪费**（线程创建需要时间）和**线程数爆炸**（可能耗尽系统资源）。线程池预先创建一组线程，重复利用它们处理任务。

#### 线程池核心组件
- **任务队列**：存储待处理的任务（比如函数指针+参数）。  
- **工作线程**：从任务队列取任务执行。  
- **管理者线程**（可选）：动态调整线程数量（比如任务多了就加线程，少了就减）。

#### 代码例子（最简线程池实现）
```c
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>
#include <unistd.h>

// 任务结构体（队列中的元素）
typedef struct Task {
    void (*func)(void*);  // 任务函数
    void* arg;            // 函数参数
    struct Task* next;    // 指向下一个任务
} Task;

// 线程池结构体
typedef struct {
    Task* task_head;      // 任务队列头
    Task* task_tail;      // 任务队列尾
    pthread_mutex_t mutex;  // 保护任务队列的锁
    pthread_cond_t cond;    // 任务队列有任务时的条件变量
    pthread_t* threads;     // 工作线程数组
    int thread_count;       // 线程数量
    int shutdown;           // 是否关闭线程池（1=是）
} ThreadPool;

// 初始化线程池
ThreadPool* thread_pool_init(int thread_count) {
    ThreadPool* pool = (ThreadPool*)malloc(sizeof(ThreadPool));
    pool->task_head = pool->task_tail = NULL;
    pool->thread_count = thread_count;
    pool->shutdown = 0;
    pthread_mutex_init(&pool->mutex, NULL);
    pthread_cond_init(&pool->cond, NULL);

    // 创建工作线程
    pool->threads = (pthread_t*)malloc(sizeof(pthread_t) * thread_count);
    for (int i = 0; i < thread_count; i++) {
        pthread_create(&pool->threads[i], NULL, (void* (*)(void*))worker_thread, pool);
    }
    return pool;
}

// 工作线程函数（从任务队列取任务执行）
void worker_thread(ThreadPool* pool) {
    while (1) {
        pthread_mutex_lock(&pool->mutex);

        // 没有任务且未关闭时，等待条件变量
        while (pool->task_head == NULL && !pool->shutdown) {
            pthread_cond_wait(&pool->cond, &pool->mutex);
        }

        // 如果线程池关闭且任务队列为空，退出线程
        if (pool->shutdown && pool->task_head == NULL) {
            pthread_mutex_unlock(&pool->mutex);
            pthread_exit(NULL);
        }

        // 取出任务（队头）
        Task* task = pool->task_head;
        pool->task_head = task->next;
        if (pool->task_head == NULL) {
            pool->task_tail = NULL;
        }

        pthread_mutex_unlock(&pool->mutex);

        // 执行任务
        task->func(task->arg);
        free(task);  // 释放任务内存
    }
}

// 向线程池添加任务
void thread_pool_add_task(ThreadPool* pool, void (*func)(void*), void* arg) {
    Task* task = (Task*)malloc(sizeof(Task));
    task->func = func;
    task->arg = arg;
    task->next = NULL;

    pthread_mutex_lock(&pool->mutex);

    // 添加到任务队列尾部
    if (pool->task_tail == NULL) {
        pool->task_head = pool->task_tail = task;
    } else {
        pool->task_tail->next = task;
        pool->task_tail = task;
    }

    pthread_cond_signal(&pool->cond);  // 通知工作线程：有新任务了！
    pthread_mutex_unlock(&mutex);
}

// 销毁线程池
void thread_pool_destroy(ThreadPool* pool) {
    pool->shutdown = 1;  // 标记关闭
    pthread_cond_broadcast(&pool->cond);  // 唤醒所有等待的线程

    // 等待所有工作线程退出
    for (int i = 0; i < pool->thread_count; i++) {
        pthread_join(pool->threads[i], NULL);
    }

    // 释放资源
    free(pool->threads);
    Task* task = pool->task_head;
    while (task != NULL) {
        Task* temp = task;
        task = task->next;
        free(temp);
    }
    pthread_mutex_destroy(&pool->mutex);
    pthread_cond_destroy(&pool->cond);
    free(pool);
}

// 测试任务函数（打印参数）
void test_task(void* arg) {
    int num = *(int*)arg;
    printf("处理任务%d\n", num);
    sleep(1);  // 模拟任务耗时
}

int main() {
    // 创建线程池（3个工作线程）
    ThreadPool* pool = thread_pool_init(3);

    // 添加10个任务
    int args[10];
    for (int i = 0; i < 10; i++) {
        args[i] = i;
        thread_pool_add_task(pool, test_task, &args[i]);
    }

    sleep(5);  // 等待任务处理完成（实际应更优雅）
    thread_pool_destroy(pool);  // 销毁线程池
    return 0;
}
```

#### 关键逻辑
- **任务队列**：用链表实现，互斥锁保护。  
- **工作线程**：循环从队列取任务，没有任务时用条件变量等待。  
- **添加任务**：将任务包装成 `Task` 结构体，添加到队列尾部，通知线程。  


---


## 三、进程间通信（IPC）：进程如何“跨办公室传文件”
### 3.1 管道（Pipe）：最简单的IPC（单向，父子进程专用）
#### 知识点说明
管道是**半双工**（只能单向通信）的，且只能用于**有亲缘关系的进程**（比如父进程和子进程）。管道的本质是内核中的一块缓冲区。

#### 代码例子（父进程发消息，子进程收消息）
```c
#include <stdio.h>
#include <unistd.h>
#include <string.h>

int main() {
    int pipefd[2];  // pipefd[0]读端，pipefd[1]写端
    if (pipe(pipefd) == -1) {  // 创建管道
        perror("pipe失败");
        return -1;
    }

    pid_t pid = fork();  // 创建子进程
    if (pid == -1) {
        perror("fork失败");
        return -1;
    }

    if (pid == 0) {  // 子进程（读数据）
        close(pipefd[1]);  // 关闭写端（不用写）
        char buf[100];
        int len = read(pipefd[0], buf, sizeof(buf));  // 从管道读数据
        printf("子进程收到：%s\n", buf);
        close(pipefd[0]);
    } else {  // 父进程（写数据）
        close(pipefd[0]);  // 关闭读端（不用读）
        const char* msg = "你好，子进程！";
        write(pipefd[1], msg, strlen(msg) + 1);  // 向管道写数据
        close(pipefd[1]);
        waitpid(pid, NULL, 0);  // 等待子进程结束
    }
    return 0;
}
```

#### 输出结果
```
子进程收到：你好，子进程！
```


### 3.2 共享内存（Shared Memory）：最快的IPC（直接共享内存）
#### 知识点说明
共享内存让多个进程**共享同一块物理内存**（类似“共享文件柜”），是**最快的IPC方式**（无需拷贝数据）。但需要配合**互斥锁/信号量**避免并发访问问题。

#### 代码例子（进程A写数据，进程B读数据）
```c
// 进程A（写数据）
#include <stdio.h>
#include <sys/shm.h>
#include <string.h>

#define SHM_KEY 1234  // 共享内存的key（唯一标识）
#define SHM_SIZE 1024  // 共享内存大小（字节）

int main() {
    // 创建/获取共享内存（IPC_CREAT表示不存在则创建）
    int shmid = shmget(SHM_KEY, SHM_SIZE, IPC_CREAT | 0666);
    if (shmid == -1) {
        perror("shmget失败");
        return -1;
    }

    // 附加共享内存到当前进程地址空间
    char* shm_addr = (char*)shmat(shmid, NULL, 0);
    if (shm_addr == (char*)-1) {
        perror("shmat失败");
        return -1;
    }

    // 写数据到共享内存
    strcpy(shm_addr, "共享内存测试数据");
    printf("进程A已写入：%s\n", shm_addr);

    // 分离共享内存（不删除，只是断开连接）
    shmdt(shm_addr);
    return 0;
}
```

```c
// 进程B（读数据）
#include <stdio.h>
#include <sys/shm.h>

#define SHM_KEY 1234

int main() {
    // 获取共享内存（无需创建）
    int shmid = shmget(SHM_KEY, 0, 0);
    if (shmid == -1) {
        perror("shmget失败");
        return -1;
    }

    // 附加共享内存
    char* shm_addr = (char*)shmat(shmid, NULL, 0);
    if (shm_addr == (char*)-1) {
        perror("shmat失败");
        return -1;
    }

    // 读数据
    printf("进程B读取到：%s\n", shm_addr);

    // 分离并删除共享内存（IPC_RMID标记删除，所有进程断开后真正删除）
    shmdt(shm_addr);
    shmctl(shmid, IPC_RMID, NULL);
    return 0;
}
```

#### 运行步骤
```bash
./进程A  # 先运行A写数据
./进程B  # 再运行B读数据
```

#### 输出结果
```
进程A已写入：共享内存测试数据
进程B读取到：共享内存测试数据
```

#### 注意
- 共享内存**不会自动删除**，需用 `shmctl(shmid, IPC_RMID, NULL)` 标记删除（所有进程断开后才会释放）。  
- 多个进程同时修改共享内存时，必须用互斥锁或信号量同步（否则数据会乱）。


### 3.3 信号（Signal）：进程的“紧急通知”
#### 知识点说明
信号是**异步事件**（比如 `Ctrl+C` 发送 `SIGINT` 信号终止进程）。进程可以**注册信号处理函数**（收到信号时执行自定义操作）。

#### 代码例子（捕获 `SIGINT` 信号，避免被终止）
```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

// 信号处理函数（收到SIGINT时执行）
void sigint_handler(int sig) {
    printf("收到SIGINT信号（Ctrl+C），但我不退出！\n");
}

int main() {
    // 注册信号处理函数（SIGINT对应Ctrl+C）
    signal(SIGINT, sigint_handler);

    while (1) {
        printf("运行中...（按Ctrl+C试试）\n");
        sleep(1);
    }
    return 0;
}
```

#### 运行结果
```
运行中...（按Ctrl+C试试）
运行中...（按Ctrl+C试试）
收到SIGINT信号（Ctrl+C），但我不退出！
运行中...（按Ctrl+C试试）
...
```


---


## 四、TCP网络编程：从“打电话”到“发微信”（附服务器-客户端代码）
### 4.1 TCP基础：三次握手与四次挥手（用“打电话”类比）
- **三次握手（建立连接）**：  
  客户端→服务器：“喂，能听到吗？”（SYN）  
  服务器→客户端：“能听到，你能听到我吗？”（SYN+ACK）  
  客户端→服务器：“能听到，开始聊吧！”（ACK）  

- **四次挥手（断开连接）**：  
  客户端→服务器：“我说完了。”（FIN）  
  服务器→客户端：“收到，我还没说完。”（ACK）  
  服务器→客户端：“我也说完了。”（FIN）  
  客户端→服务器：“收到，结束。”（ACK）  


### 4.2 TCP服务器-客户端代码实战（C语言）
#### 服务器端代码（持续接收客户端消息）
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 8888  // 监听端口
#define BUF_SIZE 1024

int main() {
    int server_fd, client_fd;
    struct sockaddr_in server_addr, client_addr;
    socklen_t client_addr_len = sizeof(client_addr);
    char buf[BUF_SIZE];

    // 1. 创建socket（相当于买电话）
    server_fd = socket(AF_INET, SOCK_STREAM, 0);  // AF_INET=IPv4，SOCK_STREAM=TCP
    if (server_fd == -1) {
        perror("socket失败");
        return -1;
    }

    // 2. 绑定端口（相当于设置电话号码）
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;  // 监听所有网卡
    server_addr.sin_port = htons(PORT);  // 端口号转网络字节序（大端）

    if (bind(server_fd, (struct sockaddr*)&server_addr, sizeof(server_addr)) == -1) {
        perror("bind失败");
        close(server_fd);
        return -1;
    }

    // 3. 监听（相当于待机，等待来电）
    if (listen(server_fd, 5) == -1) {  // 最大等待连接数5
        perror("listen失败");
        close(server_fd);
        return -1;
    }
    printf("服务器启动，监听端口%d...\n", PORT);

    // 4. 接受客户端连接（相当于接电话）
    client_fd = accept(server_fd, (struct sockaddr*)&client_addr, &client_addr_len);
    if (client_fd == -1) {
        perror("accept失败");
        close(server_fd);
        return -1;
    }
    printf("客户端%s:%d连接成功\n", 
           inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));

    // 5. 接收/发送数据（相当于通话）
    while (1) {
        int len = read(client_fd, buf, BUF_SIZE - 1);
        if (len == -1) {
            perror("read失败");
            break;
        }
        if (len == 0) {  // 客户端断开连接
            printf("客户端断开\n");
            break;
        }
        buf[len] = '\0';
        printf("客户端消息：%s\n", buf);

        // 回复消息
        char* reply = "已收到消息";
        write(client_fd, reply, strlen(reply));
    }

    // 6. 关闭连接（挂电话）
    close(client_fd);
    close(server_fd);
    return 0;
}
```

#### 客户端代码（连接服务器并发送消息）
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define SERVER_IP "127.0.0.1"  // 服务器IP（本地测试用回环地址）
#define PORT 8888
#define BUF_SIZE 1024

int main() {
    int client_fd;
    struct sockaddr_in server_addr;
    char buf[BUF_SIZE];

    // 1. 创建socket
    client_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (client_fd == -1) {
        perror("socket失败");
        return -1;
    }

    // 2. 连接服务器（拨打电话）
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(PORT);
    if (inet_pton(AF_INET, SERVER_IP, &server_addr.sin_addr) <= 0) {  // IP转二进制
        perror("IP地址无效");
        close(client_fd);
        return -1;
    }

    if (connect(client_fd, (struct sockaddr*)&server_addr, sizeof(server_addr)) == -1) {
        perror("connect失败");
        close(client_fd);
        return -1;
    }
    printf("连接服务器%s:%d成功\n", SERVER_IP, PORT);

    // 3. 发送/接收数据
    while (1) {
        printf("请输入要发送的消息（输入exit退出）：");
        fgets(buf, BUF_SIZE, stdin);  // 从标准输入读消息
        buf[strcspn(buf, "\n")] = '\0';  // 去掉换行符

        if (strcmp(buf, "exit") == 0) {  // 输入exit退出
            break;
        }

        write(client_fd, buf, strlen(buf));  // 发送消息

        int len = read(client_fd, buf, BUF_SIZE - 1);  // 接收回复
        if (len == -1) {
            perror("read失败");
            break;
        }
        buf[len] = '\0';
        printf("服务器回复：%s\n", buf);
    }

    // 4. 关闭连接
    close(client_fd);
    return 0;
}
```

#### 运行步骤
1. 先启动服务器：`./server`  
2. 再启动客户端：`./client`  
3. 客户端输入消息（如“Hello”），服务器会打印并回复“已收到消息”。


### 4.3 常见问题：TCP粘包
- **问题描述**：TCP是“流”协议（没有消息边界），客户端连续发送多条短消息，服务器可能一次性收到多条（比如发“12”和“34”，服务器收到“1234”）。  
- **解决方法**：  
  - 固定消息长度（比如每条消息100字节，不足补0）。  
  - 用特殊分隔符（比如每条消息以`\n`结尾）。  
  - 在消息头加长度字段（比如前4字节表示消息体长度）。  


---


## 五、文件管理与内存映射（mmap）：高效读写大文件
### 5.1 普通文件操作（open/read/write）
#### 知识点说明
- `open()`：打开文件，返回文件描述符（类似“文件的身份证”）。  
- `read()`/`write()`：通过文件描述符读写文件。  
- `close()`：关闭文件描述符（释放资源）。

#### 代码例子（复制文件）
```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <stdlib.h>

#define BUF_SIZE 1024

int main() {
    int src_fd = open("src.txt", O_RDONLY);  // 只读打开源文件
    if (src_fd == -1) {
        perror("open src.txt失败");
        return -1;
    }

    int dest_fd = open("dest.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);  // 写打开目标文件（不存在则创建，清空内容）
    if (dest_fd == -1) {
        perror("open dest.txt失败");
        close(src_fd);
        return -1;
    }

    char buf[BUF_SIZE];
    ssize_t len;
    while ((len = read(src_fd, buf, BUF_SIZE)) > 0) {  // 读源文件
        if (write(dest_fd, buf, len) != len) {  // 写目标文件
            perror("write失败");
            close(src_fd);
            close(dest_fd);
            return -1;
        }
    }

    if (len == -1) {
        perror("read失败");
        close(src_fd);
        close(dest_fd);
        return -1;
    }

    close(src_fd);
    close(dest_fd);
    printf("文件复制完成\n");
    return 0;
}
```


### 5.2 内存映射（mmap）：把文件当内存用
#### 知识点说明
`mmap()` 能将**文件内容映射到进程的虚拟内存**（相当于“文件内容直接出现在内存里”）。后续读写内存就相当于读写文件，**无需调用 `read/write`**，适合处理大文件（比如几GB的日志）。

#### 代码例子（用mmap复制大文件）
```c
#include <stdio.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main() {
    int src_fd = open("big_file.bin", O_RDONLY);
    if (src_fd == -1) {
        perror("open源文件失败");
        return -1;
    }

    // 获取文件大小
    struct stat src_stat;
    if (fstat(src_fd, &src_stat) == -1) {
        perror("fstat失败");
        close(src_fd);
        return -1;
    }
    off_t file_size = src_stat.st_size;

    // 内存映射源文件（可读）
    char* src_map = mmap(NULL, file_size, PROT_READ, MAP_SHARED, src_fd, 0);
    if (src_map == MAP_FAILED) {
        perror("mmap源文件失败");
        close(src_fd);
        return -1;
    }

    // 创建目标文件并设置大小（必须！否则mmap会失败）
    int dest_fd = open("big_file_copy.bin", O_RDWR | O_CREAT | O_TRUNC, 0644);
    if (dest_fd == -1) {
        perror("open目标文件失败");
        munmap(src_map, file_size);
        close(src_fd);
        return -1;
    }
    if (ftruncate(dest_fd, file_size) == -1) {  // 设置文件大小
        perror("ftruncate失败");
        munmap(src_map, file_size);
        close(src_fd);
        close(dest_fd);
        return -1;
    }

    // 内存映射目标文件（可读写）
    char* dest_map = mmap(NULL, file_size, PROT_READ | PROT_WRITE, MAP_SHARED, dest_fd, 0);
    if (dest_map == MAP_FAILED) {
        perror("mmap目标文件失败");
        munmap(src_map, file_size);
        close(src_fd);
        close(dest_fd);
        return -1;
    }

    // 复制内存（即复制文件）
    memcpy(dest_map, src_map, file_size);

    // 同步内存到磁盘（确保数据写入）
    if (msync(dest_map, file_size, MS_SYNC) == -1) {
        perror("msync失败");
    }

    // 解除映射并关闭文件
    munmap(src_map, file_size);
    munmap(dest_map, file_size);
    close(src_fd);
    close(dest_fd);
    printf("大文件复制完成（大小%d字节）\n", (int)file_size);
    return 0;
}
```

#### 优点
- **高效**：减少 `read/write` 系统调用次数（普通IO需要“用户空间→内核空间”两次拷贝，mmap直接内存映射）。  
- **方便**：可以像操作数组一样操作文件内容（比如 `src_map[100]` 直接访问文件第100字节）。  


---


## 总结：学习路径建议
1. **先跑通例子**：把文中的代码自己敲一遍，编译运行，观察输出（比如线程同步的例子不加锁会怎样）。  
2. **动手改代码**：尝试修改线程数、队列大小、网络端口，看结果如何变化（比如把生产者-消费者的队列大小改小，观察等待频率）。  
3. **结合工具调试**：用 `strace` 跟踪系统调用（比如 `strace ./thread_demo` 看 `pthread_create` 调用了哪些内核函数），用 `gdb` 调试多线程程序。  
4. **查手册**：遇到不熟悉的函数（如 `pthread_cond_wait`），查 `man` 手册（`man pthread_cond_wait`），看参数和返回值。  

这些概念刚开始可能有点抽象，但通过**动手写代码+观察现象**，很快就能掌握。遇到问题别慌，90%的错误都能通过查日志（`perror` 输出）和调试解决~