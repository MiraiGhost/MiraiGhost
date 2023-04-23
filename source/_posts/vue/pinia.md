---
title: pinia安装和使用
date: 2023-04-23 15:23:01
tags:
---

## Pinia
vue存储库，允许跨组件/页面共享状态

## 安装
```bash
pnpm install pinia --save
```

## 使用
### 1.创建 store/index.js 文件
```js
//引入 pinia
import {defineStore} from 'pinia'
const store = defineStore('counterStore',{
    // state存放所有状态
    state(){
        userId:0
    },
    // action修改状态
    actions:{
        editUserId(val: number)
        {
            this.userId = val
        }
    }
})
```
### 2.全局引入
在main.js文件中引入并使用
```js
...
import {createPinia} from 'pinia'
...
const app = createApp(App)
app.use(createPinia())
...
```

### 3.使用


## Pinia持久化
刷新页面后全局状态的值会被清空
因此要做持久化处理

### 安装 pinia-plugin-persist
```bash
pnpm install pinia-plugin-persist
```

### 1.引入
在引入pinaia的基础上再次引用piniaPluginPersist
```js
...
import piniaPluginPersist from 'pinia-plugin-persist'
...
app.use(createPinia().use(piniaPluginPersist))
...
```

### 2.使用
```js
// 状态管理
persist:{
    // 开启
    enable: true,
    strategies: {
        //指定key，key会在浏览器本地存储中生成对应的name
        key:'site',
        // 自定义存储方式
        storage: localStorage,
        // 要保存的对象，默认所有
        paths:{'userId'}
    }
}
```