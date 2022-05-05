# MCU概述

[toc]

## A.成果-MCU实例

1. 40nm工艺
   好像200-300M都能跑的到
2. ARM M3 MCU
   1. gate count 160k
      * 逻辑部分
   2. memory 17
      * 4*4 memoery + 1 rom
      * 前端大memory可以分割成小memory,方便布局布线
   3. ADC 1
   4. PLL 1
   5. always on power domain
      * The Always-On (AON) power domain is the first part of the circuit supplied at power-up
      * 大概有20k的逻辑,对于项目来说是大了 ==为什么==
3. 30 IO PADs & 16 PG PADs(PADS 其实指的是硅片的管脚)
   1. IO PADs:(处于芯片版图的外圈,其实就是GPIO吧?)
   定义:IO pad是一个芯片管脚处理模块，即可以将芯片管脚的信号经过处理送给芯片内部，又可以将芯片内部输出的信号经过处理送到芯片管脚。输入信号处理包含时钟信号，复位信号等，输出信号包含观察时钟、中断等。IO pad模块可以控制输入输出信号的电平、驱动电流等，同时还包含了检测功能
   2. PG PADs:
   ==定义:==
   3. 小于48脚采用QFP48封装
   4. 因为封装四周有脚,所以后端的Floorplan的分布的均匀些,所以前端和后端要多沟通
4. 100Mhz
   不需要太快的,MCU用的工艺都比较老,我们更看重低功耗
5. Die SIze
   1.6605mm^2
6. Physical DRC clean
   应该是指通过了版图的DRC设计规范吧

## B.MCU市场概述

## C.Cortex-M3项目介绍

## D.Designstart的使用

## E.如何*快速*实现MCU

## F.MCU项目改进和完善

## G.
