---
title: Go中经典nil!=nil
author: ymkNK
categories: [Go]
tags: [Go]
date: 2022-04-21 22:02:27 +0800
img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/post/17.jpeg
slug: 2022/4/go-nil-equal
toc: true
---
# 背景
最近在开发过程中遇到了一个奇怪的现象，对传入参数interface{}的做nil判断的时候，发现有些情况不符合预期

# 示例代码
可以猜测一下下方代码执行后的打印结果
```
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




# 原因解析
- nil 在 Go语言中只能被赋值给指针和接口。
- 但是接口在底层的实现有两个部分：
  - type 
  - data
- 在源码中，显式地将 nil 赋值给接口时，接口的 type 和 data 都将为 nil。此时，接口与 nil 值判断是相等的
- 但如果将一个带有类型的 nil 赋值给接口时，只有 data 为 nil，而 type 为 nil，此时，接口与 nil 判断将不相等。


# 参考文档
- https://learnku.com/go/t/46496
- https://www.cnblogs.com/chenqionghe/p/11357013.html
- https://gfw.go101.org/article/nil.html