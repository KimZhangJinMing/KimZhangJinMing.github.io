### 1.搭建环境

#### 1.1 下载npm和node

#### 1.2 修改npm源

#### 1.3 安装ElementUI

```javascript
npm install --save element-plus
```

在main.js引入elementUI,

```java
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import store from "./store"
import ElementPlus from 'element-plus';
import 'element-plus/lib/theme-chalk/index.css';


createApp(App).use(ElementPlus).use(router).use(store).mount('#app')
```



### 2.axios

#### 2.1 封装请求，拦截响应对错误统一处理

通过拦截器拦截response，对服务器返回的错误信息统一处理。

`api.js`

```javascript
import {ElMessage} from "element-plus";
import axios from "axios";
import {baseUrl} from "../../config";

// 拦截axios响应，对错误统一处理
axios.interceptors.response.use(result => {
	// data为后端返回的数据
	const data = result.data;
	// 业务的错误，直接显示后端返回的错误信息
	if (result.status && result.status === 200 && data.status === 500) {
		ElMessage.error(data.message)
		return;
	}
	return data;
}, error => {
	// 请求状态码不是200，有可能后端返回了错误信息
	const data = error.response;
	if (data.message) {
		ElMessage.error(data.error)
	} else if (data.status === 404) {
		ElMessage.error("您访问的页面可能在火星...");
	} else if (data.status === 500) {
		ElMessage.error("服务器内部错误...");
	} else {
		ElMessage.error("未知错误...");
	}
})

```

对键值对的POST请求进行封装：

`api.js`

```javascript
// 封装键值对的POST请求
const postKeyValueRequest = (url, data) => {
	return axios({
		url: `${baseUrl}${url}`,
		method: "post",
		data,
		headers:{
			'Content-Type':"application/x-www-form-urlencoded"
		},
		transformRequest: data => {
			let transformData = ""
			for (const key in data) {
				transformData += encodeURIComponent(key) + "=" + encodeURIComponent(data[key]) + "&"
			}
			return transformData;
		}
	})
}

const getRequest=(url,data)=> {
	return axios({
		url:`${baseUrl}${url}`,
		method:"get",
		data
	})
}

const postRequest=(url,data)=> {
	return axios({
		url:`${baseUrl}${url}`,
		method:"post",
		data
	})
}

const putRequest=(url,data)=> {
	return axios({
		url:`${baseUrl}${url}`,
		method:"put",
		data
	})
}

const deleteRequest=(url,data)=> {
	return axios({
		url:`${baseUrl}${url}`,
		method:"delete",
		data
	})
}

export {
	postKeyValueRequest,
	getRequest,
	postRequest,
	putRequest,
	deleteRequest
}

```

封装之后，要想使用还得在使用的页面导入：

```javascript
import {postKeyValueRequest} from "../util/api";
```

这样太麻烦了，可以把它们定义成全局的插件使用，使用的时候无需导入，通过this去调用。

`mian.js`定义成全局属性：

```javascript
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import store from "./store"
import ElementPlus from 'element-plus';
import 'element-plus/lib/theme-chalk/index.css';
// 这里开始
import {postKeyValueRequest} from "./util/api";
import {getRequest} from "./util/api";
import {postRequest} from "./util/api";
import {putRequest} from "./util/api";
import {deleteRequest} from "./util/api";


const app = createApp(App);
// 自定义全局组件
app.config.globalProperties.postKeyValueRequest = postKeyValueRequest
app.config.globalProperties.getRequest = getRequest
app.config.globalProperties.postRequest = postRequest
app.config.globalProperties.putRequest = putRequest
app.config.globalProperties.deleteRequest = deleteRequest
app.use(ElementPlus)
	.use(router)
	.use(store)
	.mount('#app')
```

使用的时候:

```javascript
// 无需导入，通过this调用
this.postKeyValueRequest("/doLogin",this.user).then(data => {
    if(data) {
        ElMessage.success(data.message)
        this.$router.replace("/home")
    }
})
```



#### 2.2 跨域

前端是在8080端口启动，后端是在8081端口启动，在开发的时候，当前端去访问后端的时候，由于端口不一致，会导致跨域问题。

需要配置代理，在根目录下创建`vue.config.js`文件(文件名称固定),

代理和baseUrl不能同时设置，否则代理不生效。

```javascript
const proxy = {
	// 设置监听的别名，下面替换掉
	'/': {
		// 服务器接口
		target: "http://localhost:8081",
		// 是否代理websocket
		ws: false,
		// 是否使用https
		secure: false,
		// 是否跨域
		changeOrigin: true,
		// 替换别名
		pathRewrite: {
			'^/': ''
		}
	}
}

// 这里是exports
module.exports= {
    // 开发服务器,前端使用的端口
	devServer: {
		host:'localhost',
		port:'8080',
		proxy
	}
}

```



#### 2.3 页面跳转

使用路由进行页面跳转有两种方式：

* replace:不能返回跳转前的页面
* push:可以通过浏览器返回跳转前的页面

```javascript
this.$router.replace("/home")
```



#### 2.4 session操作

```javascript
// 保存到session,数据只能是String类型
window.sessionStorage.setItem("user",JSON.stringify(data))
// 从session中获取,解析json字符串
JSON.parse(window.sessionStorage.getItem("user"))
```



#### 2.5 获取路由参数

children中的路由只能在它父级路由的<router-view>才能使用

```javascript
const routes = [
  {
    path: '/',
    name: 'Login',
    component: Login,
    hidden:true
  },
  {
    path: '/home',
    name: '菜单',
    component: Home,
    hidden: false,
    children:[
      {
        path: '/test1',
        name: '选项一',
        component: Test1
      },
      {
        path: '/test2',
        name: '选项二',
        component: Test2
      }
    ]
  },
]

```

从路由中获取参数，动态渲染菜单：

注意this.$router在setup函数中是不能使用this的

```javascript
<template>
	<el-container>
		<el-aside class="aside">
			<el-row>
				<el-col>
					<el-menu default-active="1" router v-for="(item,index) in routers" :key="index">
						<el-submenu index="1" v-if="!item.hidden">
							<template #title>
								<em class="el-icon-location"></em>
								<span>{{item.name}}</span>
							</template>
							<el-menu-item :index="children.path" v-for="(children,index) in item.children" :key="index">{{children.name}}</el-menu-item>
						</el-submenu>
					</el-menu>
				</el-col>
			</el-row>
		</el-aside>
		<el-main class="main">
			<router-view/>
		</el-main>
	</el-container>


</template>

<script>
	export default {
		name: "Menu",
		data(){
			return {
				routers:this.$router.options.routes
			}
		}
	}
</script>

<style scoped>

</style>

```



### 菜单

#### 处理后端返回数据

从服务器端返回的数据中，component是字符串类型的，而router中的component是对象类型，需要进行转换。

我们封装一个`menu.js`来进行转换。

```javascript
import {getRequest} from "./api";

let initMenu = (router, store) => {
	// 已经有数据了，不再渲染
	if (store.state.routes.length > 0) {
		return;
	}
	// 向服务器请求菜单数据
	getRequest("/system/menus").then(data => {
		if (data) {
			let routers = data.data;
			routers = trans(routers);
			console.log(routers)
			// todo addRoutes is not defined
			// router.addRoutes(routers);
			// 保存到store
			store.commit('initRouters', routers);
		}
	})

}

// 将服务器返回的菜单数据转换成router,并返回格式化后的router
const trans = (routers) => {
	let formatRouters = []
	routers.forEach( router => {
		let {path, name, component, iconCls, children, meta} = router;
		// 如果有children,递归处理
		if (children) {
			children = trans(children);
		}

		let formatRouter = {
			path,
			name,
			iconCls,
			meta,
			children,
			component: () => import(`@/views${component}`)
		}
		formatRouters.push(formatRouter);
	})
	return formatRouters;
}


export {
	initMenu
}

```

同时，需要把数据保存在store中，这样才能全局使用。

```javascript
import { createStore } from 'vuex'

export default createStore({
  state: {
    routes:[]
  },
  mutations: {
    initRouters(state,data) {
      state.routes = data;
    }
  },
  actions: {
  },
  modules: {
  }
})

```



#### 加载菜单的时机

我们可以在加载Home页面的时候初始化菜单数据，但是如果采用这种方式，跳转到别的页面，又需要加载多一次菜单，这就不得不在每个页面都写上initMenu的方法。

可以使用`导航守卫`来监控路由，由跳转的路由来决定是否加载菜单数据。

`mian.js`中注册前置导航守卫：

* to:对象类型,代表即将跳转的路由
* from:对象类型,代表从哪个路由过来的
* next:是一个函数，只有执行next()才会往下继续执行，关键所在

```javascript
router.beforeEach( (to, from ,next) => {
	// 跳转到登录页
	if(to.path === "/" ){
		next();
	}else {
		initMenu(router,store);
		next();
	}
})
```

当然，在注销登录的时候也需要清空store.state.routes的值，否则退出登录后，看到的还是上一个人的菜单数据。

```javascript
logout() {
    this.$confirm('确认注销登录吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
    }).then(() => {
        this.ElMessage.success("注销成功");
        // 清除路由数据
        this.$store.state.routes = []
        // 返回到登录页面
        this.$router.replace("/");
    }).catch(() => {
        this.ElMessage.success("撤销操作")
    });
},
```

#### 使用图标库

使用font-awesome的图标库需要先安装，然后在main.js中导入对应的css文件

```javascript
npm install font-awesome

import 'font-awesome/css/font-awesome.min.css';
```



