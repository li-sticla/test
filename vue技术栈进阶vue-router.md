#  						  vue技术栈:vue-router

------

## （一）路由基础

### 1）router-link和router-view

* router-link组件

封装了一个`<a>`标签，点击可进行路由的跳转

具有专用属性`to`，可指定要跳转的路径，如`<router-link to='/path'>`或指定跳转的对象，此时需要使用v-bind进行绑定，如`<router-link v-bind:to='{name：‘pathname’}'>`

**若中间包裹内容，需要闭合标签，不能使用开标签，如：`</router-link>`**

![image-20201122182425997](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122182425997.png) 

* router-view组件 

视图渲染组件，通过router-link跳转到某个页面所加载的组件会在router-view内渲染

可同时存在多个router-view

### 2）路由配置

新建文件router.js用于配置路由列表

![image-20201122183535423](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122183535423.png) 

新建文件index.js用于创建路由实例

![image-20201122183659837](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122183659837.png) 



#### a.动态路由

* 动态路由匹配

使用动态路径参数，一个“路径参数”使用冒号 `:` 标记。当匹配到一个路由时，参数值会被设置到 `this.$route.params`，可以在每个组件内使用。

![image-20201122201622497](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122201622497.png)  

user.vue组件通过路由获取参数：

![image-20201122201751162](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122201751162.png) 

可以在一个路由中设置多段“路径参数”.对应的值都会设置到 `$route.params` 中。

| 模式                          | 匹配路径            | **$route.params**                    |
| ----------------------------- | ------------------- | ------------------------------------ |
| /user/:username               | /user/evan          | { username: 'evan' }                 |
| /user/:username/post/:post_id | /user/evan/post/123 | { username: 'evan', post_id: '123' } |

**使用路由参数时，例如从`/user/a`导航到`/user/b`，相应的组件实例会被复用。两个路由渲染同一个组件，比起销毁再创建，复用显得更加高效。因此，组件的生命周期钩子不会再调用。**

#### b.嵌套路由

实际生活中的应用界面，通常由多层嵌套的组件组合而成。URL 中各段动态路径也按某种结构对应嵌套的各层组件。

创建父路由parent，由组件parent渲染，子路由child，由组件child渲染。要在嵌套的出口中渲染组件，需要在 `VueRouter` 的参数中使用 `children` 配置：

![image-20201122205131198](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122205131198.png) 

**注意：子路由的path不需要加`/`**

Parent组件中包含自己的嵌套 `<router-view>`，用于渲染子路由的组件

![image-20201122205443465](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122205443465.png) 

#### c.命名路由

可以使用名称来标识路由

![image-20201122212711641](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122212711641.png) 

要链接到一个命名路由，可以给`<router-link`的`to`属性传一个对象

![image-20201122212603287](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122212603287.png) 

这和代码调用 `router.push()` 是一回事：

![image-20201122212630141](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122212630141.png) 

#### d.命名视图

想同时 (同级) 展示多个视图，而不是嵌套展示，可以使用命名视图。一个视图使用一个组件渲染，因此对于同个路由，多个视图就需要多个组件。在components内可指定命名视图需要的组件，如果 `router-view` 没有设置名字，那么默认为 `default`。

![image-20201122212924547](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122212924547.png) 



### 3）JS操作路由(编程式导航)

* #### router.push

在Vue实例内部，可以通过$router访问路由实例。想要导航到不同的URL，可以使用`router.push`方法。该方法会向history栈中添加一个新的记录，当用户点击浏览器回退按钮时会回到跳转前URL。

当点击`<router-link>`时，会在内部调用该方法，该方法的参数可以是字符串路径，也可以是描述地址的对象。

![image-20201122220602524](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122220602524.png) 

**注意：如果提供了path，params将被忽略 **

![image-20201122220803087](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122220803087.png) 

可以使用JS操作该方法：

![image-20201122221732413](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122221732413.png) 

![image-20201122222031174](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122222031174.png) 

* #### router.replace

与`router.push`类似，不同的是，该方法不是向history栈中添加新纪录，而是取代当前的记录

* #### router.go(n)

该方法的参数为一个整数，意为在history记录中前进或后退几步

![image-20201122222255456](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122222255456.png) 

### 4）重定向和别名

* #### 重定向

可在routes内进行配置：

 ![image-20201122214611500](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122214611500.png) 

重定向的目标也可以是一个命名的路由：

![image-20201122214435459](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122214435459.png) 

也可传入方法：

![image-20201122214705928](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122214705928.png) 

![image-20201122214855559](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122214855559.png) 

* ####  别名 

路由可以取别名，使得访问路由别名对应的路径时可以导向该路由：

![image-20201122215509124](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122215509124.png) 

例如，当用户访问`/home`时，URL会保持为`/home`,但是路由匹配为`/`

# （二）路由进阶

## 1）路由组件传参

在页面中如果要通过路由获取参数以进行逻辑处理，可以通过route实例获取相关参数。在组件中使用 `$route` 会使之与其对应路由形成高度耦合，从而使组件只能在某些特定的 URL 上使用，限制了其灵活性。

![image-20201122223603695](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122223603695.png) 

### a.布尔模式

如果是动态路由，此时可使用 `props` 将组件和路由解耦，`props:true`表示props中的参数会默认将`$route.params`作为组件的属性：

![image-20201122223737565](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122223737565.png) 

### b.对象模式

如果是非动态路由：

![image-20201122230150927](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122230150927.png) 

![image-20201122230210523](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122230210523.png) 

### c.函数模式

可以创建一个函数返回 `props`。

例子中此时props的参数为route对象，通过对route对象进行操作可以设置传入组件的属性值：

![image-20201122231938420](C:\Users\yuanz\AppData\Roaming\Typora\typora-user-images\image-20201122231938420.png) 

URL `/about?food=banana` 会将 `{query: 'banana'}` 作为属性传递给 `About `组件。

## 2）History模式

## 3）导航守卫

## 4）路由元信息

## 5）过渡效果











