# dash.js如何调用某一具体ABRRule（未完）






## 一、定位：load按钮之后的逻辑

### doLoad(): `./samples/dash-if-reference-player/app/main.js`

选择元素，定位到Load按钮上，它对应的html元素中，`ng-click="doLoad()"`中的ng-click是AngularJS的一种指令，后面跟的是普通的expression表达式，语法规则为：

```html
<element ng-click="expression"></element>
```

<img src="https://gitee.com/tanneho/pic/raw/master/img/202111262228745.png" style="zoom: 80%;" />

即点击就会执行doLoad函数。这个函数定义在main.js中，那么它干了什么呢。这条路我昨天找没找通，是不是应该反过来找啊！
