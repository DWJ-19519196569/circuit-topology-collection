# SWD 调试接口电路

## 拓扑描述
由 **目标 MCU** 与 **调试器（如 ST-Link、J-Link）** 通过 SWDIO、SWCLK 两根信号线连接而成，配 VCC、GND 以及可选的复位 NRST 与串行跟踪输出 SWO 线。电路共有 6 个关键节点：**节点① SWCLK 时钟线**（调试器 SWCLK 与目标 SWCLK 相连）；**节点② SWDIO 数据线**（调试器 SWDIO 与目标 SWDIO 相连，半双工双向）；**节点③ 复位线 NRST**（调试器 nRESET 与目标 NRST 相连）；**节点④ 串行跟踪线 SWO**（目标 SWO 与调试器 SWO 相连，可选）；**节点⑤ 电源节点**（调试器供电 VCC 与目标 VDD）；**节点⑥ 地节点 GND**。

各元件管脚连接：**目标 MCU** 的 SWDIO 脚接节点②、SWCLK 脚接节点①、NRST 脚接节点③（可选）、SWO 脚接节点④（可选）、正电源脚 VDD 接节点⑤、地脚 GND 接节点⑥；**调试器** 的 SWDIO 脚接节点②、SWCLK 脚接节点①、nRESET 脚接节点③、SWO 脚接节点④、VCC 脚接节点⑤ 的 $+V_{DD}$、GND 脚接节点⑥；**SWDIO 信号线上** 外接一个上拉电阻 $R_{pu}$（常用 $10\ \text{kΩ}$）接到节点⑤ 的 $+V_{DD}$，保证空闲时为高电平并改善上升沿。调试器作为主机在节点① 上驱动 SWCLK 时钟，边沿控制节点② 上的双向数据收发，共同构成半双工调试链路。

由此构成调试器主机经 SWDIO/SWCLK 双线与目标 Cortex-M 内核通信、可选 NRST 复位与 SWO 跟踪输出的 SWD 调试组态，用于程序下载、在线调试与跟踪。

## 工作原理
调试器作为主机驱动 SWCLK 时钟，通过 SWDIO 双向线以分组的包（请求/应答）对 AP/DP 寄存器进行读写，进而访问内核与存储空间；SWDIO 在时钟控制下于收发方向间切换实现半双工通信。

## 关键计算公式

- 时钟周期：
$$t_{SWCLK} = \frac{1}{f_{SWCLK}}$$

- 上拉电阻取值（保证边沿并兼顾低电平）：
$$R_{pu} \approx \frac{V_{DD}}{I_{pu}},\qquad \text{常用 } 10\ \text{kΩ}$$

- 单次传输总线占用时间（N 位）：
$$t_{bit} = \frac{N}{f_{SWCLK}}$$

| 符号 | 含义 | 单位 |
|------|------|------|
| $f_{SWCLK}$ | SWD 时钟频率 | Hz |
| $t_{SWCLK}$ | 时钟周期 | s |
| $V_{DD}$ | 电源电压 | V |
| $I_{pu}$ | 上拉电流 | A |
| $R_{pu}$ | 上拉电阻 | Ω |

## 元件调整影响
- **SWCLK 频率↑** → 下载/调试更快，但对走线质量和目标稳定度要求提高。
- **上拉电阻 R↑** → 上升沿变慢，过高可能导致高速时钟下采样错误。
- **上拉电阻 R↓** → 边沿更陡，但静态功耗增大。
- **引线长度↑** → 寄生电容增大，最高可靠时钟下降，长线应降速。

## 典型参数示例
- 取 $f_{SWCLK}=4\ \text{MHz}$：$t_{SWCLK}=\dfrac{1}{4\times10^6}=250\ \text{ns}$。
- $V_{DD}=3.3\ \text{V}$、上拉 $R_{pu}=10\ \text{kΩ}$：$I_{pu}=\dfrac{3.3}{10\times10^3}=0.33\ \text{mA}$。

## 常见器件型号
- 调试器：ST-Link/V2、J-Link、DAP-Link、CMSIS-DAP。
- 目标：STM32F1/F4、GD32 等 Cortex-M 系列 MCU。
- 电平匹配：必要时用 TXS0102（3.3 V↔1.8 V）。

## 注意事项
- SWDIO、SWCLK 建议外接上拉电阻并靠近目标侧放置。
- 使用引脚数少的飞行线时，尽量短并降低 SWCLK 频率。
- 若 SWD 引脚被复用为普通 IO 或下载后无法连接，需先拉 NRST 复位再连接。
- 供电电压需与调试器和目标一致，避免电平失配损坏。

## 参考资料
- ARM CoreSight / SWD 协议规范（ARM Debug Interface Architecture）。
- STM32 参考手册（调试支持章节）。
- Wikipedia: JTAG（含 SWD 相关说明）。
