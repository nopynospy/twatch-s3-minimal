# twatch-s3-minimal

适用于 **LilyGO T-Watch S3** (ESP32-S3, AXP2101 PMU, PCF8563 RTC, ST7789 显示屏) 的精简、低功耗固件基底。

基于官方 [LilyGO TTGO_TWatch_Library](https://github.com/Xinyuan-LilyGO/TTGO_TWatch_Library) 出厂示例代码重构与精简，旨在提供一个适合日常佩戴、轻量且省电的手表系统基础。

---

## 主要改进与功耗优化

- **功耗深度优化：**
  - 将 CPU 工作主频由默认的 160/240 MHz 降低至 **80 MHz** (`setCpuFrequencyMhz(80)`)，在保证 LVGL UI 流畅度的同时显著降低待机电流。
  - 在开机阶段禁用未使用的外设硬件驱动（I2S 麦克风、LoRa 射频、音频播放器/DAC、红外遥控发射器）。
  - 分级更新节奏：时间界面维持 1 秒常规刷新，RTC/电量/步数等 I2C 传感器查询则降低频率至 30 秒或 60 秒触发一次，大幅减少 I2C 总线占用与处理器唤醒开销。
- **极简表盘设计 (`minimalClock`)：**
  - 移除原版内置的位图资源、冗余按钮与出厂二维码页面。
  - 采用高对比度 Fira Code 等宽字体排版。
  - 综合状态栏显示：
    - **顶部栏：** 电池电量百分比与芯片温度。
    - **日期进度栏：** 星期、月份、日期、两位年份、年累计天数与周数进度 (`FRI, OCT 16 '26 | D289/365 W42`)。
    - **中心：** 冒号闪烁数字时钟。
    - **底部：** BMA423 实时计步器读数。
- **智能硬件 RTC 校准：**
  - 边沿触发式对时：仅在插入 USB 充电线时检测并触发 Wi-Fi 联网校准，电池供电状态下绝不自动开启 Wi-Fi 耗电。
  - 采用稳定可靠的 **HTTP HEAD** 标头时间校准方案，彻底规避部分路由器或热点对标准 NTP UDP 123 端口的屏蔽问题。
  - 结合 `secrets.h` 支持多 Wi-Fi 凭证切换。
- **实体按键多功能交互（返回主页 / 熄屏休眠）：**
  - 利用 AXP2101 电源按键实现了类似主流智能手表的实体交互逻辑。
  - 在任何二级应用页面（如 Wi-Fi 扫描、设备信息等）单击侧边按键，界面将平滑退回 `minimalClock` 主表盘。
  
---

## 硬件规格

| 组件 | 型号 | 通信接口 |
| :--- | :--- | :--- |
| **主控** | ESP32-S3 (双核 Xtensa LX7, 8MB PSRAM, 16MB Flash) | — |
| **电源芯片** | AXP2101 PMU | I2C |
| **硬件时钟** | PCF8563 实时时钟 (RTC) | I2C |
| **加速度计** | BMA423 三轴加速度传感器（计步器） | I2C |
| **显示屏** | 1.54 英寸 IPS LCD (ST7789V, 240×240) | SPI |
| **触摸屏** | FT6236 电容触控芯片 | I2C |

---

## 快速上手

### 1. Wi-Fi 配置 (`secrets.h`)

为了防止个人 Wi-Fi 凭证意外上传到公开仓库，本项目默认忽略 `secrets.h` 文件。

在项目源码目录下新建 `secrets.h` 文件并填入网络信息：

```cpp
#pragma once

struct WifiCredential {
    const char* ssid;
    const char* password;
};

// 在此处添加你常用的 Wi-Fi 网络
const WifiCredential KNOWN_NETWORKS[] = {
    { "YOUR_WIFI_SSID",     "YOUR_WIFI_PASSWORD" },
    { "YOUR_PHONE_HOTSPOT", "HOTSPOT_PASSWORD"   }
};

const int KNOWN_NETWORKS_COUNT = sizeof(KNOWN_NETWORKS) / sizeof(KNOWN_NETWORKS[0]);
```
### 2. 编译与烧录 (PlatformIO)
推荐使用 VS Code 搭配 PlatformIO 扩展进行开发与烧录。

克隆本仓库：

```Bash
git clone [https://github.com/nopynospy/twatch-s3-minimal.git](https://github.com/nopynospy/twatch-s3-minimal.git)
cd twatch-s3-minimal
```
在 VS Code 中打开项目文件夹。

使用 USB-C 数据线连接 T-Watch S3 与电脑。

点击 PlatformIO 侧边栏的 Upload，或在终端执行：

```Bash
pio run -e twatch-s3 -t upload
```
打开串口监视器查看运行日志（波特率 115200）：

Bash
```
pio device monitor -b 115200
```
### 开源协议
本项目沿用上游 LilyGO TTGO_TWatch_Library 的 Apache-2.0 / MIT 开源协议。