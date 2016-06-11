---
title: Web Audio 实现音乐的可视化
date: 2016-06-11 10:47:29
tags:
  - WebAudio
  - Canvas
---

> 你在哪里, 亚特兰蒂斯

<canvas id="canvas" height="200" style="display: block;"></canvas>
<audio id="audio" src="/downloads/music/faded.mp3" controls preload="auto" style="display: block; width: 100%;"></audio>
<script>
var audioCtx = new window.AudioContext();
var audio = document.querySelector('audio');

var sourceNode = audioCtx.createMediaElementSource(audio);
var analyserNode = audioCtx.createAnalyser();

sourceNode.connect(analyserNode);
sourceNode.connect(audioCtx.destination);

analyserNode.fftSize = 1024;
var dataArray = new Uint8Array(analyserNode.frequencyBinCount);

var canvas = document.querySelector('#canvas');
var canvasCtx = canvas.getContext('2d');
canvas.width = canvas.parentElement.clientWidth;

canvasCtx.fillStyle = 'black';
canvasCtx.fillRect(0, 0, canvas.width, canvas.height);

var lineargradient = canvasCtx.createLinearGradient(0, 0, 0, canvas.height);
lineargradient.addColorStop(0, 'orange');
lineargradient.addColorStop(1, '#339933');

var requestID = null;

function playAnimation () {
  requestID = window.requestAnimationFrame(playAnimation);
  analyserNode.getByteFrequencyData(dataArray);

  canvasCtx.fillStyle = 'black';
  canvasCtx.fillRect(0, 0, canvas.width, canvas.height);

  canvasCtx.fillStyle = lineargradient;

  dataArray.forEach(function (h, idx) {
    canvasCtx.fillRect(idx * 2, canvas.height - h * 0.8, 2, h * 0.8);
  });
}

function stopAnimation () {
  window.cancelAnimationFrame(requestID);
}

audio.addEventListener('playing', function () { playAnimation(); });

audio.addEventListener('pause', function () { stopAnimation(); });
audio.addEventListener('ended', function () { stopAnimation(); });
</script>

# Web Audio

[Web Audio] API 提供了在Web上控制音频的一个非常有效通用的系统，允许开发者来自选音频源，对音频添加作用，创建可视化音频，应用空间效果（如平移），等等。

一般的 [Web Audio] 玩法如下:

1. 创建音频环境
2. 在音频环境里创建源
3. 创建效果节点，例如混响、双二阶滤波器、平移、压缩
4. 为音频选择一个目地，例如你的系统扬声器
5. 连接源到效果器，以及效果器和目地

{% asset_img web-audio.png web audio %}

在这里我们只用将 **源节点** 连接到 **分析节点** 就可以拿到音频的数据了。

# 创建音频图

首先，我们需要一个音频上下文

```js
var audioCtx = new window.AudioContext()
```

然后创建源节点，这里我们使用 audio 标签来作为源节点

```html
<audio id="audio" src="music.mp3" controls></audio>
```

```js
var audioEle = document.querySelector('#audio')
var sourceNode = audioCtx.createMediaElementSource(audioEle)
```

然后创建分析节点

```js
var analyserNode = audioCtx.createAnalyser()
```

最后把它们都连接起来

```js
sourceNode.connect(analyserNode)
sourceNode.connect(audioCtx.destination)
```

# 从分析节点获取数据

现在音频图已经连接完毕，接着我们开始设置分析节点，并从里面拿数据

```js
analyserNode.fftSize = 1024
var dataArray = new Uint8Array(analyserNode.frequencyBinCount)
```

我们创建了一个缓冲区来接受音频数据。然后可以同过 `getByteFrequencyData` 来获得频率域的数据。

```js
analyserNode.getByteFrequencyData(dataArray)
```

# 使用 Canvas 绘制

拿到数据后，我们就可以用 [Canvas] 来绘制我们的频谱图了。

```js
var canvas = document.querySelector('#canvas')
var canvasCtx = canvas.getContext('2d')
// 初始宽度
canvas.width = canvas.parentElement.clientWidth

// 线性渐变
var lineargradient = canvasCtx.createLinearGradient(0, 0, 0, canvas.height)
lineargradient.addColorStop(0, 'orange')
lineargradient.addColorStop(1, '#339933')

// 绘制函数
function draw () {
  requestAnimationFrame(draw)
  analyserNode.getByteFrequencyData(dataArray)

  canvasCtx.fillStyle = 'black'
  canvasCtx.fillRect(0, 0, canvas.width, canvas.height)

  canvasCtx.fillStyle = lineargradient

  dataArray.forEach(function (h, idx) {
    canvasCtx.fillRect(idx * 2, canvas.height - h * 0.8, 2, h * 0.8)
  })
}
```

在这里使用 `requestAnimationFrame` 以获得更好的动画性能。

## 参考

- [Web Audio] API
- [Canvas] 教程

[Web Audio]: https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Audio_API
[Canvas]: https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial
