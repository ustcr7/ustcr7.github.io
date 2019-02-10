---
title: linux 网络工具 - iftop
date: 2019-01-28 01:00:38
tags: devops
---

### 一、iftop简介  
iftop 是linux实时网络带宽监控工具，可以有效统计linux机器的网络带宽负载情况。
常见使用场景：找出机器上的异常流量来源/监控机器的网络负载变化情况。

<!-- more -->

### 二、iftop使用示例  
iftop结果示例:
![iftop-w50](/images/iftop.png)
* 首部:刻度行  
表示速率柱状图的刻度。

* 中间部分:展示host之间的带宽速率  
带宽速率分in/out两个方向，右侧三列分表表示最近 2s/10s/40s内的平均带宽速率 (bits/sec).
例如图中svr1 到 svr2 之间的带宽速率是:
> svr1 --> svr2 近2s/10s/40s平均带宽速率分表是 4.94Mb/2.65Mb/1.05Mb。
> svr2 --> svr1 近2s/10s/40s平均带宽速率分表是 0.99Gb/453Mb/197Mb。  

白色柱状横条表示10s内平均带宽速率的最大值，其数值可以观察面板顶部的刻度行来获得。例如图中的数值大概为430 - 450 Mb左右。

* 底部:整体统计数据  
TX:发送带宽, RX:接收带宽, Total:总带宽
cum: iftop运行至当前使用的带宽总量(单位:Bytes)
peak: 从iftop运作至当前的带宽流量峰值
rates: 近 2s/10s/40s 的带宽速率均值
    
### 三、iftop常用参数:  
-n 地址显示为ip字符串，而不是hostname
-P 显示端口
-N 端口数显示为数字,而不是service name
-B 带宽速率为 bytes/秒, 而不是 bits/秒
