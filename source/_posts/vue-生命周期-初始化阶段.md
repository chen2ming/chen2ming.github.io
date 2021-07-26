---
title: vue生命周期-初始化阶段
tag: vue2.0
date: 2021-7-26
---
# new Vue()
初始化阶段所做的第一件事就是new Vue()创建一个Vue实例，那么new Vue()的内部都干了什么呢？ 我们知道，new 关键字在 JS中表示从一个类中实例化出一个对象来，由此可见， Vue 实际上是一个类。所以new Vue()实际上是执行了Vue类的构造函数

## 做了什么
```js
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}
```
可以看到，Vue类的定义非常简单，其构造函数核心就一行代码：``` this._init(options) ```

调用原型上的_init(options)方法并把用户所写的选项options传入。那这个_init方法是从哪来的呢？
```js
initMixin(Vue)
```
这一行代码执行了initMixin函数，那initMixin函数又是从哪儿来的呢？该函数定义位于源码的src/core/instance/init.js 中，如下：
```js
export function initMixin (Vue) {
  Vue.prototype._init = function (options) {
    const vm = this
    vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
    )
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')

    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
}
```
可以看到，在initMixin函数内部就只干了一件事，那就是给Vue类的原型上绑定_init方法，同时_init方法的定义也在该函数内部。现在我们知道了，new Vue()会执行Vue类的构造函数，构造函数内部会执行_init方法，所以new Vue()所干的事情其实就是_init方法所干的事情，那么我们着重来分析下_init方法都干了哪些事情。

首先，把Vue实例赋值给变量vm，并且把用户传递的options选项与当前构造函数的options属性及其父级构造函数的options属性进行合并（关于属性如何合并的问题下面会介绍），得到一个新的options选项赋值给$options属性，并将$options属性挂载到Vue实例上，如下：
```js
vm.$options = mergeOptions(
    resolveConstructorOptions(vm.constructor),
    options || {},
    vm
)
```
接着，通过调用一些初始化函数来为Vue实例初始化一些属性，事件，响应式数据等，如下：
```js
initLifecycle(vm)       // 初始化生命周期
initEvents(vm)        // 初始化事件
initRender(vm)         // 初始化渲染
callHook(vm, 'beforeCreate')  // 调用生命周期钩子函数
initInjections(vm)   //初始化injections
initState(vm)    // 初始化props,methods,data,computed,watch
initProvide(vm) // 初始化 provide
callHook(vm, 'created')  // 调用生命周期钩子函数
```
可以看到，除了调用初始化函数来进行相关数据的初始化之外，还在合适的时机调用了callHook函数来触发生命周期的钩子，关于callHook函数是如何触发生命周期的钩子会在下面介绍，我们先继续往下看：
```js
if (vm.$options.el) {
    vm.$mount(vm.$options.el)
}
```

## callHook函数如何触发钩子函数
```js
export function callHook (vm: Component, hook: string) {
  const handlers = vm.$options[hook] // 要执行的钩子函数
  if (handlers) {
    for (let i = 0, j = handlers.length; i < j; i++) {
      try {
        handlers[i].call(vm)  // 执行钩子函数中的每一个方法
      } catch (e) {
        handleError(e, vm, `${hook} hook`)
      }
    }
  }
}
```
首先从实例的$options中获取到需要触发的钩子名称所对应的钩子函数数组handlers，每个生命周期钩子名称都对应了一个钩子函数数组。然后遍历该数组，将数组中的每个钩子函数都执行一遍。

##  initLifecycle函数

```js
export function initLifecycle (vm: Component) {
  const options = vm.$options

  let parent = options.parent
  if (parent && !options.abstract) {
    while (parent.$options.abstract && parent.$parent) {
      parent = parent.$parent
    }
    parent.$children.push(vm)
  }

  vm.$parent = parent
  vm.$root = parent ? parent.$root : vm

  vm.$children = []
  vm.$refs = {}

  vm._watcher = null
  vm._inactive = null
  vm._directInactive = false
  vm._isMounted = false
  vm._isDestroyed = false
  vm._isBeingDestroyed = false
}
```
主要是给Vue实例上挂载了一些属性并设置了默认值，**同时挂载$parent 属性和$root属性**

首先是给实例上挂载$parent属性：
```js
let parent = options.parent
if (parent && !options.abstract) {
  while (parent.$options.abstract && parent.$parent) {
    parent = parent.$parent
  }
  parent.$children.push(vm)
}

vm.$parent = parent
```
如果当前组件不是抽象组件并且存在父级，那么就通过while循环来向上循环，如果当前组件的父级是抽象组件并且也存在父级，那就继续向上查找当前组件父级的父级，直到找到第一个不是抽象类型的父级时，将其赋值vm.$parent，同时把该实例自身添加进找到的父级的$children属性中。这样就确保了在子组件的$parent属性上能访问到父组件实例，在父组件的$children属性上也能访问子组件的实例


接着是给实例上挂载$root属性
```js
vm.$root = parent ? parent.$root : vm
```
实例的$root属性表示当前实例的根实例，挂载该属性时，首先会判断如果当前实例存在父级，那么当前实例的根实例$root属性就是其父级的根实例$root属性，如果不存在，那么根实例$root属性就是它自己。这很好理解，举个例子：假如有一个人，他如果有父亲，那么他父亲的祖先肯定也是他的祖先，同理，他的儿子的祖先也肯定是他的祖先，我们不需要真正的一层一层的向上递归查找到他祖先本人，只需要知道他父亲的祖先是谁然后告诉他即可。如果他没有父亲，那说明他自己就是祖先，那么他后面的儿子、孙子的$root属性就是他自己了

## 解析事件 initEvents
在Vue中，当我们在父组件中使用子组件时可以给子组件上注册一些事件，这些事件即包括使用v-on或@注册的自定义事件，也包括注册的浏览器原生事件（需要加 .native 修饰符），如下：
```html
<child @select="selectHandler" 	@click.native="clickHandler"></child>
```

模板编译解析中，当遇到开始标签的时候，除了会解析开始标签，还会调用processAttrs 方法解析标签中的属性，processAttrs 方法位于源码的 src/compiler/parser/index.js中， 如下：
```js
export const onRE = /^@|^v-on:/
export const dirRE = /^v-|^@|^:/

function processAttrs (el) {
  const list = el.attrsList
  let i, l, name, value, modifiers
  for (i = 0, l = list.length; i < l; i++) {
    name  = list[i].name
    value = list[i].value
    if (dirRE.test(name)) {
      // 解析修饰符
      modifiers = parseModifiers(name)
      if (modifiers) {
        name = name.replace(modifierRE, '')
      }
      if (onRE.test(name)) { // v-on
        name = name.replace(onRE, '')
        addHandler(el, name, value, modifiers, false, warn)
      }
    }
  }
}
```

在对标签属性进行解析时，判断如果属性是指令，首先通过 parseModifiers 解析出属性的修饰符，然后判断如果是事件的指令，则执行 addHandler(el, name, value, modifiers, false, warn) 方法， 该方法定义在 src/compiler/helpers.js 中，如下：
```js
export function addHandler (el,name,value,modifiers) {
  modifiers = modifiers || emptyObject

  // check capture modifier 判断是否有capture修饰符
  if (modifiers.capture) {
    delete modifiers.capture
    name = '!' + name // 给事件名前加'!'用以标记capture修饰符
  }
  // 判断是否有once修饰符
  if (modifiers.once) {
    delete modifiers.once
    name = '~' + name // 给事件名前加'~'用以标记once修饰符
  }
  // 判断是否有passive修饰符
  if (modifiers.passive) {
    delete modifiers.passive
    name = '&' + name // 给事件名前加'&'用以标记passive修饰符
  }

  let events
  if (modifiers.native) {
    delete modifiers.native
    events = el.nativeEvents || (el.nativeEvents = {})
  } else {
    events = el.events || (el.events = {})
  }

  const newHandler: any = {
    value: value.trim()
  }
  if (modifiers !== emptyObject) {
    newHandler.modifiers = modifiers
  }

  const handlers = events[name]
  if (Array.isArray(handlers)) {
    handlers.push(newHandler)
  } else if (handlers) {
    events[name] = [handlers, newHandler]
  } else {
    events[name] = newHandler
  }

  el.plain = false
}
```

在addHandler 函数里做了 3 件事情，首先根据 modifier 修饰符对事件名 name 做处理，接着根据 modifier.native 判断事件是一个浏览器原生事件还是自定义事件，分别对应 el.nativeEvents 和 el.events，最后按照 name 对事件做归类，并把回调函数的字符串保留到对应的事件中。
