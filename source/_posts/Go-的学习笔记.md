---
layout: title
title: Go 的学习笔记
date: 2019-05-20 07:07:25
tags:
---
趁着B站的“go-common大事件”，学习了几天go语言，以下为整理的笔记和感悟。

#  Go的特色

## 与C相比：
1. 面向对象
2. 更安全的指针操作
3. 更自动的内存管理

## 与Java相比：
1. 支持指针
2. 高效的运行效率，不依赖于JVM
3. 语法简洁，没有历史包袱
4. 并发支持
5. 推崇组合而非继承

<!-- more -->
# 关键词
```go
break     default       func      interface    select 
case      defer         go        map          struct 
chan      else          goto      package      switch 
const     fallthrough   if        range        type 
continue  for           import    return       var
```

# 数据类型
Go语言将数据类型分为四类:基础类型、复合类型、引用类型和接口类型。

## 基础类型
用于构建程序中数据结构,是Go语言的世界的原子

### 布尔类型: true和false

### 整型
```go
rune, int8, int16, int32, int64 byte, uint8, uint16, uint32, uint64
//其中rune是int32的别称,byte是uint8的别称
```

### 浮点类型: float32和float64

### 复数: complex64和complex128

### 字符串string

## 复合类型
以不同的方式组合基本类型可以构造出来的复合数据类型。

### 数组array
数组是一个由固定长度的特定类型元素组成的序列,一个数组可以由零个或多个元素组成

### 结构体
结构体是一种聚合的数据类型,是由零个或多个任意类型的值聚合成的实体。

## 引用类型
数据种类很多，但它们都是对程序中一个变量或状态的间接引用。这意味着对任一引用类型数据的修改都会影响所有该引用的拷贝。

### 切片slice
及其他语言的“动态数组”，长度不固定，可以任意扩容

### 字典map
map是无序的，不能通过index获取，而必须通过key获取 

### 函数
函数可以让我们将一个语句序列打包为一个单元,然后可以从程序中其它地方多次调用。函数的机制可以让我们将一个大的工作分解为小的任务,这样的小任务可以让不同程序员在不同时间、不同地方独立完成。

#### 定义
```go
func name(parameter-list) (result-list) {
}

func sayHi(name string) string {
	return "hi!" + name;
}

fmt.Println(sayHi("yunlong"))
```

#### 特点
1. 多返回值
2. 第一类型
3. 匿名函数
4. 可变参数
5. Deferred函数

#### 方法
与特定类型关联的函数

##### 定义
```go
type Point struct {
	X, Y float64
}

func (p Point) Distance(q Point) float64 {
	return math.Hypot(q.x-p.x, q.y-p.y)
}
//上面的代码里那个附加的参数p,叫做方法的接收器(receiver)

p := Point(1, 2}
q := Point{4, 6}

p.Distance(q)
//这种p.Distance的表达式叫做选择器


//基于指针
func (p *Point) ScaleBy(factor float64) {
	p.X *= factor
	p.Y *= factor
}

r := &Point{1, 2}
r.ScaleBy(2)

```
