### 数组的优缺点

#### 优点

- 构建非常简单
- 能在`O(1)`的时间里根据数组的下标`index`查询某个元素

#### 缺点

- 构建时必须分配一段连续的空间
- 查询某个元素是否存在时需要遍历整个数组，耗费`O(n)`的时间（其中，`n`是元素的个数）
- 删除和添加某个元素时，同样需要耗费`O(n)`的时间

### leetcode242

```js
let isAnagram = (s, t) => {
  let m = new Map()
  let sArr = s.split('')
  let tArr = t.split('')
  let p = true
  sArr.map(val => {
    let a = m.get(val)
    if(a === undefined) {
      m.set(val, 1)
    } else {
      a++
      m.set(val, a)
    }
  })
  tArr.map(val => {
    let a = m.get(val)
    if(a === undefined) {
      p = false 
    } else {
      a--
      m.set(val, a)
    }
  })
  for(let i of m.values()) {
    if(i > 0) return false
  }
  return p
}
```

别人的优秀代码
```js
const isAnagram = function(s, t) {
    if(s.length !== t.length) return false;
    const set = new Set(s.split(""));

    for(let item of set.keys()) {
      const reg = new RegExp(item,"g");
      const s1 = s.match(reg);
      const s2 = t.match(reg);
      if(!s2 || s1.length !== s2.length) return false;
    }

    return true;
};
```