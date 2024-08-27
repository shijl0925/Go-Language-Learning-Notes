# strconv

与字符串相关的类型转换都是通过`strconv`包实现的。

### 整数类型和字符串类型

* `func Itoa(i int) string` 返回数字 `i`所表示的字符串类型的十进制数。
   Itoa 内部直接调用 FormatInt(i, 10) 实现的。

* `func Atoi(s string) (int, error)` 将字符串转换为 `int`类型。
   Atoi 是 ParseInt 的便捷版，内部通过调用 ParseInt(s, 10, 0) 来实现的

* `func ParseInt(s string, base int, bitSize int) (i int64, err error)` 将字符串转换为数字。
  * s 表示要转换的字符串
  * base 代表字符串按照给定的进制进行解释。一般的，base 的取值为 2~36，如 果 base 的值为 0，则会根据字符串的前缀来确定 base 的值:"0x" 表示 16 进制; "0" 表示 8 进制;否则就是 10 进制。
  * bitSize 表示的是整数取值范围，或者说整数的具体类型。取值 0、8、16、32 和 64 分别代表 int、int8、int16、int32 和 int64。

* `func FormatInt(i int64, base int) string` 将数字转换为字符串。
  * i 表示要转换的数字
  * base 表示转换后的字符串的进制

### 浮点类型和字符串类型

* `func ParseFloat(s string, bitSize int) (float64, error)` 将字符串转换成`float64`型
* `func FormatFloat(f float64, fmt byte, prec, bitSize int) string` 将`float64`型的数字转换成字符串。其中fmt表示格式（其值可以是'b', 'e', 'f', 或'g'）, prec表示精度，bitSize则使用32表示`float32`，用64表示`float64`

### 布尔类型和字符串类型

* `func ParseBool(str string) (bool, error)` 将字符串的 Bool类型转换成 Bool类型
* `func FormatBool(b bool) string` 将Bool类型转换成字符串类型
