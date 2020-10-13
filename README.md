
## 一. 实现功能

在App.vue中实现`commit`,`dispatch`,`state`,`getters`相应功能

```html
<template>
  <div id="app">
    <h1>Vuex - Demo</h1>
    count：{{ $store.state.count }} <br>
    msg: {{ $store.state.msg }}

    <h2>Getter</h2>
    reverseMsg: {{ $store.getters.reverseMsg }}

    <h2>Mutation</h2>
    <button @click="$store.commit('increate', 2)">Mutation</button>

    <h2>Action</h2>
    <button @click="$store.dispatch('increateAsync', 5)">Action</button>
  </div>
</template>

```

## 二. 基本结构

### 1.引入

```js
import Vuex from 'vuex'
```

在模拟完成之后要换成我们模拟文件的路径

### 2.注册到Vue中

```js
Vue.use(Vuex)
```

`Vue.use`会自动调用Vuex这个对象中的`install`方法,这个方法是需要我们去模拟实现的

### 3.导出实例化的Store

```js
export default new Vuex.Store({
  state: {},
  getters: {},
  mutations: {},
  actions: {}
  }
})
```

`new Vuex.Store`说明Vuex中还与一个Store,它是一个类,类中接受`state`,`getters`,`mutations`,`actions`参数,也是我们需要去实现的.

### 4.模拟Vuex基本结构

新建myvuex/index.js文件,穿件Store类,和install方法并导出.

```js
let _Vue = null
class Store {}

function install (Vue) {
  _Vue = Vue
}

export default {
  Store,
  install
}
```

## 三. 实现install

- 在install方法中是获取不到vue实例的,所以可以通过混入`_Vue.mixin`的方式获取
- 在`beforeCreate`中判断`$options`中是否有store,如果有说明是Vue的根实例,此时给_Vue原型注入`$options.store`,这样在所有组件中就可以通过$store去访问了.

```js
function install (Vue) {
  _Vue = Vue
  _Vue.mixin({
    beforeCreate () {
      //this就是Vue实例
      if (this.$options.store) {
        _Vue.prototype.$store = this.$options.store
      }
    }
  })
}
```

## 四. 实现Store类

- 数显需要一个钩子函数接受传入的对象,对象中有四个属性分别是`state`,`getters`,`mutations`,`actions`
  - `state`是响应式的
  - `getters`是一个对象,对象中有很多方法,一般情况下就是对state作简单的处理并返回.所以每个方法第一个参数接受的是state.那么该如何实现自动添加参数呢?这里,`可以把这些方法通过Object.defineProperty转换成this.getters对象中的get访问器,在访问的时候把调用getters中对应的方法并把stata当作参数传入`
  - 在`commit`,`dispatch`中需要调用`mutations`和`actions`中的方法,所以需要在类中定义两个内部变量以供方法中使用.分别在`mutations`和`actions`前加上`_`

```js
class Store {
  constructor (options) {
    const {
      state = {},
      getters = {},
      mutations = {},
      actions = {}
    } = options
    // 设置响应式
    this.state = _Vue.observable(state)
    this.getters = Object.create(null)
    // 把state传入到getters中,通过
    Object.keys(getters).forEach(key => {
      // key就是方法的名字
      Object.defineProperty(this.getters, key, {
        get: () => getters[key](state)
      })
    })
    this._mutations = mutations
    this._actions = actions
  }
  
  commit (type, payload) {
    this._mutations[type](this.state, payload)
  }

  dispatch (type, payload) {
    this._actions[type](this, payload)
  }
}
```

最后改变路径

```js
import Vuex from '../myvuex'
```