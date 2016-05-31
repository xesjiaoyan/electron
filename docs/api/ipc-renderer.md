# ipcRenderer

> 从渲染进程到主进程的异步通信

`ipcRenderer`模块是一个[EventEmitter](https://nodejs.org/api/events.html)类的实例。
它提供了几个从渲染进程到主进程的通讯方法，这些方法有同步的也有异步的。当然你也能够获得主进程的回复。

See [ipcMain](ipc-main.md) for code examples.

## 消息侦听

`ipcRenderer`模块有一下几个方法来侦听事件：

### `ipcRenderer.on(channel, listener)`

* `channel` String
* `listener` Function

Listens to `channel`, when a new message arrives `listener` would be called with
`listener(event, args...)`.

### `ipcRenderer.once(channel, listener)`

* `channel` String
* `listener` Function

Adds a one time `listener` function for the event. This `listener` is invoked
only the next time a message is sent to `channel`, after which it is removed.

### `ipcRenderer.removeListener(channel, listener)`

* `channel` String
* `listener` Function

Removes the specified `listener` from the listener array for the specified
`channel`.

### `ipcRenderer.removeAllListeners([channel])`

* `channel` String (optional)

Removes all listeners, or those of the specified `channel`.

## Sending Messages

The `ipcRenderer` module has the following methods for sending messages:

### `ipcRenderer.send(channel[, arg1][, arg2][, ...])`

* `channel` String
* `arg` (optional)

Send a message to the main process asynchronously via `channel`, you can also
send arbitrary arguments. Arguments will be serialized in JSON internally and
hence no functions or prototype chain will be included.

The main process handles it by listening for `channel` with `ipcMain` module.

### `ipcRenderer.sendSync(channel[, arg1][, arg2][, ...])`

* `channel` String
* `arg` (optional)

Send a message to the main process synchronously via `channel`, you can also
send arbitrary arguments. Arguments will be serialized in JSON internally and
hence no functions or prototype chain will be included.

The main process handles it by listening for `channel` with `ipcMain` module,
and replies by setting `event.returnValue`.

**Note:** Sending a synchronous message will block the whole renderer process,
unless you know what you are doing you should never use it.

### `ipcRenderer.sendToHost(channel[, arg1][, arg2][, ...])`

* `channel` String
* `arg` (optional)

Like `ipcRenderer.send` but the event will be sent to the `<webview>` element in
the host page instead of the main process.
