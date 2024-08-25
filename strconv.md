# strconv

与字符串相关的类型转换都是通过`strconv`包实现的。

### 整数类型和字符串类型

* `strconv.Itoa(i int) string` 返回数字 `i`所表示的字符串类型的十进制数。
* `strconv.Atoi(s string) int` 将字符串转换为 `int`类型。
* `strconv.ParseInt(s string, base, bitSize int) (i int, err error) ` 和 `strconv.ParseUint()` 将字符串转换为数字。
* `strconv.FormatInt()` 和 ``strconv.FormatUint()``

### 浮点类型和字符串类型

* `strconv.ParseFloat(s string, bitSize int) (f float64, err error)` 将字符串转换成`float64`型
* `strconv.FormatFloat(f float64, fmt byte, prec int, bitSize int) string` 将`float64`型的数字转换成字符串。其中fmt表示格式（其值可以是'b', 'e', 'f', 或'g'）, prec表示精度，bitSize则使用32表示`float32`，用64表示`float64`

### 布尔类型和字符串类型

* `strconv.ParseBool()` 将字符串的 Bool类型转换成 Bool类型
* `strconv.FormatBool` 将