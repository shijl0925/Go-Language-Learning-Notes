# reflect/unsafe

Go (Golang) 中的反射是一项强大的功能，它允许程序在运行时检查自身的结构和值。这一功能由 reflect 包提供。反射通常用于序列化/反序列化、构建泛型库和测试等任务。本文将概述反射在 Go 中的工作原理，并提供实际示例。

## 基本概念
* Type: 值的类型,（如 int、string、float、struct）。
* Value: 变量的实际数据。
* kind: 类型的具体类别（如 reflect.Int、reflect.Struct）

## 主要类型和函数
* reflect.TypeOf：返回给定值的类型。
* reflect.ValueOf：返回给定值的值。
* reflect.Value：表示运行时数据的结构类型。
* reflect.Type：表示运行时类型信息的结构类型。
* reflect.Kind: 类型的枚举（如 reflect.Int、reflect.Struct）。
* reflect.Value.Method：返回与给定名称的方法相对应的函数值。

### 示例 1：检查类型和值
本例演示如何使用反射来检查变量的类型和值。
```
package main

import (
 "fmt"
 "reflect"
)

func main() {
 var x float64 = 3.4

 // Getting the type and value using reflection
 v := reflect.ValueOf(x)
 t := reflect.TypeOf(x)

 fmt.Println("Type:", t)  // Type: float64
 fmt.Println("Value:", v)  // Value: 3.4
 fmt.Printf("Kind is float64: %v\n", v.Kind() == reflect.Float64)  // Kind is float64: true
 fmt.Printf("Type is float64: %v\n", t == reflect.TypeOf(float64(0)))  // Type is float64: true
}
```

### 示例 2：修改值
反射也可用于修改变量的值，但这要求变量的值是可寻址的。
```
package main

import (
 "fmt"
 "reflect"
)

func main() {
 var x float64 = 3.4
 fmt.Println("Original value:", x)  // Original value: 3.4

 // Getting the addressable value using reflection
 v := reflect.ValueOf(&x).Elem()

 // Setting the value
 v.SetFloat(7.1)
 fmt.Println("Modified value:", x) // Modified value: 7.1
}
```

### 示例 3：结构体 tag 解析
反射通常用于解析 Go 中的结构标记。这在 JSON 序列化等任务中非常有用，在这些任务中，struct 标记定义了字段的编码/解码方式。
```
package main

import (
	"fmt"
	"reflect"
)

type Person struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

func main() {
	p := Person{Name: "Alice", Age: 30}

	// 获取结构体类型
	t := reflect.TypeOf(p)
	fmt.Println("Type:", t) // Type: main.Person

	// 获取结构体值
	v := reflect.ValueOf(p)
	fmt.Println("Value:", v) // Value: {Alice 30}

	fmt.Printf("Kind is reflect.Struct: %v\n", t.Kind() == reflect.Struct) // Kind is reflect.Struct: true

	// 获取结构体字段(标签/值)和方法
	fmt.Println("NumField:", t.NumField())   // NumField: 2
	fmt.Println("NumMethod:", t.NumMethod()) // NumMethod: 0

	fmt.Println("NumField:", v.NumField())   // NumField: 2
	fmt.Println("NumMethod:", v.NumMethod()) // NumMethod: 0

	// 通过type获取字段(标签/值)
	fmt.Println(t.FieldByName("Name")) // {Name  string json:"name" 0 [0] false} true
	fmt.Println(t.FieldByName("Age"))  // {Age  int    json:"age" 16 [1] false} true

	// 通过value获取字段(值)
	fmt.Println(v.FieldByName("Name"))  // Alice
	fmt.Println(v.FieldByName("Age"))   // 30

	// 输出
	// Field: Name, Tag: name
	// Field: Age, Tag: age
	for i := 0; i < t.NumField(); i++ {
		field := t.Field(i)
		fmt.Printf("Field: %s, Tag: %s\n", field.Name, field.Tag.Get("json"))
	}

	// 可以重新对结构体进行赋值操作, 前提是获得指针
	valCanSet := reflect.ValueOf(&p)
	ptr := valCanSet.Elem()
	ptr.Field(0).SetString("Bob")
	ptr.FieldByName("Age").SetInt(35)
	fmt.Println(p) // {Bob 35}
}

```

### 示例 4：构建一个通用函数
反射可用于构建可处理任何类型的泛型函数。例如，让我们创建一个函数，打印任意结构体的字段和值。
```
package main

import (
 "fmt"
 "reflect"
)

func PrintFields(s interface{}) {
 v := reflect.ValueOf(s)

 if v.Kind() == reflect.Struct {
  t := v.Type()
  for i := 0; i < v.NumField(); i++ {
   field := t.Field(i)
   value := v.Field(i)
   fmt.Printf("%s: %v\n", field.Name, value)
  }
 } else {
  fmt.Println("Expected a struct")
 }
}

type Person struct {
 Name string
 Age  int
}

func main() {
 p := Person{Name: "Alice", Age: 30}
 PrintFields(p)
}
```
### Conclusion
Go 中的反射是一种强大的工具，允许运行时类型检查和操作。它对于需要通用编程的任务特别有用，例如构建需要动态处理不同类型的库和框架。不过，应该谨慎使用反射，因为它会增加代码的阅读和维护难度，并可能带来性能开销。
