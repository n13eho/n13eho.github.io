# 跑SIGCOMM21-5G仿真


仓库地址：[artifact/Video-Streaming/ABR-5G/run_exp at main · SIGCOMM21-5G/artifact · GitHub](https://github.com/SIGCOMM21-5G/artifact/tree/main/Video-Streaming/ABR-5G/run_exp)

## build up环境配置

### 1 修改全局的python版本，注意pip也得要

### 2 修改网卡的interface

在setup.sh中将变量interface改为自己网卡的硬件名称，可以通过命令`ifconfig`进行查看

### 3 Error: Cannot find specified qdisc on specified device.

可以认为是伪报错，因为本身不影响后续操作并且有时候自己就没了，try：

1. 单独执行一遍sudo bash setup.sh

2. type `infconfig`，发现会多了几个“网卡”

### 4 from pyvirtualdisplay import Display的SyntaxError

一连串的import到了from easyprocess import EasyProcess, EasyProcessError这里就有一个python2.7不支持的py3的语法：[typing类型注解](https://docs.python.org/zh-cn/3/library/typing.html)

这里的报错信息如下，就是在EasyProcessError这里出现了py3的语法。其实不应该啊，这个包在python2.7/下面

<img src="https://nehopicbed.oss-cn-beijing.aliyuncs.com/img/202212191126097.png" title="" alt="" data-align="center">

到了/usr/local/lib/python2.7/dist-packages/easyprocess/examples打开hello.py一看里面用的就是py3的语法。所以现在要把这个包换成2.7的。先用sudo pip uninstall卸载了，再安装。注意默认是最新版本1.1，也就是py3，上[官网](https://pypi.org/project/EasyProcess/#history)看一下之前有哪些版本，挑一个2020年左右的：0.2.9

```bash
pip install easyprocess==0.2.9
```

至此可以安心使用py2.7了

### 5 google chrome and its driver

选一个低版本的match得上的，卸载chrome：

```bash
sudo apt-get remove google-chrome-stable
```

再从外部下载deb进行安装

## 代码梳理

在run_exp/文件夹下运行bash run_5g_driving.sh（也有其他场景的sh）。一个这样的sh文件中的内容是一个for循环遍历../cooked_traces_driving这个文件夹下的所有trace文件，也就是一个背景流量。在循环体中对每一个trace文件都用不同的ABR算法跑一边。所以最小的一个测试单元是：

```bash
bash trace_run.sh ${trace_file} > ./bw_truth/bw_robustMPC_${filename} &
BACK_PID=$!
/usr/bin/python run_video.py ${server_ip} robustMPC 200 0 ${filename} 6
kill ${BACK_PID} > /dev/null 2>&1
kill $(ps aux | grep _server | awk '{print $2}') > /dev/null 2>&1
```

首先用trace_run.sh按照trace文件中的具体数值设置背景网络环境，trace_run.sh中使用的主要命令是`tc qdisc`，这里的输入参数是trace文件的路径；另外第一句末尾的&是bash语法，意味调用子进程异步执行这条命令，是后台任务。紧接着用BACK_PI存放这个后台任务的进程号。第三行就是执行run_video.py脚本，这里面的内容自然就包括了一次测试剩下的内容：打开客户端，请求时评播放，记录output；剩下两句kill是杀死本次仿真的进程，以便进行下一个算法的测试。

run_video.py

## 代码逻辑遇到的问题

### 1 server ip地址和chrome报错

遇到这样一个问题：

<img src="https://nehopicbed.oss-cn-beijing.aliyuncs.com/img/202212191239567.png" title="" alt="" data-align="left">

两方版本都是能对得上的，然而还是有unknown error。看代码
