# Router 的封装

## 路由原理

在解析源码前，先来了解下前端路由的实现原理。 前端路由实现起来其实很简单，本质就是监听 URL 的变化，然后匹配路由规则，显示相应的页面，并且无须刷新。目前单页面使用的路由就只有两种实现方式：

- hash 模式
![hash 流程图](https://user-gold-cdn.xitu.io/2018/7/11/164888109d57995f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
- history 模式
![History 流程图](https://user-gold-cdn.xitu.io/2018/7/11/164888478584a217?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## Vue Router

这个源码解析都是按照网站上的解析，加上自己的一点点理解总结的。
![](https://img-blog.csdn.net/20171008190257354?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FveGluaHVpNTIx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

我们使用 Vue Router，在初始化的时候通常经历了以下步骤：
1. `Vue.use(VueRouter)` 注册：
   * `initUse()`：调用插件 plugin 的 `install` 方法
   * `install()`：将插件的钩子函数混入到 Vue 中，注册全局组件 `RouterView` 和 `RouterLink`
2. `VueRouter` 实例化：
   * `matcher`：路由匹配对象
   * `history`：根据 `mode` 实例化 `History` 对象
3. 创建路由匹配对象，将 `routes` 对象集合映射为一张表：
   * `createMatcher()`：调用 `createRouteMap()` 创建路由映射表
   * `createRouteMap(routes)`：创建映射表，为每个配置调用 `addRouteRecord()` 添加路由记录
   * `addRouteRecord()`：为每个 `route` 添加路由记录，返回 `pathList`、`pathMap` 和 `nameMap` 三个值。它完成了：
     * 递归配置
     * 别名配置
     * 命名路由配置
4. 路由的初始化，触发组件渲染：
   1. 根组件调用 `beforeCreate()` 钩子函数，由于根组件有 `router` 属性，它的初始化会带动路由初始化
   2. 判断路由模式，添加对应的监听方法和路由跳转方法 `transitionTo()`
   3. 对组件的 `_route` 属性赋值，触发组件渲染

### 路由跳转的步骤（核心）：
1. 跳转时触发跳转方法 `transitionTo()`
2. 获取匹配的路由 `match()`：
   * 序列化 url
   * 处理命名路由
3. 创建路由 `_createRoute()`：
   1. 创建路由对象 `Route`
   2. 调用 `formatMatch()` 获得包含当前路由的所有嵌套路径片段的路由记录列表
   3. 处理重定向路由
4. 确认过渡，处理渲染节点的变化，执行导航守卫队列：
   1. 处理 `abort()` 中断
   2. 调用 `resolveQueue()` 方法对比路由记录列表，确定需要渲染的组件路由：
      * `updated`
      * `deactivated`
      * `activated`
   1. 执行导航守卫队列 `runQueue()`：
      * `extractLeaveGuards(deactivated)`：失活的组件钩子
      * `beforeHooks()`：全局 `beforeEach()` 钩子
      * `extractUpdateHooks(updated)`：当前路由改变，该组件被复用时调用
      * `beforeEnter()`：需要渲染组件守卫钩子
      * `resolveAsyncComponents(activated)`：解析异步路由组件
   2. 队列回调调用 `extractEnterGuards(activated)`，执行需要渲染组件的导航守卫钩子
   3. 执行 `beforeResolve()` 导航守卫钩子
   4. 调用 `afterEach()` 导航守卫钩子
   5. 对组件的 `_route` 属性赋值，触发组件渲染

其他方面：
* `<router-link />` 绑定了 `click()` 方法，触发 `history.push()` 或者 `history.replace()`,从而触发 `history.transitionTo()`
* 同时监控 `hashchange()` 和 `popstate()` 来对路由变化作对应的处理


---

## 参考资料
* [vuejs/vue-router](https://github.com/vuejs/vue-router/tree/65de048ee9f0ebf899ae99c82b71ad397727e55d/src)
* [VueRouter 源码深度解析](https://www.cnblogs.com/zhangycun/p/9403339.html)
* [vue-router的使用及实现原理](https://blog.csdn.net/caoxinhui521/article/details/77688512)