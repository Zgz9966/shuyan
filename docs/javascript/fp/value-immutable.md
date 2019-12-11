这一节和上节一样，都是为了减少副作用而努力。

如果说编程角度的幂等让一个数值改变的操作变为一次，那么不可变数据结构就是将其变为 0 次。

## 原始数据类型的 immutable

JavaScript 中的原始类型一直是不可变的，无法修改。

但是 JS 有一种奇怪的「装箱」行为，在访问原始类型的属性时触发。

JS 在这种情况下自动将值包裹在一个「装箱对象」中。

```js
let x = 2;
x.length = 4; // 严格模式下报错

x; // 2
x.length; // undefined

let x = new Number(2);

// works fine
x.length = 4;
```

那么，字符串呢？

```js
"use strict";

let s = new String( "hello" );

s[1];               // e
s[1] = "E";         // error
s.length = 10;      // error

s[42] = "?";        // OK

s;                  // "hello"

// 即使是装箱对象，大部分属性也不可以变
let s = new String( "hello" );

s[1] = "E";         // error
s.length = 10;      // error

s[42] = "?";        // OK

s;                  // "hello"
```

## Value to Value

首先，必须在心里上有一个清晰地认知，immutable 数据不代表在整个程序中我们不修改这个值。

immutable 的核心理念是在我们需要修改某个值的时候，我们创建一个新的值而不是对已有的值「突变」。

举个栗子

```js
let addValue = arr => [...arr, 4];

addValue([1,2,3]);    // [1,2,3,4]
```

这意味着我们可以控制我们程序的状态，除了 addValue，没有任何东西能改变状态。

我们也可以复制对象而不是产生对象的「突变」

```js
let user = {
    // ..
};

function updateLastLogin(user) {
    let newUserRecord = Object.assign({}, user);
    newUserRecord.lastLogin = Date.now();
    return newUserRecord;
}

user = updateLastLogin( user );
```