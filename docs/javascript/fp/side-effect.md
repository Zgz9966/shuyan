一个没有副作用的代码是不存在的。但是可以尽可能的限制。

举个栗子

```js
let y;
let foo = x =>  {
    y = x * 2;
}
foo(3);

// 改进

let foo = x => x * 2;
let y = foo(3);
```

很明显，前者是 _因果_ 不相关的，我们需要读 foo 的内部代码，才能知道 y 被修改了。因此可读性很差，这就是副作用。

函数引用了一个自身之外的「自由变量」时，副作用就产生了。

需要注意的是，并不是所有的「自由变量」都是负面的，但是应当要关注这些自由变量。

## Side Effects

副作用意味着代码的读者需要「人肉编译」才能跟踪某个变量的变化。

副作用的产生可能是因为一个返回值中的引用，例如引用的 state 发生了 Mutation「突变」。

因此，所有影响到函数输出的因素都应该作为函数的输入。

### 固定状态

一个约定俗称的规矩是从不覆盖函数的实现，因此函数作为自由变量引用，可以当做引用了一个常量。

考虑两点，当有一个函数时

- 给定输入，输出是不变的吗？
- 引入的自由变量每次都是不变的吗？程序中有其他地方可能会修改这个自由变量吗？

### 偶然性

随机函数也会产生副作用。

大部分语言中的随机都是采用伪随机算法，有些语言提供了起始值，因此给定相同的起始值生成的随机序列都是一致的。

但是 js 没有提供这项功能，因此 js 中偶然性也是副作用的一种

### IO Effects

最常见的副作用就是 IO 输入输出

比如 DOM，当修改 DOM 之后，事实上就已经产生了副作用

再比如两个依赖自由变量的 ajax，当异步请求的时候，如果回调执行的 **顺序** 不一致，可能会产生 bug，但是我们难以捕捉他们。

因为很多时候，这些 bug 只会带来 ui 渲染上的不一致，或者只是简单的状态改变。

## 制裁副作用

如果不得不通过副作用修改状态，制裁方式之一是 _幂等性_

### 幂等性

首先给出一个在「数学」上和「编程」上都不属于幂等的例子

```js
let updateCounter = obj => {
    if (obj.count < 10) {
        obj.count++;
        return true;
    }

    return false;
};
```

#### 数学上的幂等性

从数学角度来看，幂等性意味着在第一次调用后输出不会改变的操作。

更清晰的例子。

`f(x) = f(f(x)) = f(f(f(x)))`

在 js 内置的 Math 库中，下面这些例子显然是幂等性的。

- `Math.min(..)`
- `Math.max(..)`
- `Math.round(..)`
- `Math.floor(..)`
- `Math.ceil(..)`

我们也可以自己定义一些幂等性的运算

```js
let toPower0 = x => Math.pow( x, 0 );

let snapUp3 = x => x - (x % 3) + (x % 3 > 0 && 3);

toPower0(3) == toPower0(toPower0(3));         // true

snapUp3(3.14) == snapUp3(snapUp3(3.14));      // true
```

数学角度的幂等性并不仅仅局限于数学运算，js 中基本类型强制转换也是幂等的

```js
let x = 42, y = 'hello';

String(x) === String(String(x));              // true

Boolean(y) === Boolean(Boolean(y));           // true
```

或是

```js
let upper = v => v.toUpperCase();

let lower = v => v.toLowerCase();

let str = 'Hello World';

upper(str) == upper(upper(str));              // true

lower(str) == lower(lower(str) );             // true
```

总而言之，幂等性就是满足下列公式的数学表达式

`f(x) = f(f(x))`

#### 程序中的幂等性

程序中的幂等性没有那么正式，换句话说，当第一次调用 f(x) 与第二次调用的返回值没有区别，从程序的角度，我们认为是幂等的。

就像在 Http 的 RESTapi 中，PUT 被定义为更新服务器资源，当发送多个带有相同参数的 PUT 时，服务器资源都具备相同的结果状态。

```js
// idempotent:幂等
obj.count = 2;
a[a.length - 1] = 42;
person.name = upper(person.name);

// non-idempotent:非幂等
obj.count++;
a[a.length] = 42;
person.lastUpdated = Date.now();
```

因此，幂等性的定义是，在第一次调用之后，后续怎样的重复调用，程序状态都不会再改变。非幂等性在每次调用都会更改程序状态。

那么在 DOM 更新中如何表现呢？

```js
let hist = document.getElementById('orderHistory');

// idempotent:
hist.innerHTML = order.historyText;

// non-idempotent:
let update = document.createTextNode(order.latestUpdate);

hist.appendChild(update);
// 隐式的，当前状态是下一状态的一部分
```

我们并不能总是幂等的定义我们的操作，但这有利于减少副作用的影响。

## Pure Bliss

一个没有副作用的函数称之为 **Pure Function**，也就是「纯」函数

纯函数在编程角度来说是幂等的，因为没有任何副作用。

```js
let add = (a, b) => a + b
```

可以发现所有的输入输出都是直接的，没有引用任何自由变量，多次调用 add 函数与只调用一次没有任何区别。所以 add 是一个幂等纯函数。

但是在数学意义中，不是所有纯函数都是幂等的，因为的返回值可能不能作为他们的输入。

```js
let calculateAverage = nums => {
    let sum = 0;
    for(let num of nums) {
        sum += num;
    }
    return sum / nums.length;
}

calculateAverage([1,2,4,7,11,16,22]);         
// 9
```

在这个例子中，返回值并非一个数组，因此不可能「recall」 `calculateAverage(calculateAverage(nums))`

纯函数也可以引用自由变量和其他函数，前提是自由变量没有副作用。

```js
const PI = 3.141592;

let circleArea = radius => PI * radius * radius;

let cylinderVolume = (radius, height) => height * circleArea( radius );
```

另一个纯函数引用自由变量的例子就是 **闭包**

虽然 circleArea 是一个函数对象，我们可以给他添加一个可能「突变」的属性，但是不影响 circleArea 调用。因此只要不重新分配函数，就没有副作用。

验证纯函数的另一种方式是，相同的输入永远产生相同的输出。

如果一个函数给相同的输入产生不同的输出，那么就不是一个纯函数。

在 js 中，写带副作用的代码实在是太容易了。

## Purely Relative

```js
let rememberNumbers = nums => 
    fn => fn(nums);

let list = [1,2,3,4,5];

let simpleList = rememberNumbers(list);
```

simpleList 像一个纯函数，但实际上并不是，

```js
let median = nums => (nums[0] + nums[nums.length - 1]) / 2;

simpleList(median);       
// 3

list.push( 6 );

simpleList(median);  
// 3.5
```

显而易见，当我们修改输入引用的自由变量，返回值就发生了变化。

可以通过复制数组使得 simpleList 变成纯函数。

```js
let rememberNumbers = nums => {
    // make a copy of the array
    nums = [...nums];

    return fn => fn(nums);
}
```

但是这可能会导致一个更隐蔽的副作用潜伏。

```js
let list = [1,2,3,4,5];

// make `list[0]` be a getter with a side effect
Object.defineProperty(
    list,
    0,
    {
        get: () => {
            console.log( "[0] was accessed!" );
            return 1;
        }
    }
);

let simpleList = rememberNumbers(list);
// [0] was accessed!
```

这个副作用看起来无法避免，但是我们可以将它从函数内部转移到调用函数的地方。

```js
let rememberNumbers = (...nums) =>
    fn => fn(nums)

let simpleList = rememberNumbers(...list);
// [0] was accessed!
```

这样做的好处是，rememberNumbers 是纯函数，造成副作用的原因在于 `...`

### 纯函数 + 不纯函数 = 不纯函数

```js
// yes, a silly contrived example :)
let firstValue = nums => nums[0];

let lastValue = nums => firstValue(nums.reverse());

simpleList(lastValue);    
// 5

console.log(list);
// [1,2,3,4,5] -- OK!

simpleList(lastValue);    
// 1
```

`reverse()` 方法其实修改了原本的数组，只不过修改的是内部引用的数组。

怎样改进呢？

```js
let rememberNumbers = (...nums) => 
    fn => fn([...nums]);
```

可以通过传递一个数组拷贝来避免接受该数组的函数在调用时改变原数组。

但是仍然无法保证传递一个不纯的函数。因此，我们只能尽量的让函数 pure，为了可读性。

## Referential transparency 

判断一个函数是否纯净的第三种方式，是 Referential transparency 「引用透明性」。如果将值替换函数调用的那一行，行为上没有任何变化，那么该函数就是纯函数。

也就是说，在一个程序当中，我们用一个 val 代替了一个函数调用，在执行上我们看不出任何区别。

如果一个函数存在副作用，但是在程序的任何地方都没有被观察到或者被依赖，那么他还具备「引用透明性」吗？

```js
let sum;

let calculateAverage = nums => {
    sum = 0;
    for (let num of nums) {
        sum += num;
    }
    return sum / nums.length;
}

let numbers = [1,2,4,7,11,16,22];

let avg = calculateAverage(numbers);
```

不得不说它和纯函数几乎没有区别，但是如何看待他取决于你自己

### Performance Effects

观察这个函数

```js
let cache = [];

let specialNumber = n => {
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

我们通过一个缓存来优化多次调用这个函数的性能，但是这看起来很蠢，你可能认为他是纯函数，但是我不这么认为。

```js
let specialNumber = (function memoization(){
    let cache = [];

    return function specialNumber(n){
        // if we've already calculated this special number,
        // skip the work and just return it from the cache
        if (cache[n] !== undefined) {
            return cache[n];
        }

        let x = 1, y = 1;

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

在这里至关重要的一点，就是我们能确保，而不仅仅是心灵约定。

## Mentally Transparent

虽然有时将常数和函数调用替换对函数的执行没有影响，但是我们不应该这么做。

不仅仅是因为数据可能是变化的，还为了更好地可读性。

读者会不断地心算一个永远不会变的结果，但是纯函数能够减少这样的花销。

## 函数提纯

可以将副作用从函数内部转移到调用函数的地方，这看起来更明显。

举个栗子

```js
let addMaxNum = arr => {
    let maxNum = Math.max(...arr);
    arr.push(maxNum + 1);
}

let nums = [4,2,7,3];

addMaxNum(nums);

// [4,2,7,3,8]

//改进
let addMaxNum = arr => Math.max(...arr) + 1;

let nums = [4,2,7,3];

nums.push(addMaxNum(nums));

// [4,2,7,3,8]
```

我们可以将 addMaxNum 折叠起来了，我们不需要再去观察 addMaxNum 的内部实现。

另外一种方式是采用 immutable 数据结构来实现，我会在下一篇文章说明

我们还可以从这几个角度来分析。

### 自由变量

如果一个函数是不纯的，并且引用了外部的自由变量，我们可以使用作用域来封装。

```js
let users = {};

let fetchUserData = userId => {
    ajax(`http://some.api/user/${userId}`, function onUserData(user){
        users[userId] = user;
    });
}

// 改进
function safer_fetchUserData(userId, users) {
    // simple, naive ES6+ shallow object copy, could also
    // be done w/ various libs or frameworks
    users = Object.assign( {}, users );

    fetchUserData( userId );

    // return the copied state
    return users;


    // ***********************

    // original untouched impure function:
    function fetchUserData(userId) {
        ajax(
            `http://some.api/user/${userId}`,
            function onUserData(user){
                users[userId] = user;
            }
        );
    }
}
```

函数的纯度与否其实是由外部决定的，内部也可以适当的采用一些不纯的技术，比如为了性能使用 cache 缓存结果。

但是我们的目标是尽可能的减少副作用。

### 掩盖

有些时候，不纯的函数来自于第三方库，你无法直接将自由变量封装在函数的作用域内

```js
let nums = [];
let smallCount = 0;
let largeCount = 0;

function generateMoreRandoms(count) {
    for (let i = 0; i < count; i++) {
        let num = Math.random();

        if (num >= 0.5) {
            largeCount++;
        }
        else {
            smallCount++;
        }

        nums.push( num );
    }
}
```

我们可以通过 brute-force 算法执行以下步骤来改进：

- 保存未被影响的当前状态
- 设置初始输入状态
- 运行不纯的函数
- 捕获副作用
- 恢复原状态
- 返回副作用状态

```js
function safer_generateMoreRandoms(count,initial) {
    // (1) Save original state
    let orig = {
        nums,
        smallCount,
        largeCount
    };

    // (2) Set up initial pre-side effects state
    nums = [...initial.nums];
    smallCount = initial.smallCount;
    largeCount = initial.largeCount;

    // (3) Beware impurity!
    generateMoreRandoms( count );

    // (4) Capture side effect state
    let sides = {
        nums,
        smallCount,
        largeCount
    };

    // (5) Restore original state
    nums = orig.nums;
    smallCount = orig.smallCount;
    largeCount = orig.largeCount;

    // (6) Expose side effect state directly as output
    return sides;
}
```

需要注意的是 **掩盖** 手段只有在处理同步代码时才有效。

### 规避

当我们要处理的副作用是一个 **突变** 的输入导致的，「数组」或是「对象」我们可以这样做

```js
function handleInactiveUsers(userList, dateCutoff) {
    for (let i = 0; i < userList.length; i++) {
        if (userList[i].lastLogin == null) {
            // remove the user from the list
            userList.splice(i, 1);
            i--;
        }
        else if (userList[i].lastLogin < dateCutoff) {
            userList[i].inactive = true;
        }
    }
}
```

先做一个深拷贝

```js
function safer_handleInactiveUsers(userList,dateCutoff) {
    // make a copy of both the list and its user objects
    let copiedUserList = userList.map(function mapper(user){
        // copy a `user` object
        return Object.assign( {}, user );
    });

    // call the original function with the copy
    handleInactiveUsers(copiedUserList, dateCutoff);

    // expose the mutated list as a direct output
    return copiedUserList;
}
```

### `this` Revisited

有的时候 this 作为一个隐式的输入也可能导致副作用

```js
let ids = {
    prefix: "_",
    generate() {
        return this.prefix + Math.random();
    }
};
```

我们可以创建一个包装函数来传入可读的上下文

```js
let safer_generate = context => ids.generate.call(context);

safer_generate({ prefix: "foo" });
// "foo0.8988802158307285"
```

本质上，我们并没有消除副作用的影响，而是尽可能的做到在运行时能将 bug 定位在仍然使用副作用的代码上。