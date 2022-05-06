# MCU概述

[toc]

## A.MCU实例

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
      >因为把APB的所有模块定义到always_on里面了
      >优化方法:对于M3而言,always_on只需要定义WIC和SWJ-DP就行了
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

1. 结构-包含什么模块
2. 指令集架构
3. 生态
4. RTOS(real time OS)
5. 编译器
6. 国内市场
   1. GigaDevice
      * 主打生态系统,能做到和ST的MCU pin to pin兼容
      * 价格低
   2. MM32 MCU
7. cortex-M
   1. lowest power
      2级流水
      * M0-v6
      * M0+-v6
      增加了TrustZone(以前在A系列,现在对应IOT市场),安全性可靠
      * M23-v8
   2. performance
      三级流水线,DSP,浮点,SIMD指令,neon指令(SIMD的扩展)
      * M3-v7-gatecount大概在50k
      * M4-v7
      增加了TrustZone,DSP,浮点
      * M33-v8
   3. high performance
      6-7级流水+AXI总线
      * M7-v7-gatecount大概在300k

## C.Cortex-M3项目介绍

1. 定义架构(内部总线,外设连接关系)
   1. 架构一
      * I-code连接bus matrix
   2. 架构二
      * I-code直接连接到MUX(BOOTROM&Flash)
      好处是:取指令快,latency减小
   3. 架构三
      * I和D经过MUX连到bus matrix
      * 不能同时取数据了,逻辑上会影响性能,但其实影响不大
   4. APB架构
      1. 架构一(扩展性强)
                               ->APB_CTRL_MUX->控制寄存器
         * AHB2APB->APB_SYS_MUX->APB-PERIP_MUX->外部串口
                               ->APB_ANA_MUX->ADC
      2. 架构二(APB_mux可能要重新修改)
         * AHB2APB->APB_MUX->外设
2. Paper Floorplan(前端工程师的预规划)
   1. 根据IO,定义模块布局
   2. 定义pads,规划phy
3. Power implementation
   1. Analog电压
   2. IO电压
   3. Digital CORE电压
   4. 定义power domain
      1. always_on
         1. WIC
            * 检测中断到来唤醒模块
         2. SWJ-DP
            * 有jtag debugger相连,外部会配置寄存器,然后告诉PMU
         3. PMU
            * 控制power的swtich(PLL的时钟都可以切换给alwayon domain用)
         4. RTC
      2. system_domain可关闭的
   5. 低功耗方案
      1. sleep-CPU时钟被门口关闭
      2. stop-时钟关闭
      3. standby-数字域电压关闭-可以降低泄露
      注意:register的值会消失,要进行retention的fip_flop(数据要换到lower-power电压域内,面积会增加很多昂)
      4. run-系统时钟降频,关闭未使用外设时钟
   6. 主要时钟域规划
      1. 逻辑部分-96M
      2. 低速时钟-48M
      3. RTC-1khz
   7. 时钟设计
      1. 分频电路
      2. mux电路
      3. DFT的控制(crg)
   8. 流程管理,前端工作很多啊
      1. 前端-综合
         1. sdc
         2. constraint
      2. 前端-DFT
         1. clock reset generator
         2. DFT方案
      3. 前端-后端
         1. timing signoff
         2. 路径确认
   9. 数据管理
      1. 清晰的目录结构
          1. logic
              1. RTL
              2. model
              3. ...
          2. ==validtion==
             1. work
             2. testbench
          3. ==implemation==
             1. data
             2. log
             3. report
             4. script
             5. work(跑工具)
          4. docs
      2. 版本管理
         1. SVN
         2. git
      3. 服务器操作
         1. 可以跟踪到别人的工作==怎么做到的==

## D.Designstart

里面有subsystem,SSE-050,就是一个样板工程

## E.如何*快速*实现MCU

## F.MCU项目改进和完善

## G.
