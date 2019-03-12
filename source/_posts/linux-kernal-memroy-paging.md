---
title: 内存管理 - 分页
date: 2019-03-12 00:07:18
tags: linux
---

分页单元 负责将线性地址转换成物理地址。线性地址以页为单位，将页内连续的线性地址转换成 连续的物理地址。
<!-- more -->

### 硬件常规分页过程
<img src="/images/kernal-memory-paging.png" alt="常规分页过程" width="70%">
进程根据cr3寄存器找到 页目录表的物理内存位置，然后根据directory字段从页目录表中找到对应的条目，根据field字段找到对应的页表物理内存位置，再根据table字段找到页表中对应的条目，根据field字段，找到实际叶框的物理地址，加上offset字段后得到实际的物理地址。

### 分页保护策略
页表/页目录条目中的 特权标志位为0时，只允许cpu处于内核态时才能访问(cs寄存器的CPL字段小于3)。
read/write标志等于0，则表示页表/页是只读的。

### 加速策略

硬件高速缓存
为了缩小CPU和RAM之间的处理速度差距，引入硬件高速缓存内存(hardware cache memory)。每个cache line由几十个连续字节组成。
CPU访问RAM时会将物理地址和cache line进行比较，如果匹配则直接操作高速缓存(cache hit)。
读取：直接从cache中获取。 
写入： write-through模式既写cache，也写RAM。 write-back模式只写cache（后续刷到RAM中）。

TLB
Translation Lookaside Buffer，用于缓存线性地址转换得到的物理地址，每个CPU都有一个本地TLB，当CPU的cr3寄存器(指向页目录表)改变时，TLB自动失效。

### Linux分页
四级分页机制：Page Global Directory, Page Upper Directory, Page Middle Directory, Page Table。
未使用物理地址扩展的32位系统：Upper, Middle 位均为0
启用了物理地址扩展的32位系统：Upper 取消
64位系统：三级/四级地址划分。

linux分页的特点：
可以为不同进程设置不同的物理地址，避免寻址异常。
物理page frame中的数据可以保存到磁盘中，然后再加载到其他的page frame，而该过程对线性地址透明（虚拟内存实现基础）。

### 参考文章
<< understanding the linux kernal >>
