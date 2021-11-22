# Nodejs应用——dashjs的环境配置和相关问题解决




## 一、在服务器上安装nodejs

1. 服务器os为ubuntu，因此去到[nodejs官网](https://nodejs.org/en/download/)下载对应的`node-v15.12.0-linux-x64.tar.xz`，我下载的nodejs版本为15。

   ```bash
   wget www.downloadlinkof.nodejs
   ```

2. 解压

   ```bash
   tar -xvf node-v15.12.0-linux-x64.tar.xz
   ```

3. 重命名和创建软连接：注意这里的`/path/to/nodejs/bin/node`和`/path/to/nodejs/bin/npm`分别是到nodejs下node和npm的存放路径

   ```bash
   mv node-v15.12.0-linux-x64 nodejs
   ln -s /path/to/nodejs/bin/node /usr/local/bin/node
   ln -s /path/to/nodejs/bin/npm /usr/local/bin/npm
   ```

4. 安装完成之后进行验证

   ```bash
   node -v
   v15.12.0
   ```

## 二、首次编译

### 源文件不够，编译失败

这里是由于服务器上的dashjs原本不是照搬源码，因此有一些配置文件并没有全部放进来。所以再将源包中的`src/`、`package.json`、`githook.js`、`eslintrc`等缺失的文件原封不动的放进去即可

### 环境不完整

1. 找不到tsc

   ```bash
   npm install typescript -g
   ```

   此时仍然找不到tsc，参考stackoverflow上的一个[评论](https://stackoverflow.com/questions/39404922/tsc-command-not-found-in-compiling-typescript)，需要将nodejs的bin路径加到环境变量中去

   ```bash
   export PATH=/prefixsPath/bin:$PATH
   ```

   其中`/prefix'sPath/bin`是`nodejs/bin`的路径，即`export PATH=/root/nodejs/bin:$PATH`

   注意export只会在当前终端结束之前生效，为了避免每次都重复打这一行命令，应该直接改对所有用户生效的环境变量

   ```bash
   sudo vim /etc/profile
   ```

   添加export这一句话到末尾，然后再source一下

   ```bash
   source /etc/profile
   ```

2. rimraf

   ```bash
   sudo apt-get install
   ```

3. webpack

   ```bash
   npm install -g webpack
   npm i -g webpack-cli
   ```

### JavaScript heap out of memory 内存溢出

👉[参考](https://stackoverflow.com/questions/53230823/fatal-error-ineffective-mark-compacts-near-heap-limit-allocation-failed-javas)，和一些待后续学习的方法：[Debugging Memory Leaks in Node.js Applications](https://www.toptal.com/nodejs/debugging-memory-leaks-node-js-applications)

```bash
export NODE_OPTIONS="--max-old-space-size=512"
```

也添加到`/etc/profile`中



---



至此，dashjs环境上的问题告一段落。

后续todo：dashjs上的内存泄漏都在哪里——nodejs的内存泄漏调试办法



> 后记：我后来在树莓派4B上试了试整个环境的配置，全程丝滑顺利，让我怀疑方向是否错了？

<img src="https://gitee.com/tanneho/pic/raw/master/img/202111211818982.png" style="zoom: 80%;" />
