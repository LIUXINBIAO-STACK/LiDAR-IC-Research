# Day 4：MOS 管物理结构、栅极驱动机制与 RLC 阻尼控制

本篇整理激光雷达脉冲驱动与开关电源中功率开关管（MOSFET / GaN）的核心底层原理，涵盖 **MOS 管物理结构与寄生电容**、**栅极开通四阶段与米勒平台**、**CMOS 推挽与死区逻辑**、**栅极电阻选型**以及 **RLC 四种阻尼类型**。

---

## 一、N-MOSFET 物理结构与三大寄生电容

以增强型 N 沟道 MOSFET 为例，栅极（G）与硅衬底之间隔着一层极薄的 $\text{SiO}_2$ 绝缘层，因此**稳态时栅极不消耗直流电流**。当施加正向栅压 $V_{GS} > V_{\text{th}}$ 时，电场将衬底电子吸到绝缘层下方形成 N 型反型层导电沟道（电子桥），使漏极（D）与源极（S）低阻导通。

&lt;img src=&quot;./images/day4_1_mosfet_structure.svg&quot; alt=&quot;N-MOSFET物理截面结构与寄生电容&quot; width=&quot;680&quot; /&gt;

### 寄生电容等效电路与数据手册换算

物理结构中的金属–绝缘层–半导体天然构成三颗寄生电容：栅源电容 $C_{gs}$ 、米勒电容 $C_{gd}$ 与漏源电容 $C_{ds}$ 。

&lt;img src=&quot;./images/day4_2_mosfet_caps.svg&quot; alt=&quot;N-MOSFET寄生电容等效电路图&quot; width=&quot;680&quot; /&gt;

数据手册（Datasheet）中通常给出以下三个实测组合参数：
* **输入电容** $C_{\text{iss}} = C_{gs} + C_{gd}$ ：决定前级驱动芯片开通瞬间需要灌入的总电荷量。
* **输出电容** $C_{\text{oss}} = C_{ds} + C_{gd}$ ：决定关断瞬间与回路寄生电感产生振铃的频率。
* **反向传输电容** $C_{\text{rss}} = C_{gd}$ ：即**米勒电容**，数值越小，米勒平台越短、开关速度越快。

---

## 二、栅极驱动开通 4 阶段与米勒平台

&lt;img src=&quot;./images/day4_3_miller_waveform.svg&quot; alt=&quot;栅极开通四阶段与米勒平台波形图&quot; width=&quot;680&quot; /&gt;

1. **阶段 ① 开通延迟期**（ $t_0 \sim t_1$ ）：前级驱动瞬间灌出最大峰值电流 $I_G$ 给 $C_{gs}$ 充电；在 $V_{GS}$ 跨过阈值 $V_{\text{th}}$ 之前，管子保持截止（ $I_D = 0$ ）。
2. **阶段 ② 电流上升期**（ $t_1 \sim t_2$ ）： $V_{GS} > V_{\text{th}}$ ，沟道打开，漏极电流 $I_D$ 迅速爬升至满载电流，此时 $V_{DS}$ 仍维持高压。
3. **阶段 ③ 米勒平台期**（ $t_2 \sim t_3$ ）：在低边开关中，截止时 D 极顶着母线高压（如 $48\text{ V}$ ），导通瞬间 D 极被拉向地（ $0\text{ V}$ ），因此 **$V_{DS}$ 从高压暴跌至 $0\text{ V}$** 。根据电容方程：

$$
I_{gd} = C_{gd} \frac{d(V_G - V_D)}{dt} \approx -C_{gd} \frac{dV_D}{dt} > 0
$$

   剧烈下跌的 $V_D$ 通过 $C_{gd}$ 吸走了前级驱动送来的全部电流 $I_G$ 来完成极性反转，导致 **$V_{GS}$ 卡在平台电压上不变**。此阶段电压与电流同时很大（ $P = V_{DS} \times I_D$ ），是开关管发热最严重的时段。
4. **阶段 ④ 过驱动与稳态导通**（ $t_3 \sim t_4$ 及之后）： $V_{DS}$ 跌到接近 $0\text{ V}$ 后， $V_{GS}$ 迅速冲至 $V_{\text{drive}}$ 将导通电阻压到最低；充满电后栅极电流 $I_G$ 彻底归零。

---

## 三、CMOS 推挽驱动与“先断后通”死区逻辑

### 1. CMOS 推挽驱动电路

前级驱动芯片末级普遍采用 **PMOS 上管 + NMOS 下管** 构成的推挽结构：开通时 PMOS 导通向栅极高速灌电流（“推”），关断时 NMOS 导通将栅极电荷瞬间抽回地（“挽”），稳态时总有一管截止，实现静态零功耗。

&lt;img src=&quot;./images/day4_4_cmos_pushpull.svg&quot; alt=&quot;CMOS推挽驱动电路原理图&quot; width=&quot;680&quot; /&gt;

### 2. 先断后通（Break-Before-Make）死区逻辑

由于真实 MOS 管关断往往比开通更慢，若上下管同时切换，会出现短暂同时导通，把电源 $V_{\text{DD}}$ 直接短接到地产生致命的**直通电流（Shoot-Through）**。死区逻辑在切换瞬间强制插入一段两管同时关断的空白期 $t_d$ ，彻底杜绝直通。

&lt;img src=&quot;./images/day4_5_deadtime.svg&quot; alt=&quot;无死区直通与先断后通死区时序对比图&quot; width=&quot;680&quot; /&gt;

* **前级推挽驱动 IC 死区**：一般仅需 $1\text{–}5\text{ ns}$ 。
* **同步整流 Buck 电源死区**：一般取 $10\text{–}50\text{ ns}$ （太长会增加下管体二极管续流损耗）。

---

## 四、栅极电阻 $R_g$ 的选择与分离调速

选择 $R_g$ 是在**开关速度**与**抑制振铃**之间做权衡：
* **防振铃下限（阻尼比 $\zeta \approx 0.7 \sim 1$ ）**：

$$
R_{\text{total}} = R_{\text{drv}} + R_g + R_{g,\text{int}} \approx (1.4 \sim 2) \sqrt{\frac{L_g}{C_{\text{iss}}}}
$$

* **开关速度上限**：

$$
I_G \approx \frac{V_{\text{drive}} - V_{\text{plateau}}}{R_{\text{total}}}, \qquad t_{\text{sw}} \approx \frac{Q_g}{I_G}
$$

工程上常将开通与关断路径分开：**开通走较大的 $R_{\text{on}}$ 抑制振铃，关断经高速肖特基二极管走极小的 $R_{\text{off}}$ 强力下拉防米勒误导通**。

&lt;img src=&quot;./images/day4_6_gate_resistor.svg&quot; alt=&quot;开通与关断分离的栅极电阻驱动原理图&quot; width=&quot;680&quot; /&gt;

* **经验起手值**：eGaN 激光驱动常用 $R_{\text{on}} \approx 2\text{–}5\text{ }\Omega$ 、 $R_{\text{off}} \approx 0\text{–}1\text{ }\Omega$ ；硅 MOSFET 常用 $R_{\text{on}} \approx 4.7\text{–}10\text{ }\Omega$ 、 $R_{\text{off}} \approx 1\text{–}2.2\text{ }\Omega$ 。务必选用 0402/0603 无感贴片电阻并紧贴栅极引脚摆放。

---

## 五、RLC 串联回路的四种阻尼类型

栅极驱动回路和功率开关回路本质上都是二阶 RLC 电路，其阻尼比 $\zeta$ 定义为：

$$
\zeta = \frac{R}{2}\sqrt{\frac{C}{L}}, \qquad \omega_n = \frac{1}{\sqrt{LC}}
$$

&lt;img src=&quot;./images/day4_7_rlc_damping.svg&quot; alt=&quot;RLC四种阻尼类型阶跃响应曲线图&quot; width=&quot;680&quot; /&gt;

| 阻尼类型 | 阻尼比与电阻条件 | 波形特征 | 工程表现与调优建议 |
| :--- | :--- | :--- | :--- |
| **无阻尼** | $\zeta = 0$ （ $R = 0$ ） | 永不衰减的等幅振荡，峰值达 $2 V_{\text{in}}$ | 理想无损耗极限，实际中应极力避免 |
| **欠阻尼** | $0 < \zeta < 1$ | 冲过目标值后衰减振铃 | $\zeta < 0.3$ 极易过压击穿； **$\zeta \approx 0.707$** 速度最快且过冲仅 $4.3\%$ |
| **临界阻尼** | $\zeta = 1$ | **无任何过冲**前提下最快到达稳态 | 波形最干净，最适合耐压余量极小的 GaN 栅极 |
| **过阻尼** | $\zeta > 1$ | 完全无过冲，但上升极其缓慢 | 激光脉宽变钝、开关过渡区发热剧增 |
