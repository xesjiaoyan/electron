#安全，本机功能，和你的职责

作为一个web开发者，我们通常比较喜欢安全性强的浏览器，它能够让我们写代码的风险降低到一个相对小的位置。我们的网站在一个安全沙箱中运行，并且我们相信，我们的用户也希望浏览器有一个强大的团队支持，来响应各种安全问题。

当我们用Electron做开发的时候，我们必须明白，Electron不是浏览器。它让你用比较擅长的web技术来做桌面应用的开发，你的代码能够做更多的事情，JavaScript能够访问文件系统了，也能够使用shell，还有其他等等此前不能够做的事情。它让你能够构建高品质的本地应用的同时，安全风险也随之而来。

也就是说，会承受大量来自不受信任的代码引起的安全问题，Electron不会帮你处理。实际上，大量流行的Electron应用（Atom，Slack，Visual Studio Code等），都主要用来展示本地内容，基本不会让远程内容使用Node integration。如果你的应用执行了线上的文件，你应该承担这个风险导致的责任。

##Chromium 安全问题和更新

Electron会尽全力去支持最新的Chromium，开发者应该有所了解，更新是个很费劲的事——需要手动编辑数十上百个文件。从目前的情况来看，Electron使用的Chromium版本通常不是最新的，会落后个几天或者几周。

我觉得，当前我们的系统在Chromium更新和资源之间处在一个平衡点。在框架之上，我们有大量的可用应用程序被构建出来。我们非常高兴又越来越多的人使用Electron来做东西。提交代码和捐款等行为我们都非常支持。

## Ignoring Above Advice
A security issue exists whenever you receive code from a remote destination and
execute it locally. As an example, consider a remote website being displayed
inside a browser window. If an attacker somehow manages to change said content
(either by attacking the source directly, or by sitting between your app and
the actual destination), they will be able to execute native code on the user's
machine.

> :warning: Under no circumstances should you load and execute remote code with
Node integration enabled. Instead, use only local files (packaged together with
your application) to execute Node code. To display remote content, use the
`webview` tag and make sure to disable the `nodeIntegration`.

#### Checklist
This is not bulletproof, but at the least, you should attempt the following:

* Only display secure (https) content
* Disable the Node integration in all renderers that display remote content
  (using `webPreferences`)
* Do not disable `webSecurity`. Disabling it will disable the same-origin policy.
* Define a [`Content-Security-Policy`](http://www.html5rocks.com/en/tutorials/security/content-security-policy/)
, and use restrictive rules (ie: `script-src 'self'`)
* [Override and disable `eval`](https://github.com/nylas/N1/blob/0abc5d5defcdb057120d726b271933425b75b415/static/index.js#L6)
, which allows strings to be executed as code.
* Do not set `allowDisplayingInsecureContent` to true.
* Do not set `allowRunningInsecureContent` to true.
* Do not enable `experimentalFeatures` or `experimentalCanvasFeatures` unless
  you know what you're doing.
* Do not use `blinkFeatures` unless you know what you're doing.
* WebViews: Set `nodeintegration` to false
* WebViews: Do not use `disablewebsecurity`
* WebViews: Do not use `allowpopups`
* WebViews: Do not use `insertCSS` or `executeJavaScript` with remote CSS/JS.

Again, this list merely minimizes the risk, it does not remove it. If your goal
is to display a website, a browser will be a more secure option.
