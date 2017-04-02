# 07 - Socket.IO

本章代码在 [这里](https://github.com/verekia/js-stack-walkthrough/tree/master/07-socket-io).

> 💡 **[Socket.IO](https://github.com/socketio/socket.io)** 是一个用来处理 Websockets 的库。它的 API 设计简单，并且能为不支持 Websockets 的浏览器提供回退策略。

在本章中，我们会在客户端和服务器之间进行简单的信息交换。为了不加入新的页面 —— 新页面意味着要加入我们不感兴趣的东西 —— 信息交换的结果会展示在浏览器的 console 面板中。所有本章没有 UI 相关的操作。

- 运行 `yarn add socket.io socket.io-client`

## Server-side（服务端）

- 编辑 `src/server/index.js`：

```js
// @flow

import compression from 'compression'
import express from 'express'
import { Server } from 'http'
import socketIO from 'socket.io'

import routing from './routing'
import { WEB_PORT, STATIC_PATH } from '../shared/config'
import { isProd } from '../shared/util'
import setUpSocket from './socket'

const app = express()
// flow-disable-next-line
const http = Server(app)
const io = socketIO(http)
setUpSocket(io)

app.use(compression())
app.use(STATIC_PATH, express.static('dist'))
app.use(STATIC_PATH, express.static('public'))

routing(app)

http.listen(WEB_PORT, () => {
  // eslint-disable-next-line no-console
  console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' :
    '(development).\nKeep "yarn dev:wds" running in an other terminal'}.`)
})
```

注意，为了使用 Socket.IO，你需要使用 `http` 包的 `Server` 创建的服务器来 `listen（监听）` 请求，而不是用 Express 创建的 `app` 服务器。幸运的是，我们不用更改太多代码。Websocket 的细节写在另一个文件中，我们用 `setUpSocket` 来调用这个文件。

- 把下面的常量添加到 `src/shared/config.js`：

```js
export const IO_CONNECT = 'connect'
export const IO_DISCONNECT = 'disconnect'
export const IO_CLIENT_HELLO = 'IO_CLIENT_HELLO'
export const IO_CLIENT_JOIN_ROOM = 'IO_CLIENT_JOIN_ROOM'
export const IO_SERVER_HELLO = 'IO_SERVER_HELLO'
```

这些常量代表着浏览器和服务器会进行交换的 *消息类型*。我建议给这些常量加上 `IO_CLIENT` 或者 `IO_SERVER` 前缀，清楚地表示出 *谁* 是消息的发送方。否则，看到那么多消息类型的时候，你可能有点懵。

如你所见，有一个名为 `IO_CLIENT_JOIN_ROOM` 的消息类型。为了演示，我们让客户端加入一个房间（类似聊天室）。当向特定的用户群发送消息的时候，房间非常有用。 because for the sake of demonstration, we are going to make clients join a room (like a chatroom). Rooms are useful to broadcast messages to specific groups of users.

- 创建 `src/server/socket.js`：

```js
// @flow

import {
  IO_CONNECT,
  IO_DISCONNECT,
  IO_CLIENT_JOIN_ROOM,
  IO_CLIENT_HELLO,
  IO_SERVER_HELLO,
} from '../shared/config'

/* eslint-disable no-console */
const setUpSocket = (io: Object) => {
  io.on(IO_CONNECT, (socket) => {
    console.log('[socket.io] A client connected.')

    socket.on(IO_CLIENT_JOIN_ROOM, (room) => {
      socket.join(room)
      console.log(`[socket.io] A client joined room ${room}.`)

      io.emit(IO_SERVER_HELLO, 'Hello everyone!')
      io.to(room).emit(IO_SERVER_HELLO, `Hello clients of room ${room}!`)
      socket.emit(IO_SERVER_HELLO, 'Hello you!')
    })

    socket.on(IO_CLIENT_HELLO, (clientMessage) => {
      console.log(`[socket.io] Client: ${clientMessage}`)
    })

    socket.on(IO_DISCONNECT, () => {
      console.log('[socket.io] A client disconnected.')
    })
  })
}
/* eslint-enable no-console */

export default setUpSocket
```

下面，我们解释一下 *当客户端连接到服务端并发送消息的时候，服务端该做些什么*：

- 当客户端连接到服务端，连接信息会在服务端 log 出来，并且得到 `socket` 对象。这个对象可以用来向客户端发起通信。
- 当客户端发送 `IO_CLIENT_JOIN_ROOM` 消息时，如它所愿，我们让它加入一个 `room（房间）`。一旦客户端加入房间，我们发送三条 demo 消息：一条消息发送给所有用户，一条发送给房间内的用户，还有一条只发送给这个客户端。
- 当客户端发送 `IO_CLIENT_HELLO`，在服务端上 log 出来。
- 当客户端断开连接的时候，服务端也会 log 出来。

## Client-side（客户端）

客户端的代码相对简单：

- 编辑 `src/client/index.jsx`：

```js
// [...]
import setUpSocket from './socket'

// [at the very end of the file]
setUpSocket(store)
```

如你所见，我们把 Redux store 传给了 `setUpSocket`。这样，无论什么时候服务端向客户端推送了消息，都会改变 Redux 的 state。我们可以 `dispatch（分发）` actions，但在这个例子中，我们没有 `dispatch（分发）` 任何东西。

- 创建 `src/client/socket.js`：

```js
// @flow

import socketIOClient from 'socket.io-client'

import {
  IO_CONNECT,
  IO_DISCONNECT,
  IO_CLIENT_HELLO,
  IO_CLIENT_JOIN_ROOM,
  IO_SERVER_HELLO,
  } from '../shared/config'

const socket = socketIOClient(window.location.host)

/* eslint-disable no-console */
// eslint-disable-next-line no-unused-vars
const setUpSocket = (store: Object) => {
  socket.on(IO_CONNECT, () => {
    console.log('[socket.io] Connected.')
    socket.emit(IO_CLIENT_JOIN_ROOM, 'hello-1234')
    socket.emit(IO_CLIENT_HELLO, 'Hello!')
  })

  socket.on(IO_SERVER_HELLO, (serverMessage) => {
    console.log(`[socket.io] Server: ${serverMessage}`)
  })

  socket.on(IO_DISCONNECT, () => {
    console.log('[socket.io] Disconnected.')
  })
}
/* eslint-enable no-console */

export default setUpSocket
```

如果你已经理解了我们在服务端做的事情，那么理解客户端的东西也不难：

- 客户端连接成功后，会在 console 面板 log 出来；并且发送一条 `IO_CLIENT_JOIN_ROOM` 房间 `hello-1234`。
- 然后发送一条值 `Hello!` 的 `IO_CLIENT_HELLO` 消息。
- 如果服务端推送一条 `IO_SERVER_HELLO`，我们会在客户端 log 出来。
- 在断开连接的时候，也会有 log 信息。

🏁 运行 `yarn start` 和 `yarn dev:wds`，打开 `http://localhost:8000`。然后打开浏览器的 console 面板和 Express 服务器的命令窗口；这样，你就能看到客户端和服务端之间的通信了。

下一章：[08 - Bootstrap, JSS](08-bootstrap-jss.md#readme)

回到 [上一章](06-react-router-ssr-helmet.md#readme) 或者 [目录](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
