## Vue 3 数据绑定

### 1、ref

ref响应式数据。ref 函数传入一个值作为参数，一般传入基本数据类型，返回一个基于该值的响应式Ref对象，该对象中的值一旦被改变和访问，都会被跟踪到。

```javascript
// 定义 name 为 ref 响应式数据
const name = ref(111)
name.value = 222
// 页面中 name 的值已经变化
console.log(name.value) // 222
```

### 2、reactive

reactive是用来定义更加复杂的数据类型，但是定义后里面的变量取出来就不在是响应式Ref对象数据了

```javascript
// 定义一个复杂变量 user
const user = reactive({name: '张三', sex: '男'})
// 将 user 中的 name 属性 取出
const name = user.name
// name 变为 李四 后, user 中的 name 不会变化
name = '李四'
// name 变为 王五 后, name 不会变化
user.name = '王五'
// 将 user 中的 name 属性 取出转化为ref
const name2 = toRef(user, 'name')
// user.name 变为 李四，name2 也变为 李四
user.name = '李四'
// name2 变为 王五, user.name 不变
name2 = '王五'
```