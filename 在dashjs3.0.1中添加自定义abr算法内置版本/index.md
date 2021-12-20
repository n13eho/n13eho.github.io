# 在dash.js 3.0.1中添加自定义ABR算法——内置版本


师兄之前已经写过了在CustomRules那边[添加自定义ABR算法](https://blog.csdn.net/LvGreat/article/details/114790362?spm=1001.2014.3001.5502)的文章，并且能够用。这里从它内置的入手，看下能否做到

## 1. 让面板选项对应的ABR算法是唯一生效/工作的算法

### 先看看dashjs默认调用的ABR逻辑

参照[v4.1.0的abr调用栈](https://blog.csdn.net/LvGreat/article/details/121252622?spm=1001.2014.3001.5501)，v3.0.1的ABR调用栈丝毫没变：

1️⃣【ScheduleController】schedule()

- 2️⃣【AbrController】checkPlaybackQuality(type)
  - 3️⃣【ABRRulesCollection】getMaxQuality(rulesContext)
    - 4️⃣【各ABR算法】getMaxIndex(rulesContext)

到了第四层就是每一个ABR算法具体的内容，但是这里不深入。看关键的第三层，位于`/src/streaming/rules/abr/ABRRulesCollection.js`中的`getMaxQuality(rulesContext)`函数

```javascript
function getMaxQuality(rulesContext) {
    const switchRequestArray = qualitySwitchRules.map(rule => rule.getMaxIndex(rulesContext));
    const activeRules = getActiveRules(switchRequestArray);
    const maxQuality = getMinSwitchRequest(activeRules);

    return maxQuality || SwitchRequest(context).create();
}
```

该函数的返回值即为下一个视频块应有的质量quality。将他们都打印出来之后可以直观的得到：`qualitySwitchRules`中装载的是当前策略下的所有ABR算法，通过array的.map属性，每个AbrRule的结果存放在`switchRequestArray`中；activeRules则进一步过滤掉了相交于上一次没有质量变化的switchRequest（这是他们的返回值）；最后经过`getMinSwitchRequest`选择出active生效的Q中最小的一个。（maybe不太对，这个函数还没细看）

所以显而易见，需要改造的是到这一步之前，**qualitySwitchRules**的内容

### 按选择修改`qualitySwitchRules`

`qualitySwitchRules`在最开始初始化的时候就被定义好了，函数`initialize()`中，`qualitySwitchRules`一来就push了5个进去，也就是说默认的算法组里面有`BolaRule`、`ThroughputRule`、`InsufficientBufferRule`、`SwitchHistoryRule`和`DroppedFramesRule`这五种

```javascript
function initialize() {
    qualitySwitchRules = []; // 是个list
    abandonFragmentRules = [];

    // qualitySwitchRules一来就push了5个进去，也就是说默认的算法组里面有BolaRule、ThroughputRule、InsufficientBufferRule、SwitchHistoryRule和DroppedFramesRule这五种
    if (settings.get().streaming.abr.useDefaultABRRules) {
        // Only one of BolaRule and ThroughputRule will give a switchRequest.quality !== SwitchRequest.NO_CHANGE.
        // This is controlled by useBufferOccupancyABR mechanism in AbrController.
        qualitySwitchRules.push(
            BolaRule(context).create({
                dashMetrics: dashMetrics,
                mediaPlayerModel: mediaPlayerModel,
                settings: settings
            })
        );
        qualitySwitchRules.push(
            ThroughputRule(context).create({
                dashMetrics: dashMetrics
            })
        );
        qualitySwitchRules.push(
            InsufficientBufferRule(context).create({
                dashMetrics: dashMetrics
            })
        );
        qualitySwitchRules.push(
            SwitchHistoryRule(context).create()
        );
        qualitySwitchRules.push(
            DroppedFramesRule(context).create()
        );
        abandonFragmentRules.push(
            AbandonRequestsRule(context).create({
                dashMetrics: dashMetrics,
                mediaPlayerModel: mediaPlayerModel,
                settings: settings
            })
        );
    }

    // add custom ABR rules if any  —————— 这里是customABRRules添加进来的地方，不动
    ......
}
```

那么就依据面板选择，重写这里的逻辑。v3.0.1也是通过修改setting进行设定的，这里的接口是：settings.get().streaming.abr.ABRStrategy，类型是字符串。因此可以做一个简单的判断语句，按情况向这个list中放入rules

![](https://gitee.com/tanneho/pic/raw/master/img/202112191515653.png)

这样可以确保rules执行的唯一性

## 2. 🧐仿照内置格式写一个rule放进去（later）
