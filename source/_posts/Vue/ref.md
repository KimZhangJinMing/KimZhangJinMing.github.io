---
title: ref.md
date: 2020-12-01 22:31:35
categories: Vue
tags: ref
---

### ref的使用

废话少说，上栗子

```vue
<div id="container">
    <input type="text" v-model="age">
    {{age}}
    <button @click="change">点击</button>
</div>
```

```vue
<script>
    const { createApp, ref, reactive} = Vue

    let age =ref(18)

    const app = {
        setup(){
            function change(){
                // 因为age是全局变量,这里才能访问到
                console.log(age)
                // 因为age是ref包装的响应式对象,获取值需要.value
                console.log(age.value)
            }
			
            return {
                age,
                change
            }
        }
    }

    createApp(app).mount("#container");
</script>
```

![image-执行结果1](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201201224014403.png)

知识点：

* ref可以将基本类型的值包装成响应式对象，获取对象的值需要通过.value的形式
* 在template中渲染ref包装的响应式对象不需要通过.value，Vue会自动解析。
* 通过_v_isRef字段可以判断是否ref包装的响应式对象。也可以通过Vue对象中的isRef方法来判断
* 只有响应式对象才会进行双向绑定
* v-model只能用于input、textarea、select
* Vue2.5使用defineProperty实现响应式对象，Vue3.0使用Proxy实现响应式对象
* 