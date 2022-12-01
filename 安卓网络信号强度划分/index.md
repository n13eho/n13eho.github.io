# （未完）安卓网络信号强度划分


{{< admonition abstract >}}

结论：

WIFI：按RSSI分为五档。0: (-∞, -88)，1: [-88, -77)，2: [-77, -66)，3: [-66, -55)，4: [-55, +∞)

4G：

5G：

{{< /admonition >}}

## WIFI

### 关键源码：[link](http://aospxref.com/android-12.0.0_r3/xref/packages/modules/Wifi/framework/java/android/net/wifi/WifiManager.java#3677)

```java
public static int calculateSignalLevel(int rssi, int numLevels) {
    if (rssi <= MIN_RSSI) {
        return 0;
    } else if (rssi >= MAX_RSSI) {
        return numLevels - 1;
    } else {
        float inputRange = (MAX_RSSI - MIN_RSSI);
        float outputRange = (numLevels - 1);
        return (int)((float)(rssi - MIN_RSSI) * outputRange / inputRange);
    }
}
```

第一个参数就是rssi，整型；第二个参数是分级总级数，在安卓中一共分为0~4，5级，因此第二个参数传入值通常为5。而对于`MIN_RSSI`和`MAX_RSSI`，在此.java中也给了值：

```java
/** Anything worse than or equal to this will show 0 bars. */
private static final int MIN_RSSI = -100;

/** Anything better than or equal to this will show the max bars. */
private static final int MAX_RSSI = -55;

public static final int RSSI_LEVELS = 5;
```

### 结论

不难算出这5档的范围为：

- 0: (-∞, -88)
- 1: [-88, -77)
- 2: [-77, -66)
- 3: [-66, -55)
- 4: [-55, +∞)

{{< admonition info >}}
对于2.4GHz和5GHz，虽然都使用这同一套划分方法，但是由于距离AP同样距离时，两个频段所得到的rssi值是不一样的。
{{< /admonition >}}

## 4G

### RSRP

4GLTE的决策变量不再主要是RSSI了，还有另外一个比较重要的物理量RSRP，一些[前置知识](https://blog.csdn.net/fun_tion/article/details/100708167)

### 源码分析

在[SignalStrength.java](http://aospxref.com/android-12.0.0_r3/xref/frameworks/base/telephony/java/android/telephony/SignalStrength.java)line758，getLteLevel()获取了LTE的level

```java
public int getLteLevel() {
    return mLte.getLevel();
}
```

这个方法源自于[CellSignalStrengthLte.java](http://aospxref.com/android-12.0.0_r3/xref/frameworks/base/telephony/java/android/telephony/CellSignalStrengthLte.java)line230

```java
public int getLevel() {
    return mLevel;
}
```

这里是直接返回了`mLevel`这个全局变量。

在方法`setDefaultValues()`中，对它进行了初始化，值为`SIGNAL_STRENGTH_NONE_OR_UNKNOWN`，不难发现还有另外一个方法在更新信号的level， `updateLevel(PersistableBundle cc, ServiceState ss)`，line275

### 结论

## 5G
