---
title: 用 canvas 实现黑客帝国中的数码雨
date: 2016-02-26 15:13:09
tags:
  - canvas
  - js
---

> 你有过这种感觉没有，就是你吃不准自己是醒着还是在做梦。

{% asset_img 黑客帝国.jpg 黑客帝国 %}

《黑客帝国》这部经典科幻电影相信大家都看过，而其中的数码雨效果也是让人印象深刻。今天，我们就用 `Canvas` 来实现这个数码雨效果。

# 创建 Canvas 标签
Canvas 是 HTML5 新增的一个元素，让我们可以使用 Javascript 来绘制图形，处理图片，制作动画与游戏，甚至还可以使用 WebGL 来创建 3D 应用。好了， 我们来看看兼容性。

{% asset_img canvas-兼容性.png canvas 兼容性 %}

除了 `f**king` IE8 支持还是不错的，所以我们就放心大胆的用了。

```html
<canvas></canvas>
```

此时就已经创建了 Canvas 了，需要注意的是，canvas 的默认大小是 300x150。如果要指定大小，不要用 CSS，要使用 canvas 的 width 和 height 属性。

```html
<canvas width="300" height="300"></canvas>
```

# 绘制文字
首先我们需要获得一个绘图的上下文对象，然后我们的所有绘图操作都是基于这个上下文对象。这个有点儿类似 Windows 下编写 MFC 程序所使用的 DC (Draw Context)。

```js
var canvas = document.querySelector('canvas');
var ctx = canvas.getContext('2d');
```

拿到绘图上下文 `ctx` 后，我们先将背景涂黑。

```js
ctx.fillStyle = '#000';
ctx.fillRect(0, 0, canvas.width, canvas.height);
```

通过给 `ctx.fillStyle` 赋值，把填充颜色设为黑色，然后调用 `ctx.fillRect` 方法来绘制一个与 canvas 一样大的矩形，就达到了清屏的效果。然后我们再来把文字绘制在 canvas 上面。

```js
ctx.fillStyle = '#0f0';
ctx.font = '10px monospace';
ctx.fillText('Matrix', 100, 100);
```

此时，我们就在(100, 100)这个位置绘制出了 'Matrix' 的字样。

# 让文字动起来
现在我们已经能够绘制出文字了，那如何让文字像下雨一样动起来呢？这里就得来看看动画的原理了。

很简单，就像小时候在书脚的涂鸦一样，把图片连续播放就成了动画了。只要达到了人眼分辨不出来的帧率，就是流畅的动画了。所以我们只要把`清屏`和`绘制`的操作放到一个无限循环中就 OK 了。类似于这样。

```js
setInterval(function() {
  clearScreen();  // 清屏
  updateMatrix(); // 计算数据
  draw();         // 绘制  
}, 100);
```

为了保存文字的状态，我们需要一个数据结构，这里只用一个数组便可以搞定，数组存储的是每列文字的 y 坐标，这样我们每次更新 y 坐标便可以使文字向下流动起来了。确定要几列文字也比较简单，用 canvas 的 width 除以 font-size 就可以了。

# 组合起来
以上便是数码雨需要的所有技术，我们把他们组合起来，这里我们使用面向对象的方式来编写我们的 Matrix Rain。

先创建一个 `canvas` 标签，这里我们先不设置宽高，到后面我们让他自适应。

```html
<canvas id="matrix"></canvas>
```

然后我们定义 `Matrix` 类。

```js
// 定义构造函数，参数为一个 canvas 对象
var Matrix = function(canvas) {
  // 配置参数， 为了方便我们先硬编码
  this.opt = {
    bgcolor: '#00303e',
    opacity: 0.1,
    font: {
      size: 10,
      family: 'monospace',
      color: '#a6e22e'
    },
    speed: 100
  };
  this.canvas = canvas;
  // 取得绘图上下文
  this.ctx = this.canvas.getContext('2d');
  // 定时器 id，用于在窗口 resize 时重新设置定时器
  this.intervalId = null;
  // 初始化 canvas，准备数据结构
  this.init();
  // 添加 resize 事件
  this.addEvents();
};

// 初始化
Matrix.prototype.init = function() {
  // 让 canvas 的 width 与父容器一样
  this.canvas.width = this.canvas.parentElement.clientWidth;
  // 高度定为宽的一半
  this.canvas.height = this.canvas.width / 2;
  // 清屏
  this.clear(this.opt.bgcolor);
  // 清除定时器
  clearInterval(this.intervalId);
  // 需要几列
  var count = Math.round(this.canvas.width / this.opt.font.size);
  this.textRain = new Array();
  for (var i = 0; i < count; i++) {
    // 把随机的 y 坐标存入数组中
    this.textRain.push(Math.round(Math.random() * -1e3));
  }
  // 开始下雨
  this.rain();
};

// 绑定 resize 事件
Matrix.prototype.addEvents = function() {
  var that = this;
  window.addEventListener('resize', function() {
    // resize 的时候重新初始化
    that.init();
  });
};

// 清屏
Matrix.prototype.clear = function(color) {
  this.ctx.fillStyle = color;
  this.ctx.fillRect(0, 0, this.canvas.width, this.canvas.height);
};

// 把 #rrggbb 这种转换成 rgba 形式
Matrix.prototype.toRGBA = function(color, alpha) {
  var r = Number('0x' + color.slice(1, 3));
  var g = Number('0x' + color.slice(3, 5));
  var b = Number('0x' + color.slice(5, 7));

  return 'rgba(' + r + ',' + g + ',' + b + ',' + alpha + ')';
};

// 绘制
Matrix.prototype.draw = function() {
  // 注意这里使用 rgba 来清屏，这样就会有残影的效果了
  this.clear(this.toRGBA(this.opt.bgcolor, this.opt.opacity));
  // 设置文字颜色，字体
  this.ctx.fillStyle = this.opt.font.color;
  this.ctx.font = this.opt.font.size + 'px ' + this.opt.family;
  var that = this;
  this.textRain.forEach(function(cur, idx, arr) {
    // 生成一个随机的字符
    var text = String.fromCharCode(Math.round(1e3 + Math.random() * 33));
    // 绘制出一个文字
    that.ctx.fillText(text, idx * that.opt.font.size, cur);
    // 更新 y 坐标
    arr[idx] = (cur > that.canvas.height + Math.random() * 1e4) ? 0 : cur + that.opt.font.size;
  });
};

// 启动动画
Matrix.prototype.rain = function() {
  var that = this;
  // 设置定时器
  this.intervalId = setInterval(function() {
    that.draw();
  }, that.opt.speed);
};
```

最后构造一个 `Matrix` 对象就 OK 了。

```js
var matrix = new Matrix(document.querySelector('#matrix'));
```

看看最终的效果，如下。

<canvas id="matrix" style="display: block; margin: 0 auto"></canvas>
<script>
var Matrix = function(canvas) {
  this.opt = {
    bgcolor: '#00303e',
    opacity: 0.1,
    font: {
      size: 10,
      family: 'monospace',
      color: '#a6e22e'
    },
    speed: 100
  };
  this.canvas = canvas;
  this.ctx = this.canvas.getContext('2d');
  this.intervalId = null;
  this.init();
  this.addEvents();
};
Matrix.prototype.init = function() {
  this.canvas.width = this.canvas.parentElement.clientWidth;
  this.canvas.height = this.canvas.width / 2;
  this.clear(this.opt.bgcolor);
  clearInterval(this.intervalId);
  var count = Math.round(this.canvas.width / this.opt.font.size);
  this.textRain = new Array();
  for (var i = 0; i < count; i++) {
    this.textRain.push(Math.round(Math.random() * -1e3));
  }
  this.rain();
};
Matrix.prototype.addEvents = function() {
  var that = this;
  window.addEventListener('resize', function() {
    that.init();
  });
};
Matrix.prototype.clear = function(color) {
  this.ctx.fillStyle = color;
  this.ctx.fillRect(0, 0, this.canvas.width, this.canvas.height);
};
Matrix.prototype.toRGBA = function(color, alpha) {
  var r = Number('0x' + color.slice(1, 3));
  var g = Number('0x' + color.slice(3, 5));
  var b = Number('0x' + color.slice(5, 7));

  return 'rgba(' + r + ',' + g + ',' + b + ',' + alpha + ')';
};
Matrix.prototype.draw = function() {
  this.clear(this.toRGBA(this.opt.bgcolor, this.opt.opacity));
  this.ctx.fillStyle = this.opt.font.color;
  this.ctx.font = this.opt.font.size + 'px ' + this.opt.family;
  var that = this;
  this.textRain.forEach(function(cur, idx, arr) {
    var text = String.fromCharCode(Math.round(1e3 + Math.random() * 33));
    that.ctx.fillText(text, idx * that.opt.font.size, cur);
    arr[idx] = (cur > that.canvas.height + Math.random() * 1e4) ? 0 : cur + that.opt.font.size;
  });
};
Matrix.prototype.rain = function() {
  var that = this;
  this.intervalId = setInterval(function() {
    that.draw();
  }, that.opt.speed);
};
var matrix = new Matrix(document.querySelector('#matrix'));
</script>

你可以试着改变一下窗口的大小，可以看到我们的 `Matrix Rain` 是自适应的。

# 参考
- [MDN Canvas 教程](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial)
