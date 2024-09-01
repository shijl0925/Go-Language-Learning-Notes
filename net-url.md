# net/url

## 路由
路由： URL(统一资源定位符)，唯一定位服务器上的资源。

URL具体的语法组成如下：
* schema: 使用哪种协议，如http、ftp、file等。
* userinfo: 用户名、密码。
* host: 服务器地址，如www.baidu.com。
* port: 服务器端口，如80。
* path: 资源路径，如/index.html。
* query: 查询参数，如name=zhangsan。使用"&"连接多个键值对
* fragment: 锚点，如#top。

## 源码中关于URL的结构体定义：
```go
type URL struct {
	Scheme      string
	Opaque      string    // encoded opaque data
	User        *Userinfo // username and password information
	Host        string    // host or host:port
	Path        string    // path (relative paths may omit leading slash)
	RawPath     string    // encoded path hint (see EscapedPath method)
	OmitHost    bool      // do not emit empty host (authority)
	ForceQuery  bool      // append a query ('?') even if RawQuery is empty
	RawQuery    string    // encoded query values, without '?'
	Fragment    string    // fragment for references, without '#'
	RawFragment string    // encoded fragment hint (see EscapedFragment method)
}
```

## 实例
### 将字符串url转化成URL类型
```go
package main

import (
	"fmt"
	"net/url"
)

func main() {
	var urlString = "https://sonarcloud.io/api/issues/search?s=FILE_LINE&impactSeverities=HIGH&issueStatuses=OPEN%2CCONFIRMED&ps=100&facets=cleanCodeAttributeCategories%2CimpactSoftwareQualities%2CimpactSeverities&componentKeys=shijl0925_go-gerrit&organization=shijl0925&additionalFields=_all"
	urlParsed, _ := url.Parse(urlString)
	fmt.Printf("%#v\n", urlParsed)
}
```

输出：
```
&url.URL{Scheme:"https", Opaque:"", User:(*url.Userinfo)(nil), Host:"sonarcloud.io", Path:"/api/issues/search", RawPath:"", OmitHost:false, F
orceQuery:false, RawQuery:"s=FILE_LINE&impactSeverities=HIGH&issueStatuses=OPEN%2CCONFIRMED&ps=100&facets=cleanCodeAttributeCategories%2Cimpa
ctSoftwareQualities%2CimpactSeverities&componentKeys=shijl0925_go-gerrit&organization=shijl0925&additionalFields=_all", Fragment:"", RawFragm
ent:""}
```

# 获取请求参数并改变
```go
package main

import (
	"fmt"
	"net/url"
)

func main() {
	var urlString = "https://sonarcloud.io/api/issues/search?s=FILE_LINE&impactSeverities=HIGH&issueStatuses=OPEN%2CCONFIRMED&ps=100&facets=cleanCodeAttributeCategories%2CimpactSoftwareQualities%2CimpactSeverities&componentKeys=shijl0925_go-gerrit&organization=shijl0925&additionalFields=_all"
	urlParsed, _ := url.Parse(urlString)

    // 将查询参数键值对转化成Values类型，即map类型
	v := urlParsed.Query()
	fmt.Println(v) // map[additionalFields:[_all] componentKeys:[shijl0925_go-gerrit] facets:[cleanCodeAttributeCategories,impactSoftwareQualities,impactSeverities] impactSeverities:[HIGH] issueStatuses:[OPEN,CONFIRMED] organization:[shijl0925] ps:[100] s:[FILE_LINE]]

	fmt.Println(v.Get("additionalFields")) // _all

	v.Del("impactSeverities")
	v.Add("TYPES", "CODE_SMELL")
	v.Set("componentKeys", "shijl0925_go-sonarqube")

	urlParsed.RawQuery = v.Encode()
	fmt.Println(urlParsed.String()) // https://sonarcloud.io/api/issues/search?TYPES=CODE_SMELL&additionalFields=_all&componentKeys=shijl0925_go-sonarqube&facets=cleanCodeAttributeCategories%2CimpactSoftwareQualities%2CimpactSeverities&issueStatuses=OPEN%2CCONFIRMED&organization=shijl0925&ps=100&s=FILE_LINE
}
```


URL核心操作如下：
(1) 将字符串转化为URL类型
(2) 对请求参数的操作


**参考样例**
```go
// addOptions takes a URL string and options, returning a new URL string with the options added.
// The purpose of this function is to allow users to dynamically add parameters to a URL, facilitating HTTP request operations.
// The parameter 's' is a string representing the original URL.
// The parameter 'opt' is an interface representing the additional parameters to be added, which can be a struct or map, etc.
// If 'opt' is a pointer and nil, the function will directly return the original URL string without modification.
// The return value is a string representing the new URL after adding the parameters, and an error if any occurs.
func addOptions(s string, opt interface{}) (string, error) {
	v := reflect.ValueOf(opt)

	if v.Kind() == reflect.Ptr && v.IsNil() {
		return s, nil
	}

	urlParsed, err := url.Parse(s)
	if err != nil {
		return s, err
	}

	origValues := urlParsed.Query()

	newValues, err := query.Values(opt)
	if err != nil {
		return s, err
	}

	for k, v := range newValues {
		origValues[k] = v
	}

	urlParsed.RawQuery = origValues.Encode()

	return urlParsed.String(), nil
}
```