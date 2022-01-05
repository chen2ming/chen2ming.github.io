---
title: vue3学习
tag: vue3
date: 2022-1-5
---

# Vue3与Vue2在应用中的区别

* 响应式数据在Vue3中变得更加灵活和友善。Vue2中 data 里没有定义的属性在后续无法正常的进行响应操作，必须通过 Vue.set 这个 API 向响应式对象中添加一个 property，并确保这个新 property 同样是响应式的，且触发视图更新。它必须用于向响应式对象上添加新 property，因为 Vue 无法探测普通的新增 property (比如 this.myObject.newProperty = 'hi'); 然而在 Vue3 中我们可以通过引入 ref 来操作响应值。ref 是一个实例方法，接受一个内部值并返回一个响应式且可变的 ref 对象。ref 对象具有指向内部值的单个 property.value。
```js
const count = ref(0)
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```
Vue3 采用了 ES6的一项新特性：Proxy 来实现Vue3中数据响应式的设计。通过下面的伪代码我们可以对比一下：
```js
Object.definProperty(data,'count',{
  get(){},
  set(){}
})
new Proxy(data,'count',{
  get(key){},
  set(key,value){}
})
```
> Object.defineProperty 要修改 data 中的属性必须要明确的知道 key 值（count）, Proxy 在使用中是读取或者设置data中任意的 key，所以不管是修改已有的属性还是新增属性，都可以实现响应式的要求。

* 关于生命周期钩子函数
| vue2 | vue3 |
| ---- | ---- |
| beforeCreate() | 	use setup() |
| created() | use setup() |
| beforeMount() | onBeforeMount |
| mounted() | onMounted |
| beforeUpdate() | onBeforeUpdate |
| updated() | onUpdated |
| beforeDestory() | onBeforeUnmount |
| destoryed() | onUnmounted |
| activated() | onActivated |
| deactivated() | onDeactivated |
| errorCaptured() | onErrorCaptured |
|  | onRenderTracked(新增) — DebuggerEvent 调试用 |
|  | 	onRenderTriggered(新增) — DebuggerEvent 调试用 |

> Vue3中的钩子函数都在 setup() 中调用。

* computed，watch 可直接调用
```js
const count = ref(0)
const double = computed(() => {
    return count.value * 2
})
```
watch 接收两个参数，第一个参数是监听的属性，多个属性可传入数组， 第二个参数是一个回调函数，回调函数有两个参数（newVal, oldVal）；当 watch 的第一个参数是一个数组时，newVal 与 oldVal 对应的也是数组形式，一一对应。
```js
// 监听count
watch('count', (newVal, oldVal) => {
    console.log('newVal:', newVal)
    console.log('oldVal:', oldVal)
})
// 监听多个属性值
watch(['count', 'name'], (newVal, oldVal) => {
    console.log('newVal:', newVal) // 数组
    console.log('oldVal:', oldVal) // 数组
})
```
如果是需要监听定义在 reacitive 对象中的单一属性，需要通过函数返回值来进行监听。
```js
watch(() => data.count, (newVal, oldVal) => {
    console.log('newVal:', newVal)
    console.log('oldVal:', oldVal)
})
```
