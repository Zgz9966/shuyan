# 函数组合

![](images/Lego.jpg)

函数的组合就像拼 Lego 一样，你可以从现成的积木库中取出小方块和长条去组成程序中数据需要的管道样子。

## function outputs

在函数组合中，很容易发生这种情况，一个函数的输出作为下一个函数的输入传递，直到拿到最终的输出

```js
let toUpper = msg => msg.toUpperCase()

let welcome = msg => msg + 'GOOD NIGHT'

let username = toUpper('User')
let welcomeUser = welcome(username)


// 改进

let welcomeWord = msg => welcome(toUpper(msg))

let welcomeUser = welcomeWord('User')
```

考虑一个通用的组合函数

```js
let compose2 = (fn2,fn1) =>
    origValue => fn2(fn1(origValue));
```

注意函数的运行顺序从右向左，左边的函数在最外层最后执行。

这是大多数 FP 的库的约定俗成的规定。

需要注意的是，函数的组合需要注意类型安全，上面只是一个做作的例子。

我们可以把 2 * 2 的 Lego 放在 1 * 4 的 Lego 上面，也可以吧 1 * 4 的 Lego 放在 2 * 2 的 Lego 上面。

## 常规函数组合

我们如果可以轻易的组合两个函数，那么同样的，我们也可以组合多个函数

`finalValue <-- func1 <-- func2 <-- ... <-- funcN <-- origValue`

```js
let compose = (...fns) =>
    res => {
        let list = [...fns]
        while(list.length > 0) {
            res = list.pop()(res)
        }
        return res
    }
```

> 这里之所以使用 list copy 一份函数数组是因为，如果不拷贝的话，返回的组合函数化只能使用一次。 pop 方法会修改原数组，使得 fns 长度减小

然后我们可以对之前的函数进行组合。并且可以使用系列第三篇文章提到的 partialRight 来预设要组合的函数。

当然，也可以使用 curry 和 reverse 来从左到右的依次组合。

这和数组的 `reduce()` 方法很像，因此我们还有别的实现 compose 的方式

```js
let compose = (...fns) =>
    res => [...fns].reverse().reduce((result, fn) => fn(result), res)
```

这样实现的好处是更具备可读性，也是 FPer 喜欢的方式，他的性能也和使用 for 循环类似。

但是这样实现每次迭代函数只能接收一个参数。

当然，我们也可以使用一个惰性函数封装


```js
let compose = (...fns) =>
    fns.reverse().reduce((fn1,fn2) =>
        (...args) => fn2(fn1(...args)))
```

在每次 reduce 中，我们不再计算函数的返回值，而是将函数作为输入传递给下一次迭代的函数。这样我们可以尽可能多的传递参数，而不用受到局限。

在这样实现 compose 的技巧当中，我们运用了 **惰性计算** 的方式。

这样实现在每次调用组合函数时，将不会调用 reduce 循环

甚至，我们还可以通过递归的方式去调用 compose2 方法

```js
let compose = (...fns) => {
    // pull off the last two arguments
    let [ fn1, fn2, ...rest ] = fns.reverse()

    let composedFn = (...args) => fn2( fn1( ...args ) )

    if (rest.length == 0) return composedFn;

    return compose( ...rest.reverse(), composedFn )
}
```

递归的好处在于我们能从概念性上去去理解 compose