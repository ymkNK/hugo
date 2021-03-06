---
title: Go中经典nil!=nil问题
author: ymkNK
categories: [Go]
tags: [Go]
date: 2022-04-21 22:02:27 +0800
img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/post/17.jpeg
slug: 2022/4/go-nil-equal
toc: true
---
## 背景
最近在开发过程中遇到了一个奇怪的现象，对传入参数interface{}的做nil判断的时候，发现有些情况不符合预期，

## 示例代码
可以猜测一下下方代码执行后的打印结果
```go
func TestIsNilEqual(t *testing.T) {
	PatchConvey("test", t, func() {
		var isNil = func(v interface{}) bool {
			return v == nil
		}

		PatchConvey("test map", func() {
			var testMap map[string]interface{}
			test1 := isNil(map[string]interface{}{})
			test2 := isNil(testMap)
			test3 := testMap == nil
			test4 := isNil(nil)

			t.Logf("result 1: %v", test1)
			t.Logf("result 2: %v", test2)
			t.Logf("result 3: %v", test3)
			t.Logf("result 4: %v", test4)
		})
	})
}
```

## 原因解析

### 非空和空类型
在 go 中类型可以是空或非空
- 非空类型永远不能为 nil，并且永远不会导致nil-panic(等效于 Java 的 NullPointerException)
- 处理空类型时，我们仍须谨慎一点，可能会导致nil-panic

### 非空基本类型
在 go 中，基本类型不可为空。像这样的声明，都会有默认0值

|Types	|Zero value|
|----|----|
|int, int8, int16, int32, int64	|0|
|uint, uint8, uint16, uint32, uint64|0|
|uintptr|0|
|float32,float64	|0.0|
|byte	|0|
|rune   |0|
|string	|"" (empty string)|
|complex64, complex128	|(0,0i)|
|arrays of non-nillable types	|array of zero-values|
|arrays of nillable types	|array of nil-values|

### Non-nillable structs 非空结构体
组合的 `struct` 类型也是不可空的，并且 struct 的默认值将包含其所有字段的默认值

### Nillable 可为空类型
还有一种更高级到 `nillable` 类型，如果对应的类型未初始化，将会报错，触发 panic

这些可以为 nillabe 类型:
- 函数
- 通道
- Slice切片
- map
- 接口
- 指针

但是，nil-slice 和 nil-maps 仍然可以使用，在我们开始使用它们之前不必进行初始化。
- nil-maps 可以取值，但是不能赋值
- nil-slices 在外部引用切片将导致死机，但是`len()`和`cap()`的操作不会导致死机出现。他们只返回`0`, 因为对于未初始化的切片，其容量和长度都为零，所以可以在 nil-slice 上成功调用 append

### 可为空的指针、函数和接口类型会引起 panic
每当处理这些类型时，我们都必须考虑它们是否为零

### nil-channel 永远阻塞
尝试从 nil 通道读取或写入 nil 通道将永远受阻。关闭 nil 通道会引起 Panic

### nil预声明标识符nil没有默认类型
Go中其它的预声明标识符都有各自的默认类型，比如
- 预声明标识符true和false的默认类型均为内置类型bool。
- 预声明标识符iota的默认类型为内置类型int。
但是，预声明标识符nil没有一个默认类型，尽管它有很多潜在的可能类型。 事实上，预声明标识符nil是Go中唯一一个没有默认类型的类型不确定值。 我们必须在代码中提供足够的信息以便让编译器能够推断出一个类型不确定的nil值的期望类型。
```go
package main

func main() {
	// 代码中必须提供充足的信息来让编译器推断出某个nil的类型。
	_ = (*struct{})(nil)
	_ = []int(nil)
	_ = map[int]bool(nil)
	_ = chan string(nil)
	_ = (func())(nil)
	_ = interface{}(nil)

	// 下面这一组和上面这一组等价。
	var _ *struct{} = nil
	var _ []int = nil
	var _ map[int]bool = nil
	var _ chan string = nil
	var _ func() = nil
	var _ interface{} = nil

	// 下面这行编译不通过。
	var _ = nil
}
```

### 两个不同类型的nil值可能不能相互比较
```go
// error: 类型不匹配
var _ = (*int)(nil) == (*bool)(nil)
// error: 类型不匹配
var _ = (chan int)(nil) == (chan bool)(nil)

type IntPtr *int
// 类型IntPtr的底层类型为*int。
var _ = IntPtr(nil) == (*int)(nil)

// 任何类型都实现了interface{}类型。
var _ = (interface{})(nil) == (*int)(nil)

// 一个双向通道可以隐式转换为和它的
// 元素类型一样的单项通道类型。
var _ = (chan int)(nil) == (chan<- int)(nil)
var _ = (chan int)(nil) == (<-chan int)(nil)
```

### 同一个类型的两个nil值可能不能相互比较
在Go中，map、slice和func类型是不支持比较类型。比较同一个不支持比较的类型的两个值（包括nil值）是非法的。 比如，下面的几个比较都编译不通过。
```go
var _ = ([]int)(nil) == ([]int)(nil)
var _ = (map[string]int)(nil) == (map[string]int)(nil)
var _ = (func())(nil) == (func())(nil)
```
但是，映射类型、切片类型和函数类型的任何值都可以和类型不确定的裸nil标识符比较。
```go
var _ = ([]int)(nil) == nil
var _ = (map[string]int)(nil) == nil
var _ = (func())(nil) == nil
```

### nil!=nil
- nil 在 Go语言中只能被赋值给指针和接口。
- 但是接口在底层的实现有两个部分：
  - type
  - data
- 在源码中，显式地将 nil 赋值给接口时，接口的 type 和 data 都将为 nil。此时，接口与 nil 值判断是相等的
- 但如果将一个带有类型的 nil 赋值给接口时，只有 data 为 nil，而 type 为 nil，此时，接口与 nil 判断将不相等。

## 解决方式
这里提供了一种针对类型不为空但是值为空的判断方式
```go
// IsNil 判断指针,Slice,Map类型的值是否为空
func IsNil(i interface{}) bool {
	// 首先判断i是否为nil
	if i == nil {
		return true
	}

	// 判断值是否为nil
	vi := reflect.ValueOf(i)
	if vi.Kind() == reflect.Ptr ||
		vi.Kind() == reflect.Slice ||
		vi.Kind() == reflect.Map {
		return vi.IsNil()
	}

	return false
}
```


## 参考文档
- https://learnku.com/go/t/46496 原文作者：Summer
- https://www.cnblogs.com/chenqionghe/p/11357013.html
- https://gfw.go101.org/article/nil.html