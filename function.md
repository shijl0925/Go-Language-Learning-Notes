# 函数

## 按值传递(call by value)和按引用传递(call by reference)

Go默认使用按值传递，也就是传递参数的副本。函数接收参数副本后，在使用变量的过程中可能对副本的值进行更改，但不会影响到原先的值。
如果你希望函数可以直接修改参数的值，而不是对参数的副本进行操作，你需要将参数的地址（变量名前面添加`&`符合，比如`&var`）传递给函数，这就是按引用传递，比如`Function(&arg1)`，此时传递给函数的是一个指针。我们可以通过这个指针的值来修改这个值所指向的地址上的值。

```go
func DoSomething(s string) {
    ...
}

// 使用
s := "hello"
DoSomething(s)
```

```go
func DoSomething(s *string) {
    ...
}

// 使用
s := "hello"
DoSomething(&s)
```

## 传递变长参数
如果函数的最后一个参数采用`...type` 的形式，那个这个函数就可以处理一个变长的参数。这个长度可以为0， 这样的函数成为变参函数
```go
func MyFunc(a, b string, args ...string) {}
```
如果参数被存储在一个slice类型的变量 `slice`中，则可以通过`slice...`的形式来传递参数，调用变参函数。

```go
package main

import "fmt"

func min(s ...int) int {
    ...
}

func main() {
    s := []int{1,2,3,4,5}
    r := min(s...)
    fmt.Println(r)
}
```
