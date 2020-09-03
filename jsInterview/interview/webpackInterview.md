**Hybrid**
---
***

随着 Web技术 和 移动设备 的快速发展，在各家大厂中，Hybrid 技术已经成为一种最主流最不可取代的架构方案之一。一套好的 Hybrid 架构方案能让 App 既能拥有 **极致的体验和性能**，同时也能拥有 Web技术 **灵活的开发模式、跨平台能力以及热更新机制**。因此，相关的 Hybrid 领域人才也是十分的吃香，精通Hybrid 技术和相关的实战经验，也是面试中一项大大的加分项。

**1. 混合方案简析**
---

Hybrid App，俗称 **混合应用**，即混合了 Native技术 与 Web技术 进行开发的移动应用。现在比较流行的混合方案主要有三种，主要是在UI渲染机制上的不同:

* **Webview UI**:

    * 通过 **JSBridge** 完成 H5 与 Native 的双向通讯，并 **基于 Webview** 进行页面的渲染；

    * **优势**: 简单易用，架构门槛/成本较低，适用性与灵活性极强；

    * **劣势**: Webview 性能局限，在复杂页面中，表现远不如原生页面；

* **Native UI**:

    * 通过 **JSBridge** 赋予 H5 原生能力，并进一步将 JS 生成的虚拟节点树(Virtual DOM)传递至 Native 层，并使用 **原生系统渲染**。

    * **优势**: 用户体验基本接近原生，且能发挥 Web技术 开发灵活与易更新的特性；

    * **劣势**: 上手/改造门槛较高，最好需要掌握一定程度的客户端技术。相比于常规 Web开发，需要更高的开发调试、问题排查成本；

* **小程序**

    * 通过更加定制化的 **JSBridge**，赋予了 Web 更大的权限，并使用双 `WebView` 双线程的模式隔离了 `JS`逻辑 与 `UI`渲染，形成了特殊的开发模式，加强了 `H5` 与 `Native` 混合程度，属于第一种方案的优化版本；

    * **优势**: 用户体验好于常规 `Webview` 方案，且通常依托的平台也能提供更为友好的开发调试体验以及功能；

    * **劣势**: 需要依托于特定的平台的规范限定

**2. Webviev**

Webview 是 Native App 中内置的一款基于 Webkit内核 的浏览器，主要由两部分组成:

* **WebCore 排版引擎**；

* **JSCore 解析引擎**；

在原生开发 SDK 中 Webview 被封装成了一个组件，用于作为 Web页面 的容器。因此，作为宿主的客户端中拥有更高的权限，可以对 Webview 中的 Web页面 进行配置和开发。

Hybrid技术中双端的交互原理，便是基于 Webview 的一些 API 和特性。

**3. 交互原理**
---

Hybrid技术 中最核心的点就是 Native端 与 H5端 之间的 **双向通讯层**，其实这里也可以理解为我们需要一套 **跨语言通讯方案**，便是我们常听到的 JSBridge。

* JavaScript 通知 Native

    * **API注入**，Native 直接在 JS 上下文中挂载数据或者方法

        * 延迟较低，在安卓4.1以下具有安全性问题，风险较高

    * WebView **URL Scheme** 跳转拦截

        * 兼容性好，但延迟较高，且有长度限制

    * WebView 中的 **prompt/console/alert拦截**(通常使用 prompt)

* Native 通知 Javascript:

    * IOS: `stringByEvaluatingJavaScriptFromString`

    ```javascript
    // Swift
    webview.stringByEvaluatingJavaScriptFromString("alert('NativeCall')")
    ```

    * Android: `loadUrl` (4.4-)

    ```javascript
    // 调用js中的JSBridge.trigger方法
    // 该方法的弊端是无法获取函数返回值；
    webView.loadUrl("javascript:JSBridge.trigger('NativeCall')")
    ```

    * Android: `evaluateJavascript` (4.4+)

    ```javascript
    // 4.4+后使用该方法便可调用并获取函数返回值；
    mWebView.evaluateJavascript（"javascript:JSBridge.trigger('NativeCall')", 	 new ValueCallback<String>() {
        @Override
        public void onReceiveValue(String value) {
            //此处为 js 返回的结果
        }
    });
    ```

**4. 接入方案**
---

整套方案需要 Web 与 Native 两部分共同来完成:

* **Native**: 负责实现URL拦截与解析、环境信息的注入、拓展功能的映射、版本更新等功能；

* **JavaScirpt**: 负责实现功能协议的拼装、协议的发送、参数的传递、回调等一系列基础功能。

**接入方式**:

* **在线H5**: 直接将项目部署于线上服务器，并由客户端在 HTML 头部注入对应的 Bridge。

    * 优势: 接入/开发成本低，对 App 的侵入小；

    * 劣势: 重度依赖网络，无法离线使用，首屏加载慢；

* 内置离线包: 将代码直接内置于 App 中，即本地存储中，可由 H5 或者 客户端引用 Bridge。

    * 优势: 首屏加载快，可离线化使用；

    * 劣势: 开发、调试成本变高，需要多端合作，且会增加 App 包体积