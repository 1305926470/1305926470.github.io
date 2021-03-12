---
layout: single
title:  "Comparable"
date:   2020-04-20 10:50:46 +0800
permalink: /golang/part1/comparable
toc: true
toc_sticky: true
---



比较运算符比较两个操作数并产生一个无类型的布尔值。

```
==    equal
!=    not equal
<     less
<=    less or equal
>     greater
>=    greater or equal
```



**比较运算前提**：在任何比较中，第一个操作数必须可分配给第二个操作数的类型，反之亦然。如一个 int32 的整数无法于 int64 的直接进行比较运算。

```go
var a int32 = 1
var b int64 = 1
fmt.Println(a > b)  // invalid operation: a > b (mismatched types int32 and int64)
```



相等运算符==和!=应用于可比较的操作数。排序运算符<、<=、>和>=应用于排序的操作数。

## 可比较类型

- `布尔型` 具有可比性。如果两个布尔值都为真或都为假，则它们是相等的。
- `整数值`按照通常的方式是可比较的和有序的。但要注意区分 int8 int16 int32 int64 它们不能直接进行比较运算。
- `浮点值`是可比较的和有序的。
- `复数值`是可比较的。如果实数(u) ==实数(v)和imag(u) == imag(v)两个复数值u和v相等。
- `字符串`具有可比性，并按词法字节顺序排列。
- `指针`是可比较的。两个指针值是相等的，如果它们指向同一个变量，或者两个指针值都为nil。指向不同大小为零的变量的指针可能相等，也可能不相等。
- `chan` 是可比较的。两个通道的值是相等的，如果它们是由同一个调用创建的，或者如果两个通道的值都为nil。
- `interface{}`是可比较的。如果两个接口值具有相同的动态类型和相等的动态值，或者两个接口值都为nil，则两个接口值是相等的。
  当类型x的值是可比较的，并且x执行t时，非接口类型x的值x和接口类型t的值t是可比较的，如果t的动态类型和x相同，t的动态值等于x，则两者相等。
- 如果`结构值`的所有字段都是可比较的，则结构值是可比较的。如果两个结构值对应的非空白字段相等，则两个结构值相等。
- 如果`数组`元素类型的值是可比较的，则数组值是可比较的。如果两个数组的对应元素相等，则两个数组值相等。





## 不可比较类型

`slice` 是不可比较类型，但是数组是可比较的（当数组中的元素是可比较时）。

`map` 是不可比较类型。

`function` 是不可比较类型。



比较具有相同动态类型的两个接口值，如果该类型的值不可比较，则会导致运行时恐慌。这种行为不仅适用于直接的接口值比较，也适用于将接口值数组或结构与接口值字段进行比较。

切片、映射和函数值是不可比较的。但是，作为一种特殊情况，片、映射或函数值可以与预先声明的标识符nil进行比较。指针、通道和接口值与nil的比较也是允许的，符合上面的一般规则。



## 动手

通过反射查看是否可比较

```go
func comparableType() {
   var b bool = true
   var i int = 1
   var f float64 = 1.0
   var s string = ""
   var ptr *int = &i
   var ch = make(chan int)
   var it error = errors.New("error interface")
   var struc = struct{x int}{x: 1}
   var array = [3]int{1,2,3}
   var slice = make([]int, 0, 3)  // 不可比较
   var m = map[string]int{
      "id":1,
   }
   
   fmt.Println("bool: ", reflect.TypeOf(b).Comparable())
   fmt.Println("int: ", reflect.TypeOf(i).Comparable())
   fmt.Println("float: ", reflect.TypeOf(f).Comparable())
   fmt.Println("string: ", reflect.TypeOf(s).Comparable())
   fmt.Println("pointer: ", reflect.TypeOf(ptr).Comparable())
   fmt.Println("chan: ", reflect.TypeOf(ch).Comparable())
   fmt.Println("interface: ", reflect.TypeOf(it).Comparable())
   fmt.Println("struct: ", reflect.TypeOf(struc).Comparable())
   fmt.Println("array: ", reflect.TypeOf(array).Comparable())
   fmt.Println("slice: ", reflect.TypeOf(slice).Comparable())
   fmt.Println("map: ", reflect.TypeOf(m).Comparable())
}
```

运行结果

> Is comparable:
>
> bool:  true
> int:  true
> float:  true
> string:  true
> pointer:  true
> chan:  true
> interface:  true
> struct:  true
> array:  true
> slice:  false
> map:  false







# 扩展阅读

https://golang.org/ref/spec#Comparison_operators