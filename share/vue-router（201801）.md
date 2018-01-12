## 写在前面

用户进行了交互操作，现在要对页面内容进行变更，此时可以通过javascript进行动态替换DOM，但是其不便于分享、收藏，对于搜索引擎和用户来说都是不友好的！

**什么是前端路由？**

​	根据不同的 url 地址展示不同的内容或页面，无需依赖服务器根据不同URL进行页面展示操作

**优点**

- 用户体验好，不需要每次都从服务器全部获取，快速展现给用户

**缺点**

- 使用浏览器的前进，后退键的时候会重新发送请求，没有合理地利用缓存
- 单页面无法记住之前滚动的位置，无法在前进，后退的时候记住滚动的位置

## 简介

​	使用 Vue.js ，可以通过组合组件来组成应用程序，当你要把 vue-router 添加进来，我们需要做的是，将组件(components)映射到路由(routes)，然后告诉 vue-router 在哪里渲染它们。

```javascript
// 1. 定义、引用（路由）组件。
const Foo = { template: '<div>foo</div>' }
import Bar from '@/views/bar.vue'

// 2. 定义路由
const routes = [
  { path: '/foo', name: 'foo', component: Foo },
  { path: '/bar', name: 'bar', component: Bar }
]

// 3. 创建 router 实例，然后传 `routes` 配置
const router = new VueRouter({
  routes // （缩写）相当于 routes: routes
})

// 4. 创建和挂载根实例。通过 router 配置参数注入路
const app = new Vue({
  router
}).$mount('#app')

```

## 动态路由匹配

两种方式传递`$route.params`和`$route.query`

| 模式              | 匹配路径                  | 获取参数（路由信息对象）             |
| --------------- | --------------------- | ------------------------ |
| /user/:username | /user/ligang          | `$route.params.username` |
| /user?:username | /user?username=ligang | `$route.query.username`  |

### 响应路由参数的变化

​	当使用路由参数时，例如从 `/user/ligang` 导航到 `user/lg`，**原来的组件实例会被复用**。因为两个路由都渲染同个组件，比起销毁再创建，复用则显得更加高效。**不过，这也意味着组件的生命周期钩子不会再被调用**。

**方式一：简单地watch(监测变化)$route对象**

```javascript
 watch: {
    '$route' (to, from) {
      // 对路由变化作出响应...
    }
  }
```

**方式二：使用 2.2 中引入的 `beforeRouteUpdate` 守卫**

```javascript
beforeRouteUpdate (to, from, next) {
  // 对路由变化作出响应...不要忘记调用next()
}
```

**示例**：新增和编辑使用同一模块，从编辑切换到新增页面信息不会更新！

```javascript
{
  path: 'add',
  name: 'setting-user-manager-add',
  component: () => import('@/views/setting/user-manager/add-edit.vue'),
  meta: {name: '用户新增'}
}, {
  path: 'edit',
  name: 'setting-user-manager-edit',
  component: () => import('@/views/setting/user-manager/add-edit.vue'),
  meta: {
      name: '用户编辑',
      hidden: true
  }
}
```

## 嵌套路由

```javascript
routes: [{ 
    path: '/user/:id', 
    component: User,
    children: [
        // 匹配 /user/:id
        { path: '', component: UserHome },
        // 匹配 /user/:id/profile
        { path: 'profile', component: UserProfile },
        // 匹配 /user/:id/posts
        { path: 'posts', component: UserPosts }
    ]
}]
```

**要注意，以 / 开头的嵌套路径会被当作根路径。 这让你充分的使用嵌套组件而无须设置嵌套的路径。**

## 编程式导航

### `router.push(location, onComplete?, onAbort?)`

| 声明式                       | 编程式                |
| ------------------------- | ------------------ |
| `<router-link :to="...">` | `router.push(...)` |

```javascript
// 方式一：字符串路径
router.push('/user')
// 方式二：path对象
router.push({ path: '/user' })
// 方式三：路由名称
router.push({ name: 'user'})
```

**注意：如果提供了 path，params 会被忽略，query不会！！**

```javascript
// 不生效
router.push({ path: '/user', params: { id: 1 }})
// params生效 /user/1
router.push({ name: 'user', params: { id: 1 }}) // 使用name方式
router.push({ path: `/user/1` }) // 直接在path上扩充
// query不受影响 /user?id=1
router.push({ path: '/user', query: { id: 1 }})
```

### `router.replace(location, onComplete?, onAbort?)`

| 声明式                               | 编程式                   |
| --------------------------------- | --------------------- |
| `<router-link :to="..." replace>` | `router.replace(...)` |

跟`router.push` 很像，唯一的不同就是，它不会向 history 添加新记录！

### `router.go(n)`

在 history 记录中向前或者后退多少步，类似 `window.history.go(n)`

## 命名视图

多个非嵌套视图展示，例如创建一个布局，有`header`头信息、 `sidebar`（侧导航） 和 `main`（主内容） 两个视图。

```vue
<router-view class="view header" name="header"></router-view>
<router-view class="view sidebar" name="sidebar"></router-view>
<router-view class="view main"></router-view>
```

```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: mainComponent,
        sidebar: sidebarComponent,
        header: headerComponent
      }
    }
  ]
})
```



*gsp中header和sidebar也可以通过这种方式*



## 重定向和别名

### 重定向

```javascript
// 方式一：字符串路径path
{ path: '/a', redirect: '/b' }
// 方式二：name
{ path: '/a', redirect: {name: 'b'} }
// 方式三：动态返回重定向目标
{ path: '/a', redirect: to => {
  /* 方法接收 目标路由 作为参数；return 重定向的 字符串路径/路径对象 */
}}
```

### 别名

`/a`的别名是`/b`，意味着当用户访问`/b`时，URL会保持为`/b`，但是路由匹配则为`/a`，就像用户访问`/a`一样。

```javascript
{ path: '/a', component: A, alias: '/b' }
```

『别名』的功能让你可以自由地将 UI 结构映射到任意的 URL，而不是受限于配置的嵌套路由结构。

示例：上述【动态路由匹配】可修改成如下，可能存在name问题

```javascript
{
  path: 'add',
  name: 'setting-user-manager-add',
  component: () => import('@/views/setting/user-manager/add-edit.vue'),
  meta: {name: '用户新增'},
  alias: 'edit'
}
```



*思考：上述add、edit使用别名是否更好！*



## 向路由组件传递 props

### 路由组件传参

**默认(常规)方式：通过$route.params获取**

```vue
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
const router = new VueRouter({
  routes: [{ path: '/user/:id', component: User }]
})
```

**使用props解耦：只需要将props设置为true**

```vue
const User = {
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}
const router = new VueRouter({
  routes: [{ path: '/user/:id', component: User, props: true }]
})
```

注意：上述props不仅可以设置为布尔值，还可以设置为对象或函数，具体请查看:「[https://router.vuejs.org/zh-cn/essentials/passing-props.html](https://router.vuejs.org/zh-cn/essentials/passing-props.html)」



*poc中使用params传值，是不是又多了一种获取方式？😄*



## HTML5 History 模式

```
const router = new VueRouter({
  mode: 'history',
  routes: [...]
})
```

需要后台配置，否则输入的除首页外都为404（当然系统内跳转可以）。具体ngix、Apache、node等配置参考：「[https://router.vuejs.org/zh-cn/essentials/history-mode.html](https://router.vuejs.org/zh-cn/essentials/history-mode.html)」

**这里说一下本地webpack需要增加的配置情况：**`historyApiFallback: true`「[https://doc.webpack-china.org/configuration/dev-server/#devserver-historyapifallback](https://doc.webpack-china.org/configuration/dev-server/#devserver-historyapifallback)」vue-cli生成的默认webpack配置，

```javascript
historyApiFallback: {
  rewrites: [
    { from: /.*/, to: path.join(config.dev.assetsPublicPath, 'index.html') },
  ],
}
```

在window下特定node版本会有问题！



## 导航守卫

> 『导航』表示路由正在发生改变

导航守卫主要用来通过跳转或取消的方式守卫导航。注意**参数或查询的改变并不会触发进入/离开的导航守卫**。可以通过[观察 `$route` 对象](https://router.vuejs.org/zh-cn/essentials/dynamic-matching.html#%E5%93%8D%E5%BA%94%E8%B7%AF%E7%94%B1%E5%8F%82%E6%95%B0%E7%9A%84%E5%8F%98%E5%8C%96)来应对这些变化，或使用 `beforeRouteUpdate` 的组件内守卫。



*@上述哪个地方提到？*



### 全局守卫

使用 `router.beforeEach` 注册一个全局前置守卫

```javascript
const router = new VueRouter({ ... })
router.beforeEach((to, from, next) => {
  // ...
})
```

守卫是异步解析执行，此时导航在所有守卫 resolve 完之前一直处于 **等待中**。所以**确保要调用 next 方法，否则钩子就不会被 resolved**。



登录拦截处理！伪代码



### 全局解析守卫

在 2.5.0+ 你可以用 `router.beforeResolve` 注册一个全局守卫。这和 `router.beforeEach` 类似，区别是在导航被确认之前，**同时在所有组件内守卫和异步路由组件被解析之后**，解析守卫就被调用。



@是否可以解决异步同步差异化的问题！



### 全局后置钩子

你也可以注册全局后置钩子，然而和守卫不同的是，这些钩子不会接受 `next` 函数也不会改变导航本身：

```javascript
router.afterEach((to, from) => {
  // ...
})
```



### 路由独享的守卫

```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      }
    }
  ]
})
```

### 组件内的守卫

- `beforeRouteEnter`
- `beforeRouteUpdate` (2.2 新增)
- `beforeRouteLeave`

需要注意的是beforeRouteEnter不能访问this，可以通过传一个回调给 `next`来访问组件实例。

```javascript
beforeRouteEnter (to, from, next) {
  next(vm => {
    // 通过 `vm` 访问组件实例
  })
}
```

### 完整的导航解析流程

1. 导航被触发。
2. 在失活的**组件内**调用离开守卫`beforeRouteLeave`。
3. 调用**全局**的 `beforeEach` 守卫。
4. 在重用的**组件内**调用 `beforeRouteUpdate` 守卫 (2.2+)。
5. 在**路由**配置里调用**独享**守卫 `beforeEnter`。
6. 解析异步路由组件。
7. 在被激活的**组件内**调用 `beforeRouteEnter`。
8. 调用**全局**的 `beforeResolve` 守卫 (2.5+)。
9. 导航被确认。
10. 调用**全局**的 `afterEach` 钩子。
11. 触发 DOM 更新。
12. 用创建好的实例调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数。

## 路由元信息

`meta` 字段来设置名称、是否需要验证、是否隐藏等附加信息！！

一个路由匹配到的所有路由记录会暴露为 `$route` 对象（还有在导航守卫中的路有对象）的 `$route.matched` 数组。因此，我们需要遍历 `$route.matched` 来检查路由记录中的 `meta` 字段。

```javascript
if (to.meta.requireAuth) {   // 判断该路由是否需要登录权限
    if (store.state.token) {  // 通过vuex state获取当前的token是否存在
        next();
    }else {
        next({
            path: '/login',
            query: {redirect: to.fullPath}  // 将跳转的路由path作为参数，登录成功后跳转到该路由
        })
    }
}else {
    next();
}
```



## 数据获取

有时候，进入某个路由后，需要从服务器获取数据。

- **导航完成之后获取**：先完成导航，然后在接下来的组件生命周期钩子中获取数据。在数据获取期间显示『加载中』之类的指示。

  该方式会马上导航和渲染组件，然后在组件的 `created` 钩子中获取数据。这让我们有机会在数据获取期间展示一个 loading 状态，还可以在不同视图间展示不同的 loading 状态。

- **导航完成之前获取**：导航完成前，在路由进入的守卫中获取数据，在数据获取成功后执行导航。

  该方式在导航转入新的路由前获取数据。我们可以在接下来的**组件内**的 `beforeRouteEnter` 守卫中获取数据，当数据获取成功后只调用 `next` 方法。



使用第二种方式会有什么问题呢？脑洞～～～

beforeRouteEnter会有什么问题呢？？？



## 滚动行为

**只在 HTML5 history 模式下可用。**当切换到新路由时，想要页面滚到顶部，或者是保持原先的滚动位置，就像重新加载页面那样。 `vue-router` 能做到，而且更好，它让你可以自定义路由切换时页面如何滚动。

```javascript
const router = new VueRouter({
  routes: [...],
  scrollBehavior (to, from, savedPosition) {
    // return 期望滚动到哪个的位置
  }
})
```

参考地址：「[https://router.vuejs.org/zh-cn/advanced/scroll-behavior.html](https://router.vuejs.org/zh-cn/advanced/scroll-behavior.html)」



## 特别说明

### Router 实例

| 属性                  | 说明                 |
| ------------------- | ------------------ |
| router.app          | `router` 的 Vue 根实例 |
| router.mode         | 路由使用的模式            |
| router.currentRoute | 当前路由对应的**路由信息对象**  |

方法：router.beforeEach(guard)、router.beforeResolve(guard) 、router.afterEach(hook)、router.push(location, onComplete?, onAbort?)、router.replace(location, onComplete?, onAbort?)、router.go(n)、router.back()、router.forward()、router.getMatchedComponents(location?)、router.resolve(location, current?, append?)、router.addRoutes(routes)、router.onReady(callback, [errorCallback])、router.onError(callback)

### 路由信息对象

每次成功的导航后都会产生一个新的对象

- 在组件内，即 `this.$route`

- 在 `$route` 观察者回调内

- `router.match(location)` 的返回值

- 导航守卫的参数：

  ```javascript
  router.beforeEach((to, from, next) => {
    // to 和 from 都是 路由信息对象
  })
  ```

- `scrollBehavior` 方法的参数:

  ```javascript
  const router = new VueRouter({
    scrollBehavior (to, from, savedPosition) {
      // to 和 from 都是 路由信息对象
    }
  })
  ```


其包含的属性值：**$route.path**、**$route.params**、**$route.query**、**$route.hash**、**$route.fullPath**、**$route.matched**



> **重点强调**：this.$router.currentRoute === this.$route

### 行为表现

因为它也是个组件，所以可以配合 `<transition>` 和 `<keep-alive>` 使用。如果两个结合一起用，要确保在内层使用 `<keep-alive>`：

```vue
<transition>
  <keep-alive>
    <router-view></router-view>
  </keep-alive>
</transition>
```

Keep-alive说明一下！！！



### router-link

`<router-link>` 比起写死的 `<a href="...">` 会好一些，理由如下：

- 无论是 HTML5 history 模式还是 hash 模式，它的表现行为一致，所以，当你要切换路由模式，或者在 IE9 降级使用 hash 模式，无须作任何变动。

- 在 HTML5 history 模式下，`router-link` 会守卫点击事件，让浏览器不再重新加载页面。

- 当你在 HTML5 history 模式下使用 `base` 选项之后，所有的 `to` 属性都不需要写（基路径）了。

  base相关说明：「[https://router.vuejs.org/zh-cn/api/options.html#base](https://router.vuejs.org/zh-cn/api/options.html#base)」




## 实例

### header编写

**第一步：**获取router的全部配置信息`import {ROUTES} from '@/app.router'`，然后循环铺值（获取一级的路由）`meta.name`

**第二步：**选择header，路由跳转；主要思路：在一级组件上配置`meta.defaultRouteName`信息，获取该信息后，进行调整（如果不含有该信息，则默认第一个子路由）

**第三步：**处理当前选中的的header项目

```javascript
 watch: {
  '$route': {
    // 必须，解决路由同步加载组件时，$watch首次不执行的问题
    immediate: true,
    handler (to) {
      if (to.matched[0]) {
        this.currentRoute = to.matched[0].name
      }
    }
  }
}
```

### sidebar编写

基本思路和header相似，唯一区别是要根据当前一级路由进行铺值！

