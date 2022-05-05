# 使用CMSDK--BUS

## A. AMBA Trend

服务于ARM处理器的SOC系统

1. AMBA1
   1. ASB1
   2. APB1
2. AMBA2
   1. ASB2
   2. APB2
   3. AHB(被淘汰了)
3. AMBA3
   1. APB3
      * 解决了slave不能及时响应
   2. AHB_lite
      * 节省面积,方便收敛,针对单master
      * 还可以升级成bus Matrix,可32/64传输,最多16*16
      * 仲裁实质是做到了masterinterface上了
   3. ATB1
      * 专门做trace用的,用于追踪
   4. AXI3
      * 解决了AHB带宽不够的问题
4. AMBA4
   1. APB4
   新增了了protect信号,提供了memory属性信号;non-security or security等
   2. AXI4
   改进,删了一些,加了一些
   3. AXI4_lite
   节省面积,在AXI4基础上
   4. AXI4_stream
   数据流单向传输,主要是视频传输
   5. ACE
   AXI4的基础上加上了cache的一致性的协议,满足多核要求
   6. ACE_lite
   7. ATB1.1
5. AMBA5
   1. AHB5
   保证安全
   2. CHI
   数据的传输都是打包式的,都是节点发送的,非常高端的总线

## B.NIC-400block(对数据的吞吐量要求高)

本质是AXI的bus matrix

### a.功能

1. 主要是支持AXI,首先延迟低,其次吞吐量大.
2. 有很多NIC-400 switch去构成
   内部有很多映射地址,总线寄存器ASIB,跨时钟同步,总线桥,和缓冲FIFO可以提供配置
3. A7,GPU,DDR都是AXI到ACE结构的
4. 配置软件:
   1. AMBA Designer-gui(旧)
   2. Socrates-gui(新)

### b.key issus of AXi Bus Matrix

* Deadlock(死锁)
  * AXI是基于握手的协议,但他也支持乱序传输
  * 想要的反馈信号被打乱在后面,导致协议无法进行
* QoS(Quality of Service)
  * matrix上master和slave的性能需求
    * 尽可能减小CPU的latency
    * 尽可能增加GPU等的带宽
    * 尽可能满足实时主机的latency
  * 影响QoS优先级的方法
    * 静态RTL调整master端口
    * 可编程寄存器调整优先级
    * AXI4支持AxQoS信号,支持master发送提高优先级的信号
  * 如何保证QoS
    * 发送率的控制,控制数据量和带宽,TSPEC协议
    * outstanding transaction regulation(同时发出几个未完成的传输,提高带宽)
    * 调整发起到结束的latency
    * 调整前面一个发起之后到下一个发起的latency
  * QoS在哪
    * 矩阵内部的调整控制器中,都可以看到QoS regular的影子
  * 优先级分类
    当不满足时可以实时调节,拉高不满足的master
    1. 15-Hard real-time(escalated)还不太清晰
    2. 12-14 延迟敏感
    3. 8-9 实时
    4. 5-7 软实时
    5. 0-4 带宽敏感
  * QoS的问题
    * ==比如访问DDR有个QoS缓存,如果QoS缓存满了,哪怕优先级再高,也访问不到需要的地方==
* Routing(across bus)
  * 信号线太多,结构太多,导致性能下降

## C.Mater Types

1. 延迟敏感主机
   随latency增加性能逐渐降低
   1. 流水线CPU对memory的访问
   会影响load/store指令,导致数据相关冲突
2. 带宽敏感主机
   随latency增加,超过截止时间,性能逐渐降低
   1. DMA
   2. GPU
3. 实时主机
   随latency增加,有截止时间,在截止时间内没啥事,超过截止时间了,性能就会损失很大
   1. LCD display controller
   2. HDMI display controller
