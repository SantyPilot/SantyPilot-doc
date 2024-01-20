# 滤波原理

采用EKF做状态估计

关于EKF与PX4实现的相关背景知识您可以参考[链接](https://mp.weixin.qq.com/s?__biz=MzI1MjQyNjcwMQ==&mid=2247483717&idx=1&sn=f8eff13e1bc8e7fa282c66e2784cc040&chksm=e9e2a1e7de9528f1129bc46fef6c9d898c8ca1364250f4ba2dfbf7bb9fadca5b050695ebcc66&cur_album_id=2842458489158336513&scene=189#wechat_redirect)

## 变量设计

状态向量为
<p style="text-align: center;">
$$x=\left[\begin{array}{l}
 p_{N}, p_{E}, p_{D}, v_{N}, v_{E}, v_{D}, q_{0}, q_{1}, q_{2}, q_{3}, gb_{x}, gb_{y}, gb_{z}
\end{array}\right]^T$$
</p>

其中分量分别为
 - 地系位置(position)
 - 地系速度(velocity)
 - 姿态四元数(quaternion)
 - 陀螺仪机体系偏移量(gyro bias)

惯性测量作为输入量
<p style="text-align: center;">
$$u=\left[\begin{array}{l}
 g_{x}, g_{y}, g_{z}, a_{x}, a_{y}, a_{z}
\end{array}\right]^T$$
</p>

和其他传感器分别融合，部分观测向量为
<p style="text-align: center;">
$$y=\left[\begin{array}{l}
 p_{N}, p_{E}, p_{D}, v_{N}, v_{E}, v_{D}, mag_{x}, mag_{y}, mag_{z}, baro
\end{array}\right]^T$$
</p>

## 先验估计

先验估计包括状态预测和系统噪声协方差预测

### 状态预测
状态预测是结合状态微分方程，估计$$k+1$$时刻状态，[源码RungeKutta法](https://github.com/SantyPilot/SantyPilot/blob/master/flight/libraries/insgps13state.c#L304)
<p style="text-align: center;">
$$X_{k+1}^{-} = f(X_{k}, U_{k})$$
</p>

和PX4不同，这里没有各新息数据到来时分批处理，而是只运行一次预测

其中地系速度由机体加速度经过四元数旋转得到，新的四元数由前时刻四元数经过机体角速度旋转得到

### 系统误差协方差预测

系统误差协方差预测需要把非线性方程线性化，计算对时间一阶导和状态偏分

包括状态变换矩阵$$F$$与系统噪声传递矩阵$$G$$

#### F阵线性化

- 地系位置对地系速度 为单位阵

- 地系速度对四元数

<p style="text-align: center;">
$$\frac{\partial \dot f_{v}}{\partial q}=\left[\begin{array}{cccc}
2\left(q_{0}a_{x}-q_{3}a_{y}+q_{2}a_{z}\right) & 2\left(q_{1}a_{x}-q_{2}a_{y}+q_{3}a_{z}\right) & 2\left(q_{2}a_{x}+q_{1}a_{y}+q_{0}a_{z}\right) & 2\left(q_{3}a_{x}-q_{0}a_{y}+q_{1}a_{z}\right) \\
2\left(q_{3}a_{x}+q_{0}a_{y}-q_{1}a_{z}\right) & 2\left(q_{2}a_{x}-q_{1}a_{y}-q_{0}a_{z}\right) & 2\left(q_{1}a_{x}+q_{2}a_{y}+q_{3}a_{z}\right) & 2\left(q_{0}a_{x}-q_{3}a_{y}+q_{2}a_{z}\right) \\
2\left(-q_{2}a_{x}+q_{1}a_{y}+q_{0}a_{z}\right) & 2\left(q_{3}a_{x}+q_{0}a_{y}-q_{1}a_{z}\right) & 2\left(-q_{0}a_{x}+q_{3}a_{y}-q_{2}a_{z}\right) & 2\left(q_{1}a_{x}+q_{2}a_{y}+q_{3}a_{z}\right)
\end{array}\right]$$
</p>

- 四元数对四元数 并用小角度三角函数近似为

<p style="text-align: center;">
$$\frac{\partial \dot f_{q}}{\partial q}=\left[\begin{array}{cccc}
0 & -\frac{\omega_{x}}{2} & -\frac{\omega_{y}}{2} & -\frac{\omega_{z}}{2} \\
\frac{\omega_{x}}{2} & 0 & \frac{\omega_{z}}{2} & -\frac{\omega_{y}}{2} \\
\frac{\omega_{y}}{2} & -\frac{\omega_{z}}{2} & 0 & \frac{\omega_{x}}{2} \\
\frac{\omega_{z}}{2} & \frac{\omega_{y}}{2} & -\frac{\omega_{x}}{2} & 0
\end{array}\right]$$
</p>

- 四元数对陀螺仪角速度偏移量

来自四元数更新方程，不赘述

#### G阵线性化
考虑系统误差向量

<p style="text-align: center;">
$$Q_{x}=\left[\begin{array}{l}
 g_{x}, g_{y}, g_{z}, a_{x}, a_{y}, a_{z}, gb_{x}, gb_{y}, gb_{z}
\end{array}\right]^T$$
</p>

考虑系统误差传递矩阵$$G$$，描述部分状态变量存在系统误差，对其他状态分量传递作用

- 四元数来自陀螺仪角速度的影响

<p style="text-align: center;">
$$\frac{\partial \dot f_{q}}{\partial gyro}=\left[\begin{array}{ccc}
\frac{q_{1}}{2} & \frac{q_{2}}{2} & \frac{q_{3}}{2} \\
-\frac{q_{0}}{2} & \frac{q_{3}}{2} & -\frac{q_{2}}{2} \\
-\frac{q_{3}}{2} & -\frac{q_{0}}{2} & \frac{q_{1}}{2} \\
\frac{q_{2}}{2} & -\frac{q_{1}}{2} & -\frac{q_{0}}{2}
\end{array}\right]$$
</p>

- 四元数来自加速度计的影响

<p style="text-align: center;">
$$\frac{\partial \dot f_{q}}{\partial acc}=\left[\begin{array}{ccc}
\left(-q_{0}^2-q_{1}^2+q_{2}^2+q_{3}^2\right) & 2\left(-q_{1}a_{2}+q_{0}q_{3}\right) & -2\left(q_{1}q_{3}+q_{0}q_{2}\right) \\
-2\left(q_{1}q_{2}+q_{0}q_{3}\right) & \left(-q_{0}^2+q_{1}^2-q_{2}^2+q_{3}^2\right) & 2\left(-q_{2}q_{3}+q_{0}q_{1}\right) \\
2\left(-q_{1}q_{3}+q_{0}q_{2}\right) & -2\left(q_{2}q_{3}+q_{0}q_{1}\right) & \left(-q_{0}^2+q_{1}^2+q_{2}^2-q_{3}^2\right)
\end{array}\right]$$
</p>

则
<p style="text-align: center;">
$$P_{k+1}^{-}=F_{k} P_{k} F_{k}^{T}+G_{k} Q_{k} G_{k}^{T}$$
</p>

## 后验校正

其他传感器数据到来，结合观测方程对先验估计做校正

### 计算卡尔曼增益

<p style="text-align: center;">
$$K_{k+1}=P_{k}^{-} H_{k}^{T}(H_{k} P_{k}^{-} H_{k}^{T} + V_{k})$$
</p>

这里对观测方程非线性$$h$$函数做线性化，思路同上可参考[代码](https://github.com/SantyPilot/SantyPilot/blob/master/flight/libraries/insgps13state.c#L827)
不赘述

### 后验估计状态

<p style="text-align: center;">
$$X_{k+1}=X_{k}^{-} + K_{k} (y_{k}-h(X_{k}^{-}, U_{k}))$$
</p>

### 后验估计系统噪声协方差

<p style="text-align: center;">
$$P_{k+1}=(I - K_{k} H_{k}) P_{k}^{-}$$
</p>
