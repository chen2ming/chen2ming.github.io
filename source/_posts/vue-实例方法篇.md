---
title: vue实例方法
tag: vue2.0
data: 2021-8-2
---
+ 数据相关的方法
  + vm.$set
  + vm.$delete
  + vm.$watch
+ 事件相关的方法
  + vm.$on
  + vm.$emit
  + vm.$off
  + vm.$once
+ 生命周期相关的方法
  + vm.$mount
  + vm.$forceUpdate
  + vm.$nextTick
  + vm.$destory

# 数据相关的方法

与数据相关的实例方法有3个，分别是vm.$set、vm.$delete和vm.$watch。它们是在stateMixin函数中挂载到Vue原型上的，代码如下：
```js
import {set,del} from '../observer/index'
// 当执行stateMixin函数后，会向Vue原型上挂载上述3个实例方法
export function stateMixin (Vue) {
    Vue.prototype.$set = set
    Vue.prototype.$delete = del
    Vue.prototype.$watch = function (expOrFn,cb,options) {}
}
```
##  vm.$watch
## 用法
```js
vm.$watch( expOrFn, callback, [options] )
```
+ 参数
  + {string | Function} expOrFn
  + {Function | Object} callback
  + {Object} [options]
    + {boolean} deep
    + {boolean} immediate
+ 返回值
  {Function} unwatch
+ 用法
  观察 Vue 实例变化的一个表达式或计算属性函数。回调函数得到的参数为新值和旧值。表达式只接受监督的键路径。对于更复杂的表达式，用一个函数取代。
  注意：**在变异 (不是替换) 对象或数组时，旧值将与新值相同，因为它们的引用指向同一个对象/数组。Vue 不会保留变异之前值的副本。**
+ 示例
```js
// 键路径
vm.$watch('a.b.c', function (newVal, oldVal) {
  // 做点什么
})

// 函数
vm.$watch(
  function () {
    // 表达式 `this.a + this.b` 每次得出一个不同的结果时
    // 处理函数都会被调用。
    // 这就像监听一个未被定义的计算属性
    return this.a + this.b
  },
  function (newVal, oldVal) {
    // 做点什么
  }
)
```
vm.$watch 返回一个取消观察函数，用来停止触发回调：
```js
var unwatch = vm.$watch('a', cb)
// 之后取消观察
unwatch()
```
+ 选项：deep
  为了发现对象内部值的变化，可以在选项参数中指定 deep: true 。注意监听数组的变动不需要这么做。
```js
vm.$watch('someObject', callback, {
  deep: true
})
vm.someObject.nestedValue = 123
// callback is fired
```
+ 选项：immediate
  在选项参数中指定 immediate: true 将立即以表达式的当前值触发回调：
```js
vm.$watch('a', callback, {
  immediate: true
})
// 立即以 `a` 的当前值触发回调
```
注意在带有 immediate 选项时，你不能在第一次回调时取消侦听给定的 property。
```js
// 这会导致报错
var unwatch = vm.$watch(
  'value',
  function () {
    doSomething()
    unwatch()
  },
  { immediate: true }
)
```
如果你仍然希望在回调内部调用一个取消侦听的函数，你应该先检查其函数的可用性：
```js
var unwatch = vm.$watch(
  'value',
  function () {
    doSomething()
    if (unwatch) {
      unwatch()
    }
  },
  { immediate: true }
)
```
##  内部原理
$watch的定义位于源码的src/core/instance/state.js中，如下：
```js
Vue.prototype.$watch = function (expOrFn,cb,options) {
    const vm: Component = this
    if (isPlainObject(cb)) {
      return createWatcher(vm, expOrFn, cb, options)
    }
    options = options || {}
    options.user = true
    const watcher = new Watcher(vm, expOrFn, cb, options)
    if (options.immediate) {
      cb.call(vm, watcher.value)
    }
    return function unwatchFn () {
      watcher.teardown()
    }
  }
```
在函数内部，首先判断传入的回调函数是否为一个对象，就像下面这种形式：
```js
vm.$watch(
    'a.b.c',
    {
        handler: function (val, oldVal) { /* ... */ },
        deep: true
    }
)
```
如果传入的回调函数是个对象，那就表明用户是把第二个参数回调函数cb和第三个参数选项options合起来传入的，此时调用createWatcher函数，该函数定义如下：
```js
function createWatcher (vm,expOrFn,handler,options) {
    if (isPlainObject(handler)) {
        options = handler
        handler = handler.handler
    }
    if (typeof handler === 'string') {
        handler = vm[handler]
    }
    return vm.$watch(expOrFn, handler, options)
}
```
可以看到，该函数内部其实就是从用户合起来传入的对象中把回调函数cb和参数options剥离出来，然后再以常规的方式调用$watch方法并将剥离出来的参数穿进去
接着获取到用户传入的options，如果用户没有传入则将其赋值为一个默认空对象，如下
```js
options = options || {}
```
$watch方法内部会创建一个watcher实例，由于该实例是用户手动调用$watch方法创建而来的，所以给options添加user属性并赋值为true，用于区分用户创建的watcher实例和Vue内部创建的watcher实例，如下：
```js
options.user = true
```
接着，传入参数创建一个watcher实例，如下：
```js
const watcher = new Watcher(vm, expOrFn, cb, options)
```
接着判断如果用户在选项参数options中指定的immediate为true，则立即用被观察数据当前的值触发回调，如下：
```js
if (options.immediate) {
    cb.call(vm, watcher.value)
}
```
最后返回一个取消观察函数unwatchFn，用来停止触发回调。如下：
```js
return function unwatchFn () {
    watcher.teardown()
}
```
这个取消观察函数unwatchFn内部其实是调用了watcher实例的teardown方法，那么我们来看一下这个teardown方法是如何实现的。其代码如下：
```js
export default class Watcher {
    constructor (/* ... */) {
        // ...
        this.deps = []
    }
    teardown () {
        let i = this.deps.length
        while (i--) {
            this.deps[i].removeSub(this)
        }
    }
}
```
