# 给dash.js 3.0.1中内置BolaRule加上输出


## 1. 在树莓派上配置3.0.1所需环境
### 安装grunt

grunt是前几年使用的一个打包工具吧，现在dashjs的项目用的是webpack了。安装grunt很简单，只需要一句

```bash
npm install -g grunt-cli 
```

当然需要提前配好nodejs环境。安装完之后验证

<img src="https://gitee.com/tanneho/pic/raw/master/img/202112181559902.png"  />

### gurnt装不上怎么办

{{< admonition >}}
这里可能会出现找不到grunt的失败提示，那么多半是npm/bin的路径没有添加到环境变量中去。使用熟悉的

```bash
sudo vim /etc/profile
```

将这句话添加到结尾

```bash
export PATH=/root/nodejs/bin:$PATH
```

<img src="https://gitee.com/tanneho/pic/raw/master/img/202112181606424.png" style="zoom:67%;" />

最后记得再source一下

```bash
source /etc/profile
```

{{< /admonition >}}

至此环境就配好了

## 2. 正确添加内置Bola的输出量

### 项目install准备

[官方文档](https://github.com/Dash-Industry-Forum/dash.js/tree/v3.0.1#quick-start-for-developers)说的很清楚，这里过一遍

```bash
npm install # 注意这一步是必须要求grunt时已经安装好了的，否则就会一直被一个超时问题给卡住🙄
grunt debug
```

![](https://gitee.com/tanneho/pic/raw/master/img/202112181625688.png)

用grunt dev可以看得到我们的html目录在哪里

![](https://gitee.com/tanneho/pic/raw/master/img/202112181628209.png)

### 网站上线

通过一些顺藤摸瓜，可以得到访问目标是`http://192.168.1.167:3002/samples/dash-if-reference-player/index.html`，因此就应该把ngnix网站根目录改为当前的dashjs-301就好了

```bash
sudo vim /etc/nginx/sites-available/default # root /home/pi/dashjs-301;
sudo service nginx restart
```

访问`http://192.168.1.167/samples/dash-if-reference-player/index.html`即可



完了之后把获取视频的地址改为http://10.61.140.191:3456/video-src/bbb-4s-an.mpd，这里用路由器做了一些端口转发，直接访问的是路由器的ip。（用内网突然访问不到了，所以换成这个

### 测试输出

这里就随便在/src/streaming/rules/abr/BolaRule.js中输出一些东西，打包，运行即可。验证的确是执行了Bola算法了的即可。

![](https://gitee.com/tanneho/pic/raw/master/img/202112182237372.png)

### 添加输出量

{{< admonition >}}

这里放另外一篇（《在dash.js 3.0.1中添加自定义ABR算法——内置版本》）的子标题的链接：让面板选项对应的ABR算法是唯一生效/工作的算法

前提：让工作的只有BolaRule唯一一个内置算法，因为原生的dashjs的内置算法是混合着算的，详见[这篇](https://blog.csdn.net/LvGreat/article/details/103735968?spm=1001.2014.3001.5501)

{{< /admonition >}}

然后进行这些物理量的输出：

```javascript
/**Parameters
 * @param {string} mediaType mediaType - 
 * @param {number} last_last_bitrate bolaState.lastQuality - 上一次选择的quality
 * @param {number} bufferLevel bufferLevel - [call dashMetric API]
 * @param {number} latency latency_cus - [call dashMetric API] the time in seconds from request of segment to receipt of first byte
 * @param {number} downloadTime downloadTime - the time in seconds from first byte being received to the last byte
 * @param {number} idleTime idleTime - ?
 * @param {number} rebufferTime rebufferTime - 
 * @param {number} chunkSize chunkSize - 
 * @param {number} throughput throughput_cus - 
 * @param {number} app_throughput app_throughput - 
 * @param {number} time_request time_request - 
 * @param {number} time_response time_response - 
 * @param {number} time_finish time_finish - 
 * @param {number} cur_reward cur_reward - QoE
 */
```

检查一下输出来的chunksize是否正常，这个版本的fastSwitch是没有bug的

## 上服务器

树莓派上都能跑了的话，就把这一套搬到Ubuntu服务器上去。

