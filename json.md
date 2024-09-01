# encoding/json - json解析 #

数据结构要在网络中传输或保存到文件，就必须对其编码或解码。通过把数据结构转换成JSON格式，就可以在网络上传输或保存到文件中。
任何类型的应用都能够读取和输出，不与操作系统和编程语言的类型相关。

术语说明:
* 数据结构 -> 指定格式 = 序列化 或 编码（传输之前）
* 指定格式 -> 数据结构 = 反序列化 或 解码（传输之后）

序列化是在内存中把数据转换成指定的格式（数据 -> 字符串），反之亦然（字符串 -> 数据）。
编码也是一样，只是输出一个数据流（实现了io.Writer接口的对象），解码是读取一个数据流（实现了io.Reader接口的对象）到数据结构。

## 序列化和反序列化

### 序列化
`func Marshal(v any) ([]byte, error)` // 将一个对象编码成JSON数据，接受一个任何类型的对象，返回[]byte和error：

```go
// json.go
package main

import (
    "encoding/json"
    "fmt"
)

type Address struct {
    Type    string
    City    string
    Country string
}

type VCard struct {
    FirstName string
    LastName  string
    Addresses []*Address
    Remark    string
}

func main() {
    pa := &Address{"private", "Aartselaar", "Belgium"}
    wa := &Address{"work", "Boom", "Belgium"}
    vc := VCard{"Jan", "Kersschot", []*Address{pa, wa}, "none"}

    // JSON format:
    js, _ := json.Marshal(vc)
    fmt.Printf("JSON format: %s", js)
}
```

下面是数据编码后的 JSON 文本（实际上是一个 []byte）：
```json
{
    "FirstName": "Jan",
    "LastName": "Kersschot",
    "Addresses": [{
        "Type": "private",
        "City": "Aartselaar",
        "Country": "Belgium"
    }, {
        "Type": "work",
        "City": "Boom",
        "Country": "Belgium"
    }],
    "Remark": "none"
}
```

### 反序列化
`func Unmarshal(data []byte, v any) error` // 将一个JSON数据解码到相应的数据结构，接受一个[]byte和任何类型的对象，返回error：

>下面例子中`json.Unmarshal(jsonData, &actress)`，解析`[]byte`中的JSON数据并将结果存入指针`&actress`中。

```go
package main

import (
   "encoding/json"
   "fmt"
)

type Opus struct {
   Date string
   Title string
}

// Actress 女演员
type Actress struct {
   Name       string
   Birthday   string
   BirthPlace string
   Opus       []Opus
}

func main() {
   // 因为json.UnMarshal() 函数接收的参数是字节切片
   // 所以需要把JSON字符串转换成字节切片。   
   jsonData := []byte(`{
      "name":"迪丽热巴",
      "birthday":"1992-06-03",
      "birthPlace":"新疆乌鲁木齐市",
      "opus":[
         {
            "date":"2013",
            "title":"《阿娜尔罕》"
         },
         {
            "date":"2014",
            "title":"《逆光之恋》"
         },
         {
            "date":"2015",
            "title":"《克拉恋人》"
         }
      ]
   }`)

   var actress Actress
   err := json.Unmarshal(jsonData, &actress)
   if err != nil {
      fmt.Println("error:", err)
      return
   }
   fmt.Printf("姓名：%s\n", actress.Name)
   fmt.Printf("生日：%s\n", actress.Birthday)
   fmt.Printf("出生地：%s\n", actress.BirthPlace)
   fmt.Println("作品：")
   for _, val := range actress.Opus {
      fmt.Printf("\t%s - %s\n", val.Date, val.Title)
   }
}
```

## 编码和解码流
`json` 包提供 `Decoder` 和 `Encoder` 类型来支持常用 JSON 数据流读写。`NewDecoder()` 和 `NewEncoder()` 函数分别封装了 `io.Reader` 和 `io.Writer` 接口。

要想把 JSON 直接写入文件，可以使用 `json.NewEncoder` 初始化文件（或者任何实现 `io.Writer` 的类型），并调用 `Encode()`；反过来与其对应的是使用 `json.NewDecoder` 和 `Decode()` 函数：

```go
func NewEncoder(w io.Writer) *Encoder
func (enc *Encoder) Encode(v any) error
```

```go
func NewDecoder(r io.Reader) *Decoder
func (dec *Decoder) Decode(v interface{}) error
```

来看下接口是如何对实现进行抽象的：数据结构可以是任何类型，只要其实现了某种接口，目标或源数据要能够被编码就必须实现 `io.Writer` 或 `io.Reader` 接口。由于 Go 语言中到处都实现了 Reader 和 Writer，因此 `Encoder` 和 `Decoder` 可被应用的场景非常广泛，例如读取或写入 HTTP 连接、websockets 或文件。

样例1，将数据结构编码成 JSON 数据，并写入文件：
```go
// json.go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "os"
)

type Address struct {
    Type    string
    City    string
    Country string
}

type VCard struct {
    FirstName string
    LastName  string
    Addresses []*Address
    Remark    string
}

func main() {
    pa := &Address{"private", "Aartselaar", "Belgium"}
    wa := &Address{"work", "Boom", "Belgium"}
    vc := VCard{"Jan", "Kersschot", []*Address{pa, wa}, "none"}

    // using an encoder:
    file, _ := os.OpenFile("vcard.json", os.O_CREATE|os.O_WRONLY, 0666)
    defer file.Close()
  
    enc := json.NewEncoder(file)
    err := enc.Encode(vc)
    if err != nil {
        log.Println("Error in encoding json")
    }
}
```

  样例2， 从JSON文件中读取数据：
  ```go
  package main
  import (
      "encoding/json"
      "log"
      "os"
  )

  type Address struct {
    Type    string
    City    string
    Country string
  }

  type VCard struct {
    FirstName string
    LastName  string
    Addresses []*Address
    Remark    string
  }

  func main() {
      var vc VCard
  
      file, err := os.Open("vc.json")
      if err != nil {
          log.Fatalf("Failed to open file: %+v", err)
      }
      defer file.Close()

      // 根据io.Reader创建Decoder 然后调用Decode()方法将JSON解析成对象
      if err := json.NewDecoder(file).Decode(&vc); err != nil {
          log.Println("Error in decoding json")
      }
      log.Printf("%+v", vc)
  }
  ```
样例3, 处理http响应数据解码的两种方式:

```go
package main
import (
    "encoding/json"
    "io"
    "net/http"
)

func HandleRequest(r *http.Request, v interface{}) (*http.Response, error) {
    client := http.DefaultClient
    resp, err := client.Do(r)
    if err != nil {
        return resp, err
    }

    if v != nil {
        defer resp.Body.Close()

        data, err := io.ReadAll(r.Body) //此处的r是http请求得到的json格式数据-->然后转化为[]byte格式数据.
        if err != nil {
            return resp, err
        }
        if err := json.Unmarshal(data, v); err != nil { //经过这一步将json解码赋值给结构体，由json转化为结构体数据
            return resp, err
        }
    }
    return resp, nil
}
```

```go
package main
import (
  "encoding/json"
  "net/http"
)

func HandleRequest(r *http.Request, v interface{}) (*http.Response, error) {
  client := http.DefaultClient
  resp, err := client.Do(r)
  if err != nil {
    return resp, err
  }

  if v != nil {
    defer resp.Body.Close()
    
    if err := json.NewDecoder(r.Body).Decode(&v); err != nil {
      return resp, err
    }
  }
  return resp, nil
}
```
> 💡 **Tip**
> 
> 1、json.NewDecoder是从一个流里面直接进行解码，代码精干；
> 
> 2、json.Unmarshal是从已存在与内存中的json进行解码；
> 
> 3、相对于解码，json.NewEncoder进行大JSON的编码比json.marshal性能高，因为内部使用pool。