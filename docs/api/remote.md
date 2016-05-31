# remote

> 在渲染进程（renderer process）中使用主进程（main process）相关模块

`remote`提供了一个简单的主进程和渲染进程间通讯（IPC）的方式

在Electron，GUI（用户图形界面）相关模块，像`dialog`，`menu`等，仅在主进程中可用，渲染进程中不可用。
为了能够在渲染进程中使用，`ipc`模块必须给主进程发送进程间消息（inter-process messages）。通过`remote`模块，
你可以不必显示的发送进程间消息（inter-process messages）就能够执行主进程中的方法，类似Jave中的[RMI][rmi]。
以下是一个通过渲染进程创建BrowserWindow的例子。


```javascript
const {BrowserWindow} = require('electron').remote;

let win = new BrowserWindow({width: 800, height: 600});
win.loadURL('https://github.com');
```

**注意：**反过来，如果你想通过主进程访问渲染进程，你可以使用[webContents.executeJavascript](web-contents.md#webcontentsexecutejavascriptcode-usergesture).

## Remote Objects

通过`remote`返回主进程中的对象（包括方法），我们称它为远程对象或者远程方法。
当你调用远程对象，或者执行远程方法，以及通过远程构造函数创建一个对象，实际上你是
发送了一个同步进程间消息（inter-process messages）

在上面的例子中，`BrowserWindow`和`win`是远程对象，在渲染进程中不能使用`new BrowserWindow`创建`BrowserWindow`对象。
实际上，上例中的`win`对象是在主进程创建一个`BrowserWindow`对象，然后在渲染进程中返回相应的远程对象。

请注意，[enumerable properties](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Enumerability_and_ownership_of_properties)目前只有当远程对象被引用的时候，才能通过remote访问

## Lifetime of Remote Objects

Electron确保只要远程对象在渲染进程中存在，主进程中对应的对象就不会被释放。当远程对象被垃圾回收之后，主进程中对应的对象也就解除了对应的引用。

如果远程对象出现内存泄漏，那么主进程中对应的对象也将出现内存泄漏，所以你必须非常小心，别让远程对象出现内存泄漏。

主要的值类型，像strings和numbers，都是值传递。


##传递回调函数到主进程

主进程中的代码能够接收由渲染进程通过`remote`实例传过来的回调函数，但是，你必须非常小心的使用这个特性。

首先，为了避免死锁，传递到主进程的回调函数是通过异步触发。你别想着通过回调函数直接获取返回值。

比如，你不能再主进程中调用渲染进程中的`Array.map`方法：

```javascript
// main process mapNumbers.js
exports.withRendererCallback = (mapper) => {
  return [1,2,3].map(mapper);
};

exports.withLocalCallback = () => {
  return exports.mapNumbers(x => x + 1);
};
```

```javascript
// renderer process
const mapNumbers = require('remote').require('./mapNumbers');

const withRendererCb = mapNumbers.withRendererCallback(x => x + 1);

const withLocalCb = mapNumbers.withLocalCallback();

console.log(withRendererCb, withLocalCb); // [true, true, true], [2, 3, 4]
```

如你所见，通过渲染进程传递到主进程，然后获取回调函数的返回值，并没有达到预期的效果。同样的方法，在主进程中使用，就能够达到预期的效果。

其次，回调函数传递到主进程之后会一直保留，知道主进程对其做垃圾回收。

例如，下面的代码通过远程对象注册了一个`close`事件，乍一看，没有任何问题。


```javascript
remote.getCurrentWindow().on('close', () => {
  // blabla...
});
```
但是，你可还记得，该回调函数除非你明确的卸载它，否则它将一直存在于主进程中。如果你没有做卸载动作，那么，每次你重新加载该窗口的时候，这个回调函数都会被重新加载一次，如此反复，这样就导致了内存泄漏。

更糟糕的是，当主进程发送`close`事件时，由于注册对象被释放了（窗口关闭，主进程会将`win`对象释放），将会导致抛出错误。

为了避免这个问题，确保清除了所有渲染进程传递到主进程的回调函数。


##访问主进程的内置模块

主进程内置模块被添加在`remote`模块，你可以像`electron`模块一样直接使用它们。

```javascript
const app = remote.app;
```

## Methods

The `remote` module has the following methods:

### `remote.require(module)`

* `module` String

Returns the object returned by `require(module)` in the main process.

### `remote.getCurrentWindow()`

Returns the [`BrowserWindow`](browser-window.md) object to which this web page
belongs.

### `remote.getCurrentWebContents()`

Returns the [`WebContents`](web-contents.md) object of this web page.

### `remote.getGlobal(name)`

* `name` String

Returns the global variable of `name` (e.g. `global[name]`) in the main
process.

### `remote.process`

Returns the `process` object in the main process. This is the same as
`remote.getGlobal('process')` but is cached.

[rmi]: http://en.wikipedia.org/wiki/Java_remote_method_invocation
