# 串口实验

## 1. 总线

总线是一种特殊的通信线路，用于连接多个设备，使得多个设备之间可以相互通信，总线的特点是：

1. 通信双方共享同一条总线，即多个设备共用一条总线。
2. 总线上的设备可以通过地址来区分，即每个设备都有一个地址。
3. 总线上的设备可以通过数据来通信，即每个设备都可以发送数据和接收数据。

总线的分类：

1. 串行总线：一次只能传输一个 bit，如 I2C、SPI、UART。
2. 并行总线：一次可以传输多个 bit，如 8 位总线、16 位总线、32 位总线。

系统总线：用于连接 CPU 和外设，如 AHB、APB。

## 2. 串口

串口是一种串行总线，用于连接 CPU 和外设，串口的特点是：

1. 串口是一种异步通信方式，即发送方和接收方的时钟不同步。
2. 串口是一种双线制，即发送方和接收方分别有一根数据线和一根时钟线。
3. 串口是一种全双工通信方式，即发送方和接收方可以同时发送和接收数据。

串口的分类：

1. 串行通信接口：用于连接 CPU 和外设，如 UART、USART、SPI、I2C。

## 3. UART

UART 是一种串行通信接口，用于连接 CPU 和外设。

### UART 寄存器

UART 寄存器分为两类：

1. 控制寄存器：用于配置 UART 的工作模式，如波特率、数据位、停止位、校验位等。
2. 数据寄存器：用于发送和接收数据。

### UART 工作模式

UART 有两种工作模式：

1. 8 位数据位、1 位停止位、无校验位。
2. 8 位数据位、1 位停止位、1 位校验位。

### UART 通信流程

UART 通信流程如下：

1. 配置 UART 工作模式。
2. 配置 UART 波特率。
3. 配置 UART 数据位。
4. 配置 UART 停止位。
5. 配置 UART 校验位。

## 4. 串口实验

### 4.1. 实验一

实验一：使用串口发送数据。

实验原理：配置 UART 工作模式、波特率、数据位、停止位、校验位，然后发送数据。

```c
#include "stm32f10x.h"

void UART1_Init(u32 pclk2, u32 bound) {
    float temp;
    u16 mantissa;
    u16 fraction;
    temp = (float)(pclk2 * 1000000) / (bound * 16);
    mantissa = temp;
    fraction = (temp - mantissa) * 16;
    mantissa <<= 4;
    mantissa += fraction;
    RCC->APB2ENR |= 1 << 2;
    RCC->APB2ENR |= 1 << 14;
    GPIOA->CRH &= 0xFFFFF00F;
    GPIOA->CRH |= 0x000008B0;
    RCC->APB2RSTR |= 1 << 14;
    RCC->APB2RSTR &= ~(1 << 14);
    USART1->BRR = mantissa;
    USART1->CR1 |= 0x200C;
}

void UART1_Send(u8 data) {
    USART1->DR = (data & (u16)0x01FF);
    while (!(USART1->SR & 0x40));
}

void UART1_Send_String(u8* buf) {
    while ((*buf) != '\0') {
        UART1_Send(*buf);
        buf++;
    }
}

int main(void) {
    UART1_Init(72, 115200);
    while (1) {
        UART1_Send_String("Hello World!\r\n");
        for (int i = 0; i < 10000000; i++);
    }
}
```
