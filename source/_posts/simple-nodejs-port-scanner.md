---
title: 用 NodeJS 实现简易端口扫描器
date: 2016-05-17 17:50:05
tags:
  - NodeJS
  - 端口扫描
---

闲来无事，用 [NodeJS] 来耍点杂技，我们来实现一个简易的 **端口扫描器** 。

# 原理
本杂技主要使用了 [NodeJS] 的 `net` 模块，通过遍历主机的所有端口，尝试建立 TCP 连接，来查看此端口是否对外开放。

<!-- more -->

```js
'use strict'

const net = require('net')

// 要扫描的目标主机
const host = 'www.baidu.com'
// 开始端口
const start = 0
// 结束端口
const end = 65536
// 超时
const timeout = 5000

// 遍历端口
for (let port = start; port < end; port++) {
  // 新建 socket
  let socket = new net.Socket()

  // 设置超时
  socket.setTimeout(timeout, () => socket.destroy())

  // 连接失败时销毁此 socket
  socket.on('error', () => socket.destroy())

  // 如果连接上，纪录下来
  socket.connect({ port, host }, () => {
    console.log(`${host}: ${port}`)
    socket.destroy()
  })
}
```


[NodeJS]: https://nodejs.org
