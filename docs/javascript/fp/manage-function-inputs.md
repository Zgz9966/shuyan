## 一元函数

给函数单一的参数

这样做可以避免参数穿透，举个栗子

```js
['1', '2', '3'].map(parseInt)
// [1, NaN, NaN]
```

我们可以写一个帮助函数来过滤多余的参数

```js
let unary = fn => arg => fn(arg)

['1', '2', '3'].map(unary(parseInt))
```

甚至你还可以用的更花哨

来看下面这个函数

```js
let identity = v => v;
```

这个函数看起来就像人们口中的 shit

但是只有脑子 shit 的开发者

```js
let words = "   Now is the time for all...  ".split( /\s|\b/ )

words
// ["","Now","is","the","time","for","all","...",""]

words.filter( identity )
// ["Now","is","the","time","for","all","..."]
```

还不过瘾？再看下面这个

```js
let output = (msg, formatFn = identity) => {
    msg = formatFn(msg)
    console.log(msg)
}

let upper = txt => txt.toUpperCase()

output("Hello World", upper)     // HELLO WORLD
output("Hello World")            // Hello World
```

一元函数还有别的方式，比如某些不能传递值参数但能传递函数的方法

如 js 中的 Promise 的 then 方法

```js
// error
promise1.then(foo).then(p2).then(bar)

// success
promise1.then(foo).then(() => p2).then(bar)
```

我们可以写一个值转换函数

```js
let constant = v => () => v

promise1.then(foo).then(constant(v)).then(bar)
```

## 参数的解构和聚合

某些时候，你会有两个不兼容的函数，在无法更改声明的情况下，如何一起使用他们呢？

```js
let foo = (x, y) => console.log(x + y)

let bar = fn => fn([3, 9])

bar(foo) // error

let spreadArgs = fn => 
    argsArr => fn(...argsArr)

bar(spreadArgs(foo)) // 12
```

考虑反向操作

```js
let gatherArgs = fn => 
    (...argsArr) => fn(argsArr)

let combineFirstTwo = ([ v1, v2 ]) => v1 + v2

[1,2,3,4,5].reduce(gatherArgs(combineFirstTwo))
// 15
```

## 偏函数

```js
let partial = (fn,...presetArgs) => 
    (...laterArgs) => fn(...presetArgs, ...laterArgs)
```

js 中的 bind 也可实现上面的功能，但是 FPer 不太喜欢，因为绑定 this 上下文和偏函数应用很多时候并不同时需要这两个功能

反转参数

```js
let reverseArgs = fn => 
    (...args) => fn(...args.reverse())

let cache = {};

let cacheResult = reverseArgs(
    partial(reverseArgs( ajax ), function onResult(obj){
        cache[obj.id] = obj
    })
)

// later:
cacheResult("http://some.api/person", {user: CURRENT_USER_ID })
```

某些场景下，你只需要反转最右边的参数，因此可以编写一个基于反转参数的函数

```js
function partialRight(fn,...presetArgs) {
    return reverseArgs(
        partial(reverseArgs(fn), ...presetArgs.reverse())
    )
}

let cacheResult = partialRight( ajax, function onResult(obj){
    cache[obj.id] = obj
})

// later:
cacheResult( "http://some.api/person", { user: CURRENT_USER_ID } )
```

当然，可以用更直接的技巧

```js
let partialRight = (fn,...presetArgs) => 
    (...laterArgs) => fn( ...laterArgs, ...presetArgs )
```

## 柯里化

柯里化和偏函数很像，只不过柯里化每次都只接收一个参数，然后将参数传递给下一次调用，考虑如下代码

```js
let curry = (fn, len = fn.length) => 
    (nextCurry = prevParam => 
        nextParam => {
            let params = [...prevParam, nextParam]

            if(params.length >= len) {
                return fn(...params)
            } else {
                return nextCurry(params)
            }
    })([])
```

上面的 len 不是必须要传递的参数，但是当你要 curry 的函数是不定参数的函数，则需要手动传入一个期望长度

### 为什么使用柯里化和偏函数？

- 提高可读性

- 拆分会扰乱可读性的其他参数信息

### 更宽松的柯里化

事实上 js 内部实现的柯里化都是这样，上面我们实现的柯里化可以理解为「strict」

```js
let looseCurry = (fn, len = fn.length) => 
    (nextCurried = prevArgs => 
        (...nextArgs) => {
            let args = [...prevArgs, ...nextArgs]

            if (args.length >= len) {
                return fn(...args)
            }
            else {
                return nextCurried(args)
            }
        }
    )([])
```

### 取消柯里化

```js
let unCurry = fn => (...args) => {
    let ret = fn

    for (let arg of args) {
        ret = ret(arg)
    }

    return ret
}
```

需要注意的是，当你没有向取消柯里化中传入应有数量的参数，返回的仍然是一个偏函数

### 高级实现

上面的柯里化和偏函数都只能按照参数的顺序，开发者不可能每次都 reverse 参数

考虑下面的代码

它使得参数的位置没那么重要

```js
let partialProps = (fn, presetPropsObj) => 
    propsObj => 
        fn(Object.assign({}, presetPropsObj, propsObj))


let curryProps = (fn, len = 1) =>
    (nextCurried = nowObjProps => 
        (nextObjProps = {}) =>
            {
                let props = Object.assign({}, nowObjProps, nextObjProps)
                if(Object.keys(props).length >= len){
                    return fn(props)
                } else {
                    return nextCurried(props) 
                }   
            }
    )({})
```

可是有的时候我们没法更改传入的函数的参数，这使得我们不能轻易的使用解构。

幸好 js 有一个内置方法叫做 `toString()` 可以让我们拿到函数的参数列表，再通过类似前面 spreadArgs 的转换，实现一个函数装饰器

## Point Style

当我们遇见某些函数，他们接收参数，并将参数原封不动的 **转发** 给另一个函数，我们可以把它砍掉

```js
let addThree = v => v + 3
[1, 2, 3, 4, 5].map(v => addThree(v))

// 优化
[1, 2, 3, 4, 5].map(addThree)
```

如果是之前 `parseInt()` 的那个例子，则可以通过 `unary` 方法实现这一编程风格

又比如你有两个完全相反的判断

```js
let isShortEnough = msg => msg.length <= 5
let isLongEnough = msg => msg.length > 5

// 
let not = fn => (...args) => !fn(...args)
let isLongEnough = not(isShortEnough)  
```
