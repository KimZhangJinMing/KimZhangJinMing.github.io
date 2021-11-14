在做这个菜单功能的时候，花了好几天。现在终于做出来了。但是还是有好多的疑问。

`松哥有时间教教我这个菜鸟..`

这里先给出完整的`menu.js`：

```javascript
import {getRequest} from "./api";

export const initMenu = (router, store) => {
	if (store.state.routes.length > 0) {
		return;
	}
	getRequest("/system/menus").then(data => {
		if (data) {
			let fmtRoutes = formatRoutes(data.data);
			// 不加这行代码,addRoute无效
			router.options.routes = fmtRoutes;
			fmtRoutes.forEach((item) => {
				router.addRoute(item)
			})
			store.commit('initRoutes', fmtRoutes);
		}
	})
}
export const formatRoutes = (routes) => {
	let fmRoutes = [];
	routes.forEach(router => {
		let {
			path,
			component,
			name,
			meta,
			iconCls,
			children
		} = router;
		if (children) {
			children = formatRoutes(children);
		}
		let fmRouter = {
			path:path,
			name,
			iconCls,
			meta,
			children:children,
			component: () => {
				if (component.startsWith("Home")) {
					return import('../views/Home');
				} else if (component.startsWith("Emp")) {
					return import(`../views/emp/${component}`);
				} else if (component.startsWith("Per")) {
					return import(`../views/per/${component}`);
				} else if (component.startsWith("Sal")) {
					return import(`../views/sal/${component}`);
				} else if (component.startsWith("Sta")) {
					return import(`../views/sta/${component}`);
				} else if (component.startsWith("Sys")) {
					return import(`../views/sys/${component}`);
				}
			}
		}
		fmRoutes.push(fmRouter);
	})
	return fmRoutes;
}

```

### 1.动态加载路由

如果使用以下代码动态加载路由:

```javascript
component(resolv) {
    if(..){
        require(['../views/emp'+component+'.vue'],resolve)  // 使用这种方式..
    }
}
```

浏览器会报错如下:

![image-20210120212655214](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210120212655214.png)



### 2.addRoutes()

我使用的是vue-router4.0,好像是没用addRoutes()这个方法的。我在控制台打印出router对象，只看到了addRoute方法？ 

官方文档中好像也没看到由addRoutes()的方法？



### 3. addRoute()无效

既然没有addRoutes(),我就尝试着使用循环addRoute的方式添加路由，发现也无效。

后来，加上下面一句代码就可以了，原因不详...

```javascript
router.options.routes = fmtRoutes;
fmtRoutes.forEach((item) => {
    router.addRoute(item)
})
```

### 4. ‘员工资料’菜单不能正确跳转(已解决)

数据库中存在两个`员工资料`的菜单，一个是一级菜单，一个是二级菜单。

当我使用addRoute添加路由的时候,一级菜单不能正确跳转。

查阅官方文档知道，addRoute在name相同的时候，会先移除第一个。由于这里是先加载一级菜单，后添加二级菜单，当添加二级菜单的`员工资料`时,一级菜单的`员工资料`就被移除了。导致一级菜单不能正确跳转。



### 5. ES6语法问题

在ES6语法的时候，我记得当key和value的名称一样的时候，可以不写value。比如name:name简写为name.

但是在这里，我省略name,iconCls,meta属性的时候是可以的，但是path，children简写的时候就出问题了。

如果简写了path,children，有时候会出问题，有时候又可以？ 

```javascript
let fmRouter = {
    path:path,
    name,
    iconCls,
    meta,
    children:children,
```



### 6.登录进去首页的时候，有时候不加载菜单

这种情况偶尔才会出现，重新刷新就不会了，也是想不懂。。

![image-20210120214223398](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210120214223398.png)