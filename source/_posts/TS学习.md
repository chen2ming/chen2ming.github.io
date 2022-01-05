---
title: ts学习
tag: ts
date: 2021-12-29
---

# 基础类型

1. 布尔值
  ```js
  let isDone:boolean = false
  ```
2. 数字
  ```js
  let decLiteral: number = 6;
  let hexLiteral: number = 0xf00d;
  let binaryLiteral: number = 0b1010;
  let octalLiteral: number = 0o744;
  ```
3. 字符串
  ```js
  let name: string = "bob";
  name = "smith";
  ```
4. 数组
  ```js
  let list: number[] = [1, 2, 3];
  // 泛型
  let list: Array<number> = [1, 2, 3];
  ```
5. 元组 Tuple
  ```js
  let x: [string, number];
  x = ['hello', 10]; // OK
  x = [10, 'hello']; // Error
  ```
6. 枚举
  ```js
  enum Color {Red, Green, Blue}
  let c: Color = Color.Green;
  ```
7. Any  （随便什么类型都可以）
  ```js
  let notSure: any = 4;
  notSure = 
  ```
8. Void  没有任何类型
  只能为它赋予undefined和null
  ```js
  function warnUser(): void {
    console.log("This is my warning message");
  }

  let unusable: void = undefined;
  ```
  TypeScript里，undefined和null两者各自有自己的类型分别叫做undefined和null。 和 void相似，它们的本身的类型用处不是很大：
  ```js
  let u: undefined = undefined;
  let n: null = null;
  ```
  默认情况下null和undefined是所有类型的子类型。 就是说你可以把 null和undefined赋值给number类型的变量。
