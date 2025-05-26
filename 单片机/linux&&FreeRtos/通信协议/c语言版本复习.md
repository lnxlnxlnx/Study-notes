
明白你的需求了！我整理了**C语言版本**的通信协议示例，直接给出完整代码（不拆文件），并重点标注关键API和开发细节。以下是六大协议的C语言实战代码，结合Linux系统特性讲解。


---

## 一、TCP通信（C语言）


### 1.1 核心API说明（C语言）
- `socket()`：创建套接字（`AF_INET` IPv4，`SOCK_STREAM` TCP）
- `bind()`：绑定本地IP+端口（服务端必须）
- `listen()`：开启监听（服务端）
- `accept()`：等待客户端连接（服务端，返回新套接字）
- `connect()`：客户端连接服务端
- `send()`/`recv()`：收发数据（基于新套接字）


### 1.2 示例代码：TCP服务端+客户端（C语言）
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

// TCP服务端函数
void tcp_server() {
    // 1. 创建TCP套接字
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (server_fd < 0) {
        perror("socket创建失败");
        exit(1);
    }

    // 2. 绑定IP和端口（允许地址重用）
    struct sockaddr_in server_addr = {
        .sin_family = AF_INET,
        .sin_port = htons(8888),      // 端口8888（网络字节序）
        .sin_addr.s_addr = INADDR_ANY  // 监听所有网卡
    };
    int opt = 1;
    setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt)); // 避免端口占用
    if (bind(server_fd, (struct sockaddr*)&server_addr, sizeof(server_addr)) < 0) {
        perror("bind失败");
        exit(1);
    }

    // 3. 开启监听（最多5个客户端排队）
    if (listen(server_fd, 5) < 0) {
        perror("listen失败");
        exit(1);
    }
    printf("TCP服务端启动，等待连接...\n");

    // 4. 接受客户端连接（阻塞）
    struct sockaddr_in client_addr;
    socklen_t client_len = sizeof(client_addr);
    int client_fd = accept(server_fd, (struct sockaddr*)&client_addr, &client_len);
    if (client_fd < 0) {
        perror("accept失败");
        exit(1);
    }
    printf("客户端 %s:%d 已连接\n", 
           inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));

    // 5. 收发数据
    char buf[1024] = {0};
    ssize_t recv_len = recv(client_fd, buf, sizeof(buf), 0);
    if (recv_len < 0) {
        perror("recv失败");
        close(client_fd);
        close(server_fd);
        return;
    }
    printf("收到数据：%s\n", buf);

    // 6. 回复数据
    const char* response = "已收到数据，TCP连接正常";
    send(client_fd, response, strlen(response), 0);

    // 7. 关闭连接
    close(client_fd);
    close(server_fd);
}

// TCP客户端函数
void tcp_client() {
    // 1. 创建TCP套接字
    int client_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (client_fd < 0) {
        perror("socket创建失败");
        exit(1);
    }

    // 2. 连接服务端
    struct sockaddr_in server_addr = {
        .sin_family = AF_INET,
        .sin_port = htons(8888),
        .sin_addr.s_addr = inet_addr("127.0.0.1") // 服务端IP
    };
    if (connect(client_fd, (struct sockaddr*)&server_addr, sizeof(server_addr)) < 0) {
        perror("connect失败");
        exit(1);
    }

    // 3. 发送数据
    const char* send_data = "你好，TCP服务端！";
    send(client_fd, send_data, strlen(send_data), 0);

    // 4. 接收回复
    char buf[1024] = {0};
    ssize_t recv_len = recv(client_fd, buf, sizeof(buf), 0);
    if (recv_len < 0) {
        perror("recv失败");
        close(client_fd);
        return;
    }
    printf("服务端回复：%s\n", buf);

    // 5. 关闭连接
    close(client_fd);
}

int main(int argc, char* argv[]) {
    if (argc != 2) {
        printf("用法：%s [server/client]\n", argv[0]);
        return 1;
    }
    if (strcmp(argv[1], "server") == 0) {
        tcp_server();
    } else if (strcmp(argv[1], "client") == 0) {
        tcp_client();
    } else {
        printf("参数错误\n");
    }
    return 0;
}
```

**编译运行**：  
`gcc tcp_demo.c -o tcp_demo`  
服务端：`./tcp_demo server`  
客户端：`./tcp_demo client`


### 1.3 开发对比与避坑
- **C语言 vs Python**：C语言需要手动处理字节序（`htons`/`ntohs`）、内存分配，错误处理更底层（`perror`），但性能更高（适合嵌入式实时场景）。
- **易混淆点**：`bind()`的`INADDR_ANY`表示监听所有网卡，若只想监听本地回环（`127.0.0.1`），需设置`sin_addr.s_addr = inet_addr("127.0.0.1")`。
- **端口占用**：服务端关闭后，端口会被系统保留一段时间（`TIME_WAIT`状态），用`SO_REUSEADDR`选项可跳过等待。


---

## 二、UDP通信（C语言）


### 2.1 核心API说明（C语言）
- `socket()`：`SOCK_DGRAM`指定UDP
- `bind()`：服务端绑定端口（客户端可选）
- `sendto()`：发送数据（需指定目标地址）
- `recvfrom()`：接收数据（返回发送方地址）


### 2.2 示例代码：UDP广播通信（C语言）
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

// UDP发送端（广播）
void udp_sender() {
    // 1. 创建UDP套接字
    int sock = socket(AF_INET, SOCK_DGRAM, 0);
    if (sock < 0) {
        perror("socket创建失败");
        exit(1);
    }

    // 2. 允许广播（关键！）
    int broadcast = 1;
    if (setsockopt(sock, SOL_SOCKET, SO_BROADCAST, &broadcast, sizeof(broadcast)) < 0) {
        perror("设置广播失败");
        exit(1);
    }

    // 3. 目标地址（广播地址255.255.255.255，端口8889）
    struct sockaddr_in broadcast_addr = {
        .sin_family = AF_INET,
        .sin_port = htons(8889),
        .sin_addr.s_addr = inet_addr("255.255.255.255")
    };

    // 4. 发送广播数据
    const char* send_data = "这是一条UDP广播消息！";
    ssize_t send_len = sendto(sock, send_data, strlen(send_data), 0,
                             (struct sockaddr*)&broadcast_addr, sizeof(broadcast_addr));
    if (send_len < 0) {
        perror("sendto失败");
        exit(1);
    }
    printf("UDP广播已发送\n");

    close(sock);
}

// UDP接收端
void udp_receiver() {
    // 1. 创建UDP套接字
    int sock = socket(AF_INET, SOCK_DGRAM, 0);
    if (sock < 0) {
        perror("socket创建失败");
        exit(1);
    }

    // 2. 绑定端口（接收广播必须绑定目标端口）
    struct sockaddr_in local_addr = {
        .sin_family = AF_INET,
        .sin_port = htons(8889),
        .sin_addr.s_addr = INADDR_ANY
    };
    if (bind(sock, (struct sockaddr*)&local_addr, sizeof(local_addr)) < 0) {
        perror("bind失败");
        exit(1);
    }
    printf("UDP接收端启动，等待广播...\n");

    // 3. 循环接收数据
    while (1) {
        char buf[1024] = {0};
        struct sockaddr_in sender_addr;
        socklen_t sender_len = sizeof(sender_addr);
        ssize_t recv_len = recvfrom(sock, buf, sizeof(buf), 0,
                                   (struct sockaddr*)&sender_addr, &sender_len);
        if (recv_len < 0) {
            perror("recvfrom失败");
            continue;
        }
        printf("收到来自 %s:%d 的广播：%s\n",
               inet_ntoa(sender_addr.sin_addr), ntohs(sender_addr.sin_port), buf);
    }
}

int main(int argc, char* argv[]) {
    if (argc != 2) {
        printf("用法：%s [sender/receiver]\n", argv[0]);
        return 1;
    }
    if (strcmp(argv[1], "sender") == 0) {
        udp_sender();
    } else if (strcmp(argv[1], "receiver") == 0) {
        udp_receiver();
    } else {
        printf("参数错误\n");
    }
    return 0;
}
```

**编译运行**：  
`gcc udp_demo.c -o udp_demo`  
发送端：`./udp_demo sender`  
接收端：`./udp_demo receiver`（需用`sudo`运行，否则可能无广播权限）


### 2.3 开发对比与避坑
- **C语言优势**：UDP数据报的边界性在C中更明显（`recvfrom`按包接收），而Python的`recvfrom`可能因缓冲区设置丢失边界。
- **广播权限**：Linux默认禁止普通用户发送广播，需用`sudo`运行发送端（或关闭防火墙）。
- **多网卡处理**：若设备有多个网卡，需用`setsockopt`指定发送广播的网卡（`SO_BINDTODEVICE`）。


---

## 三、串口通信（C语言）


### 3.1 核心API说明（C语言）
- `open()`：打开串口设备（`O_RDWR | O_NOCTTY`）
- `tcgetattr()`：获取当前串口配置（`struct termios`）
- `cfsetispeed()`/`cfsetospeed()`：设置输入/输出波特率（`B115200`等宏）
- `tcsetattr()`：应用配置（`TCSANOW`立即生效）
- `read()`/`write()`：收发数据


### 3.2 示例代码：串口读写（C语言）
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <termios.h>

int uart_init(const char* dev_path, speed_t baudrate) {
    // 1. 打开串口设备
    int fd = open(dev_path, O_RDWR | O_NOCTTY | O_NDELAY);
    if (fd < 0) {
        perror("打开串口失败");
        return -1;
    }

    // 2. 获取串口配置
    struct termios tio;
    if (tcgetattr(fd, &tio) < 0) {
        perror("tcgetattr失败");
        close(fd);
        return -1;
    }

    // 3. 配置波特率
    cfsetispeed(&tio, baudrate);  // 输入波特率
    cfsetospeed(&tio, baudrate);  // 输出波特率

    // 4. 配置数据位、停止位、校验位（8N1）
    tio.c_cflag &= ~CSIZE;   // 清除数据位掩码
    tio.c_cflag |= CS8;      // 8位数据位
    tio.c_cflag &= ~CSTOPB;  // 1位停止位
    tio.c_cflag &= ~PARENB;  // 无校验

    // 5. 禁用硬件流控制
    tio.c_cflag &= ~CRTSCTS;

    // 6. 启用接收和本地模式
    tio.c_cflag |= (CLOCAL | CREAD);

    // 7. 应用配置（立即生效）
    if (tcsetattr(fd, TCSANOW, &tio) < 0) {
        perror("tcsetattr失败");
        close(fd);
        return -1;
    }

    return fd;
}

int main() {
    const char* uart_dev = "/dev/ttyUSB0";  // 你的串口设备（如USB转串口）
    int uart_fd = uart_init(uart_dev, B115200);
    if (uart_fd < 0) {
        return 1;
    }
    printf("串口已初始化：%s\n", uart_dev);

    // 发送数据
    const char* send_data = "Hello UART!\r\n";
    ssize_t send_len = write(uart_fd, send_data, strlen(send_data));
    if (send_len < 0) {
        perror("write失败");
        close(uart_fd);
        return 1;
    }
    printf("已发送：%s", send_data);

    // 接收数据（等待1秒）
    char buf[1024] = {0};
    ssize_t recv_len = read(uart_fd, buf, sizeof(buf));
    if (recv_len < 0) {
        perror("read失败");
        close(uart_fd);
        return 1;
    } else if (recv_len == 0) {
        printf("未收到数据\n");
    } else {
        printf("收到数据：%s", buf);
    }

    close(uart_fd);
    return 0;
}
```

**编译运行**：  
`gcc uart_demo.c -o uart_demo`  
`./uart_demo`（需用`sudo`运行，否则可能无串口权限）


### 3.3 开发对比与避坑
- **C语言细节**：波特率用`B115200`等宏（定义在`termios.h`），需与实际设备一致（如蓝牙模块默认9600）。
- **阻塞模式**：默认`read()`是阻塞的（`O_NDELAY`设为非阻塞），可通过`tcsetattr`的`VMIN`和`VTIME`配置最小字节数和超时。
- **串口权限**：普通用户无`/dev/ttyUSB0`权限，需添加用户到`dialout`组：  
  `sudo usermod -aG dialout $USER`（重启生效）


---

## 四、SPI通信（C语言）


### 4.1 核心API说明（C语言）
- `open()`：打开SPI设备（`/dev/spidev<bus>.<cs>`）
- `ioctl()`：配置SPI模式（`SPI_IOC_WR_MODE`）、速率（`SPI_IOC_WR_MAX_SPEED_HZ`）
- `spi_ioc_transfer`：结构体用于全双工传输（`write()`和`read()`合并操作）


### 4.2 示例代码：SPI读写（C语言，以ADXL345为例）
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <linux/spi/spidev.h>

#define SPI_DEVICE "/dev/spidev0.0"  // SPI设备（总线0，片选0）

int spi_init(int* fd, uint8_t mode, uint32_t speed) {
    // 1. 打开SPI设备
    *fd = open(SPI_DEVICE, O_RDWR);
    if (*fd < 0) {
        perror("打开SPI设备失败");
        return -1;
    }

    // 2. 配置SPI模式（CPOL/CPHA）
    if (ioctl(*fd, SPI_IOC_WR_MODE, &mode) < 0) {
        perror("设置SPI模式失败");
        close(*fd);
        return -1;
    }

    // 3. 配置最大速率（Hz）
    if (ioctl(*fd, SPI_IOC_WR_MAX_SPEED_HZ, &speed) < 0) {
        perror("设置SPI速率失败");
        close(*fd);
        return -1;
    }

    return 0;
}

// SPI全双工传输函数
int spi_transfer(int fd, uint8_t* tx_buf, uint8_t* rx_buf, size_t len) {
    struct spi_ioc_transfer tr = {
        .tx_buf = (unsigned long)tx_buf,  // 发送缓冲区指针
        .rx_buf = (unsigned long)rx_buf,  // 接收缓冲区指针
        .len = len,                       // 数据长度（字节）
        .delay_usecs = 0,                 // 传输后延时
        .speed_hz = 5000000,              // 本次传输速率（可覆盖全局设置）
        .bits_per_word = 8,               // 8位/字
    };

    // 执行传输（一次ioctl完成收发）
    if (ioctl(fd, SPI_IOC_MESSAGE(1), &tr) < 0) {
        perror("SPI传输失败");
        return -1;
    }
    return 0;
}

int main() {
    int spi_fd;
    if (spi_init(&spi_fd, 0b11, 5000000) < 0) {  // 模式3（CPOL=1, CPHA=1），5MHz
        return 1;
    }

    // 读取ADXL345设备ID（寄存器0x00）
    uint8_t tx_buf[2] = {0x80 | 0x00, 0x00};  // 0x80表示读操作
    uint8_t rx_buf[2] = {0};
    if (spi_transfer(spi_fd, tx_buf, rx_buf, 2) < 0) {
        close(spi_fd);
        return 1;
    }
    printf("ADXL345设备ID：0x%02X（应为0xE5）\n", rx_buf[1]);

    close(spi_fd);
    return 0;
}
```

**编译运行**：  
`gcc spi_demo.c -o spi_demo`  
`./spi_demo`（需确保SPI设备已启用：`sudo modprobe spidev`）


### 4.3 开发对比与避坑
- **C语言优势**：`spi_ioc_transfer`结构体直接操作硬件，比Python的`spidev`更高效（适合高频传感器）。
- **SPI模式**：ADXL345用模式3（`0b11`），而有些传感器用模式0（`0b00`），需根据手册调整`mode`参数。
- **片选控制**：Linux下SPI设备的片选（`spidev0.0`中的`.0`）由驱动自动管理，无需手动控制GPIO。


---

## 五、I2C通信（C语言）


### 5.1 核心API说明（C语言）
- `open()`：打开I2C设备（`/dev/i2c-<bus>`）
- `ioctl()`：设置从机地址（`I2C_SLAVE`）
- `write()`：发送“寄存器地址+数据”（写操作）
- `read()`：读取从机返回的数据（读操作）


### 5.2 示例代码：I2C读写EEPROM（C语言，以AT24C02为例）
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <linux/i2c-dev.h>

#define I2C_DEVICE "/dev/i2c-1"  // I2C设备（总线1）
#define EEPROM_ADDR 0x50         // AT24C02的7位地址（A0-A2接地）

int i2c_write(int fd, uint8_t reg_addr, uint8_t data) {
    // 构造写命令：[寄存器地址, 数据]
    uint8_t buf[2] = {reg_addr, data};
    if (write(fd, buf, sizeof(buf)) != sizeof(buf)) {
        perror("I2C写失败");
        return -1;
    }
    usleep(5000);  // EEPROM写入需要5ms
    return 0;
}

uint8_t i2c_read(int fd, uint8_t reg_addr) {
    // 先写寄存器地址（告诉从机要读哪个寄存器）
    if (write(fd, &reg_addr, 1) != 1) {
        perror("I2C写寄存器地址失败");
        return 0xFF;
    }

    // 读取数据（从机返回该寄存器的值）
    uint8_t data;
    if (read(fd, &data, 1) != 1) {
        perror("I2C读数据失败");
        return 0xFF;
    }
    return data;
}

int main() {
    // 1. 打开I2C设备
    int fd = open(I2C_DEVICE, O_RDWR);
    if (fd < 0) {
        perror("打开I2C设备失败");
        return 1;
    }

    // 2. 设置从机地址（7位地址左移1位，最低位为0表示写）
    if (ioctl(fd, I2C_SLAVE, EEPROM_ADDR) < 0) {
        perror("设置I2C从机地址失败");
        close(fd);
        return 1;
    }

    // 3. 向地址0x01写入0xAA
    uint8_t write_addr = 0x01;
    uint8_t write_data = 0xAA;
    if (i2c_write(fd, write_addr, write_data) < 0) {
        close(fd);
        return 1;
    }
    printf("已写入：地址0x%02X，数据0x%02X\n", write_addr, write_data);

    // 4. 从地址0x01读取数据
    uint8_t read_data = i2c_read(fd, write_addr);
    if (read_data != 0xFF) {
        printf("读取到：地址0x%02X，数据0x%02X\n", write_addr, read_data);
    }

    close(fd);
    return 0;
}
```

**编译运行**：  
`gcc i2c_demo.c -o i2c_demo`  
`./i2c_demo`（需确保I2C设备已启用：`sudo modprobe i2c-dev`）


### 5.3 开发对比与避坑
- **C语言细节**：I2C写操作需先发送寄存器地址，再发送数据；读操作需先“虚写”寄存器地址，再读取数据（类似“指针定位”）。
- **地址混淆**：7位地址（如0x50）在`ioctl(I2C_SLAVE, addr)`中直接使用，驱动会自动处理读写位（写时最低位0，读时1）。
- **上拉电阻**：I2C必须接4.7kΩ上拉电阻（SCL/SDA到VCC），否则无法产生“高”电平。


---

## 六、CAN通信（C语言，socketcan）


### 6.1 核心API说明（C语言）
- `socket()`：创建CAN套接字（`PF_CAN`，`SOCK_RAW`）
- `bind()`：绑定CAN接口（如`can0`）
- `send()`/`recv()`：发送/接收`struct can_frame`结构体


### 6.2 示例代码：CAN总线收发（C语言，模拟汽车传感器）
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <sys/ioctl.h>
#include <net/if.h>
#include <linux/can.h>
#include <linux/can/raw.h>

int can_init(const char* can_interface) {
    // 1. 创建CAN套接字
    int sock = socket(PF_CAN, SOCK_RAW, CAN_RAW);
    if (sock < 0) {
        perror("socket创建失败");
        return -1;
    }

    // 2. 获取CAN接口索引
    struct ifreq ifr;
    strncpy(ifr.ifr_name, can_interface, IFNAMSIZ - 1);
    if (ioctl(sock, SIOCGIFINDEX, &ifr) < 0) {
        perror("获取CAN接口索引失败");
        close(sock);
        return -1;
    }

    // 3. 绑定CAN接口
    struct sockaddr_can addr = {
        .can_family = AF_CAN,
        .can_ifindex = ifr.ifr_ifindex
    };
    if (bind(sock, (struct sockaddr*)&addr, sizeof(addr)) < 0) {
        perror("bind失败");
        close(sock);
        return -1;
    }

    return sock;
}

int main(int argc, char* argv[]) {
    if (argc != 2) {
        printf("用法：%s [sender/receiver]\n", argv[0]);
        return 1;
    }

    const char* can_interface = "can0";
    int can_sock = can_init(can_interface);
    if (can_sock < 0) {
        return 1;
    }

    if (strcmp(argv[1], "sender") == 0) {
        // 发送CAN帧（发动机转速2500rpm）
        struct can_frame frame = {
            .can_id = 0x123,          // 11位标准ID
            .can_dlc = 2,             // 数据长度2字节
            .data = {0x09, 0xC4}      // 2500 = 0x09C4
        };
        if (send(can_sock, &frame, sizeof(frame), 0) != sizeof(frame)) {
            perror("send失败");
            close(can_sock);
            return 1;
        }
        printf("已发送CAN帧：ID=0x%03X，数据=[0x%02X, 0x%02X]\n",
               frame.can_id, frame.data[0], frame.data[1]);
    } else if (strcmp(argv[1], "receiver") == 0) {
        // 接收CAN帧
        printf("CAN接收端启动，等待数据...\n");
        while (1) {
            struct can_frame frame;
            ssize_t recv_len = recv(can_sock, &frame, sizeof(frame), 0);
            if (recv_len < 0) {
                perror("recv失败");
                continue;
            }
            if (frame.can_id == 0x123 && frame.can_dlc == 2) {
                uint16_t rpm = (frame.data[0] << 8) | frame.data[1];
                printf("收到发动机转速：%d rpm\n", rpm);
            }
        }
    }

    close(can_sock);
    return 0;
}
```

**编译运行**：  
`gcc can_demo.c -o can_demo`  
1. 启用CAN接口（需先配置位速率）：  
   `sudo ip link set can0 up type can bitrate 500000`  
2. 发送端：`./can_demo sender`  
3. 接收端：`./can_demo receiver`（另一个终端）


### 6.3 开发对比与避坑
- **C语言优势**：直接操作`struct can_frame`结构体，与硬件层交互更紧密（适合汽车电子的实时性要求）。
- **CAN接口启用**：Linux默认关闭CAN接口，需用`ip link`命令启用（`bitrate`需与硬件一致）。
- **扩展帧支持**：若使用29位ID，需设置`frame.can_id |= CAN_EFF_FLAG`，并调整`can_dlc`（数据长度0-8字节）。


---

## 总结
以上代码覆盖了嵌入式最常用的六大通信协议，均为**C语言+Linux系统**实现。动手调试时注意：
- 串口：用串口助手（如`minicom`）验证收发。
- SPI/I2C：用逻辑分析仪抓波形，确认时序是否正确。
- CAN：用`candump can0`工具监控总线（`sudo candump can0`）。

遇到问题时，优先看`perror`的输出（如“权限拒绝”“设备未找到”），再结合协议手册排查时序或配置错误。嵌入式通信的核心是“理解协议时序→验证硬件连接→调试软件逻辑”，多动手才能快速掌握！