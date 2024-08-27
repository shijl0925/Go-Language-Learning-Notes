# bytes - byte slice 便利操作 #

内置库bytes提供了与strings相似的操作，但bytes操作的是字节数组，而不是字符串。其实两者可以相互转换，所以只需记住一个常用的用法即可。

* 字节数组转换为字符串：
  ```go
  func BytesToString(b []byte) string {
    return string(b)
  }
  ```
  或者使用强转换, 性能更好：
  ```go
  func BytesToString(b []byte) string {
    return *(*string)(unsafe.Pointer(&b))
  }
  ```

* 字符串转换为字节数组：
  ```go
  func StringToBytes(s string) []byte {
    return []byte(s)
  }
  ```
  或者使用强转换：
  ```go
  func StringToBytes(s string) []byte {
    return *(*[]byte)(unsafe.Pointer(&s))
  }
  ```
  
除此之外，bytes还有Buffer和Reader两个结构体，用于操作字节数组。
在网络请求过程中经常使用Http.NewRequest来创建新的请求，请求函数的定义如下：
```go
func NewRequest(method, urlStr string, body io.Reader) (*Request, error)
```
其中body的类型是io.Reader，而io.Reader接口定义了Read方法。如下所示：
```go
type Reader interface {
	Read(p []byte) (n int, err error)
}
```
可以看出io.Reader是接口类型，bytes中的Buffer / Reader 和 strings中的Reader对应的结构体实现了Read方法，也就实现了io.Reader接口。

```go
import (
	"bytes"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"
	"strings"
)

func HttpByBytes() {
	url := "http://httpbin.org/anything?name=xix"

	var body map[string]string
	body = make(map[string]string)
	body["age"] = "20"
	body["school"] = "ShangHai"

	by, _ := json.Marshal(body)

	request, _ := http.NewRequest(http.MethodPost, url, bytes.NewBuffer(by))
	client := http.DefaultClient
	response, err := client.Do(request)
	if err != nil {
		panic(err)
	}
	content, _ := ioutil.ReadAll(response.Body)
	fmt.Println(string(content))

}

func HttpByByteNewReader() {
	url := "http://httpbin.org/anything?name=xix"

	var body map[string]string
	body = make(map[string]string)
	body["age"] = "20"
	body["school"] = "ShangHai"

	by, _ := json.Marshal(body)

	request, _ := http.NewRequest(http.MethodPost, url, bytes.NewReader(by))
	client := http.DefaultClient
	response, err := client.Do(request)
	if err != nil {
		panic(err)
	}
	content, _ := ioutil.ReadAll(response.Body)
	fmt.Println(string(content))
}

func HttpByStrings() {
	url := "http://httpbin.org/anything?name=xix"

	request, _ := http.NewRequest(http.MethodPost, url, strings.NewReader(`{"name":"XieWei", "school":"ShangHai"}`))
	client := http.DefaultClient
	response, err := client.Do(request)
	if err != nil {
		panic(err)
	}
	content, _ := ioutil.ReadAll(response.Body)
	fmt.Println(string(content))
}

```
