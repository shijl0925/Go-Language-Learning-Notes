# error - 错误处理

## error接口
如果想要自定义项目的错误类型，实现error接口即可。
```go
type error interface {
    Error() string
}
```

样例1：
```go
type ErrorResponse struct {
	Code     int `json:"code"`
	Message  string `json:"message"`
}

func (e *ErrorResponse) Error() string {
    return fmt.Sprintf("Code: %d, Message: %s", e.Code, e.Message)
}
```

样例2:
```go
package sonarqube

import (
	"encoding/json"
	"fmt"
	"net/http"
	"strings"
)

type ErrorResponse struct {
	StatusCode int
	Errors     []struct {
		Msg string `json:"msg"`
	} `json:"errors"`
}

func (e *ErrorResponse) Error() string {
	messages := make([]string, len(e.Errors))
	for i := 0; i < len(e.Errors); i++ {
		messages[i] = e.Errors[i].Msg
	}
	messagesString := strings.Join(messages, ",")

	return fmt.Sprintf("received non 2xx status code (%d): %s", e.StatusCode, messagesString)
}

func ErrorResponseFrom(res *http.Response) (*ErrorResponse, error) {
	errorResponse := &ErrorResponse{}
	err := json.NewDecoder(res.Body).Decode(&errorResponse)
	if err != nil {
		return nil, fmt.Errorf("could not decode response into ErrorResponse: %+v", err)
	}
	errorResponse.StatusCode = res.StatusCode
	return errorResponse, nil
}
```

样例3:
```go
type ErrorResponse struct {
	Response *http.Response
	Message  string
}

func (e *ErrorResponse) Error() string {
	path, _ := url.QueryUnescape(e.Response.Request.URL.Path)
	u := fmt.Sprintf("%s://%s%s", e.Response.Request.URL.Scheme, e.Response.Request.URL.Host, path)
	return fmt.Sprintf("%s %s: %d %s", e.Response.Request.Method, u, e.Response.StatusCode, e.Message)
}

// CheckResponse checks the API response for errors, and returns them if present.
func CheckResponse(r *http.Response) error {
	switch r.StatusCode {
	case 200, 201, 202, 204, 304:
		return nil
	}

	errorResponse := &ErrorResponse{Response: r}
	data, err := io.ReadAll(r.Body)
	if err == nil && data != nil {
		errorResponse.Message = string(data)
	}

	return errorResponse
}
```