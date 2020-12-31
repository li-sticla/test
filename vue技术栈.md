#  						  **vue-router**

------

## （一）路由基础

### 1）router-link和router-view

* #### router-link组件

封装了一个`<a>`标签，点击可进行路由的跳转

具有专用属性`to`，可指定要跳转的路径，如`<router-link to='/path'>`或指定跳转的对象，此时需要使用v-bind进行绑定，如`<router-link v-bind:to='{name："pathname"}'>`

**若中间包裹内容，需要闭合标签，不能使用开标签，如：`</router-link>`**

```html
<router-link to='/'>Home<router-link/>
```

* #### router-view组件 

视图渲染组件，通过`router-link`跳转到某个页面所加载的组件会在`router-view`内渲染,可同时存在多个`router-view`

```html
<router-view>在此处渲染<router-view/>
<router-view name='a'><router-view/>
```



### 2）路由配置

新建文件`router.js`用于配置路由列表

```js
import Home from '@/views/Home'
import Login from '@/views/Login'

export default [
    {
        path:'/',
        name:'home',
        alias:'/home',
        component:Home
    },
    {
        path: '/login',
        name: 'login',
        component: Login
    },
    {
        path: '/register',
        name: 'register',
            // 懒加载,访问页面时加载vue组件
        component: () => import('@/views/Register.vue')
    }
]

```

新建文件`index.js`用于创建路由实例

```js
import Vue from 'vue'
import Router from 'vue-router'
import routes from './router'
// 加载router
Vue.use(Router)
// 创建实例
const router= new Router({
  routes,
  mode:'history'
})
//暴露路由接口
export default router 
```

#### a.动态路由

* 动态路由匹配

使用动态路径参数，一个“路径参数”使用冒号 `:` 标记。当匹配到一个路由时，参数值会被设置到 `this.$route.params`，可以在每个组件内使用。

```js
    {
        path:'/user/:name',
        component: () =>import('@/views/User.vue'),
    },
```

user.vue组件通过路由获取参数：

```html
<template>
  <div>
    {{ $route.params.name }}
  </div>
</template>
```

可以在一个路由中设置多段“路径参数”.对应的值都会设置到 `$route.params` 中。

| 模式                          | 匹配路径            | **$route.params**                    |
| ----------------------------- | ------------------- | ------------------------------------ |
| /user/:username               | /user/evan          | { username: 'evan' }                 |
| /user/:username/post/:post_id | /user/evan/post/123 | { username: 'evan', post_id: '123' } |

**使用路由参数时，例如从`/user/a`导航到`/user/b`，相应的组件实例会被复用。两个路由渲染同一个组件，比起销毁再创建，复用显得更加高效。因此，组件的生命周期钩子不会再调用。**

#### b.嵌套路由

实际生活中的应用界面，通常由多层嵌套的组件组合而成。URL 中各段动态路径也按某种结构对应嵌套的各层组件。

创建父路由parent，由组件parent渲染，子路由child，由组件child渲染。要在嵌套的出口中渲染组件，需要在 `VueRouter` 的参数中使用 `children` 配置：

```js
   {
        path:'/parent',
        name:'parent',
        component: () => import('@/views/Parent.vue'),
        children:[
                {
                    path:'child',
                    component: () => import('@/views/child.vue')
                }
        ]
    } 
```

**注意：子路由的path不需要加`/`**

Parent组件中包含自己的嵌套 `<router-view>`，用于渲染子路由的组件

```html
<template>
  <div>
parent
    <router-view />
  </div>
</template>
```

#### c.命名路由

可以使用名称来标识路由

```js
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',
      component: User
    }
  ]
})
```

要链接到一个命名路由，可以给`router-link`的`to`属性传一个对象

```html
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
```

这和代码调用 `router.push()` 是一回事：

```js
router.push({ name: 'user', params: { userId: 123 }})
```

#### d.命名视图

想同时 (同级) 展示多个视图，而不是嵌套展示，可以使用命名视图。一个视图使用一个组件渲染，因此对于同个路由，多个视图就需要多个组件。在components内可指定命名视图需要的组件，如果 `router-view` 没有设置名字，那么默认为 `default`。

```js
    {
        path:'/settings',
        components: {
            default: () => import('@/components/UserSettings.vue'),
            foot: () => import('@/components/foot.vue')
        }
```

```html
<template>
  <div id="app">
    <router-view />
    <router-view name="foot"/>
  </div>
</template>
```

### 3）JS操作路由(编程式导航)

* #### router.push

在Vue实例内部，可以通过`$router`访问路由实例。想要导航到不同的URL，可以使用`router.push`方法。该方法会向history栈中添加一个新的记录，当用户点击浏览器回退按钮时会回到跳转前URL。

当点击`<router-link>`时，会在内部调用该方法，该方法的参数可以是字符串路径，也可以是描述地址的对象。



```js
// 字符串
router.push('home')

// 对象
router.push({ path: 'home' })

// 命名的路由
router.push({ name: 'user', params: { userId: '123' }})

// 带查询参数，变成 /register?plan=private
router.push({ path: 'register', query: { plan: 'private' }})
```

 

**注意：如果提供了`path`，`params`将被忽略 **



```js
const userId = '123'
router.push({ name: 'user', params: { userId }}) // -> /user/123
router.push({ path: `/user/${userId}` }) // -> /user/123
// 这里的 params 不生效
router.push({ path: '/user', params: { userId }}) // -> /user
```



可以使用JS操作该方法：

```html
<template>
    <div class="home">
        <button @click="handleClick('back')">点击后退</button>
        <button @click="handleClick('push')">点击前进</button>
        <button @click="handleClick('replace')">点击替换</button>
    </div>
</template>
```

 ```js
<script>
export default {
    name:'home',
    methods:{
        handleClick(type){
            if(type == 'back'){
                this.$router.back()
            }
            else if(type == 'push'){
                this.$router.push({
                    name: 'parent'
                })
            }
            else if(type == 'replace'){
                this.$router.replace({
                    name:'parent'
                })
            }
        }
    }
}
</script>
 ```

* #### router.replace

与`router.push`类似，不同的是，该方法不是向history栈中添加新纪录，而是取代当前的记录

* #### router.go(n)

该方法的参数为一个整数，意为在history记录中前进或后退几步



```js
// 在浏览器记录中前进一步，等同于 history.forward()
router.go(1)

// 后退一步记录，等同于 history.back()
router.go(-1)

// 前进 3 步记录
router.go(3)

// 如果 history 记录不够用，那就默默地失败呗
router.go(-100)
router.go(100)
```

 

### 4）重定向和别名

* #### 重定向

可在`routes`内进行配置：

```js
    {
       path:'/personal' ,
       component: () => import('@/components/Personal.vue'),
       redirect: '/'
       }
    }
```

 

重定向的目标也可以是一个命名的路由：

```js
    {
       path:'/personal' ,
       component: () => import('@/components/Personal.vue'),
       redirect: {name:'parent'}
    },
```

 

也可传入方法：

```
    {
       path:'/personal' ,
       component: () => import('@/components/Personal.vue'),
       redirect: to => {
			console.log(to)
       }
    }
```

```js
    {
       path:'/personal' ,
       component: () => import('@/components/Personal.vue'),
       redirect: to => {
           return{
            //    '/parent'
               name:'parent'
           }
       }
    } 
```

* ####  别名 

路由可以取别名，使得访问路由别名对应的路径时可以导向该路由：

```js
    {
        path:'/',
        name:'home',
        alias:'/home',
        component:Home
    } 
```

例如，当用户访问`/home`时，URL会保持为`/home`，但是路由匹配为`/`

# （二）路由进阶

## 1）路由组件传参

在页面中如果要通过路由获取参数以进行逻辑处理，可以通过`route`实例获取相关参数。在组件中使用 `$route` 会使之与其对应路由形成高度耦合，从而使组件只能在某些特定的 URL 上使用，限制了其灵活性。



```js
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User }
  ]
})
```

 

### a.布尔模式

如果是动态路由，此时可使用 `props` 将组件和路由解耦，`props:true`表示`props`中的参数会默认将`$route.params`作为组件的属性：



```js
const User = {
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User, props: true },

    // 对于包含命名视图的路由，你必须分别为每个命名视图添加 `props` 选项：
    {
      path: '/user/:id',
      components: { default: User, sidebar: Sidebar },
      props: { default: true, sidebar: false }
    }
  ]
})
```

 

### b.对象模式

如果是非动态路由, 传入的`props` 可以是一个对象，它会被按原样设置为组件属性。当 `props` 是静态的时候有用。

```html
<template>
    <div>
        <p>{{food}}</p>
    </div>
</template>
```

```js
<script>
export default {
    props:{
        food: {
            type:String,
            default:'apple'
        }
    }
}
</script>
```

```js
    {
        path: '/about',
        name: 'about',
        component: () => import('@/views/About.vue'),
            //传入静态数据
       props:{
           food: 'banana'
       }
    }
```



### c.函数模式

可以创建一个函数返回 `props`。

例子中此时`props`的参数为`route`对象，通过对`route`对象进行操作可以设置传入组件的属性值：

```js
    {
        path: '/about',
        name: 'about',
        component: () => import('@/views/About.vue'),
       props:route =>({
           food: route.query.food
       })
    },
```

URL `/about?food=banana` 会将 `{query: 'banana'}` 作为属性传递给 `About `组件。



## 2）History模式

### 为何要使用History模式？

vue-router默认使用hash模式，使用URL的hash来模拟一个完整的URL。hash即地址栏URL中的`#`（锚点）,表示网页中的一个位置，改变`#`后的部分不会重新加载页面，即hash不会被包括在http请求中，对后端没有影响。每次hash值的改变都会在浏览器的访问记录中增加一条记录，所以hash模式通过锚点的值的改变，根据不同的值，在DOM处渲染不同的数据。

但是，在URL中包含`#`并不美观，使用History模式即可实现在修改URL的同时不向后端发送请求，这种模式充分利用了html5 history interface中新增的`pushState()`和`replaceState()`方法。除此之外，History模式还包含以下优点：

1. `pushState()`设置的新URL可以是与当前URL同源的URL，而hash只可修改`#`后的值，设置的URL只能是与当前同文档的URL。
2. `pushState()`设置的新URL可以与当前URL相同，这样也会把记录添加到栈中，而hash设置的 URL必须与原URL不同才会触发动作将记录添加到栈中。

3. `pushState()`通过`stateObject`参数可以添加任意类型的数据到记录中，而hash只可以添加短字符串。
4. `pushState()`可额外设置`title`属性供后续使用。

当然History模式也并不是完全优于hash模式。在用户手动输入URL后回车时，或是浏览器刷新时，hash模式下仅`#`符号之前的内容会被包含到请求中，对于后端来说，即使没有做到路由全覆盖，也不会出现404错误。而History模式下前端URL必须和向后端发送请求的URL完全一致，如果后端缺少对路由的覆盖处理，就会返回404错误。因此要使用History模式需要对后端进行配置。

```js
const router = new VueRouter({
routes,
mode:'History'
})
```

## 3）导航守卫

## 4）路由元信息

## 5）过渡效果

# **vueX**

----

## （一）状态管理Bus

一般在项目中，状态管理都是使用Vue官方提供的Vuex

当在多组件之间共享状态变得复杂时，使用Vuex，此外也可以使用Bus来进行简单的状态管理

### 1）父子组件通信

---

例子：v-model双向绑定的实现

子组件 :

```javascript
<template>
    <input @input="changeData" :value='value'>
</template>
<script>
export default {
    name:'Ainput',
    props:{
        value: {
           type: [String,Number],
           default: '',
        }
    },
    methods:{
        changeData(event){
            const value = event.target.value
            this.$emit('input',value)
        }
    }
}
</script>
```

父组件：

```js
<template>
  <div>
    <Ainput @input="handleInput" :value="inputValue"/> //等效于 v-model=‘inputValue’
    <p>{{ inputValue }}</p>
  </div>
</template>
<script>
import Ainput from "@/components/Ainput.vue";

export default {
  name: "store",
  data() {
    return {
      inputValue: "aaaaa"
    };
  },
  components: {
    Ainput
  },
  methods: {
    handleInput(val) {
      this.inputValue = val;
    }
  }
};
</script>
```



过程：

子组件向父组件发送数据，通过`$emit`触发`input`事件，来调用父组件中的方法，修改父组件中的值。

这里子组件将value的值以参数的形式，通过事件提交给父组件。

父组件接收后，通过`input`的事件监听器调用`handleInput`方法，修改`value`属性的值。

父子组件通信完成。

### 2）兄弟组件通信

---









