# 路由模式

到现在为止，前端的路由可以分为两大模式，一种是 hash，另一种是 history。而这两种模式的诞生是源于 SPA 的广泛应用。那么 SPA 是什么呢？

单页 Web 应用（SPA），就是只有一张 Web 页面的应用。浏览器一开始会加载必需的 HTML、CSS和JavaScript，所有的操作都在这张页面上完成，都由 JavaScript 来控制。

因此，前端开发者希望在用户切换页面的时候保持单页面不变，即不再向服务器去请求 HTML、CSS和JavaScript。但是浏览器并不知道当前的页面是 SPA，还是传统的页面，所以为了兼容历史版本，它还默认为是传统的 Web 应用——不同页面对应不同 HTML。所以当你的 URL 改变时，浏览器依然会向新的 URL 发起请求，但这是我们不希望浏览器做的。

随着时代进步，开发者越来越意识到限制浏览器发送 URL 请求的必要性。伴随者 MVVM 开发模式的兴起，Ajax 请求技术的进步，所以两种路由模式就诞生了。

## 1. hash

### 1.1 特点

hash 模式最大的特点就是在浏览器地址栏上有一个`#`号。`#`是一个特殊的标志符，代表其后的字符串不会放入到HTTP请求中。

比如，访问下面的网址，
```
　　http://www.example.com/index.html#print
```

浏览器实际发出的请求是这样的：
```
　　GET /index.html HTTP/1.1
　　Host: www.example.com
```

这种 “截断” 路径的方式就是 hash 的设计本质。因为在相同的请求路由下，浏览器一般认为：现在已经用户在这个 URL 请求的页面下了，如果你再发一份同样的 URL，浏览器就觉得是请求相同内容，那再没有必要去请求一份了。所以我们可以得出：

* 浏览器不会将 URL `#` 后面的内容放在 HTTP 请求中发送到后端；
* 更改`#`后面的内容不会重新加载页面，也不会重新向服务器请求 index.html；

再比如，访问下面的网址
```
https://router.vuejs.org/zh/guide/advanced/navigation-guards.html#%E5%85%A8%E5%B1%80%E5%89%8D%E7%BD%AE%E5%AE%88%E5%8D%AB
```

我们可以发现地址栏上`#`后面出现的是 “全局前置守卫”。使用`window.location.hash`可以得到：

``` javascript
console.log(window.location.hash) //"#%E5%85%A8%E5%B1%80%E5%89%8D%E7%BD%AE%E5%AE%88%E5%8D%AB"
```

按下 `F12`，尝试点击侧边栏的 进阶-导航首位 的另外一些页面，可以发现`#`后面的文字发生了变化，并且每一次的 hash 都发生了变化，但是浏览器并没有发出获取 HTML、JS、CSS 的新请求。
当我们尝试点击浏览器左上角的返回按钮时，我们可以发现页面可以回退到上一个页面。我们也可以尝试删掉一些字符，回车，我们发现页面没有变化，按下回退键还可以回到上一次删掉字符的URL。因此我们又可以得到：

* 改变`#`号会改变浏览器的访问历史；
* `#`后面的值不存在时不会出现 404；

### 1.2 流程

让我们来整理思路，假如我们要用 hash 的模式实现一个路由，那么流程应该是这样的：

![hash](https://segmentfault.com/img/remote/1460000013243919?w=1043&h=1199)

### 1.3 使用方法

* 使用`window.location.hash`可以读取路由哈希值；当写入时，浏览器会创建一个历史记录。
* 使用`onhashchange`事件监测 hash 的变更：
  * 改变onhashchange函数：`window.onhashchange = func;`
  * 使用监听回调：`window.addEventListener("hashchange", func, false);`
  * 在 body 标签中添加：`<body onhashchange="func();">`

实例代码：
``` javascript
function link(url) {
  console.log('link to: ' + url);
  //实现跳转页面（没有发送HTTP请求）
  window.location = '#' + url;
}

// 监听 hashchange，执行相应的回调
window.addEventListener("hashchange", function() {
  var hash = window.location.hash;
  console.log('current hash: ' + hash);
}, false);
```


### 1.4 缺陷

* 无法SEO。页面都变成了全JS生成，搜索引擎及第三方统计无法进行抓起。
* 后端有时需要`#`后面的路由时前端无法方便的提供。

## 2. hisotry

> 如果不想要很丑的 hash，我们可以用路由的 history 模式

### 2.1 特点

与 hash 模式相比，history 少了`#`，地址栏更加优雅美观了。但是随之而来的问题是浏览器会默认将 URL 发送到服务器，需要服务器的支持。后台开发人员需要去处理这些路由，把所有路由都重定向到根页面。如果出现路由不匹配的情况，需要返回 404 或者根页面。

### 2.2 流程

同样，我们来理清下思路，这样写起代码才更得心应手~

![history](https://segmentfault.com/img/remote/1460000013243920)

### 2.3 使用方法

* 使用`window.history.pushState`来切换前端路由，并创建历史记录，同时触发`popState`事件；
* 使用`window.history.replaceState`来替换当前的前端路由，同时触发`popState`事件；

实例代码：
``` javascript
let title = "page1";
let URL = "/route"
let stateObj = {
    foo: "bar",
};

// 创建并激活新的历史记录
history.pushState(stateObj, title, URL);
// 获取 stateObj
console.log(history.state);
```

## 3. hash 与 history 的区别

模式|hash | history
-|-|-
url显示|会显示`#`|不会显示`#`
请求URL|`#`后面不会发送|全部发送
页面传参|字符串编码与解析|直接可以传入对象

history 的优势：
* 新的 URL 可以是与当前 URL 同源的任意 URL。相反，只有在修改哈希时，设置 `window.location` 才能是同一个 document。
* 如果你不想改 URL，就不用改。相反，设置 `window.location = "#foo";` 在当前哈希不是 `#foo` 时， 才能创建新的历史记录项。
* 你可以将任意数据和新的历史记录项相关联。而基于哈希的方式，要把所有相关数据编码为短字符串。 
* 如果 标题 随后还会被浏览器所用到，那么这个数据是可以被使用的（哈希则不是）。

## 4. 框架的封装

### 4.1 Vue Router

vue-router 默认 hash 模式 —— 使用 URL 的 hash 来模拟一个完整的 URL，于是当 URL 改变时，页面不会重新加载。

如果不想要很丑的 hash，我们可以用路由的 history 模式，这种模式充分利用 `history.pushState` API 来完成 URL 跳转而无须重新加载页面。

``` javascript
const router = new VueRouter({
  mode: 'history',
  routes: [...]
})
```

### 4.2 React Router

React Router 是建立在 history 之上的。 简而言之，一个 history 知道如何去监听浏览器地址栏的变化， 并解析这个 URL 转化为 location 对象， 然后 router 使用它匹配到路由，最后正确地渲染对应的组件。

常用的 history 有三种形式， 但是你也可以使用 React Router 实现自定义的 history。

* browserHistory

``` javascript
render(
  <Router history={browserHistory} routes={routes} />,
  document.getElementById('app')
)
```
* hashHistory

> Hash history 不需要服务器任何配置就可以运行，如果你刚刚入门，那就使用它吧。但是我们不推荐在实际线上环境中用到它，因为每一个 web 应用都应该渴望使用 browserHistory。

* createMemoryHistory

---
## 参考资料
* [前端路由的前生今世及实现原理](https://segmentfault.com/a/1190000011967786)
* [前端路由的两种实现原理](https://segmentfault.com/a/1190000007238999)
* [Manipulating the browser history](https://developer.mozilla.org/zh-CN/docs/Web/API/History_API)
* [Vue Router 前端路由源码](https://github.com/vuejs/vue-router/tree/dev/src/history)