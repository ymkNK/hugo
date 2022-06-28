---
title: Go实践：将Struct转为Query参数字符串
author: ymkNK
categories: [Go]
tags: [Go]
date: 2022-06-28 18:01:27 +0800
img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/post/6.jpeg
slug: 2022/6/build-api-query
toc: true
---
## 背景
在写go的过程中，经常需要拼接一些url来进行http的请求，代码起来感觉非常的不优雅，全是fmt，因此写了一个小工具，将简单的struct，通过json的tag，转化为对应的query string

## 代码
```Go
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

// ParseToRequestQuery 将简单json结构体转成query参数
func ParseToRequestQuery(p interface{}) string {
	paramArr := buildParamArr(p)

	return strings.Join(paramArr, "&")
}

func buildParamArr(p interface{}) []string {
	v := reflect.ValueOf(p)
	t := v.Type()

	paramArr := make([]string, 0)

	for i := 0; i < t.NumField(); i++ {
		// Interface()：获取字段对应的值
		paramVal := v.Field(i).Interface()
		if IsNil(paramVal) {
			continue
		}

		// 取每个字段的json tag
		paramKey := t.Field(i).Tag.Get("json")

		// 取出指针中的值
		if reflect.ValueOf(paramVal).Kind() == reflect.Ptr {
			paramVal = reflect.ValueOf(paramVal).Elem().Interface()
		}

		// 如果是struct 那么看看里面的字段
		if reflect.ValueOf(paramVal).Kind() == reflect.Struct {
			paramArr = append(paramArr, buildParamArr(paramVal)...)
			continue
		}

		paramArr = append(paramArr, fmt.Sprintf("%s=%v", paramKey, paramVal))
	}

	return paramArr
}
```

## 效果
```Go
type TestStruct struct {
	A string `json:"a"`
	B int64  `json:"b"`
}

type TestStruct1 struct {
	C *bool `json:"c"`
	D bool  `json:"d"`
}

type TestStruct2 struct {
	E *string `json:"e"`
}

type TestStructTotal struct {
	F     string       `json:"f"`
	Test  TestStruct   `json:"test"`
	Test1 *TestStruct1 `json:"test_1"`
	TestStruct2
}

func TestParseToRequestQuery(t *testing.T) {
	type args struct {
		p interface{}
	}
	type testConfig struct {
		args args
		want string
	}
	PatchConvey("test", t, func() {
		//your mock code...
		PatchConvey("test case1", func() {
			c := true
			e := "test"
			tt := testConfig{
				args: args{},
				want: "f=f&a=&b=0&c=true&d=true&e=test",
			}

			got := ParseToRequestQuery(TestStructTotal{
				F: "f",
				Test: TestStruct{
					A: "",
					B: 0,
				},
				Test1: &TestStruct1{
					C: &c,
					D: true,
				},
				TestStruct2: TestStruct2{
					E: &e,
				},
			})
			So(got, ShouldEqual, tt.want)
		})
		PatchConvey("test case2", func() {
			tt := testConfig{
				args: args{
					p: TestStructTotal{
						Test: TestStruct{
							A: "valueA",
							B: 2,
						},
						Test1: &TestStruct1{
							C: nil,
							D: false,
						},
					},
				},
				want: "f=&a=valueA&b=2&d=false",
			}
			got := ParseToRequestQuery(tt.args.p)
			So(got, ShouldEqual, tt.want)
		})
	})
}
```

## 实际使用
```go
    // param 为参数结构体
	apiPath := fmt.Sprintf(
		"/api/path?%s", beanutil.ParseToRequestQuery(param),
	)
```