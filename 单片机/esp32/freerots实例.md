




### 说明  
以下实例基于 **ESP32-DevKitC 核心板**（板载 LED 在 GPIO2，UART0 用于调试），使用 **ESP-IDF 5.1+** 框架，无需外接元件。每个实例包含 **完整代码**、**关键函数解析** 和 **运行效果**，覆盖 FreeRTOS 核心功能（任务创建、队列、信号量、软件定时器）。


### 实例1：双任务串口打印（基础任务调度）  
**功能**：创建两个 FreeRTOS 任务，分别以不同周期打印日志到串口，演示任务调度机制。  

#### 代码（`main.c`）  
```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_log.h"

// 任务标签（用于日志分类）
static const char *TAG_TASK1 = "TASK1";
static const char *TAG_TASK2 = "TASK2";

// 任务1：每1000ms打印一次
void task1(void *arg) {
    while (1) {
        ESP_LOGI(TAG_TASK1, "Hello from Task1! FreeRTOS tick: %lu", xTaskGetTickCount());
        vTaskDelay(pdMS_TO_TICKS(1000)); // 延迟1000ms（释放CPU）
    }
}

// 任务2：每500ms打印一次
void task2(void *arg) {
    while (1) {
        ESP_LOGI(TAG_TASK2, "Hi from Task2! FreeRTOS tick: %lu", xTaskGetTickCount());
        vTaskDelay(pdMS_TO_TICKS(500)); // 延迟500ms
    }
}

void app_main(void) {
    // 创建任务（参数：任务函数、任务名、栈大小、参数、优先级、任务句柄）
    xTaskCreate(task1, "Task1", 2048, NULL, 1, NULL);
    xTaskCreate(task2, "Task2", 2048, NULL, 1, NULL);
}
```

#### 关键函数  
- `xTaskCreate()`：创建 FreeRTOS 任务，参数包括任务函数、名称、栈大小（单位：字）、优先级（数值越大优先级越高）。  
- `vTaskDelay(pdMS_TO_TICKS(ms))`：任务主动挂起，释放 CPU 使用权，参数为延迟毫秒数（转换为系统滴答）。  
- `ESP_LOGI()`：ESP-IDF 日志宏（`I` 表示信息级，支持颜色输出）。  

#### 运行效果  
串口终端会交替输出两个任务的日志（时间间隔分别为 1s 和 0.5s）：  
```
[I][TASK1:12] task1: Hello from Task1! FreeRTOS tick: 1000
[I][TASK2:17] task2: Hi from Task2! FreeRTOS tick: 1000
[I][TASK2:17] task2: Hi from Task2! FreeRTOS tick: 1500
[I][TASK1:12] task1: Hello from Task1! FreeRTOS tick: 2000
[I][TASK2:17] task2: Hi from Task2! FreeRTOS tick: 2000
...
```


### 实例2：串口接收回显（队列实现任务通信）  
**功能**：通过 FreeRTOS 队列实现“串口接收→处理→发送”流程：任务1监听串口数据，通过队列发送给任务2；任务2从队列取出数据并回发到串口。  

#### 代码（`main.c`）  
```c
#include <stdio.h>
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/queue.h"
#include "driver/uart.h"
#include "esp_log.h"

// 串口配置（UART0，默认连接USB转串口）
#define UART_NUM UART_NUM_0
#define BUF_SIZE 128
#define QUEUE_SIZE 5 // 队列最大存储5条数据

// 定义队列句柄
QueueHandle_t uart_queue;

// 任务1：监听串口数据，发送到队列
void uart_receive_task(void *arg) {
    uint8_t data[BUF_SIZE];
    while (1) {
        // 从串口读取数据（阻塞，超时100ms）
        int len = uart_read_bytes(UART_NUM, data, BUF_SIZE, pdMS_TO_TICKS(100));
        if (len > 0) {
            // 将数据长度和内容打包为结构体
            typedef struct {
                int length;
                uint8_t buf[BUF_SIZE];
            } uart_packet_t;
            uart_packet_t packet = {.length = len};
            memcpy(packet.buf, data, len);

            // 发送到队列（阻塞，超时100ms）
            if (xQueueSend(uart_queue, &packet, pdMS_TO_TICKS(100)) != pdPASS) {
                ESP_LOGE("UART_RX", "Queue send failed!");
            }
        }
    }
}

// 任务2：从队列取出数据，回发到串口
void uart_send_task(void *arg) {
    typedef struct {
        int length;
        uint8_t buf[BUF_SIZE];
    } uart_packet_t;
    uart_packet_t packet;

    while (1) {
        // 从队列接收数据（阻塞，超时1000ms）
        if (xQueueReceive(uart_queue, &packet, pdMS_TO_TICKS(1000)) == pdPASS) {
            // 回发数据到串口
            uart_write_bytes(UART_NUM, (const char *)packet.buf, packet.length);
            ESP_LOGI("UART_TX", "Echo: %.*s", packet.length, packet.buf);
        }
    }
}

void app_main(void) {
    // 初始化串口（波特率115200，8位数据，1位停止位，无校验）
    uart_config_t uart_cfg = {
        .baud_rate = 115200,
        .data_bits = UART_DATA_8_BITS,
        .stop_bits = UART_STOP_BITS_1,
        .parity = UART_PARITY_DISABLE,
        .flow_ctrl = UART_HW_FLOWCTRL_DISABLE
    };
    uart_param_config(UART_NUM, &uart_cfg);
    uart_driver_install(UART_NUM, BUF_SIZE * 2, BUF_SIZE * 2, 10, &uart_queue, 0);

    // 创建队列（存储uart_packet_t类型，最大5条）
    uart_queue = xQueueCreate(QUEUE_SIZE, sizeof(uart_packet_t));

    // 创建任务
    xTaskCreate(uart_receive_task, "UART_RX", 2048, NULL, 2, NULL);
    xTaskCreate(uart_send_task, "UART_TX", 2048, NULL, 2, NULL);
}
```

#### 关键函数  
- `xQueueCreate()`：创建 FreeRTOS 队列，参数为队列长度和单条数据大小。  
- `xQueueSend()`：向队列发送数据（阻塞等待空位）。  
- `xQueueReceive()`：从队列接收数据（阻塞等待数据）。  
- `uart_read_bytes()`/`uart_write_bytes()`：ESP-IDF 串口读写函数（需先初始化驱动）。  

#### 运行效果  
通过串口助手（如 Putty）发送任意字符（例如“Hello”），串口会立即回显“Hello”，并打印日志：  
```
[I][UART_TX:38] uart_send_task: Echo: Hello
```


### 实例3：板载LED闪烁与串口同步（信号量同步任务）  
**功能**：两个任务通过二值信号量同步：任务1控制板载 LED 闪烁，每闪烁一次释放信号量；任务2获取信号量后通过串口报告 LED 状态。  

#### 代码（`main.c`）  
```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/semphr.h"
#include "driver/gpio.h"
#include "esp_log.h"

// 板载LED引脚（ESP32-DevKitC为GPIO2）
#define LED_PIN GPIO_NUM_2

// 定义信号量句柄
SemaphoreHandle_t led_semaphore;

// 任务1：控制LED闪烁，释放信号量
void led_task(void *arg) {
    gpio_set_direction(LED_PIN, GPIO_MODE_OUTPUT);
    while (1) {
        gpio_set_level(LED_PIN, 1); // LED亮
        vTaskDelay(pdMS_TO_TICKS(500));
        gpio_set_level(LED_PIN, 0); // LED灭
        vTaskDelay(pdMS_TO_TICKS(500));
        xSemaphoreGive(led_semaphore); // 释放信号量（任务2可继续）
    }
}

// 任务2：获取信号量，串口报告状态
void log_task(void *arg) {
    static uint32_t count = 0;
    while (1) {
        // 等待信号量（阻塞，超时1000ms）
        if (xSemaphoreTake(led_semaphore, pdMS_TO_TICKS(1000)) == pdPASS) {
            count++;
            ESP_LOGI("LED_LOG", "LED toggled %d times!", count);
        } else {
            ESP_LOGE("LED_LOG", "Wait semaphore timeout!");
        }
    }
}

void app_main(void) {
    // 创建二值信号量（初始为未占用）
    led_semaphore = xSemaphoreCreateBinary();

    // 创建任务（LED任务优先级更高）
    xTaskCreate(led_task, "LED_Task", 2048, NULL, 3, NULL);
    xTaskCreate(log_task, "LOG_Task", 2048, NULL, 2, NULL);
}
```

#### 关键函数  
- `xSemaphoreCreateBinary()`：创建二值信号量（0或1状态）。  
- `xSemaphoreGive()`：释放信号量（状态置1）。  
- `xSemaphoreTake()`：获取信号量（状态置0，若状态为0则阻塞）。  
- `gpio_set_level()`：控制 GPIO 电平（LED 亮/灭）。  

#### 运行效果  
LED 每 1 秒闪烁一次（亮 500ms+灭 500ms），串口同步输出闪烁次数：  
```
[I][LED_LOG:23] log_task: LED toggled 1 times!
[I][LED_LOG:23] log_task: LED toggled 2 times!
[I][LED_LOG:23] log_task: LED toggled 3 times!
...
```


### 实例4：软件定时器周期任务（定时串口输出）  
**功能**：创建一个 FreeRTOS 软件定时器，每 2 秒通过串口打印系统运行时间，演示软件定时器的使用。  

#### 代码（`main.c`）  
```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/timers.h"
#include "esp_log.h"

// 定义定时器句柄
TimerHandle_t periodic_timer;

// 定时器回调函数（必须为void (*)(TimerHandle_t)类型）
void timer_callback(TimerHandle_t xTimer) {
    uint32_t timer_id = (uint32_t)pvTimerGetTimerID(xTimer); // 获取定时器ID（可自定义）
    uint32_t ticks = xTaskGetTickCount();
    ESP_LOGI("TIMER", "Timer %d triggered! System tick: %lu", timer_id, ticks);
}

void app_main(void) {
    // 创建软件定时器（参数：名称、周期（ms转换为滴答）、是否自动重载、ID、回调函数）
    periodic_timer = xTimerCreate(
        "PeriodicTimer",
        pdMS_TO_TICKS(2000), // 2000ms周期
        pdTRUE,              // pdTRUE=自动重载（重复触发）
        (void *)1,           // 定时器ID（可自定义）
        timer_callback       // 回调函数
    );

    // 启动定时器（阻塞，超时100ms）
    if (xTimerStart(periodic_timer, pdMS_TO_TICKS(100)) != pdPASS) {
        ESP_LOGE("TIMER", "Start timer failed!");
    }
}
```

#### 关键函数  
- `xTimerCreate()`：创建软件定时器，参数包括周期（滴答数）、是否自动重载（`pdTRUE` 为循环触发）。  
- `xTimerStart()`：启动定时器（需在创建后调用）。  
- `pvTimerGetTimerID()`：获取定时器自定义 ID（用于多定时器场景）。  

#### 运行效果  
串口每 2 秒输出一次系统运行时间：  
```
[I][TIMER:11] timer_callback: Timer 1 triggered! System tick: 2000
[I][TIMER:11] timer_callback: Timer 1 triggered! System tick: 4000
[I][TIMER:11] timer_callback: Timer 1 triggered! System tick: 6000
...
```


### 如何在 Windows 下运行？  
1. **安装 ESP-IDF**：  
   - 下载 [ESP-IDF 工具链](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32/get-started/windows-setup.html)，按向导安装（推荐使用 `ESP-IDF 5.1`）。  
   - 安装完成后，打开 `ESP-IDF Command Prompt`（Windows 开始菜单）。  

2. **创建工程**：  
   ```bash
   idf.py create-project -p my_esp32_project -b esp32
   cd my_esp32_project
   ```  
   将上述实例代码替换 `main/main.c`。  

3. **配置串口**：  
   - 连接 ESP32 到电脑，查看设备管理器中的 COM 端口（如 `COM3`）。  
   - 运行 `idf.py menuconfig` → 进入 `Component config → ESP System → UART for console`，确保波特率为 `115200`（默认）。  

4. **编译、烧录、监控**：  
   ```bash
   idf.py build           # 编译工程
   idf.py -p COM3 flash   # 烧录到 ESP32（COM3 替换为你的端口）
   idf.py -p COM3 monitor # 打开串口监控（Ctrl+] 退出）
   ```  


### 扩展建议  
- **任务优先级**：修改任务优先级（`xTaskCreate` 的第5个参数），观察高优先级任务是否抢占低优先级任务。  
- **队列优化**：在实例2中尝试增大 `QUEUE_SIZE`（如 10），测试连续发送大量数据时的队列溢出处理。  
- **LED 亮度调节**：结合实例3和 LEDC 定时器（PWM），实现 LED 呼吸灯效果（替换 `gpio_set_level` 为 PWM 占空比调整）。  

需要其他类型的实例（如事件组、互斥锁）或具体功能扩展，可以随时告诉我！