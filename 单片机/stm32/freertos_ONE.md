![[c823ac749a1055ae31a45dcdc0b81a70.png]]
### 实现思路

#### 1. OLED 显示任务
- **功能**：显示 LED1 和 LED2 任务的状态。
- **实现**：
  - 使用 CubeMX 初始化 OLED 相关的 GPIO 和外设。
  - 创建一个任务函数，定期调用 `OLED_ShowString` 函数显示任务状态。
  - 使用 FreeRTOS 的 `vTaskDelay` 函数控制显示任务的执行周期。

#### 2. LED1 任务
- **功能**：控制 LED1 以 500ms 周期翻转亮灭。
- **实现**：
  - 使用 CubeMX 初始化 LED1 相关的 GPIO。
  - 创建一个任务函数，定期切换 LED1 的状态。
  - 使用 FreeRTOS 的 `vTaskDelay` 函数控制任务的执行周期。

#### 3. LED2 任务
- **功能**：按键 B 按下时创建或删除任务，控制 LED2 以 1000ms 周期翻转亮灭。
- **实现**：
  - 使用 CubeMX 初始化 LED2 和按键 B 相关的 GPIO。
  - 创建一个任务函数，定期切换 LED2 的状态。
  - 使用 FreeRTOS 的 `vTaskDelay` 函数控制任务的执行周期。
  - 在按键任务中，根据按键 B 的状态创建或删除 LED2 任务。

#### 4. 按键任务
- **功能**：按键 A 按下时挂起或解挂 LED1 任务，按键 B 按下时创建或删除 LED2 任务。
- **实现**：
  - 使用 CubeMX 初始化按键 A 和按键 B 相关的 GPIO。
  - 创建一个任务函数，检测按键状态并执行相应的操作。
  - 使用 FreeRTOS 的 `vTaskSuspend` 和 `vTaskResume` 函数挂起或解挂 LED1 任务。
  - 使用 FreeRTOS 的 `xTaskCreate` 和 `vTaskDelete` 函数创建或删除 LED2 任务。

### 引脚配置
- **LED1**：PA0
- **LED2**：PA1
- **按键 A**：PB0
- **按键 B**：PB1

### HAL 库实现

```c
#include "FreeRTOS.h"
#include "task.h"
#include "stm32f1xx_hal.h"

// 引脚配置
#define LED1_PIN GPIO_PIN_0
#define LED2_PIN GPIO_PIN_1
#define KEY_A_PIN GPIO_PIN_0
#define KEY_B_PIN GPIO_PIN_1
#define LED1_GPIO_PORT GPIOA
#define LED2_GPIO_PORT GPIOA
#define KEY_A_GPIO_PORT GPIOB
#define KEY_B_GPIO_PORT GPIOB

// 任务句柄
TaskHandle_t xOLEDTaskHandle;
TaskHandle_t xLED1TaskHandle;
TaskHandle_t xLED2TaskHandle;
TaskHandle_t xKeyTaskHandle;

// 按键状态
volatile uint8_t key_a_pressed = 0;
volatile uint8_t key_b_pressed = 0;

// OLED 显示任务
void OLED_Task(void *pvParameters) {
    while (1) {
        // 显示 LED1 和 LED2 任务的状态
        OLED_ShowString(1, 1, "LED1: ", 2);
        OLED_ShowString(1, 4, "LED2: ", 2);
        HAL_GPIO_TogglePin(LED1_GPIO_PORT, LED1_PIN);
        HAL_GPIO_TogglePin(LED2_GPIO_PORT, LED2_PIN);
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}

// LED1 任务
void LED1_Task(void *pvParameters) {
    while (1) {
        HAL_GPIO_TogglePin(LED1_GPIO_PORT, LED1_PIN);
        vTaskDelay(500 / portTICK_PERIOD_MS);
    }
}

// LED2 任务
void LED2_Task(void *pvParameters) {
    while (1) {
        HAL_GPIO_TogglePin(LED2_GPIO_PORT, LED2_PIN);
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}

// 按键任务
void Key_Task(void *pvParameters) {
    while (1) {
        if (HAL_GPIO_ReadPin(KEY_A_GPIO_PORT, KEY_A_PIN) == GPIO_PIN_SET) {
            key_a_pressed = 1;
            if (xLED1TaskHandle != NULL) {
                vTaskSuspend(xLED1TaskHandle);
            } else {
                xTaskCreate(LED1_Task, "LED1_Task", 128, NULL, 1, &xLED1TaskHandle);
            }
        }
        if (HAL_GPIO_ReadPin(KEY_B_GPIO_PORT, KEY_B_PIN) == GPIO_PIN_SET) {
            key_b_pressed = 1;
            if (xLED2TaskHandle != NULL) {
                vTaskDelete(xLED2TaskHandle);
                xLED2TaskHandle = NULL;
            } else {
                xTaskCreate(LED2_Task, "LED2_Task", 128, NULL, 1, &xLED2TaskHandle);
            }
        }
        vTaskDelay(100 / portTICK_PERIOD_MS);
    }
}

int main(void) {
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();
    MX_OLED_Init();

    xTaskCreate(OLED_Task, "OLED_Task", 128, NULL, 1, &xOLEDTaskHandle);
    xTaskCreate(Key_Task, "Key_Task", 128, NULL, 1, &xKeyTaskHandle);

    vTaskStartScheduler();

    while (1) {
    }
}

void MX_GPIO_Init(void) {
    GPIO_InitTypeDef GPIO_InitStruct = {0};

    __HAL_RCC_GPIOA_CLK_ENABLE();
    __HAL_RCC_GPIOB_CLK_ENABLE();

    GPIO_InitStruct.Pin = LED1_PIN | LED2_PIN;
    GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
    HAL_GPIO_Init(LED1_GPIO_PORT, &GPIO_InitStruct);
    HAL_GPIO_Init(LED2_GPIO_PORT, &GPIO_InitStruct);

    GPIO_InitStruct.Pin = KEY_A_PIN | KEY_B_PIN;
    GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    HAL_GPIO_Init(KEY_A_GPIO_PORT, &GPIO_InitStruct);
    HAL_GPIO_Init(KEY_B_GPIO_PORT, &GPIO_InitStruct);
}
```

### 标准库实现

```c
#include "FreeRTOS.h"
#include "task.h"
#include "stm32f10x.h"

// 引脚配置
#define LED1_PIN GPIO_Pin_0
#define LED2_PIN GPIO_Pin_1
#define KEY_A_PIN GPIO_Pin_0
#define KEY_B_PIN GPIO_Pin_1
#define LED1_GPIO_PORT GPIOA
#define LED2_GPIO_PORT GPIOA
#define KEY_A_GPIO_PORT GPIOB
#define KEY_B_GPIO_PORT GPIOB

// 任务句柄
TaskHandle_t xOLEDTaskHandle;
TaskHandle_t xLED1TaskHandle;
TaskHandle_t xLED2TaskHandle;
TaskHandle_t xKeyTaskHandle;

// 按键状态
volatile uint8_t key_a_pressed = 0;
volatile uint8_t key_b_pressed = 0;

// OLED 显示任务
void OLED_Task(void *pvParameters) {
    while (1) {
        // 显示 LED1 和 LED2 任务的状态
        OLED_ShowString(1, 1, "LED1: ", 2);
        OLED_ShowString(1, 4, "LED2: ", 2);
        GPIO_ToggleBits(LED1_GPIO_PORT, LED1_PIN);
        GPIO_ToggleBits(LED2_GPIO_PORT, LED2_PIN);
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}

// LED1 任务
void LED1_Task(void *pvParameters) {
    while (1) {
        GPIO_ToggleBits(LED1_GPIO_PORT, LED1_PIN);
        vTaskDelay(500 / portTICK_PERIOD_MS);
    }
}

// LED2 任务
void LED2_Task(void *pvParameters) {
    while (1) {
        GPIO_ToggleBits(LED2_GPIO_PORT, LED2_PIN);
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}

// 按键任务
void Key_Task(void *pvParameters) {
    while (1) {
        if (GPIO_ReadInputDataBit(KEY_A_GPIO_PORT, KEY_A_PIN) == BIT_SET) {
            key_a_pressed = 1;
            if (xLED1TaskHandle != NULL) {
                vTaskSuspend(xLED1TaskHandle);
            } else {
                xTaskCreate(LED1_Task, "LED1_Task", 128, NULL, 1, &xLED1TaskHandle);
            }
        }
        if (GPIO_ReadInputDataBit(KEY_B_GPIO_PORT, KEY_B_PIN) == BIT_SET) {
            key_b_pressed = 1;
            if (xLED2TaskHandle != NULL) {
                vTaskDelete(xLED2TaskHandle);
                xLED2TaskHandle = NULL;
            } else {
                xTaskCreate(LED2_Task, "LED2_Task", 128, NULL, 1, &xLED2TaskHandle);
            }
        }
        vTaskDelay(100 / portTICK_PERIOD_MS);
    }
}

int main(void) {
    SystemInit();
    SystemClock_Config();
    GPIO_InitTypeDef GPIO_InitStruct = {0};

    // 初始化 OLED
    OLED_Init();

    // 初始化 LED 和按键 GPIO
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA | RCC_APB2Periph_GPIOB, ENABLE);

    GPIO_InitStruct.GPIO_Pin = LED1_PIN | LED2_PIN;
    GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(LED1_GPIO_PORT, &GPIO_InitStruct);
    GPIO_Init(LED2_GPIO_PORT, &GPIO_InitStruct);

    GPIO_InitStruct.GPIO_Pin = KEY_A_PIN | KEY_B_PIN;
    GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
    GPIO_Init(KEY_A_GPIO_PORT, &GPIO_InitStruct);
    GPIO_Init(KEY_B_GPIO_PORT, &GPIO_InitStruct);

    xTaskCreate(OLED_Task, "OLED_Task", 128, NULL, 1, &xOLEDTaskHandle);
    xTaskCreate(Key_Task, "Key_Task", 128, NULL, 1, &xKeyTaskHandle);

    vTaskStartScheduler();

    while (1) {
    }
}
```

### platformio.ini 配置文件
```ini
[env:stm32f103vet6]
platform = ststm32
board = stm32f103vet6
framework = arduino
monitor_speed = 115200
upload_speed = 921600
lib_deps = 
    FreeRTOS
```
![[Pasted image 20250514224816.png]]