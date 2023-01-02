本文介绍一道常见的面试题：typescript中 **type **和 **interface** 的异同点，以及什么时候该使用它们。
# 相同的用法
## 定义object类型
```typescript
interface User {
  name: string
  age: number
}

type User = {
  name: string
  age: number
}
```
## 定义function类型
```typescript
interface Fn {
  (name: string) => void
}

type Fn = (name: string) => void
```
## 继承（拓展）
```typescript
// interface

interface Name {
  name: string
}

interface User extends Name {
  age: number
}


// type

type Name = {
  name: string
}

type User = Name & {
  age: number
}
```
### interface 继承 type
```typescript
type Name = {
  name: string
}

interface User extends Name {
  age: number
}
```
### type 继承 interface
```typescript
interface Name {
  name: string
}

type User = Name & {
  age: number
}
```
# 不同点
## 实现类型: 
type 可以定义基本类型，元组， 联合类型。interface 不行
```typescript
// 基本类型
type Name = string

// 联合类型
interface ZhangSan {
  name: string
}

interface LiSi {
  age: number
}

type People = ZhangSan | LiSi

// 元组
type Perple = [ZhangSan, LiSi]

```
## 声明合并: 
type 不能重复定义相同的类型名，interface 定义相同的类型名会进行声明合并。
### interface：
```typescript
// interface

interface User {
  name: string
}

interface User {
  age: number
}

// 属性会被合并到User中
const user: User = {
  name: '',
  age: 18
}

```
### type：
不能重复声明<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/2180858/1672631924059-c0c750f0-7edd-40c3-b50e-ba43adc83dbc.png#averageHue=%23fefdfc&clientId=u14a10d80-7230-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=250&id=uc24c08ae&margin=%5Bobject%20Object%5D&name=image.png&originHeight=500&originWidth=788&originalType=binary&ratio=1&rotation=0&showTitle=false&size=52665&status=done&style=stroke&taskId=u65bc1dfd-9139-46bb-af00-e3002cedb68&title=&width=394)
# 总结
## 相同点

1. 都能定义object类型和function类型
2. 都能被继承，interface使用extends，type使用&
## 区别总结：

1. 实现类型: type 可以定义基本类型，元组， 联合类型。interface 不行
2. 声明合并: type 不能重复定义相同的类型名，interface 定义相同的类型名会进行声明合并
## 什么时候使用type

1. 定义基础类型
2. 定义函数类型
3. 定义联合类型
4. 定义元组类型
5. 定义映射类型
## 什么时候使用interface

1. 需要声明合并时
2. 定义object类型时
