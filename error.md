# error

如果想要自定义项目的错误类型，实现error接口即可。
```go
type error interface {
    Error() string
}
```

样例：
```go
type ErrorResponse struct {
	Code     int `json:"code"`
	Message  string `json:"message"`
}

func (e *ErrorResponse) Error() string {
    return fmt.Sprintf("Code: %d, Message: %s", e.Code, e.Message)
}
```