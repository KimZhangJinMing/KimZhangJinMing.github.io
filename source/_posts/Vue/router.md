---
title: 路由.md
date: 2020-12-06 20:03:35
categories: Vue
tags: router
---

### 切换路由

* 导入router，调用push方法

```vue
<script>
import router from './router'

export default {
  setup () {
    const currentRouter = window.location.pathname
    function changeView () {
      router.push('/about')
    }
    return {
      currentRouter,
      changeView
    }
  }
}

</script>
```

**知识点：**

* 当前访问路径可以通过window.location.pathname 获取
* import的时候，如果目录下只有一个index.js，可以写成import router from './router' 而不用写成 import router from './router/index.js'

### router-view监听子组件的事件

如果有多层组件，子组件的事件只能一层层往上冒泡，顶层不能直接监听到子组件的事件。

栗子：App.vue 包含了Home.vue，Home.vue包含了组件swiper



1.swiper向Home.vue组件传递事件img-click

```vue
<template>
  <img :src="src" :alt="alt" class="swiper" @click="$emit('img-click')">
</template>

<script>
export default {
  props: ['src', 'alt'],
  name: 'swiper'
}
</script>
```

2.Home.vue监听到img-click事件，向App.vue传递事件home-click

```vue
<template>
  <div class="home">
    <swiper :src="pic" :alt="漂亮的图片" @img-click="onHomeClick"></swiper>
  </div>
</template>

<script>
// @ is an alias to /src
import swiper from '../components/swiper'
import pic from '@/assets/beautiful.jpg'

export default {
  setup (props, context) {
    function onHomeClick () {
      console.log('接收到img-click事件')
      context.emit('home-click')
    }
    return {
      pic,
      onHomeClick
    }
  },
  name: 'Home',
  components: {
    // eslint-disable-next-line vue/no-unused-components
    swiper
  }
}
</script>

```

3.App.vue不能直接监听到img-click事件，只能监听Home.vue的home-click事件，监听到事件后，路由跳转到/about页面

```vue
<template>
  <div id="nav">
    <button @click="changeView">点击切换页面</button>
    <router-link to="currentRouter" ></router-link>
  </div>
  <router-view @home-click="onImgClick"/>
</template>

<script>
import router from './router'

export default {
  setup () {
    function onImgClick () {
      console.log('接收到home-click事件')
      router.push('/about')
    }
    const currentRouter = window.location.pathname
    function changeView () {
      router.push('/about')
    }
    return {
      currentRouter,
      changeView,
      onImgClick
    }
  }
}

</script>
```

**注意点：**

* 路由的监听事件是写在router-view中，而不是router-link



### 嵌套路由

* 配置路由的children属性，是一个数组
* 在页面中使用<router-view/>才能显示子页面
* 使用children属性配置的路由，只有在父页面的router-view才能显示

栗子：在Detail页面中，跳转到Detail页面的子页面SubDetail

1.Detail.vue

```vue
<template>
  <div class="about">
    <h1>This is an about page</h1>
    <button @click="goSubPage">点击去到子页面</button>
    <router-view/>
  </div>
</template>

<script>
import router from '@/router'
export default {
  setup () {
    // eslint-disable-next-line no-unused-vars
    function goSubPage () {
      router.push('/detail/sub-detail')
    }
    return {
      goSubPage
    }
  }
}
</script>

```

2.SubDetail.vue

```vue
<template>
  <div>
    <p>this is sub-detail page</p>
  </div>
</template>

<script>
export default {
  name: 'SubDetail'
}
</script>

<style scoped>

</style>
```

3.路由配置

**这里配置的sub-detail只有在/detail页面的<router-view/>才能显示。**

```js
const routes = [
  {
    path: '/detail',
    name: 'Detail',
    // route level code-splitting
    // this generates a separate chunk (about.[hash].js) for this route
    // which is lazy-loaded when the route is visited.
    component: () => import(/* webpackChunkName: "about" */ '../views/Detail.vue'),
    children: [
      {
        // 这里不能加/
        path: 'sub-detail',
        name: SubDetail,
        component: SubDetail
        // 两种写法都可以
        // component: () => import('../views/SubDetail')
      }
    ]
  }
]
```

一种更好的做法是：

​	不配置children属性，而是配置成平行的。子组件通过传递同一个自定义事件到最上层，通过自定义事件将路由传递到最上层，统一由最上层的事件进行router的跳转。



栗子：App.vue是最上层，监听着子组件的home-click事件。

Home.vue 和 Detail.vue 都可以通过向App.vue传递home-click自定义事件，带上自己跳转的路由参数，统一由App.vue进行路由的跳转。



1.App.vue

```vue
<router-view @home-click="onImgClick"/>

<script>
import router from './router'

export default {
  setup () {
    function onImgClick (event) {
      console.log('接收到home-click事件')
      router.push(event)
    }
    return {
      onImgClick
    }
  }
}
```

2.Home.vue

```vue
<swiper :src="pic" :alt="漂亮的图片" @img-click="onHomeClick"></swiper>

<script>
 setup (props, context) {
    function onHomeClick () {
      console.log('接收到img-click事件')
	  // 传递/detail
      context.emit('home-click', '/detail')
    }
    return {
      pic,
      onHomeClick
    }
  }
</script>
```

3.Detail.vue

```vue
<button @click="goSubPage">点击去到子页面</button>

<script>
export default {
  setup (props, context) {
   	// 传递/sub-detail
    function goSubPage () {
      context.emit('home-click', '/sub-detail')
    }
    return {
      goSubPage
    }
  }
}
</script>
```

### 路由参数

1.定义路由参数 :参数名 ， 如果定义了参数而不传参，会找不到路由，出现空白页面

```js
const routes = [
  {
    path: '/detail/:id',
    name: 'Detail',
    component: Detail
  }
]
```

2.传递参数

```js
function onHomeClick () {
	console.log('接收到img-click事件')
	context.emit('home-click', '/detail/3')
}
```

3.接收参数

通过router可以获取到当前路由currentRoute，再通过value.params.参数名获取路由参数

```js
 console.log(router.currentRoute.value.params.id)
```



### 命名路由

可以通过name来当路由，写法是一个对象

```js
context.emit('home-click', {
    name: 'Detail',
    params: {
        id: 3
    }
})
// 没有参数时，传递空对象
 context.emit('home-click', {
        name: 'SubDetail',
        params: {}
})
```

跳转路由时可以直接将子组件传递的对象当为参数。但是需要保证每个子组件传递的参数格式一致。

```js
 function onImgClick (event) {
     console.log('接收到home-click事件')
     console.log(event)
     router.push(event)
}
```



### 组件切换的几种方法

* 使用router
* 使用v-if隐藏显示
* 使用<component>动态组件



### 动态组件和缓存组件的状态

<component>的is属性能根据组件的名称，动态切换组件

```vue
<template>
  <button @click="changeView('keep')">切换keep组件</button>
  <button @click="changeView('stat')">切换stat组件</button>
  <keep-alive exclude="keep">
  	<component :is="currentPage"></component>
  </keep-alive>
</template>

<script>
import keep from '@/components/keep.vue'
import stat from '@/components/stat.vue'
import { ref } from 'vue'

export default {
  setup () {
    const currentPage = ref('keep')
    function changeView (c) {
      this.currentPage = c
    }
    return {
      currentPage,
      changeView
    }
  },
  components: {
    keep,
    stat
  }
}
</script>
```

currentPage属性控制着应该显示哪个组件。通过按钮点击动态改变currentPage的值，达到动态切换组件。

**注意，这里的currentPage应该是响应式对象才能切换。**

当组件切换后，原有组件的值不会被保存。此时可以使用keep-alive来缓存切换前组件的值。

keep-alive有两个属性include和exclude，可以根据组件的name来包含或排除应该缓存的组件。

**注意，这里组件的值是大小写敏感的。多个组件的值可以使用，分割。**



### 全局状态变量传参

**在store中定义方法，在其他组件中使用store.commit方法触发方法，并传递参数。**

方法的第一个参数为store中定义的state

* 传递单个参数

store:

```js
export default createStore({
  state: {
    id: '',
    url: ''
  },
  mutations: {
    getParams (state, url) {
      console.log(url)
      state.url = url
    }
  },
  actions: {
  },
  modules: {
  }
})
```

app.vue:

```js
function changeView (url) {
    this.currentPage = url
    const id = 3
    store.commit('getParams', url)
}
```

* 传递多个参数

store：多个参数使用payload接收

```js
export default createStore({
  state: {
    id: '',
    url: ''
  },
  mutations: {
    getParams (state, payload) {
      console.log(state)
      console.log(payload.id)
      console.log(payload.url)
      state.url = payload.url
      state.id = payload.id
    }
  },
  actions: {
  },
  modules: {
  }
})
```

App.vue：commit的时候使用对象传递多个参数，这里使用的是ES6的语法。

```js
function changeView (url) {
    this.currentPage = url
    const id = 3
    store.commit('getParams', {
        url,
        id
    })
}
```

commit的参数也能写成一个对象的形式，使用type属性指明要触发的函数：

```js
function changeView (url) {
    this.currentPage = url
    const id = 3
    store.commit({
        type: 'getParams',
        id,
        url
    })
}
```



### 通过全局参数切换路由

之前的切换路由的方法，如果一个页面包含很多子组件，只能一层层往上传递事件。

使用全局参数，可以实现切换路由。思路如下：

* 在子组件中修改全局参数的值
* 在父组件监听全局参数的值，一旦发生改变，通过router.push完成路由切换



1.子组件Home.vue修改全局参数url的值

```js
function onHomeClick () {
    // 直接修改全局变量url
    store.state.url = '/detail'
}
```

直接修改state的方式太粗暴了，可以通过传参的方式改变全局参数的值：

```js
function onHomeClick () {
    // 直接修改全局变量url
    store.commit('changeView', '/detail')
}
```

当然，在全局参数store的文件中：

```js
export default createStore({
    state: {
        url: ''
    },
    mutations: {
        changeView (state, url) {
            state.url = url
        }
    },
    actions: {
    },
    modules: {
    }
})
```



2.父组件中不再使用事件，直接监听全局参数的值

```js
watch(() => store.state.url, (newVal, oldVal) => {
    console.log(store.state.url)
    router.push(newVal)
})
```

也可以通过监听computed properties：

```js
const targetRoute = computed(() => store.state.url)
watch(targetRoute, (newVal, oldVal) => {
    console.log(store.state.url)
    router.push(newVal)
})
```



### 带参数的全局参数路由跳转

设置一个对象类型的全局参数，其格式与router.push的参数相同。当子组件设置全局参数时，直接监听全局参数的改变，跳转路由。

store:

```js
export default createStore({
  state: {
    routerParams: {}
  },
  mutations: {
    changeView (state, payload) {
      state.routerParams = payload.routerParams
    }
  },
  actions: {
  },
  modules: {
  }
})
```

子组件设置全局参数，参数的形式与router.push的参数格式一致

```js
function onHomeClick () {
    store.commit('changeView', {
        routerParams: {
            name: 'Detail',
            params: {
                id: 3
            }
        }
    })
}
```

在App.vue中监听全局参数的变化，跳转路由：

```js
watch(() => store.state.routerParams, (newVal, oldVal) => {
    console.log(newVal)
    router.push(newVal)
})
```

