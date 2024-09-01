# regexp — 正则表达式

正则表达式使用单个字符串来描述、匹配一系列符合某个句法规则的字符串。正则表达式为文本处理提供了强大的功能。Go 作为一门通用语言，自然提供了对正则表达式的支持。

`regexp` 包实现了正则表达式搜索。

正则表达式采用 RE2 语法（除了 \c、\C），和 Perl、Python 等语言的正则基本一致。确切地说是兼容 `RE2` 语法。相关资料：[http://code.google.com/p/re2/wiki/Syntax](http://code.google.com/p/re2/wiki/Syntax)，[包：regexp/syntax](http://docs.studygolang.com/pkg/regexp/syntax/)

注意：`regexp` 包的正则表达式实现保证运行时间随着输入大小线性增长的（即复杂度为 O\(n\)，其中 n 为输入的长度），这一点，很多正则表达式的开源实现无法保证，参见：RSC 的 [《Regular Expression Matching Can Be Simple And Fast   
\(but is slow in Java, Perl, PHP, Python, Ruby, ...\)》](http://swtch.com/~rsc/regexp/regexp1.html)

另外，所有的字符都被视为 utf-8 编码的码值 \(Code Point\)。

## 匹配
如果要判断字符串是否与正则表达式匹配，那么就可以调用`Match_X`函数，所有的匹配规则需要经过`Compile`函数编译之后才能使用，匹配函数只负责确认是否匹配，返回值为布尔类型。
```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	reg, _ := regexp.Compile(`^Hello`)

	if reg.Match([]byte("Hello Regular Expression.")) {
		fmt.Println("byte: Match")
	}

	// Will print 'Match'
	if reg.MatchString("Hello Regular Expression.") {
		fmt.Println("string: Match")
	}
}
```
输出
```
byte: Match
string: Match
```

## 搜索
`Match_X`函数用于判断内容是否匹配。除此之外，正则表达式的另一个常用功能是搜索。
```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	reg, _ := regexp.Compile(`^Hello`)
	Slogan := "Hello, world!"

	v := reg.Find([]byte(Slogan))
	fmt.Println(string(v))  // Hello

	v2 := reg.FindString(Slogan)
	fmt.Println(v2)  // Hello
}
```

Regexp 类型提供了多达 16 个方法，用于匹配正则表达式并获取匹配的结果。它们的名字满足如下正则表达式：

```
Find(All)?(String)?(Submatch)?(Index)?
```

## 替换
另外，可以根据指定的匹配规则，使用正则表达式进行替换操作。
* ReplaceAll 和 ReplaceAllString
```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	reg, _ := regexp.Compile(`^Hello`)
	Slogan := "Hello, world!"

	result0 := reg.ReplaceAll([]byte(Slogan), []byte("Hi"))
	fmt.Println(string(result0))

	result := reg.ReplaceAllString(Slogan, "Hi")
	fmt.Println(result)
}
```

* ReplaceAllFunc 函数处理替换源字串
```go
func (re *Regexp) ReplaceAllFunc(src []byte, repl func([]byte) []byte) []byte
```
初始化实例已经包含了正则，只需要传入：原字串（src）、要替换的字串（repl）。
repl是一个函数，传入值是正则匹配到的字串，传出一个经该函数处理过的值。

```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	reg, _ := regexp.Compile("\\w+$")
	url := "www.google.com"

	result := reg.ReplaceAllFunc([]byte(url), func(match []byte) []byte {
		var data []byte
		data = append(data, match...)
		data = append(data, ".hk"...)
		return data
	})
	fmt.Println(string(result))  // www.google.com.cn
}
```

## 分割
```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	reg, _ := regexp.Compile(`\s|\,|\.`) // 匹配空格、逗号、句号
	url := "www.google.com is Google web site"

	result := reg.Split(url, -1)
	fmt.Println(result)
}
```

## 参考文章
[https://github.com/StefanSchroeder/Golang-Regex-Tutorial](https://github.com/StefanSchroeder/Golang-Regex-Tutorial)