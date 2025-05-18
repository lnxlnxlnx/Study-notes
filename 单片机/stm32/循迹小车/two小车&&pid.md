
以下是修正后的内容，主要解决了代码块符号冲突、公式格式问题及Markdown语法规范问题：


## 一、STM32 开发基础知识

### （一）STM32 的基本架构
- STM32 是基于 ARM Cortex-M 内核的 32 位微控制器。它采用哈佛架构，拥有独立的指令总线、数据总线和 I/O 总线，这种架构允许程序、数据和 I/O 的访问同时进行，显著提升处理器性能。
- 内部存储器包括 Flash 存储器和 SRAM。Flash 用于存储程序代码和固定数据，SRAM 用于存储运行时数据（如变量、堆栈等）。

### （二）STM32 的开发环境
- **硬件工具**：至少需要一块 STM32 开发板和一个下载器（如 ST - LINK）。
- **软件工具**：常用集成开发环境（IDE）为 STM32CubeIDE。它提供图形化代码配置工具，支持寄存器配置、时钟设置等操作，并可通过代码生成器自动生成初始化代码。


## 二、PID 控制器设计（以循迹小车的速度控制为例）

### （一）PID 控制原理
- **比例（P）控制**：比例项反映当前误差大小，误差越大，控制作用越强。计算公式为：  
  \[ P = K_p \times e(t) \]  
  其中 \(K_p\) 为比例系数，\(e(t)\) 为当前误差（期望值与实际值之差）。

- **积分（I）控制**：积分项反映误差累积，用于消除系统静差。计算公式为：  
  \[ I = K_i \times \int e(t)dt \]  
  \(K_i\) 为积分系数。

- **微分（D）控制**：微分项反映误差变化率，可预测系统趋势，提升响应速度和稳定性。计算公式为：  
  \[ D = K_d \times \frac{de(t)}{dt} \]  
  \(K_d\) 为微分系数。

### （二）PID 控制器的实现
1. **初始化 PID 参数**  
   - 根据经验设定初始 \(K_p\)、\(K_i\)、\(K_d\) 值。例如，简单速度控制系统可初设 \(K_p = 1.0\)、\(K_i = 0.1\)、\(K_d = 0.01\)。  
   - 定义变量保存误差累积值（积分项）和误差变化率（微分项），示例代码：  
     ```c
     float error_sum = 0.0; // 误差累积
     float error_prev = 0.0; // 上一次的误差
     ```

2. **计算 PID 输出**  
   - 在控制循环中计算当前误差 \(e(t)\)。例如，目标速度为 `target_speed`，实际速度为 `actual_speed`，则：  
     \[ e(t) = \text{target\_speed} - \text{actual\_speed} \]  
   - 更新误差累积值：  
     ```c
     error_sum += e(t) * dt; // dt 为采样时间间隔
     ```  
   - 计算误差变化率：  
     ```c
     float error_rate = (e(t) - error_prev) / dt;
     ```  
   - 计算 PID 输出：  
     ```c
     float pid_output = K_p * e(t) + K_i * error_sum + K_d * error_rate;
     ```  
   - 更新误差变量：  
     ```c
     error_prev = e(t);
     ```

3. **将 PID 输出应用于执行器**  
   - 将 `pid_output` 转换为执行器控制信号。例如，电机速度控制中，可将其作为 PWM 占空比调整值：正数增加占空比（提转速），负数减少占空比（降转速）。

### （三）PID 参数调试
- **逐步调整法**：先调比例参数 \(K_p\)，逐渐增大至系统振荡，再减小至振荡消失；接着增加积分参数 \(K_i\) 减小稳态误差；最后调微分参数 \(K_d\) 减小超调量、改善稳定性。  
- **Ziegler-Nichols 法**：先找系统临界增益 \(K_u\)（使系统等幅振荡的比例增益）和振荡周期 \(T_u\)，再按经验公式计算：  
  \[ K_p = 0.6K_u, \quad K_i = \frac{1.2K_u}{T_u}, \quad K_d = 0.075K_uT_u \]


## 三、卡尔曼滤波器设计（以加速度传感器数据滤波为例）

### （一）卡尔曼滤波原理
- **系统模型**：基于线性动态系统，状态方程为：  
  \[ \mathbf{x}_k = \mathbf{F}_k \mathbf{x}_{k-1} + \mathbf{B}_k \mathbf{u}_k + \mathbf{w}_k \]  
  其中 \(\mathbf{x}_k\) 为状态向量，\(\mathbf{F}_k\) 为状态转移矩阵，\(\mathbf{B}_k\) 为控制输入矩阵，\(\mathbf{u}_k\) 为控制输入向量，\(\mathbf{w}_k\) 为过程噪声。

- **观测模型**：观测值与状态关系由观测方程表示：  
  \[ \mathbf{z}_k = \mathbf{H}_k \mathbf{x}_k + \mathbf{v}_k \]  
  其中 \(\mathbf{z}_k\) 为观测向量，\(\mathbf{H}_k\) 为观测矩阵，\(\mathbf{v}_k\) 为观测噪声。

### （二）卡尔曼滤波器的实现
1. **初始化滤波器状态**  
   - 初始化状态向量 \(\mathbf{x}_0\)（基于先验知识或初始测量值，如加速度传感器首次测量值）。  
   - 初始化状态协方差矩阵 \(\mathbf{P}_0\)（表示状态估计不确定性，初始可设较大值），示例代码：  
     ```c
     float P[2][2] = {{100.0, 0.0}, {0.0, 100.0}}; // 二维状态协方差矩阵
     ```

2. **预测阶段**  
   - 根据状态方程预测下一时刻状态，示例（匀速运动模型）：  
     ```c
     float F[2][2] = {{1.0, dt}, {0.0, 1.0}}; // dt 为时间步长
     float x_pred[2] = {x_prev[0] + x_prev[1] * dt, x_prev[1]}; // 状态预测
     ```  
   - 预测协方差矩阵：  
     ```c
     float Q[2][2] = {{0.01*dt*dt, 0.0}, {0.0, 0.01*dt}}; // 过程噪声协方差
     float P_pred[2][2];
     // 矩阵运算：P_pred = F * P_prev * F^T + Q
     ```

3. **更新阶段**  
   - 计算卡尔曼增益（假设观测矩阵 \(\mathbf{H}\) 为单位矩阵）：  
     ```c
     float H[2][2] = {{1.0, 0.0}, {0.0, 1.0}}; // 观测矩阵
     float R[2][2] = {{0.1, 0.0}, {0.0, 0.1}}; // 观测噪声协方差
     // 矩阵运算：K = P_pred * H^T * (H * P_pred * H^T + R)^(-1)
     ```  
   - 更新状态估计：  
     ```c
     float z[2] = {meas_accel, meas_velocity}; // 观测值
     float y[2] = {z[0] - x_pred[0], z[1] - x_pred[1]}; // 观测残差
     float x_update[2] = {x_pred[0] + K[0][0]*y[0] + K[0][1]*y[1],
                          x_pred[1] + K[1][0]*y[0] + K[1][1]*y[1]}; // 状态更新
     ```  
   - 更新协方差矩阵：  
     ```c
     // 矩阵运算：P = (I - K * H) * P_pred
     ```


## 四、常用 STM32 API 函数（基于 HAL 库）

### （一）GPIO 操作
1. **初始化 GPIO 引脚**  
   ```c
   GPIO_InitTypeDef GPIO_InitStruct = {0};
   GPIO_InitStruct.Pin = GPIO_PIN_0;          // 配置引脚 0
   GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP; // 推挽输出模式
   GPIO_InitStruct.Pull = GPIO_NOPULL;         // 无上拉/下拉
   GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW; // 低速
   HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);      // 初始化 GPIOA 引脚 0
   ```

2. **设置 GPIO 引脚输出电平**  
   ```c
   HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_SET); // 高电平
   HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_RESET); // 低电平
   ```

3. **读取 GPIO 引脚输入电平**  
   ```c
   GPIO_PinState state = HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0); // 返回 GPIO_PIN_SET 或 GPIO_PIN_RESET
   ```

### （二）ADC 操作
1. **初始化 ADC**  
   ```c
   ADC_HandleTypeDef hadc1;
   hadc1.Instance = ADC1;
   hadc1.Init.ClockPrescaler = ADC_CLOCK_SYNC_PCLK_DIV2; // 时钟分频
   hadc1.Init.Resolution = ADC_RESOLUTION_12B;           // 12 位分辨率
   hadc1.Init.ScanConvMode = DISABLE;                    // 禁用扫描模式
   hadc1.Init.ContinuousConvMode = DISABLE;              // 禁用连续转换
   hadc1.Init.ExternalTrigConv = ADC_SOFTWARE_START;     // 软件触发
   hadc1.Init.DataAlign = ADC_DATAALIGN_RIGHT;           // 右对齐
   HAL_ADC_Init(&hadc1);
   ```

2. **启动 ADC 转换并获取值**  
   ```c
   HAL_ADC_Start(&hadc1);             // 启动转换
   HAL_ADC_PollForConversion(&hadc1, HAL_MAX_DELAY); // 等待完成
   uint16_t adc_value = HAL_ADC_GetValue(&hadc1);     // 获取结果（0-4095）
   ```

### （三）TIM 操作（用于 PWM 信号生成）
1. **初始化 TIM 用于 PWM**  
   ```c
   TIM_HandleTypeDef htim2;
   htim2.Instance = TIM2;
   htim2.Init.Prescaler = 0;                 // 预分频器（不分频）
   htim2.Init.CounterMode = TIM_COUNTERMODE_UP; // 向上计数模式
   htim2.Init.Period = 1000 - 1;             // 自动重装载值（决定频率）
   htim2.Init.AutoReloadPreload = TIM_AUTORELOAD_PRELOAD_ENABLE;
   HAL_TIM_PWM_Init(&htim2);
   ```

2. **配置 PWM 通道**  
   ```c
   TIM_OC_InitTypeDef sConfigOC = {0};
   sConfigOC.OCMode = TIM_OCMODE_PWM1;       // PWM 模式 1
   sConfigOC.Pulse = 500;                    // 初始占空比（50%）
   sConfigOC.OCPolarity = TIM_OCPOLARITY_HIGH; // 高电平有效
   HAL_TIM_PWM_ConfigChannel(&htim2, &sConfigOC, TIM_CHANNEL_1); // 配置通道 1
   ```

3. **启动 PWM 信号输出**  
   ```c
   HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_1); // 启动通道 1
   __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, new_pulse); // 更新占空比
   ```

### （四）串口通信（用于与上位机通信）
1. **初始化 UART**  
   ```c
   UART_HandleTypeDef huart1;
   huart1.Instance = USART1;
   huart1.Init.BaudRate = 115200;           // 波特率
   huart1.Init.WordLength = UART_WORDLENGTH_8B; // 8 位数据
   huart1.Init.StopBits = UART_STOPBITS_1;   // 1 位停止位
   huart1.Init.Parity = UART_PARITY_NONE;     // 无校验
   huart1.Init.Mode = UART_MODE_TX_RX;       // 收发模式
   HAL_UART_Init(&huart1);
   ```

2. **发送数据**  
   ```c
   uint8_t tx_buffer[] = "Hello STM32!";
   HAL_UART_Transmit(&huart1, tx_buffer, sizeof(tx_buffer)-1, HAL_MAX_DELAY);
   ```

3. **接收数据**  
   ```c
   uint8_t rx_data;
   HAL_UART_Receive(&huart1, &rx_data, 1, HAL_MAX_DELAY); // 接收单个字节
   ```


**修正说明**：  
1. 所有代码块统一使用 ```c 包裹，避免与 Markdown 符号冲突。  
2. 数学公式使用 \[ \] 包裹，确保公式正确渲染。  
3. 修正了部分代码中的标点符号全角/半角问题（如中文引号改为英文引号）。  
4. 调整了代码缩进，增强可读性。