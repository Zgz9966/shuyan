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

> 这里之所以使用 list copy 一份函数数组是因为，如果不拷贝的话，返回的组合函数只能使用一次。 pop 方法会修改原数组，使得 fns 长度减小

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

递归的好处在于我们能从概念上去理解 compose

## pipe

pipe 与 compose 相同，只不过 pipe 是从左往右

```js
let pipe = reverseArgs(compose)
```

你可能会困惑，我们给出详细实现。

```js
let pipe = (...fns) => 
    result => {
        let list = [...fns]

        while (list.length > 0) {
            // take the first function from the list
            // and execute it
            result = list.shift()( result );
        }

        return result;
    };
```

在某些需要 reverse 右边参数的场景，使用 pipe 会更有效率

## Abstraction

抽象是一个重要的能力，他让我们的一些代码只需要写一遍。

思考

```js
let saveComment = txt => {
    if (txt != "") comments[comments.length] = txt;
}

let trackEvent = evt => {
    if (evt.name !== undefined) {
        events[evt.name] = evt;
    }
}
```

我们可以很轻松的发现，上面代码的共性就是存储一个值

```js
function storeData(store, location, value) {
    store[location] = value;
}

function saveComment(txt) {
    if (txt != "") {
        storeData( comments, comments.length, txt );
    }
}

function trackEvent(evt) {
    if (evt.name !== undefined) {
        storeData( events, evt.name, evt );
    }
}
```

上面体现了抽象的一个原则，那就是不要重复。

但是注意抽象不要过头。

我们可以隐藏一些细节，就像黑盒子那样

但是被隐藏的细节应该是相对的，比如我们有一个相互依赖的功能 x 和 y

当我们专注 x 的时候 y 是无关紧要的 相反，我们专注 y 的时候 x 是无关紧要的。

**我们抽象的目的不是隐藏细节，而是调整聚焦**。

请时刻记住，函数式编程的本质目的是写出更多可读性良好，可维护的代码。

为了分离两个概念，我们会插入一个语义级的分界，在大多数情况下，这个边界就是函数的名称。我们调用时，只在意名称和他的输出。

我们把 _怎样_ 和 _什么_ 分离开来

命令式编程风格说明 _怎样_

而声明式风格注重 _什么_ 也就是输出，声明式关心结果，将如何实现交给别人。

声明式代码实现了从 how 到 what 的一个抽象

我们应该在声明式和命令式之间找到一个平衡。

声明式简单的将 做什么 和 如何做 分开

## compose VS abstract

compose 也是 声明式 的抽象

总而言之，compose 是一项非常有用的技能来将我们命令式的代码转换为可读性更好的声明式的代码。

在 FP 中 compose是极其重要的一种方式，它可能是函数间除了副作用传递数据的唯一方法。