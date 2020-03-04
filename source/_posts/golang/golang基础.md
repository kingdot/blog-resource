---
title: golang基础
date: 2020-2-23 17:10:01
tag: golang基础
category: go
---

### 指针、取址、和值

```
var v = 1   // v 保存的是值1
ptr := &v  // ptr保存的是v的内存地址(0x000AF)
p := *ptr // p和v相等
```

### 声明变量和实例化对象的几种方式

1. var、new、make

    - `var` 用于定义一个变量，且为它分配零zhi返回的是对象本身，
    
    - new返回的是对象的引用，
    
    - make

2. 实例化结构体的几种方法

```
type TypeA struct {
    Name string
}

vaule := TypeA{}  // 实例化一个对象

addr1 := &TypeA{} // 返回对象的地址

addr2 := new(TypeA) // new操作符返回的是新对象的引用地址

var a * Model // 未初始化，a是指向model的指针类型

make(map[string]string) // map 和 数组 不make的话，会得到nil，无法使用其方法


注意：变量在使用前最好初始化

```

### 函数


### 结构体


### 方法

>Go 没有类。

**不过你可以为结构体类型定义方法**。

方法就是一类带特殊的 `接收者` 参数的函数。

方法接收者在它自己的参数列表内，位于 `func` 关键字和方法名之间。

方法只是个带接收者参数的函数。

只能为在同一包内定义的类型的接收者声明方法，而不能为其它包内定义的类型（包括 int 之类的内建类型）的接收者声明方法。

（译注：就是接收者的类型定义和方法声明必须在同一包内；不能为内建类型声明方法。）

```golang
type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := Vertex{3, 4}
	fmt.Println(v.Abs())
}

```

与下面的方法在功能上没有区别

```golang
type Vertex struct {
	X, Y float64
}

func Abs(v Vertex) float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := Vertex{3, 4}
	fmt.Println(Abs(v))
}
```

#### 结构体可以有方法


#### 指针类型的接收者

你可以为指针接收者声明方法。

这意味着对于某类型 T，接收者的类型可以用 *T 的文法。（此外，T 不能是像 *int 这样的指针。）

指针接收者的方法可以修改接收者指向的值（就像 Scale 在这做的）。**由于方法经常需要修改它的接收者，指针接收者比值接收者更常用**。

若使用值接收者，那么 Scale 方法会对原始 Vertex 值的**副本进行操作**。（对于函数的其它参数也是如此。）Scale 方法必须用指针接受者来更改 main 函数中声明的 Vertex 的值。

```
package main

import (
	"fmt"
	//"math"
)

type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() int32 {
	//return math.Sqrt(v.X*v.X + v.Y*v.Y)
	v.X = 1
	v.Y =10
	fmt.Println(v) // {1, 10}
	return 0
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	//v.Scale(10) // 
	fmt.Println(v.Abs(), v) // v: {3, 4}
}
```

> 使用结构体的时候，注意值传递和引用传递的区别

带指针参数的函数必须接受一个指针：

```
type Vertex struct {
	X, Y float64
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func ScaleFunc(v *Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(2)
	ScaleFunc(&v, 10)

	p := &Vertex{4, 3}
	p.Scale(3)
	ScaleFunc(p, 8)

	fmt.Println(v, p)
}
```

比较前两个程序，你大概会注意到带指针参数的函数必须接受一个指针：

```
var v Vertex
ScaleFunc(v, 5)  // 编译错误！
ScaleFunc(&v, 5) // OK
```

而以指针为接收者的方法被调用时，接收者既能为值又能为指针：

```
var v Vertex
v.Scale(5)  // OK
p := &v
p.Scale(10) // OK
```

对于语句 v.Scale(5)，即便 v 是个值而非指针，带指针接收者的方法也能被直接调用。 

也就是说，由于 Scale 方法有一个指针接收者，为方便起见，Go 会将语句 v.Scale(5) 解释为 (&v).Scale(5)。

### 数组

for 循环的 range 形式可遍历切片或映射。

当使用 for 循环遍历切片时，每次迭代都会返回两个值。第一个值为当前元素的下标，第二个值为该下标所对应元素的一份副本。

```
var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}

func main() {
	for i, v := range pow {
		fmt.Printf("2**%d = %d\n", i, v)
	}
}
```

range（续）
可以将下标或值赋予 _ 来忽略它。

for i, _ := range pow
for _, value := range pow
若你只需要索引，忽略第二个变量即可。

for i := range pow