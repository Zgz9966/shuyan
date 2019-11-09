
### 原型链是什么

> 原型链就是下面代码中用`__proto__`串起来的链

```js
class User{}
let xiaoming = new User()
console.log(xiaoming.__proto__ === User.prototype) // true
console.log(xiaoming.__proto__.constructor === User) // true

class Student extends User{}
console.log(Student.prototype.__proto__ === User.prototype) // true
```