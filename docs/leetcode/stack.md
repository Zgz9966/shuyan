
### leetcode20

>用一个栈压入，如果和上一个相同则两个弹出，然后递归，若栈不为空，则不符合。

```js

/**
 * @param {string} s
 * @return {boolean}
 */
var isValid = function(s) {
    let sArr = s.split('')
    let arr = []
    let len = 0
    function filter(sarr) {
        let arr = []
        sarr.map(s => {
            if(arr.length === 0) arr.push(s)
            else if((s === '(' && arr[arr.length - 1] === ')') || (s === ')' && arr[arr.length - 1] === '(')){
                arr.pop()
            }
            else if((s === '{' && arr[arr.length - 1] === '}') || (s === '}' && arr[arr.length - 1] === '{')){
                arr.pop()
            }
            else if((s === '[' && arr[arr.length - 1] === ']') || (s === ']' && arr[arr.length - 1] === '[')){
                arr.pop()
            } else {
                arr.push(s)
            }
        })
        return arr
    }
    arr = filter(sArr)
    while(arr.length > 0 && arr.length !== len) {
        arr = filter(arr)
        len = arr.length
    }
    if(!arr.length) return true
    return false
};
```

### leetcode739


>在栈中压入下标和值，如果后面的值大于他，则找到弹出直到栈中没有下标和值

```js
var dailyTemperatures = function(T) {
    let stack = []
    let mapStack = []
  let result = []
  stack.push(0)
    mapStack.push(T[0])
  for(let i = 1, len = T.length; i < len; i++) {
      for(let j = stack.length - 1; j >= 0; j--) {
          if(mapStack[j] < T[i]) {
              result[stack[j]] = i - stack[j]
              stack.pop()
              mapStack.pop()
          }
      }
      stack.push(i)
       mapStack.push(T[i])
  }
  for(let j = stack.length - 1; j >= 0; j--) {
    result[stack[j]] = 0
    stack.pop()
      mapStack.pop()
  }
  console.log(result)
  return result
};


```

### leetcode224

>

```js

```