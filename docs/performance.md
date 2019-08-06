#### 运行原理

``uni-app`` 在 App 端或小程序端运行时，从架构上分为逻辑层和视图层两个部分。逻辑层负责储存数据和执行业务逻辑，视图层负责页面渲染。

目前，视图层以 WebView 作为渲染载体进行渲染，以 evaluateJavascript 的方式在视图层和逻辑层进行数据通讯，evaluateJavascript 通讯是异步操作，所以数据到达视图层不是实时的。

数据通讯时，框架会将要传递的数据转换为字符串进行传递，然后将字符串接成一份 JS 脚本，再通过执行 JS 脚本的形式传递到逻辑层和视图层，此操作意味着数据的大小会影响性能。

#### 优化建议

**使用自定义组件模式**

使用自定义组件模式，在manifest中配置自定义组件模式（HBuilderX1.9起新建项目默认即为自定义组件模式）。

在复杂页面中，页面中嵌套大量组件，如果是非自定义组件模式，更新一个组件会导致整个页面数据更新。而自定义组件模式则可以单独更新一个组件的数据。

在App端，除了上述好处，自定义组件模式还新增了一个独立的js引擎，加快启动速度、减少js阻塞。

之前的非自定义组件模式已经不再推荐，请尽快升级你的应用。

**避免使用大图**

页面中若大量使用大图资源，会造成页面切换的卡顿，导致系统内存升高，甚至白屏崩溃。

尤其是不要把多张大图缩小后显示在一个屏幕内，比如上传图片前选了数张几M体积的照片，然后缩小在一个屏幕中展示多张几M的大图，非常容易白屏崩溃。

**优化数据更新**

在 ``uni-app`` 中，定义在 data 里面的数据每次变化时都会通知视图层重新渲染页面。 所以如果不是视图所需要的变量，可以不定义在 data 中，可在外部定义变量或直接挂载在vue实例上，以避免造成资源浪费。

**减少一次性渲染的节点数量**

页面初始化时，逻辑层如果一次性向视图层传递很大的数据，使视图层一次性渲染大量节点，可能造成通讯变慢、页面切换卡顿，所以建议以局部更新页面的方式渲染页面。如：服务端返回100条数据，可进行分批加载，一次加载20条，500ms 后进行下一次加载。

**减少节点嵌套层级**

 深层嵌套的节点在页面初始化构建时往往需要更多的内存占用，并且在遍历节点时也会更慢些，所以建议减少深层的节点嵌套。

**避免视图层和逻辑层频繁进行通讯**

* 减少 scroll-view 组件的 scroll 事件监听，当监听 scroll-view 的滚动事件时，视图层会频繁的向逻辑层发送数据；
* 监听 scroll-view 组件的滚动事件时，不要实时的改变 scroll-top/scroll-left 属性，因为监听滚动时，视图层向逻辑层通讯，改变 scroll-top/scroll-left 时，逻辑层又向视图层通讯，这样就可能造成通讯卡顿。
* 注意 onPageScroll 的使用，onPageScroll 进行监听时，视图层会频繁的向逻辑层发送数据；

**优化页面切换动画**

* 页面初始化时若存在大量图片渲染和大量数据通讯，很有可能造成页面切换卡顿、掉帧。建议延时100ms~300ms渲染图片，分批进行数据通讯，以减少一次性渲染的节点数量。
* App端动画效果可以设置，popin/popout的双窗体联动挤压动画效果对资源的消耗更大，如果动画期间页面里在执行耗时的js，可能会造成动画掉帧。此时可以使用消耗资源更小的动画效果，比如slide-in-right/slide-out-right。

**优化样式渲染速度**

将样式写在 ``App.vue`` 里，可以加速页面样式渲染速度。``App.vue`` 里面的样式是全局样式，每次新开页面会优先加载 ``App.vue`` 里面的样式，然后加载普通 vue 页面的样式。尤其是背景为深色的页面，如果不能及时设置样式，会造成进入转场动画时闪烁。

**使用 nvue 代替 vue**

在 App 端 ```uni-app``` 的 nvue 页面可是基于 [weex](https://weex.apache.org/) 定制的原生渲染引擎，实现了页面原生渲染能力、提高了页面流畅性。若对页面性能要求较高可以使用此方式开发，详见：[nvue](/use-weex)。

**优化App启动速度的注意**

* 工程代码越多，包括背景图和本地字体文件越大，对App的启动速度有影响，应注意控制体积。<image>组件引用的前景图不影响性能。
* App端的 splash 关闭有白屏检测机制，如果首页一直白屏或首页本身就是一个空的中转页面，可能会造成 splash 10秒才关闭，可参考此文解决[http://ask.dcloud.net.cn/article/35565](http://ask.dcloud.net.cn/article/35565)
* App端使用自定义组件模式的话，首页为nvue页面可比vue页面大幅提升启动速度，nvue页面一般2、3秒即可完成启动。

**优化包体积**

* uni-app发行到小程序时，自带引擎只有几十K，主要是一个定制过的vue.js核心库。如果使用了es6转es5、css对齐的功能，可能会增大代码体积，目前可以在cli中配置这些编译功能是否开启，后续在HBuilderX中也会提供配置。
* uni-app的App端，因为自带了一个独立v8引擎和小程序框架，所以比HTML5Plus或mui等普通hybrid的App引擎体积要大。基础引擎约10M+。App还提供了扩展模块，比如地图、蓝牙等，打包时如不需要这些模块，可以裁剪掉，以缩小发行包体积。在 manifest.json-App模块权限 里可以选择。
* uni-app的H5端，自带了vue.js、小程序ui库（如picker、switch等）、小程序的对齐js api，目前包体积比较大。下版会提供裁剪方案，按需加载组件和API。