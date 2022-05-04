
# CMSDK 开发 M3 核心笔记

[TOC]

## 1.定制化将产品从PCB到SOC

1. 成本
2. 可维护性
3. 尺寸
4. 节能,高效,电源管理方便
5. 可靠性高
6. 差异性提高,不太好抄板
7. IP保护,黑盒SOC

## 2.如何和ARM做定制化产品

参加Designstart,更好利用ARM的原生态

### i.for Soc/Asic

1. MCU M0,M3 没有授权费的
2. linux A5
3. 物理IP,自造要求高
   1. 基本Cell
   2. 基本库

ARM将其分为了两个版本  
获取网站  
designstart.arm.com

注意,可以参加ARM的研讨会

#### 评估版 Eval

不用签订任何合同,除了商业化一切的创新型工作都行

#### 商业版 pro

当评估完成后,采用流片前需要将其转为商业版

#### 配套参考模块

1. 总线
2. 电源管理
3. 外设

#### 简单流程

1. Soc 定义
   1. 外设
   2. 参数
   3. 指标
   4. ...
2. 挑选合适的IP
3. 设计整合
4. 验证
5. 实现

#### 技术支持

1. flow流程里面任何步骤都能找到ARM的支持
2. ARM合作的design house
3. EDA的简单获取

### ii.for FPGA

应用场景

1. M1,M3
2. 软IP,并应用与Xilinx
3. 全部免费,xilinx SDK中可以直接使用

#### FPGA项目嵌入CPU

软IP在vivado的设计套件里,在FPGA中需要处理器场景时非常快速且方便
  
主要目的还是降低流片成本

面对小规模应用是大大降低成本的,且比通用性MCU更加安全,毕竟FPGA是bit流

## 3.designstart提供了什么?

1. cortex核心
2. cmsdk ip工具
3. cmsdk样板工程

## 4.CMSDK

Cortex-M system design kit  
==目的:不要重复造轮子,几乎不用verilog去开发==

套件按包含了什么:

1. 总线IP
2. 关键外设ip
3. 样板工程
   1. 非常完善
   2. 不要魔改(不推荐)模块封装很乱
   3. 以样板工程为参考,自己搭建soc eg:
      1. 全局同步时钟,FPGA M3核心频率不高,导致多时钟域的必要性不是很大,全局就可以简化操作
      2. BRAM的readmemh/readmemb;要求体现ROM和RAM,BRAM自带ROM和RAM特性,这两个系统函数可以综合,keil都可以下到bram里
4. 编译仿真脚本
5. 软件驱动

### i.提供详细内容

套件路径:
Designstart/cmsdk/logic

1. APB
2. AHB_lite
3. memory_models
4. verification
5. ...

### ii.软件需求

1. 仿真modelsim
2. 综合
   1. xilinx:vivado
   2. intel:quartus
3. WSL  
windows下子linux
   1. make
   2. minicom 配合uart的linux环境的串口助手
4. keil软件调试

### iii.M3开发实例

#### a.cmsdk_ahb_busmatrix

1. 连接关系: core master -> matrix slaveport -> matrix masterport -> device salve
2. 定义:
   1. 矩阵的连接关系
   2. 决定存储的参数
      1. 稀疏,紧密
      2. 仲裁逻辑
         1. fixed 严格固定优先级,会打断brust传输
         2. brust 固定优先级,不会打断brust传输
         3. run   循环仲裁,上次传了下次优先级就低了
      3. ==...==
3. 实现:
   1. 进入cmsdk中xml文件夹
   2. example是arm提供的模板,pl是perl脚本-help运行可以看到xml使用方法
   3. global definitions
   4. 根据core master 设定矩阵的slaveinterface,有几个写几段
      1. slaveinterface的名字
      2. 该slaveinterface连接的所有的masterinterface的名字
      3. masterinterface分配对应的地址
         1. address_region:物理地址
         2. remap_region:  物理地址重映射,可以通过外部IO控制REMAP参数去控制硬件映射方式
   5. 定义device slave 的masterinterface的名字
   6. 完成makefile文件,在readme里面有说明
   7. 获得代码
4. 注意事项:
   1. 矩阵会在第一次仲裁时,会有一个延迟
   2. 矩阵在之后的传输,后面的路径都是组合路径,如果多个矩阵链接可能导致一条很长的路径,==FPGA的长路径会经过较多的MUX逻辑,这使得延迟可能会爆炸==;==ASIC的话长路径也会带来的电容,但应该可以在后端插入反相器链条,可能影响不会那么大==,既对FPGA而言布线长度比逻辑深度更重要,ASIC相反

#### b.cmsdk_ahb_to_sram

#### c.cmsdk_fpga_sram

魔改sram,可以直接改成系统函数读文件,这个是能综合的

#### d.cmsdk_ahb_to_apb

1. 只截取HADDR低16bit
2. APB只支持32bit传输
3. cmsdk_apb_slave_mux
   1. DECODE使用PADDR高4bit译码
   2. 生产16个选通信号
   3. 用低12位控制每个外设(000-fff)
   4. 绝对地址还是32位的

#### e.cmsdk_ahb_to_ahb_sync

1. 目的:解决总线矩阵组合逻辑通路问题,在多级矩阵中打断AHB的长组合逻辑,内部有寄存器会全部打拍
2. 问题:延迟会变长啊,核心搬数据可能要6个cycle
3. 解决方法:
   1. 使用DMA去完成大数据的搬运(buffer+状态机)
   2. 存储器放在高级的总线上

#### f.cmsdk_ahb_default_slave

#### g.cmsdk_ahb_master_mux

1. 多个master合并,在M3的权威指南上有说明
I-code 和 D-code 两个总线结构由master_mux合并,就不用去写矩阵了(现在矩阵也脚本化了,这个可能会比较少用)

#### h.cmsdk_ahb_slave_mux

1. 麻烦点,传统的ahb,ahb_lite控制多从机的方式
2. 相比矩阵不会有延迟了

### iiii.软件开发

#### a.软件驱动

1. cmsis header
定义了amsdk 的apb结构体,功能,指令
   1. core_cm3.h
   2. core_cmFunc.h
   3. core_cminstr.h
2. driver.h driver.c
定义了外设驱动的函数实现
3. startup_CM3DS.s  
启动代码
4. system.c  
 系统初始化
5. CortexM3.h  
软件工程头文件
6. mian.c
7. ...

#### b.软件移植流程

1. CortexM3.h 定义你的struct
   1. 有哪些驱动寄存器
   2. 寄存器的控制信息
   3. 外设基础地址
   4. 外设的结构体声明 链接 结构体和基地址
2. driver.h结构体放入,函数声明
3. driver.c 具体实现功能的函数可以移植sdk
4. system.c 系统时钟常数,时钟中断,函数初始化,初始化屏幕
5. start.c 启动代码 所有软件机器码的第一个是在keil设置的

#### c.开发外设

1. GPIO 处理器直接控制外设,如LCD(映射IO)
2. 指令型的,用状态机去控制buffer送数据,CPU可以去干别的事(帧缓存)
