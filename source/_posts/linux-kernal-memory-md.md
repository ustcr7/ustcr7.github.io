---
title: 内存寻址 - 分段
date: 2019-02-20 00:45:23
tags: linux
---

CPU使用的是物理地址，而应用程序操作的则是逻辑地址，二者之间通过MMU来转换: 逻辑地址 --> [segmentation unit] --> 线性地址 --> [paging unit] --> 物理地址。本文主要描述80x86 CPU的内存分段以及linux的内存分段过程。
<!-- more -->

### 内存操作流程

<img src="/images/memory_cpu.png" alt="cpu操作内存流程" width="30%" height="30%">
地址总线用来指定内存的地址。地址线的根数决定了最大寻址空间。eg: 32根地址线，最大寻址范围是: 2^32 Bytes (内存的最小单元是字节)。
数据总线决定了cpu读写内存的速度。
控制总线用来决定对内存的操作类型(eg:读/写内存)。


### 8086 内存分段
8086 cpu的寄存器是16位，而内存地址总线是20位。为了充分使用1MB的寻址空间，使用 段地址 * 16 + 偏移地址 的方法来表示内存地址。
段地址存储在段寄存器里 (cs代码段 ds数据段 ss 堆栈寄存器 es附加寄存器)，例如cpu总是从 CS:IP (代码段寄存器:指令寄存器) 指向的内容读取指令。  
由于程序可以修改CS IP寄存器，因此不同程序之间的内存很容易被互相破坏。

### 80286 内存分段
80286 cpu新增了保护模式(protected mode)，改进了分段的实现方式。新增了几个概念：
>  *Segment Descriptor*:      描述段的相关数据，例如段的起始线性地址(Base)，地址区间最大长度(Limit)，段的访问特权级别(DPL)等。
>  *Global Descriptor Table*: GDT存储segment descriptor的全局table。
>  *Local Descriptor Table*:  LDT存储segment descriptor的local table(每个进程如有需要可以自行创建)。
>  *Segment Selector*:        逻辑地址的段标识符部分，用来从GDT/LDT中定位到具体的segment descriptor的。  

逻辑地址由两部分组成: segment selector + 偏移量。 其中segment selector 长度为16bit，存储在各个段寄存器中(段寄存器不直接存储各个段的起始地址)，其包含字段如下:  

<img src="/images/linux-kernal-memory-md/segment_selector_fields.png" alt="segment selector字段" width="50%" height="50%">  

TI 表示segment descriptor是在GDT还是LDT中。
index 表示segment descriptor 在GDT/LDT中的位置。
RPL 对应的segment selector 装入cs寄存器时用于表示CPU的执行特权级别。

逻辑地址到线性地址转换过程:
1 根据segment selector TI标记确定 segment descritor存储在GDT或LDT中
2 根据segment selector 的index字段 从 GDT/LDT中找到下标对应的segment descriptor
3 从segment descriptor 中获取段的起始地址
4 段起始地址 + 偏移地址 得到最终的线性地址

<img src="/images/linux-kernal-memory-md/segment_unit.png" alt="逻辑地址到线性地址转换过程" width="70%">  

应用程序可以修改段寄存器中的segment selector，但是无法修改GDT/LDT中的值(由操作系统负责修改) ，使用一个无效的逻辑地址可能会找不到合法的segment descriptor 从而产生segmentation fault错误。


### linux内存分段
linux为了可移植性 以非常有限的方式使用分段，所有的用户进程/内核进程共享相同的segment descriptor来进行指令和数据寻址。这组segment descriptor对应的segment selector用宏直接定义 __USER_CS, __USER_DS, __KERNEL_CS, __KERNEL_DS，且这些段的起始地址都是0(逻辑地址的偏移字段和线性地址是一致的)。CS，DS寄存器在CPU特权级别变化时会自动装载对应的segment selector。
linux的GDT主要包括了用户/内核的代码/指令4个段，3个线程局部存储段等。大多数linux程序不使用LDT，因此内核在GDT中定义了一个默认的LDT供多数进程共享。

### 参考文章
<< understanding the linux kernal >>
<< 汇编语言 - 王爽 >>
[8026与保护模式](https://zhuanlan.zhihu.com/p/27401519)
[CPU的运行级别和保护机制](https://zhuanlan.zhihu.com/p/55478625)
