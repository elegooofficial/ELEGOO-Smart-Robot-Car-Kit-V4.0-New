# ELEGOO Smart Robot Car Kit V4.0 New

[English README](./README.md)

ELEGOO Smart Robot Car Kit V4.0 New 是一套基于 Arduino 的智能小车程序，适用于 ELEGOO Smart Robot Car Kit V4.0 平台。当前固件版本为 `2.1.2`，主程序入口为 `ELEGOO-Smart-Robot-Car-Kit-V4.0-New.ino`（Arduino 工具链要求工程文件夹与该文件同名）。

本仓库包含小车的运动控制、循迹、避障、跟随、红外遥控、串口 JSON 控制、RGB 灯光、舵机、超声波测距、电压检测和按键处理等核心逻辑。

## 功能特性

- 基础运动控制：前进、后退、左转、右转、停止
- 自动模式：循迹、避障、跟随
- 遥控模式：红外遥控和串口摇杆控制
- 传感器读取：三路循迹传感器、超声波距离、电压检测、MPU6050
- 外设控制：RGB LED、双轴舵机、电机驱动
- 串口通信：通过 JSON 命令控制小车并返回执行结果
- 看门狗：启用 AVR watchdog，提高运行稳定性

## 文件结构

```text
.
|-- ELEGOO-Smart-Robot-Car-Kit-V4.0-New.ino  # Arduino 主程序入口
|-- ApplicationFunctionSet.*     # 应用层状态机、串口命令、模式切换
|-- MotionControl.*              # 运动控制、循迹、避障、跟随算法
|-- DeviceDriverSet.*            # 电机、舵机、红外、超声波、MPU6050 等驱动
|-- RgbControl.*                 # RGB LED 控制
|-- Utils.*                      # 工具函数
`-- ArduinoJson-v6.11.1.h        # 项目内置 ArduinoJson 头文件
```

## 社区硬件与许可说明

套件自带的底盘为原厂亚克力板。围绕套件自带硬件（电机、车轮、UNO 与扩展板、舵机云台、传感器、电池盒）设计的 3D 打印底盘模型，在社区分支 https://github.com/westondunn/elegoo-smart-car-v4 中维护（见 Pull Request [#1](https://github.com/westondunn/elegoo-smart-car-v4/pull/1) 与 [#2](https://github.com/westondunn/elegoo-smart-car-v4/pull/2)），不涉及固件改动。

本仓库未附带许可证文件，固件仍遵循 ELEGOO 的条款。来自 ELEGOO 官方仓库的资料（原理图、数据手册、底盘板模型）仅以固定提交号引用而不复制，社区分支也不会为其指定任何许可证。

## 硬件与引脚

默认引脚已经在代码中定义，通常不需要手动修改。

| 模块 | 引脚 |
| --- | --- |
| RGB LED | D4 |
| 按键 | D2 |
| 红外接收 | D9 |
| 左循迹传感器 | A2 |
| 中循迹传感器 | A1 |
| 右循迹传感器 | A0 |
| 电压检测 | A3 |
| 右电机 PWM | D5 |
| 左电机 PWM | D6 |
| 右电机方向 | D7 |
| 左电机方向 | D8 |
| 电机 STBY | D3 |
| 超声波 TRIG | D13 |
| 超声波 ECHO | D12 |
| Z 轴舵机 | D10 |
| Y 轴舵机 | D11 |
| MPU6050 | I2C SDA/SCL |

## 开发环境

推荐使用：

- Arduino IDE 2.x
- 或 Visual Studio Code + Arduino 扩展
- 开发板选择：`Arduino Uno`
- 串口波特率：`9600`

需要安装的 Arduino 库：

- `FastLED`
- `Servo`
- `IRremote`
- `MPU6050`
- `Wire`，Arduino 内置库

项目已包含 `ArduinoJson-v6.11.1.h`，一般不需要额外安装 ArduinoJson。

## 编译与上传

### Arduino IDE

1. 打开 `ELEGOO-Smart-Robot-Car-Kit-V4.0-New.ino`
2. 选择 `工具 > 开发板 > Arduino AVR Boards > Arduino Uno`
3. 通过 USB 连接小车主控板
4. 选择正确的串口，例如 `COM3`
5. 点击 `验证` 检查编译
6. 点击 `上传` 烧录程序

上传成功后打开串口监视器，波特率设置为 `9600`。复位后应看到：

```text
Smart Robot Car V4.0 - System Ready
```

### arduino-cli

编译：

```bash
arduino-cli compile --fqbn arduino:avr:uno .
```

上传时将 `COM3` 替换为实际端口：

```bash
arduino-cli upload -p COM3 --fqbn arduino:avr:uno .
```

## 串口 JSON 控制

程序通过串口接收 JSON 命令。消息必须以 `{` 开始、以 `}` 结束，最大长度为 `128` 字节。

常用字段：

| 字段 | 说明 |
| --- | --- |
| `N` | 命令编号 |
| `H` | 命令序号，用于响应跟踪 |
| `D1`、`D2`、`D3`、`D4` | 命令参数 |
| `T` | 定时时间，单位毫秒 |

示例：控制小车以速度 `120` 前进 `1000` 毫秒后停止。`N=2` 为定时命令；`N=3` 不定时，忽略 `T`。

```json
{"N":2,"H":"1","D1":3,"D2":120,"T":1000}
```

示例：切换到循迹模式。

```json
{"N":101,"H":"2","D1":1}
```

示例：将 RGB 灯设为红色并保持 `1000` 毫秒。`N=7` 为定时命令；`N=8` 不定时，忽略 `T`。

```json
{"N":7,"H":"3","D1":1,"D2":255,"D3":0,"D4":0,"T":1000}
```

## 主要命令

| `N` | 功能 |
| --- | --- |
| `1` | 单个或全部电机控制 |
| `2` / `3` | 小车方向与速度控制（`2` 按 `T` 定时，`3` 不定时） |
| `4` | 左右电机独立速度控制 |
| `5` | 舵机角度控制 |
| `7` / `8` | RGB 灯光控制（`7` 按 `T` 定时，`8` 不定时） |
| `21` | 获取超声波状态或距离 |
| `22` | 获取循迹传感器状态 |
| `23` | 获取离地/地面检测状态 |
| `100` | 清除功能并进入待机 |
| `101` | 模式切换：`1` 循迹，`2` 避障，`3` 跟随 |
| `102` | 摇杆控制模式 |
| `105` | LED 亮度调整 |
| `106` | 舵机方向控制 |
| `110` | 清除功能并进入空闲状态 |

## 运行模式

| 模式 | 说明 |
| --- | --- |
| `Standby` | 待机 |
| `TraceBased` | 循迹 |
| `ObstacleAvoidance` | 避障 |
| `Follow` | 跟随 |
| `Rocker` | 摇杆或遥控 |
| `DoNothing` | 空闲 |

## 常见问题

### 编译提示找不到 `FastLED.h`

在 Arduino IDE 中打开库管理器，搜索并安装 `FastLED`。

### 编译提示找不到 `IRremote.hpp` 或 `MPU6050.h`

安装对应库后重新编译。不同版本的库 API 可能有差异，建议优先使用 ELEGOO 官方资料包中提供的库版本。

### 上传失败

检查以下项目：

- 开发板是否选择 `Arduino Uno`
- 串口是否选择正确
- USB 数据线是否支持数据传输
- 是否有串口监视器或其他软件占用了 COM 口
- 主控板驱动是否安装正确

### 小车不运动

检查电池电量、主板电源开关、电机接线和电机驱动 STBY 控制。程序中电池电压阈值为 `7.00V`，电量过低可能导致行为异常。

## 版本信息

- 当前版本：`2.1.2`
- 主程序：`ELEGOO-Smart-Robot-Car-Kit-V4.0-New.ino`
- 作者：ELEGOO

