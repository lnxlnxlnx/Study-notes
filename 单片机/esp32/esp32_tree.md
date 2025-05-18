# ESP32 知识树（无线MCU）

## 一、硬件基础
- 核心参数：
  - CPU：Xtensa双核32位（240MHz，支持浮点）
  - 内存：520KB SRAM，4MB Flash（可扩展）
  - 无线：Wi-Fi（802.11b/g/n）、蓝牙5.0（经典/BLE）
  - 外设：34×GPIO（支持I2C/SPI/UART/PWM/ADC/DAC）、2×SAR ADC（12位，18通道）、温度传感器

## 二、开发环境
- 主流框架：
  1. **ESP-IDF**（官方原生，C/C++）：
     - 工具链：CMake构建，支持VS Code/CLion
     - 核心API：`esp_system.h`（系统）、`esp_wifi.h`（Wi-Fi）、`nvs_flash.h`（存储）
  2. **Arduino-ESP32**（简化开发，C++）：
     - 兼容Arduino语法，库丰富（如`WiFi.h`、`BLE.h`）
     - 典型函数：`setup()/loop()`（主循环）、`WiFi.begin()`（连接Wi-Fi）

## 三、核心功能模块
### 1. 基础外设
- **GPIO**：
  - ESP-IDF：`gpio_set_direction()`（设置方向）、`gpio_set_level()`（写电平）、`gpio_get_level()`（读电平）
  - Arduino：`pinMode()`（配置模式）、`digitalWrite()`（写）、`digitalRead()`（读）
- **ADC**：
  - ESP-IDF：`adc1_config_width()`（配置位宽）、`adc1_get_raw()`（读取原始值）
  - Arduino：`analogRead()`（直接读取，返回0-4095）

### 2. 无线通信
- **Wi-Fi**（STA模式）：
  - ESP-IDF：
    ```c
    #include "esp_wifi.h"
    void wifi_init_sta() {
        wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
        esp_wifi_init(&cfg);          // 初始化Wi-Fi
        esp_wifi_set_mode(WIFI_MODE_STA); // 设置STA模式
        wifi_config_t sta_cfg = {
            .sta = {.ssid = "MyWiFi", .password = "123456"}
        };
        esp_wifi_set_config(WIFI_IF_STA, &sta_cfg); // 配置SSID/密码
        esp_wifi_start();               // 启动Wi-Fi
        esp_wifi_connect();             // 连接AP
    }