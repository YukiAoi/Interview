## 代码优化

### 对象取值（对象解构）

```js
// before
const obj = {
  a:1,
  b:2,
  c:3,
  d:4,
  e:5
}

const a = obj.a
const b = obj.b
const c = obj.c
const d = obj.d
const e = obj.e
const f = obj.a + obj.d
const g = obj.c + obj.e

// after
const { a, b, c, d, e } = obj
const f = a + d
const g = c + e

// 想创建的变量名和对象的属性名不一致
const { a: a1 } = obj
console.log(a1) // 1

//要注意解构的对象不能为undefined、null。否则会报错，故要给被解构的对象一个默认值
const { a, b, c, d, e } = obj || {}
```

### 合并数据（扩展运算符）

```js
// before
const a = [ 1, 2, 3 ]
const b = [ 1, 5, 6 ]
const c = a.concat(b) // [1,2,3,1,5,6]
const obj1 = { a: 1 }
const obj2 = { b: 1 }
const obj = Object.assign({}, obj1, obj2) // {a:1,b:1}

// after
const a = [ 1, 2, 3 ]
const b = [ 1, 5, 6 ]
const c = [ ...new Set([ ...a, ...b ])] // [1,2,3,5,6]
const obj1 = { a: 1 }
const obj2 = { b: 1 }
const obj = { ...obj1, ...obj2} // {a:1,b:1}
```

### 拼接字符串（字符串模板可以运行表达式）

```js
// before
const name = '小明'
const score = 59
let result = ''
if ( score > 60 ) {
  result = `${name}的考试成绩及格` 
} else {
  result = `${name}的考试成绩不及格`
}

// after
const name = '小明'
const score = 59
const result = `${name}${score > 60 ? '的考试成绩及格' : '的考试成绩不及格'}`
```

### if中判断条件（includes）

```js
// before
if( type == 1 || type == 2 || type == 3 || type == 4 ){
  //...
}

// after
const condition = [ 1, 2, 3, 4]
if( condition.includes(type) ){
  //...
}
```

### 列表搜索（find）

```js
// before
// 搜索一般分为精确搜索和模糊搜索。搜索也叫过滤，一般用filter来实现
const a = [ 1, 2, 3, 4, 5 ]
const result = a.filter( 
  item =>{
    return item === 3
  }
)

// after
// find方法中找到符合条件的项，就不会继续遍历数组
const a = [ 1, 2, 3, 4, 5]
const result = a.find( 
  item =>{
    return item === 3
  }
)
```

### 扁平化数组（flat和Infinity）

```js
// before
// 一个部门JSON数据中，属性名是部门id，属性值是个部门成员id数组集合，现在要把有部门的成员id都提取到一个数组集合中
const deps = {
  a: [ 1, 2, 3 ],
  b: [ 5, 8, 12 ],
  c: [ 5, 14 ,79 ],
  d: [ 3, 64 ,105 ]
}
let member = []
for (let item in deps ){
  const value = deps[item]
  if ( Array.isArray(value) ) {
    member = [ ...member, ...value ]
  }
}
member = [ ...new Set(member) ]

// after
// Infinity作为flat的参数，使得无需知道被扁平化的数组的维度
// flat方法不支持IE浏览器
const deps = {
  a: [ 1, 2, 3 ],
  b: [ 5, 8, 12 ],
  c: [ 5, 14 ,79 ],
  d: [ 3, 64 ,105 ]
}
let member = Object.values(deps).flat(Infinity)
```

### 获取对象属性值（可选链操作符）

```js
// before
const name = obj && obj.name

// after
const name = obj ?. name
```

### 动态对象属性（[]）

```js
// before
let obj = {}
let index = 1
let key = `topic${ index }`
obj[key] = '话题内容'

// after
let obj = {}
let index = 1
obj[ `topic${ index }` ] = '话题内容'
```

### 输入框非空的判断（空值合并运算符）

```js
// before
if ( value !== null && value !== undefined && value !== '' ){
  //...
}

// after
// 左侧操作数为null或者undefined时，返回右侧操作数，否则返回左侧操作数
if (( value ?? '' ) !== '') {
  //...
}
```

### 异步函数（async/await）

```js
// before
const fn1 = () =>{
  return new Promise(( resolve, reject ) => {
    setTimeout(() => {
      resolve(1)
    }, 300)
  })
}
const fn2 = () =>{
  return new Promise(( resolve, reject ) => {
    setTimeout(() => {
      resolve(2)
    }, 600)
  })
}
const fn = () =>{
  fn1().then(res1 => {
    console.log(res1) // 1
    fn2().then(res2 => {
      console.log(res2) // 2
    })
  })
}

// after
// 容易形成回调地狱，建议使用async和await转为同步函数
const fn = async () =>{
  const res1 = await fn1()
  const res2 = await fn2()
  console.log(res1) // 1
  console.log(res2) // 2
}
// 但是要做并发请求时，还是要用到Promise.all()
// 如果并发请求时，只要其中一个异步函数处理完成，就返回结果，要用到Promise.race()
const fn = () =>{
  Promise.all([ fn1(), fn2() ]).then(res => {
    console.log(res) // [1,2]
  }) 
}

```