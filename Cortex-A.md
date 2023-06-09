# Cortex-A

## ARM

### 1. ARM 的含义

ARM 一般有两个含义：

- ARM 公司，不生产 CPU，只进行 CPU 的框架设计。
- ARM 架构，ARMv8-A(Cortex-A)，ARMv8-R(Cortex-R)，ARMv8-M(Cortex-M)。

### 2. 什么是裸机编程

裸机编程就是在没有操作系统的情况下，直接对硬件进行编程，可以是汇编，也可以是 C 语言。
bootloader 是一个裸机程序。
在 ubuntu 上运行的程序不是裸机程序，因为 ubuntu 是一个操作系统。

> bootloader 是用来加载操作系统的，它是一个裸机程序，它的作用是加载操作系统，然后把控制权交给操作系统。

### 3. ARM 内核的发展

- ARMv1 - ARMv3：32 位内核，不支持 Thumb 指令集。
- ARMv4：32 位内核，支持 Thumb 指令集。
- ARMv7：32 位内核，支持 Thumb-2 指令集。
- ARMv8：64 位内核，支持 Thumb-2 指令集。

在 ARMv7 之前所对应的 CPU 核心名称为 ARM7、ARM9、ARM11 等，而在 ARMv7 之后，ARM 公司将 CPU 核心名称统一为 Cortex，如 Cortex-A、Cortex-R、Cortex-M 等。
A 系列：面向应用的处理器，如手机、平板电脑等。
R 系列：面向实时应用的处理器，如汽车电子、工业控制等。
M 系列：面向嵌入式应用的处理器，如智能家电、智能穿戴等。

### 4. 指令集、CPU 核心、CPU、SoC 的区别

- 指令集通常是指 ARMv7 指令、ARMv8 指令等这些指令集架构，这部分定义了整体的架构，例如定义了 ARM 的工作模式、中断的处理方式、寄存器的定义等。
- CPU 核心是指令集的具体实现，例如 ARMv7 指令集的 CPU 核心有 Cortex-A7、Cortex-A8、Cortex-A9 等，ARMv8 指令集的 CPU 核心有 Cortex-A53、Cortex-A57 等。
- 一个 CPU 中可以包含多个 CPU 核心，例如 RK3288 芯片中就包含了 4 个 Cortex-A17 核心。STM32MP157A 芯片中包含了 2 个 Cortex-A7 核心和 1 个 Cortex-M4 核心。
- SoC 是 System on Chip 的缩写，是指一个芯片内部除了有 CPU 之外，还集成了很多控制器单元(外设)，例如 USB 控制器、LCD 控制器、SD 控制器、UART 控制器、NAND Flash 控制器等，**这些控制器单元可以通过总线连接到 CPU 核心，CPU 核心可以通过总线访问这些控制器单元，从而实现对外设的控制。**例如 STM32MP157A 芯片就是一个 SoC。

> 单片机(MCU(Micro Controller Unit))：单片机是指只有一个 SoC 的嵌入式系统。
> 开发板(Development Board)：开发板是指包含了单片机芯片以及其他外设的嵌入式系统。

### 5. STM32MP157 系列

STM32MP157 有 A、C、D、F 四个系列，我们开发板使用的型号是 STM32MP157AAA3，该 CPU 有两个 Cortex-A7 核心，主频 800MHz，一个 Cortex-M4 核心，主频 209MHz。
在我们学习一款芯片之前需要在该厂商的官网的官网上下载该芯片的数据手册、参考手册、编程手册和勘误手册。

数据手册：该芯片的硬件资料，包含了该芯片的管脚定义、外设的寄存器定义、时序图等。软件工程师一般会用它来查管脚。
参考手册：主要是对数据手册的细化，会介绍每个寄存器的使用，与软件开发关系密切。
编程手册：主要讲述和芯片相关的体系结构的一些指令，例如 ARMv7 指令集、ARMv8 指令集等。
勘误手册：主要讲述该芯片目前已知的一些问题，用于避错。

> 主频：CPU 的工作频率，单位是 MHz，表示 CPU 每秒钟可以执行的指令条数。

## FS-MP1A
