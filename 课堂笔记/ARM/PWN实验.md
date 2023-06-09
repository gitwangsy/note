# PWM

## 1. PWM 简介

PWM（Pulse Width Modulation）即脉宽调制，是一种通过改变脉冲的宽度来控制电路的一种技术。PWM 信号的占空比是可以调节的，**占空比越大，输出电压越高**，反之亦然。PWM 信号的频率是固定的，一般为几十 KHz。

## 2. PWM 的应用

PWM 信号可以用来控制电机的转速，控制 LED 的亮度，控制电压的大小等等。

## 3. PWM 的实现

### 3.1. 用定时器实现 PWM

定时器的工作原理是：定时器的计数器会不断地累加，当计数器的值达到设定的值时，定时器会产生一个中断，然后计数器会被清零，重新开始计数。定时器的计数器的值可以用来控制 PWM 信号的占空比。

定时器的计数器的值越大，PWM 信号的占空比越大，输出电压越高，反之亦然。

### 3.2. 用比较器实现 PWM

比较器的工作原理是：比较器会不断地比较定时器的计数器的值和比较器的值，当定时器的计数器的值小于比较器的值时，比较器会输出高电平，当定时器的计数器的值大于比较器的值时，比较器会输出低电平。比较器的值可以用来控制 PWM 信号的占空比。

比较器的值越大，PWM 信号的占空比越大，输出电压越高，反之亦然。

## 4. PWM 的实验

### 4.1. 实验原理

本实验使用定时器实现 PWM，定时器的计数器的值越大，PWM 信号的占空比越大，输出电压越高，反之亦然。

### 4.2. 实验代码

```c
#include "stm32f10x.h"

void RCC_Configuration(void);
void GPIO_Configuration(void);
void TIM_Configuration(void);

int main(void)
{
 RCC_Configuration();
 GPIO_Configuration();
 TIM_Configuration();

 while (1)
 {
 }
}

void RCC_Configuration(void)
{
 RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
 RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);
}

void GPIO_Configuration(void)
{
 GPIO_InitTypeDef GPIO_InitStructure;

 GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
 GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
 GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
 GPIO_Init(GPIOA, &GPIO_InitStructure);
}

void TIM_Configuration(void)
{
 TIM_TimeBaseInitTypeDef TIM_TimeBaseStructure;
 TIM_OCInitTypeDef TIM_OCInitStructure;

 TIM_TimeBaseStructure.TIM_Period = 1000 - 1;
 TIM_TimeBaseStructure.TIM_Prescaler = 72 - 1;
 TIM_TimeBaseStructure.TIM_ClockDivision = TIM_CKD_DIV1;
 TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;
 TIM_TimeBaseInit(TIM2, &TIM_TimeBaseStructure);

 TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;
 TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable;
 TIM_OCInitStructure.TIM_Pulse = 500 - 1;
 TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;
 TIM_OC1Init(TIM2, &TIM_OCInitStructure);

 TIM_OC1PreloadConfig(TIM2, TIM_OCPreload_Enable);

 TIM_ARRPreloadConfig(TIM2, ENABLE);

 TIM_Cmd(TIM2, ENABLE);
}
```
