# LiDAR-IC-Research

本项目用于整理和记录激光雷达（dToF LiDAR）相关电源管理芯片与激光驱动芯片的研究资料、电路设计与仿真实验。

## 1. 目录结构

| 文件夹 | 内容说明 | 状态 |
| :--- | :--- | :---: |
| `01_System_Overview` | 系统架构与指标拆解 | 进行中 |
| `02_Power_Management` | Buck / Boost 电源拓扑与计算 | 待补充 |
| `03_Laser_Driver` | 激光器驱动原理与电路 | 待补充 |
| `05_Simulation` | 仿真工程与波形记录 | 待开始 |

## 2. 常用公式渲染测试（GitHub 原生支持 LaTeX）

Buck 降压电路在连续导通模式（CCM）下的占空比关系：

$$D = \frac{V_{out}}{V_{in}}$$

电感电流纹波计算：

$$\Delta I_L = \frac{V_{out}(1 - D)}{f_{sw} \cdot L}$$

## 3. 代码块测试

```c
// 示例：简单的状态标记
#define SYSTEM_READY 1
