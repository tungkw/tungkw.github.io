{%- inclade header_latex.html -%}

# 定义

- 状态 $S_{0:k} = \left\{s_0, s_1, \cdots, s_k\right\}$
- 动作 $A_{0:k} = \left\{a_0, a_1, \cdots , a_k\right\}$
- 观测 $O_{0:k} = \left\{o_0, o_1, \cdots , o_k\right\}$
- 路标 $L = \left\{l_0, l_1, \cdots , l_n\right\}$

# 问题

给定：观测序列，动作序列，初始状态
求解：当前状态，路标

$$
p(s_k, L | O_{0:k}, A_{0:k}, s_0)
$$

将$s_k$和$L$合并成$x_k$

$$
p(x_k | O_{0:k}, A_{0:k}, x_0) \sim \mathcal{N} (\mu_k, \Sigma_k)
$$

# 推导

## Bayes定理

最新观测下的后验概率，转为求最新观测的似然 $\times$ 先验概率

$$
\begin{aligned}
& p(x_k | O_{0:k}, A_{0:k}, x_0) \\
\propto \ & p(o_k | O_{0:k-1}, A_{0:k}, x_k, x_0) \cdot p(x_k | O_{0:k-1}, A_{0:k}, x_0)
\end{aligned}
$$

## 先验概率

### 马尔科夫性

以$k-1$时刻状态的条件概率展开先验概率

$$
p(x_k | O_{0:k-1}, A_{0:k}, x_0) = \int p(x_k | x_{k-1}, O_{0:k-1}, A_{0:k}, x_0) \cdot p(x_{k-1} | O_{0:k-1}, A_{0:k}, x_0) \mathrm{d}x_{k-1}
$$

假设当前状态之和前一时刻状态、当前运动有关

$$
p(x_k | x_{k-1}, O_{0:k-1}, A_{0:k}, x_0) = p(x_k | a_k, x_{k-1})
$$


### 高斯线性运动模型（predict step）

$$
x_k = W^x_k x_{k-1} + a_k + b^x_k
$$

使用均值作为前一时刻状态的估计 $x_{k-1} = \mu_{k-1}$

$$
\begin{aligned}
x_k =& W^x_k \mu_{k-1} + a_k + b^x_k \\
\sim& \mathcal{N} (W^x_k \mu_{k-1} + a_k, W^x_k \Sigma_{k-1} {W^x_k}^T + \Sigma^{b^x}) \\
=& \mathcal{N} (\check{\mu}_k, \check{\Sigma}_k)
\end{aligned} 
$$

则先验概率的积分中第一部分和$x_{k-1}$无关

$$
\begin{aligned}
p(x_k | O_{0:k-1}, A_{0:k}, x_0) &= \int p(x_k | a_k, x_{k-1}) \cdot p(x_{k-1} | O_{0:k-1}, A_{0:k}, x_0) \mathrm{d}x_{k-1} \\
&= p(x_k | a_k, x_{k-1}) \int p(x_{k-1} | O_{0:k-1}, A_{0:k}, x_0) \mathrm{d}x_{k-1} \\
&= p(x_k | a_k, x_{k-1})
\end{aligned}
$$


## 似然

### 观测独立

每次观测独立，且和动作无关

$$
p(o_k | O_{0:k-1}, A_{0:k}, x_k, x_0) = p(o_k | x_k)
$$

### 高斯线性观测模型

$$
\begin{aligned}
o_k =& W^o_k x_k + b^o_k \\
\sim& \mathcal{N} (W^o_k x_k, \Sigma^{b^o})
\end{aligned}
$$


## 后验概率

两个高斯的乘积仍为高斯

$$
\begin{aligned}
& p(x_k | O_{0:k}, A_{0:k}, x_0) \\
\propto \ & p(o_k | x_k) \cdot p(x_k | a_k, x_{k-1})
\end{aligned} 
$$

配平两边指数部分

$$
\begin{aligned}
&(x_k - \mu_k)^T \Sigma^{-1}_k (x_k - \mu_k) \\
=& (o_k - W^o_k x_k)^T (\Sigma^{b^o})^{-1} (o_k - W^o_k x_k) \\
+& (x_k - \check{\mu}_{k-1})^T \check{\Sigma}_k^{-1} (x_k - \check{\mu}_{k-1})
\end{aligned}
$$

### 二次项系数有

$$
\Sigma^{-1}_k = {W^o_k}^T (\Sigma^{b^o})^{-1} W^o_k + \check{\Sigma}_k^{-1}
$$

设（卡尔曼增益）

$$
\begin{aligned}
K =& \Sigma_k {W^o_k}^T (\Sigma^{b^o})^{-1} \\
(woodbary\ identity) =& \check{\Sigma}_k {W^o_k}^T (W^o_k \check{\Sigma}_k {W^o_k}^T + \Sigma^{b^o})^{-1}
\end{aligned}
$$

则有

$$
\begin{aligned}
\Sigma^{-1}_k &= {W^o_k}^T (\Sigma^{b^o})^{-1} W^o_k + \check{\Sigma}_k^{-1} \\

I &= \Sigma_k {W^o_k}^T (\Sigma^{b^o})^{-1} W^o_k + \Sigma_k \check{\Sigma}_k^{-1} \\

I &= K W^o_k + \Sigma_k \check{\Sigma}_k^{-1} \\

\Sigma_k &= (I - K W^o_k) \check{\Sigma}_k
\end{aligned}
$$

### 一次项系数

$$
\begin{aligned}
- \mu_k^T \Sigma^{-1}_k x_k 
=& - o_k^T (\Sigma^{b^o})^{-1} W^o_k x_k - \check{\mu}_{k-1}^T \check{\Sigma}_k^{-1} x_k \\

\Sigma^{-1}_k \mu_k =& {W^o_k}^T (\Sigma^{b^o})^{-1} o_k + \check{\Sigma}_k^{-1} \check{\mu}_{k-1} \\

\mu_k =& \Sigma_k {W^o_k}^T (\Sigma^{b^o})^{-1} o_k + \Sigma_k \check{\Sigma}_k^{-1} \check{\mu}_{k-1} \\

\mu_k =& K o_k + (I - K W^o_k)\check{\mu}_{k-1} \\

\mu_k =& \check{\mu}_{k-1} + K (o_k - W^o_k \check{\mu}_{k-1})

\end{aligned}
$$

### 整合（update step）


$$
\begin{aligned}

K =& \check{\Sigma}_k {W^o_k}^T (W^o_k \check{\Sigma}_k {W^o_k}^T + \Sigma^{b^o})^{-1} \\

\mu_k =& \check{\mu}_{k-1} + K (o_k - W^o_k \check{\mu}_{k-1}) \\

\Sigma_k =& (I - K W^o_k) \check{\Sigma}_k

\end{aligned}
$$


# 性质

由于k时刻机器人的位置估计有偏差，对应的地图估计的绝对位置也会有偏差，但是同一次观测中的两个地标之间的相对位置是比较准确的，形成一种相关性（correlation）。再之后的观测中，对同一地标的更新会顺着之前建立的相关性，更新新的观测中没有感知到的旧路标。随着这种基于相关性的图越来越壮大，地标的位置估计，即地图的构建将收敛。而机器人的位置估计则将简化成在已知地图中的定位。