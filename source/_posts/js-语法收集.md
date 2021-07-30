---
title: js语法收集
tag: js
date: 2021-7-17
---
# 判断执行
```js
if(el){
 query(el)
}
el && query(el)

if(el){
 el = query(el)
}
el = el && query(el);

if(!el) return
el = query(el)
```
