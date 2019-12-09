写一个没有副作用的程序是不可能的。但是我们可以尽可能的限制他们。

思考下面的代码

```js
function foo(x) {
    y = x * 2;
}

var y;

foo( 3 );

////

function foo(x) {
    return x * 2;
}

var y = foo( 3 );
```

很明显前者是 因 果 不相关的，因此可读性很差，这也就是我们所说的副作用

当函数引用了一个自身之外的变量时，这个变量叫做 **自由变量**

并不是所有的 自由变量 都会带来负面影响，但是我们应当要小心。

## 了解副作用

副作用意味着您的代码读者需要人肉编译你的代码才能知道某个变量的变化。比如你在某个函数中修改了某个外部变量

副作用还有可能是因为你在返回值中的引用，如果引用的状态发生了变化，函数就产生了副作用。因此，所有影响到函数输出的东西都应该作为函数的输入

### 固定状态

一个约定俗称的规矩是从不覆盖函数实现，因此函数作为自由变量引用，我们可以当做我们引用了一个常量。

考虑两点，当你有一个函数时

- 当你给定输入，输出是不变的吗？
- 你引入的自由变量每次都是不变的吗？程序中有其他地方可能会修改这个自由变量吗？

### 偶然性

随机函数也可能产生副作用。

大部分语言中的随机都是采用伪随机算法，有些语言提供了起始值，因此给定相同的起始值生成的随机序列都是一致的。

但是 js 没有提供这项功能，因此 js 中偶然性也是副作用的一种

## IO Effects

最常见的副作用就是 IO 输入输出

比如 DOM，当我们修改了 DOM 事实上就已经产生了副作用

再比如两个依赖自由变量的 ajax，当异步请求的时候，如果回调执行的顺序不一致，可能会产生 bug，但是我们难以捕捉他们，因为很多时候，这些 bug 只会带来 ui 渲染上的不一致，或者只是简单的状态改变。

## 制裁副作用

如果你不得不通过副作用修改状态，制裁他们的一种方式是 _幂等性_

### 幂等性

首先给出一个在 数学 上和 编程 上都 **不属于** 幂等的例子

```js
function updateCounter(obj) {
    if (obj.count < 10) {
        obj.count++;
        return true;
    }

    return false;
}
```

#### 数学上的幂等性

从数学角度来看，幂等性意味着在第一次调用后输出不会改变的操作。

更清晰的例子

`f(x) = f(f(x)) = f(f(f(x)))`

在 js 内置的 Math 库中，下面这些例子显然是幂等性的。

- `Math.min(..)`
- `Math.max(..)`
- `Math.round(..)`
- `Math.floor(..)`
- `Math.ceil(..)`

我们也可以自己定义一些幂等性的运算

```js
function toPower0(x) {
    return Math.pow( x, 0 );
}

function snapUp3(x) {
    return x - (x % 3) + (x % 3 > 0 && 3);
}

toPower0( 3 ) == toPower0( toPower0( 3 ) );         // true

snapUp3( 3.14 ) == snapUp3( snapUp3( 3.14 ) );      // true
```

数学角度的幂等性并不仅仅局限于数学运算，来看看 js 基本类型强制转换

```js
var x = 42, y = "hello";

String( x ) === String( String( x ) );              // true

Boolean( y ) === Boolean( Boolean( y ) );           // true
```

或者是

```js
function upper(x) {
    return x.toUpperCase();
}

function lower(x) {
    return x.toLowerCase();
}

var str = "Hello World";

upper( str ) == upper( upper( str ) );              // true

lower( str ) == lower( lower( str ) );              // true
```

总而言之，幂等性就是满足下列公式的数学表达式

`f(x) = f(f(x))`


#### 程序中的幂等性

程序中的幂等性没有那么正式，换句话说，第一次调用 f(x) 与第二次调用的返回值没有区别。

就像在 Http 的 RESTapi 中，PUT 被定义为更新服务器资源，当我们发送了多个带有相同数据的 PUT 时，服务器资源都应该具有相同的结果状态

```js
// idempotent:幂等性
obj.count = 2;
a[a.length - 1] = 42;
person.name = upper( person.name );

// non-idempotent:非幂等性
obj.count++;
a[a.length] = 42;
person.lastUpdated = Date.now();
```

在这里，幂等性的概念是，在第一次调用之后，后续无论怎样的重复调用，程序状态都不会改变。而非幂等性每次调用都会更改程序状态。

那么在 DOM 更新中如何表现呢？

```js
var hist = document.getElementById( "orderHistory" );

// idempotent:
hist.innerHTML = order.historyText;

// non-idempotent:
var update = document.createTextNode( order.latestUpdate );
hist.appendChild( update );
// 隐式的，当前状态是下一状态的一部分
```

我们并不能总是幂等的定义我们的操作，但这有利于减少副作用的影响

## Pure Bliss

一个没有副作用的函数我们称之为 **Pure Function**

纯函数在编程角度来说是幂等的，因为没有任何副作用

```js
let add = (a, b) => a + b
```

可以发现所有的输入输出都是直接的，没有引用任何自由变量，多次调用 add 函数与只调用一次没有任何区别。所以 add 是一个幂等纯函数。

但是在数学意义中，不是所有纯函数都是幂等的，因为他们不必返回适合他们的输入的输出。

```js
function calculateAverage(nums) {
    var sum = 0;
    for (let num of nums) {
        sum += num;
    }
    return sum / nums.length;
}

calculateAverage( [1,2,4,7,11,16,22] );         // 9
```

返回值并非一个数组，因此不可能 `calculateAverage(calculateAverage(nums))`

纯函数也可以引用自由变量和函数值，只要这个变量是没有副作用的。

```js
const PI = 3.141592;

function circleArea(radius) {
    return PI * radius * radius;
}

function cylinderVolume(radius,height) {
    return height * circleArea( radius );
}
```

另一个纯函数引用自由变量的例子就是 **闭包**

虽然 fn 是一个函数对象，我们可以给他分配属性「突变」，但是这不影响函数值调用，因此只要不重新分配函数，他就无副作用。

表达纯函数的另一种方式是，给他相同的输入，他总是产生相同的输出。

如果一个函数给相同的输入产生不同的输出，那么就不是一个纯函数。产生副作用也是不纯的。

在 js 中，产生副作用实在是太容易了。

## Purely Relative

```js
function rememberNumbers(nums) {
    return function caller(fn){
        return fn( nums );
    };
}

var list = [1,2,3,4,5];

var simpleList = rememberNumbers( list );
```

simpleList 看起来像是一个纯函数，实际上并不是，

```js
function median(nums) {
    return (nums[0] + nums[nums.length - 1]) / 2;
}
simpleList( median );       // 3
// ..
list.push( 6 );
// ..
simpleList( median );  // 3.5
```

显而易见，当我们修改数组，他的返回值就发生了变化。

我们可以通过复制数组让他变成纯函数。

```js
function rememberNumbers(nums) {
    // make a copy of the array
    nums = [...nums];

    return function caller(fn){
        return fn( nums );
    };
}
```

但是可能会有一个更隐蔽的副作用潜伏。

```js
var list = [1,2,3,4,5];

// make `list[0]` be a getter with a side effect
Object.defineProperty(
    list,
    0,
    {
        get: function(){
            console.log( "[0] was accessed!" );
            return 1;
        }
    }
);

var simpleList = rememberNumbers( list );
// [0] was accessed!
```

因此，我们可以通过这样

```js
function rememberNumbers(...nums) {
    return function caller(fn){
        return fn( nums );
    };
}

var simpleList = rememberNumbers( ...list );
// [0] was accessed!
```

这样做的好处是，rememberNumbers 是纯函数，造成副作用的原因在于 `...`

纯函数 + 不纯函数 = 不纯函数

```js
// yes, a silly contrived example :)
function firstValue(nums) {
    return nums[0];
}

function lastValue(nums) {
    return firstValue( nums.reverse() );
}

simpleList( lastValue );    // 5

list;                       // [1,2,3,4,5] -- OK!

simpleList( lastValue );    // 1
```

`reverse()` 方法其实修改了原本的数组

所以我们可以这样做

```js
function rememberNumbers(...nums) {
    return function caller(fn){
        // send in a copy!
        return fn( [...nums] );
    };
}
```

传递一个复制了的数组来避免接受的函数在行为上改变原数组。

但是我们仍然无法保证我们传递一个不纯的函数进去。因此，我们只能尽量的让函数 pure，这能提高可读性

## Referential transparency 

判断一个函数是否纯净的第三种方式，是 Referential transparency。如果将一个函数和他的输出对换，行为上没有任何变化，那么就是纯函数。

也就是说，在一个程序当中，我们用一个 val 代替了一个函数调用，在执行上我们看不出任何区别，那么这个函数就是纯函数。

如果一个函数存在副作用，但是在程序的任何地方都没有被观察到或者被依赖，那么他还具备引用透明性吗？

```js
function calculateAverage(nums) {
    sum = 0;
    for (let num of nums) {
        sum += num;
    }
    return sum / nums.length;
}

var sum;
var numbers = [1,2,4,7,11,16,22];

var avg = calculateAverage( numbers );
```

不得不说他和纯函数几乎没有区别，但是如何看待他取决于你自己

### Performance Effects

观察这个函数

```js
var cache = [];

function specialNumber(n) {
    // if we've already calculated this special number,
    // skip the work and just return it from the cache
    if (cache[n] !== undefined) {
        return cache[n];
    }

    var x = 1, y = 1;

    for (let i = 1; i <= n; i++) {
        x += i % 2;
        y += i % 3;
    }

    cache[n] = (x * y) / (n + 1);

    return cache[n];
}

specialNumber( 6 );             // 4
specialNumber( 42 );            // 22
specialNumber( 1E6 );           // 500001
specialNumber( 987654321 );     // 493827162
```

我们通过一个缓存来优化多次调用这个算法的性能，但是这看起来很蠢，你可能认为他是纯函数，但是我不这么认为。

```js
var specialNumber = (function memoization(){
    var cache = [];

    return function specialNumber(n){
        // if we've already calculated this special number,
        // skip the work and just return it from the cache
        if (cache[n] !== undefined) {
            return cache[n];
        }

        var x = 1, y = 1;

        for (let i = 1; i <= n; i++) {
            x += i % 2;
            y += i % 3;
        }

        cache[n] = (x * y) / (n + 1);

        return cache[n];
    };
})();
```

我们通过 IIFE 来 **确保** 函数的其他部分不能访问到 cache，而不仅仅是 **不允许**

## Mentally Transparent

虽然有时将常数和函数调用替换对函数的执行没有影响，但是我们不应该这么做。

不仅仅是因为数据可能是变化的，还为了更好地可读性。

读者会不断地心算一个永远不会变的结果，但是纯函数能够减少这样的花销。
