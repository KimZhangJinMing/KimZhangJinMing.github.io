---
title: watch.md
date: 2020-11-29 22:31:35
categories: Vue
tags: watch
---

### watch的写法 -> Vue2

1.通过watch对象，指定要监听的属性为方法名称，方法有两个参数newVal，oldValue

```vue
	watch:{
			firstName(newVal,oldVal){
				this.fullName = newVal + this.lastName
			},
			lastName(newVal,oldVal){
				this.fullName = this.firstName + newVal
			}
		}
```

2.通过钩子函数，this.$watch

```vue
	created(){
			this.$watch('firstName',(newVal,oldVal) => {
				this.fullName = newVal + this.lastName
			})
			this.$watch('lastName',(newVal,oldVal) => {
				this.fullName = this.firstName + newVal
			})
		}
```

### watch写法 -> Vue3

> 监听对象的一个属性

```vue
const { createApp, ref ,reactive, watch} = Vue

    let firstName = ref('')
    let lastName = ref('')
    let fullName = ref('')

    const rootComponent = {
    setup(){
        watch(firstName,(newVal,oldVal) => {
            console.log(newVal)
                fullName.value = newVal + lastName.value
        });
        watch(lastName,(newVal) => {
            console.log(newVal)
                fullName.value = firstName.value + newVal
        })
            return {
            firstName,
            lastName,
            fullName,
        }
    }
}
```

> 监听对象

```vue
const { createApp, ref ,reactive, watch} = Vue

    const name = reactive({
        firstName: "",
        lastName:""
    })

    let fullName = ref('')

    const rootComponent = {
    setup(){
        // 只要对象的属性变化,整个对象也会变化
        watch(name,(newValue) => {
            // 监听的是一个对象,newVal是一个Proxy对象
            console.log(newValue)
                fullName.value = name.firstName + name.lastName
        })
            return {
            name,
            fullName,
        }
    }
}
```

* reactive包装的响应式对象是嵌套的。对象中的属性也会被包装成响应式对象，因此对象的属性变化也会进行双向绑定。
* 

> 监听对象的某个属性

```vue
	const { createApp, ref ,reactive, watch} = Vue

	const name = reactive({
		firstName: "",
		lastName:""
	})

	let fullName = ref('')

	const rootComponent = {
		setup(){
			// 监听对象中的某个属性
			watch(() => name.firstName,(newValue) => {
				// 监听的是一个对象,newVal是一个Proxy对象
				console.log(newValue)
				fullName.value = newValue + name.lastName
			})

			watch(() => name.lastName,(newVal) => {
				fullName.value = name.firstName + newVal
			})
			return {
				name,
				fullName,
			}
		}
}
```



### computed的使用

```vue
const { createApp, ref ,reactive, watch, computed} = Vue

	const firstName = ref('')
	const lastName = ref('')


	const rootComponent = {
		setup(){
			const fullName = computed(() => firstName.value + lastName.value)
			console.log(fullName)
			return {
				firstName,
				lastName,
				fullName
			}
		}
	}
```

* computed也是需要return的
* computed的返回值是只读的，它是一个对象

![image-computed返回的对象](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201202214941679.png)

### computed的setter方法

```vue
const firstName = ref('1')
	const lastName = ref('2')


	const rootComponent = {
		setup(){
			const fullName = computed({
				get : () => firstName.value + lastName.value,
				set : (value) => lastName.value = value
			})
			// 加了setter,fullName才可以修改。而firstName和lstName本身就可以进行修改,不需要setter
			// 修改fullName,最终的值取决于setter的逻辑
			// firstName.value = 7  72
			// fullName.value = '8';   18

			return {
				firstName,
				lastName,
				fullName
			}
		}
```

