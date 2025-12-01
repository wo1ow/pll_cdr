## HW05- 理论分析



### 1. $D_{in}$的生成

输入信号$D_{in}(t)$ 本质上是一个时间被“扭曲”了的理想数字序列。我们可以拆解为两部分：**理想数据**和**相位抖动**。

#### 1.1 理想数据(Ideal NRZ Data)

理想的10Gbps NRZ 信号$D_{ideal}(t)$ 是由一系列二进制$b_k$ 组成的矩形脉冲序列：
$$
D_{ideal}(t) = \sum_{k=-\infty}^{\infty}b_{k}\cdot p(t-k\cdot T_{b})
$$
其中：

+ $b_k \in {0, 1}$  是随机数据。
+ $T_b$ 是比特周期（Unit Interval, UI）。对于 10Gbps，$T_b = \frac{1}{10 \times 10^9} = 100 \text{ps}$.
+ $p(t)$ 是单位矩形脉冲（宽度为 $T_b$）。 

### 1.2 抖动信号 (Jitter Profile)

抖动是**正弦抖动 (Sinusoidal Jitter, SJ)**。抖动通常定义为实际边沿时间与理想边沿时间的偏差 $\Delta t(t)$。

**频率 ($f_{jitter}$):** 1 MHz。

**幅度 ($J_{pp}$):** 0.2 UI。

- 这里的 0.2 UI 指的是**峰峰值 (Peak-to-Peak)**。
- 1 UI = $T_b = 100 \text{ps}$。
- 所以抖动的峰峰值时间为 $0.2 \times 100 \text{ps} = 20 \text{ps}$。
- 正弦波的**振幅 (Amplitude)** $A_j$ 是峰峰值的一半：$A_j = 10 \text{ps}$。

抖动的时间函数表达为：  
$$
\Delta t(t) = A_j \cdot \sin(2\pi f_{jitter} t) = 10\text{ps} \cdot \sin(2\pi \cdot 10^6 \cdot t)
$$


### 1.3 合成输入信号 (Combined Signal)

将抖动施加到理想数据上，意味着数据的“时间轴”发生了伸缩。如果 $t$ 时刻存在抖动 $\Delta t(t)$，则实际信号 $D_{in}(t)$ 等价于理想信号在时间轴上进行了平移：
$$
D_{in}(t) = D_{ideal}(t - \Delta t(t))
$$
也就是当 $\Delta t(t) > 0$ 时，我们读取的是“过去”时刻的理想值，意味着信号**延迟 (Lag)** 了；当 $\Delta t(t) < 0$ 时，信号**超前 (Lead)** 了。

---



### 2. 
