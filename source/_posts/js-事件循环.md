---
title: JS 事件循环、微任务和宏任务
date: 2023-2-28
tag: js
type: js
---

JS 是单线程执行的，所有 JS 代码都要放在主线程中运行。 如果把异步 IO 等耗时较长的任务也放在主线程中处理，会阻塞后续同步代码的执行，造成卡顿等现象。因此，浏览器等运行环境额外设置了异步处理线程，专门用于处理异步事件。
事件循环描述了 JS 的运行机制，也就是同步和异步任务的执行过程。

## 循环过程

    1. 拿到一段代码并执行
    2. 将代码中的同步任务交给主线程，形成执行栈
    3. 将代码中的异步（宏）任务交给异步处理线程
    4. 将异步处理的事件回调推入任务队列
    5. 当执行栈中的同步任务执行完成后，调用任务队列中的异步回调
    6. 重复步骤1

    整个 script 脚本将开启一次事件循环，而每个宏任务都将开启一次新的事件循环。

## JS 为什么是单线程执行的？

     * JS 可以操作 DOM 节点。如果 JS 是多线程的话，多个线程可以同时操作同一个 DOM 节点，比如一个在修改，另一个却要删除，这样太过混乱，导致浏览器很难处理。
     * 虽然上面说到异步处理线程，但它和 JS 的执行无关。比如一个 ajax 请求，在发送请求时，浏览器将请求交给异步线程处理；请求完成后，异步线程将事件回调推入任务队列，等待 JS 主线程调用；请求的实现是由浏览器 IO 线程和服务器完成的。

## 宏任务和微任务的区别

  ### 宏任务包括

    ajax 等 IO 交互消息
    onclick 等 UI 交互消息
    setImmediate、setInterval、setTimeout、requestAnimationFrame 等执行环境消息

  ## 微任务包括

    promise.then
    MutationObserver（监听 DOM 节点的变化）
    process.nextTick （Node.js）
    Object.obseve（监听对象的变化，已废弃）

    宏任务，依赖浏览器等宿主环境； 微任务，在 JS 引擎中执行，不会造成阻塞，也不参与事件循环。

## 微任务的执行时机

    JS 在执行一段代码的时候，除了会把同步任务放入执行栈，还会把微任务放到执行栈后面，形成一个微任务队列（ JS 中可访问 queueMicroTask）。 在执行栈中的同步任务执行完成后，JS 会先调用微任务队列中的任务，然后再去调用宏任务队列。

    因此，在同一次循环中，微任务比宏任务优先执行；在整个执行过程中，微任务复用一个队列，而宏任务共用一个队列。

## 微任务和宏任务的执行顺序

  在同一次循环中，微任务比宏任务优先执行，任务按照推入队列的顺序执行（FIFO）。
  
  * 微任务不参与事件循环，微任务会被推到当前循环对应的微任务队列中，即使是微任务中的微任务。
  * 宏任务将开启新的事件循环。如果宏任务中包含微任务，这个微任务会被带到下一次循环中执行。


```js

let results = []
function microTask (name, callback) {
 new Promise((resolve) => {
  resolve()
 }).then(() => {
  results.push(name)
  callback && callback()
 })
}

function macroTask (name, callback) {
 setTimeout(() => {
  results.push(name)
  callback && callback()
 }, 0)
}

function tick3 () {
 microTask('w1', () => {
  macroTask('h1', () => {
   microTask('w2')
  })
 })
 microTask('w3', () => {
  microTask('w4')
 })
}

tick3()

setTimeout(() => {
 console.log(results.join(' ')) // w1 w3 w4 h1 w2
}, 1000)
```
