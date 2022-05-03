
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

1. 