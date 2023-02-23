# Bellman Equation: Gridworld


## 问题描述

游戏规则：正常移动一次的奖励为0；出界时回到当前位置，奖励为-1；A位置选择任意方向都到达A’，奖励为+10；B位置选择任意方向都到达B’，奖励为+5； 折扣率y = 0.9。

假定策略为；从每个格子等概率选择四个移动方向（行为）

<div align=center><img src="https://nehopicbed.oss-cn-beijing.aliyuncs.com/img/202212012302219.png" width="1300"></div>

用Bellman Equation复现右边5*5的状态估值

## 解

记号规定：obj移动到左上角的状态：s(0, 0)/s(0)，移动到右下角的状态：s(4, 4)/s(24)

带入Bellman Equation：

<div align=center><img src="https://nehopicbed.oss-cn-beijing.aliyuncs.com/img/202212020131357.png" width="400"></div>

对于在位置(0, 0)的状态：

<div align=center><img src="https://nehopicbed.oss-cn-beijing.aliyuncs.com/img/202212020132263.png" width="400"></div>

对于这里的

<div align=center><img src="https://nehopicbed.oss-cn-beijing.aliyuncs.com/img/202212020133409.png" width="300"></div>

除了特殊位置A和B外，其余都是`0 * [] + 0 * [] + 0 * [] + 1 * [0 + 0.9 * 下一个状态的估值]`，所以对于位置(0, 0)：

<div align=center><img src="https://nehopicbed.oss-cn-beijing.aliyuncs.com/img/202212020134900.png" width="400"></div>

以此类推其余地方的点都一样，除了A点s(1)和B点s(3)

<div align=center><img src="https://nehopicbed.oss-cn-beijing.aliyuncs.com/img/202212020134499.png" width="200"></div>

然后就是matlab coding得解：

```matlab
syms v0 v1 v2 v3 v4 v5 v6 v7 v8 v9 v10 v11 v12 v13 v14 v15 v16 v17 v18 v19 v20 v21 v22 v23 v24;

eqns=[
    v0 == (9 / 40) * v1 + 9 / 20 * v0 + (9 / 40) * v5 - 0.5,
    v1 == 10 + 0.9 * v21,
    v2 == (9 / 40) * (v3 + v1 + v2 + v7) - 0.25,
    v3 == 5 + 0.9 * v13,
    v4 == (9 / 40) * (v3 + v9 + 2 * v4) - 0.5,
    v5 == (9 / 40) * (v5 + v6 + v0 + v10) - 0.25,
    v6 == (9 / 40) * (v1 + v5 + v7 + v11),
    v7 == (9 / 40) * (v2 + v6 + v8 + v12),
    v8 == (9 / 40) * (v3 + v7 + v9 + v13),
    v9 == (9 / 40) * (v4 + v8 + v9 + v14) - 0.25,
    v10 == (9 / 40) * (v5 + v10 + v11 + v15) - 0.25,
    v11 == (9 / 40) * (v6 + v10 + v12 + v16),
    v12 == (9 / 40) * (v7 + v11 + v13 + v17),
    v13 == (9 / 40) * (v8 + v12 + v14 + v18),
    v14 == (9 / 40) * (v9 + v13 + v14 + v19) - 0.25,
    v15 == (9 / 40) * (v15 + v10 + v16 + v20) - 0.25,
    v16 == (9 / 40) * (v11 + v15 + v17 + v21),
    v17 == (9 / 40) * (v12 + v16 + v18 + v22),
    v18 == (9 / 40) * (v13 + v17 + v19 + v23),
    v19 == (9 / 40) * (v14 + v18 + v19 + v24) - 0.25,
    v20 == (9 / 40) * (v15 + v21 + 2 * v20) - 0.5,
    v21 == (9 / 40) * (v16 + v20 + v21 + v22) - 0.25,
    v22 == (9 / 40) * (v17 + v21 + v22 + v23) - 0.25,
    v23 == (9 / 40) * (v18 + v22 + v23 + v24) - 0.25,
    v24 == (9 / 40) * (v19 + v23 + 2 * v24) - 0.5,
    ]; 
vars=[v0, v1, v2, v3, v4, v5, v6, v7, v8, v9, v10, v11, v12, v13, v14, v15, v16, v17, v18, v19, v20, v21, v22, v23, v24];
ret=solve(eqns,vars);
ret = [
[roundn(double(ret.v0),-1), roundn(double(ret.v1),-1) ,roundn(double(ret.v2),-1), roundn(double(ret.v3),-1), roundn(double(ret.v4),-1)],
[roundn(double(ret.v5),-1), roundn(double(ret.v6),-1) ,roundn(double(ret.v7),-1), roundn(double(ret.v8),-1), roundn(double(ret.v9),-1)],
[roundn(double(ret.v10),-1), roundn(double(ret.v11),-1) ,roundn(double(ret.v12),-1), roundn(double(ret.v13),-1), roundn(double(ret.v14),-1)],
[roundn(double(ret.v15),-1), roundn(double(ret.v16),-1) ,roundn(double(ret.v17),-1), roundn(double(ret.v18),-1), roundn(double(ret.v19),-1)],
[roundn(double(ret.v20),-1), roundn(double(ret.v21),-1) ,roundn(double(ret.v22),-1), roundn(double(ret.v23),-1), roundn(double(ret.v24),-1)]
]
```

但是结果出来ret.v0=211544423854643298169167140/63930087070970054436332951，G啊，咋回事？

---

噢噢看错了，是对的，中间还有个除号，下面是最后的结果：

<div align=center><img src="https://nehopicbed.oss-cn-beijing.aliyuncs.com/img/202212020129949.png" width="600"></div>

对得上
