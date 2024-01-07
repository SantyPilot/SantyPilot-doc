# 软件架构

## 嵌入式层

### STM32驱动与微内核

采用FreeRTOS微内核，支持多种外设STM32驱动，   
自定义新增传感器相对简单

`flight/pios`[文件夹](https://github.com/SantyPilot/SantyPilot/tree/master/flight/pios)
```
.
├── pios.h
├── inc
├── osx
├── common/ 通用设备驱动
|   ├── libraries 文件系统，微内核
|   |── pios_sensors.c 传感器封装
|   └── ...
|   
└── stm32f4xx/ 型号特定驱动
    ├── link_STM32F4xx_RM_fw_memory.ld 链接脚本指定内存布局
    |── libraries CMSIS, USB库
    └── pios_i2c.c 协议封装
```

`flight/targets/boards/revolution`[文件夹](https://github.com/SantyPilot/SantyPilot/tree/master/flight/targets/boards/revolution)
```
.
├── boad_hw_defs.c 硬件引脚配置
├── firmware/ 
    ├── revolution.cpp 入口主函数
    |── pios_board.c   驱动初始化
    └── inc/
        |── pios_config.h 驱动剪裁宏 外设选择 
```

## 应用层

### 通信机制

uavobject是类似于uORB的轻量级通信机制，而与PX4不同的是，    
uavobject也用于飞控与地面站通信，而PX4采用MAVLink    

uavo消息由XML定义，您可以参考`shared/uavobjectdefinition`了解全部消息定义   
生成的源码则文件位于 `build/uavobject-synthetics/`    
如果您对xml->src机制感兴趣，可以参考`ground/uavobjgenerator`  

对于应用层业务模块间通信,机制上得益于FreeRTOS的msg queue    
而接口层面依赖于`flight/uavobjects`定义的一些接口   
您也可以参考`flight/Example`了解其用法和场景    

对于地面站通信，无论是有线还是无线，在硬件驱动层面都做了抽象   
您可以参考`flight/uavobjects`了解消息的序列化和反序列化   
`flight/uavtalk`是处理消息的一些工具封装

### 应用模块

得益于微核调度和通信机制的封装    
应用层的业务模块可以实现隔离

为了满足不同场景调度需求，    
当前支持普通task，DelayCallback等    
您可以参考`flight/Example`和`CallbackTest`了解它们的使用方法     

您也可以参考如下模块了解自定义模块开发流程    

| 模块              | 描述             |
| -------------     | ---------------- |
| `Actuator`        | 计算电机输出     |
| `FlightPlan`      | 航路点路径规划   |
| `FirmwareIAP`     | 板子型号认证     |
| `GPS`             | GPS传感器        |
| `Logging`         | 日志操作控制     |
| `ManualControl`   | 手动输入信号控制 |
| `Receiver`        | 接收机信号处理   |
| `Sensors`         | 传感器数据整合   |
| `Stabilization`   | 姿态控制器       |
| `StateEstimation` | EKF状态估计      |

### 模块性能
