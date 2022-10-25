# Dart 基础

## 一、语法

### 1.1  标识符

> 标识符是为变量、方法、枚举、类、接口等指定的名字

- 小驼峰命名法

  单词首字母大写，其余小写，第一个单词首字母小写。用于变量名

- 大驼峰命名法

  单词首字母大写，其余字母小写。用于类名

- 下划线命名法

  所有单词小写，单词之间下划线分割。用于库名、包名

### 1.2  关键字

> 具有特殊含义的单词，又被称为保留字，应避免将关键字作为标识符。

final：只能赋值一次，可在初始化时赋值

const 修饰的变量必须在声明时赋值，且必须是编译时常量

### 1.3 变量

> 使用 runtimeType 获取 运行时对象
>
> 未声明的变量默认值为空

### 1.4 数据类型

> Dart 会进行类型推断

- 数字类型 num

  int、double

- 字符串类型 String

  单引号、双引号、三单引号、三双引号，使用 ${} 获取表达式值，$变量名获取属性值

- 布尔类型

- 集合

  List：有序集合，可插入任意类型元素，使用 [] 创建

  Set：无序集合，不可重复，使用 {} 创建

  map：键值对

- 符文 Runes

  用于字符串种表示Unicode， \u 开头

### 1.5 函数

```dart
/// 函数简写方式
bool isEvent(int x) => x%2 == 0 ? true : false;
```

- 可选命名参数

```dart
/// 可选命名参数, 花括号
var message(String from, String content,{DateTime time, String device}){
    // 方法体
}
// 可选命名参数调用
message( 'from', 'content', time: DateTime.now)

```

- 可选位置参数

```dart
/// 可选位置参数, 中括号
var message(String from, String content,[DateTime time, String device]){
    // 方法体
}
// 可选位置参数调用, 参数顺序必须固定
message( 'from', 'content', DateTime.now)

```

- 匿名函数

```dart
var list = [1,2,3]
list.forEach((Object element){
    print(element)
})
```

### 1.6 作用域

### 1.7 闭包

一个函数对象，即函数对象的调用在其原始作用域之外，依然可以访问其语法作用域内的变量

```dart
Function makeAdder(num addBy){
	return (num i) => i + addBy; 
}
/// add2 = (num i) => i + 2;
var add2 = makeAdder(2);
var result = add2(6);
```

### 1.8 回调函数

```dart
void add(Function(int) callback){
    for(int progress = 0; process < 4; progress++ ){
        callback(progress);
    }
}

add((int i){
    print(i);
})
```

### 1.9 运算符

- 逻辑运算符

- 算数运算符

- 关系运算符

- 类型测试运算符

  as、is、is！

- 位运算符

- 条件运算符

```dart
/// 三元条件表达式
// tem 作为表达式, 结果为 true 时返回 res1, 否则返回 res2
tem ? res1 : res2
/// 二元条件表达式
// tem 不为空时返回自己, 否则返回 tem1
tem ?? tem1
```

- 赋值运算符