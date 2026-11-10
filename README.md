# 麦轮机器人 · RoboMaster Mecanum-Wheel Robot

基于 RoboMaster 开发板 C 型官方例程开发的**麦轮全向底盘机器人**完整嵌入式工程，覆盖底盘、云台、发射、裁判系统、视觉通信、UI 绘制等任务，可直接用 Keil MDK 编译下载。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![MCU](https://img.shields.io/badge/MCU-STM32F407IGHx-blue.svg)
![RTOS](https://img.shields.io/badge/RTOS-FreeRTOS-green.svg)
![Toolchain](https://img.shields.io/badge/Toolchain-Keil%20MDK%205.x-orange.svg)

## 目录

- [项目简介](#项目简介)
- [功能特性](#功能特性)
- [硬件平台](#硬件平台)
- [软件环境](#软件环境)
- [目录结构](#目录结构)
- [编译与烧录](#编译与烧录)
- [调试工具](#调试工具)
- [注意事项](#注意事项)
- [参与贡献](#参与贡献)
- [许可证](#许可证)

## 项目简介

本项目是 RoboMaster 2023 赛季的麦轮底盘机器人工程，在 RoboMaster 官方开发板 C 型例程的基础上重构为分层结构（`application` / `bsp` / `components`），便于按任务扩展：

- **应用层**（`application/`）只关心业务逻辑，一个 `.c/.h` 对应一类任务；
- **板级支持层**（`bsp/boards/`）封装外设，向上提供统一接口；
- **组件层**（`components/`）沉淀可复用算法与设备驱动（PID、卡尔曼滤波、四元数 EKF、AHRS、FIFO、CRC 等）。

工程使用 STM32CubeMX 管理外设配置（`standard_robot.ioc`），外设初始化代码由 CubeMX 生成，业务代码与其分离，重新生成时不会覆盖。

## 功能特性

| 模块 | 说明 | 相关文件 |
| --- | --- | --- |
| 底盘 | 麦轮全向运动解算、跟随/小陀螺等行为切换、功率与速度限幅 | `application/chassis_task.c`、`chassis_behaviour.c` |
| 云台 | 角度环 + 速度环串级 PID，支持视觉辅助瞄准 | `application/gimbal_task.c`、`gimbal_behaviour.c` |
| 发射 | 摩擦轮转速闭环与拨弹控制 | `application/shoot_task.c` |
| 裁判系统 | 串口读取比赛数据、绘制自定义 UI | `application/referee.c`、`RM_Cilent_UI.c`、`UI_task.c` |
| 视觉通信 | 接收视觉帧，解算弹道俯仰角补偿 | `application/vision.c`、`vision_task.c` |
| 姿态解算 | BMI088 + IST8310，四元数 EKF / AHRS 输出姿态 | `application/ins_task.c`、`components/algorithm/` |
| 离线检测 | 按接收时间判定各设备在线状态并触发保护 | `application/detect_task.c` |
| 辅助 | 遥控、OLED、电压监测、LED 流水灯、USB 虚拟串口 | `remote_control.c`、`oled_task.c`、`voltage_task.c`、`usb_task.c` |

## 硬件平台

| 部件 | 型号 / 说明 |
| --- | --- |
| 主控 | RoboMaster 开发板 C 型，STM32F407IGHx（Cortex-M4F，168 MHz，1 MB Flash / 192 KB RAM） |
| 底盘电机 | M3508 无刷减速电机 + C620 电调（CAN 总线，电流/转速闭环） |
| 惯性传感器 | BMI088（六轴 IMU）+ IST8310（三轴磁力计），C 板板载 |
| 遥控 | DBUS 接收机（DT7 + DR16 系列） |
| 外设 | OLED（I2C）、蜂鸣器、激光、舵机 PWM、USB CDC |

## 软件环境

| 项目 | 版本 / 说明 |
| --- | --- |
| IDE | Keil MDK 5.x（ARM Compiler 5 或 6） |
| 器件包 | Keil.STM32F4xx_DFP |
| 配置工具 | STM32CubeMX（`standard_robot.ioc`，可选，仅在改动外设时需要） |
| RTOS | FreeRTOS 10 + CMSIS-RTOS v1 |
| DSP | CMSIS-DSP（`components/algorithm/arm_cortexM4lf_math.lib`） |

## 目录结构

```text
.
├── application/          # 应用层：底盘、云台、发射、裁判、视觉、UI、检测等任务
├── bsp/boards/           # 板级支持包：CAN、USART、SPI、I2C、ADC、PWM、Flash、DWT 等
├── components/           # 通用组件
│   ├── algorithm/        # AHRS、四元数 EKF、卡尔曼滤波、CMSIS-DSP
│   ├── controller/       # PID
│   ├── devices/          # BMI088、IST8310 驱动
│   └── support/          # FIFO、CRC8/CRC16、内存管理、链表
├── Drivers/              # STM32F4 HAL 驱动与 CMSIS
├── Inc/ Src/             # CubeMX 生成的外设初始化与 main
├── Middlewares/          # FreeRTOS、STM32 USB Device Library
├── MDK-ARM/              # Keil 工程（standard_robot.uvprojx）
├── RMTools_9.0.0.6 Stu/  # 第三方调参上位机（仅作调试参考，非本项目源码）
├── 规则/                  # RoboMaster 2023 比赛规则与机器人制作规范（官方文档）
├── standard_robot.ioc    # STM32CubeMX 工程文件
└── keilkilll.bat         # 一键清理 Keil 中间文件
```

## 编译与烧录

1. 安装 Keil MDK 5.x 与 `Keil.STM32F4xx_DFP` 器件包；
2. 打开 `MDK-ARM/standard_robot.uvprojx`；
3. 确认 Target 为 `standard_robot`、器件为 `STM32F407IG`；
4. `Rebuild`（F7）编译，确认 0 Error；
5. 用 ST-Link / J-Link 下载，或使用生成的 `.hex` 通过其他工具烧录。

> 仓库不包含编译产物（`.o`、`.crf`、`.axf`、`.hex`、`.map` 等），首次编译请执行完整的 `Rebuild`。需要清理中间文件时可直接运行 `keilkilll.bat`。

## 调试工具

`RMTools_9.0.0.6 Stu` 为第三方调参上位机（.NET 程序），可通过 USB 虚拟串口在线修改 PID 参数、观察波形。该目录仅作调试参考随仓库附带，版权归其原作者所有。

## 注意事项

- 上电前请核对电机 ID、CAN 接线与电源极性；首次调试建议架空底盘、限制最大电流；
- 修改 `standard_robot.ioc` 后重新生成代码时，请注意 CubeMX 会覆盖 `main.c` 中 `USER CODE` 标记之外的区域；
- `Inc/`、`Src/`、`Drivers/`、`Middlewares/` 中的 ST 官方代码遵循其原始许可（SLA0044 等），不在本项目 MIT 许可覆盖范围内；
- `规则/` 中的比赛规则文档版权归 RoboMaster 组委会所有，仅作学习参考。

## 参与贡献

欢迎提交 Issue 与 Pull Request，具体流程与代码风格约定见 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 许可证

本项目（`application/`、`bsp/`、`components/` 及工程配置）以 **MIT License** 开源，详见 [LICENSE](LICENSE)。

第三方组件遵循各自许可：STM32 HAL / CMSIS（ST SLA0044、Apache-2.0）、FreeRTOS（MIT）、RMTools（原作者所有）。
