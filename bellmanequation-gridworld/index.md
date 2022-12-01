# Bellman Equation: Gridworld


## 问题描述

游戏规则：正常移动一次的奖励为0；出界时回到当前位置，奖励为-1；A位置选择任意方向都到达A’，奖励为+10；B位置选择任意方向都到达B’，奖励为+5； 折扣率y = 0.9。

假定策略为；从每个格子等概率选择四个移动方向 （行为）

![](https://nehopicbed.oss-cn-beijing.aliyuncs.com/img/202212012302219.png)

用Bellman Equation复现右边5*5的状态估值

## 解

记号规定：obj移动到左上角的状态：s(0, 0)/s(0)，移动到右下角的状态：s(4, 4)/s(24)

带入Bellman Equation：

$$
v_{\pi}(s) = \sum_{a}\pi(a|s)\sum_{s', r}p(s', r|s, a)[r + \gamma v_{\pi}s']
$$

对于在位置(0, 0)的状态：

$$
\begin{align}
v_{\pi}(0) =
 &\frac{1}{4}\sum_{s', r}p(s', r|s, a)[r + \gamma v_{\pi}s(1)]\quad向右走\\
+&\frac{1}{4}\sum_{s', r}p(s', r|s, a)[r + \gamma v_{\pi}s(😫)]\quad向左走\\
+&\frac{1}{4}\sum_{s', r}p(s', r|s, a)[r + \gamma v_{\pi}s(5)]\quad向下走\\
+&\frac{1}{4}\sum_{s', r}p(s', r|s, a)[r + \gamma v_{\pi}s(😫)]\quad向上走\\


\end{align}
$$

对于这里的

$$
\sum_{s', r}p(s', r|s, a)[r + \gamma v_{\pi}s']
$$

除了特殊位置A和B外，其余都是`0 * [] + 0 * [] + 0 * [] + 1 * [0 + 0.9 * 下一个状态的估值]`，所以对于位置(0, 0)：

$$
\begin{align}
v_{\pi}(0) =
 &\frac{1}{4}[0 + 0.9  v_{\pi}s(1)]\quad向右走\\
+&\frac{1}{4}[-1 + 0.9 v_{\pi}s(0)]\quad向左走\\
+&\frac{1}{4}[0 + 0.9 v_{\pi}s(5)]\quad向下走\\
+&\frac{1}{4}[-1 + 0.9 v_{\pi}s(0)]\quad向上走\\
\end{align}
$$
