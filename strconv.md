# strconv

与字符串相关的类型转换都是通过`strconv`包实现的。

### 整数类型和字符串类型

* `strconv.Itoa(i int) string` 返回数字 `i`所表示的字符串类型的十进制数。
* `strconv.Atoi(s string) (int, error)` 将字符串转换为 `int`类型。
* `strconv.ParseInt(s string, base int, bitSize int) (i int64, err error) ` 将字符串转换为数字。
  * s 表示要转换的字符串
  * base 表示转化后所需要的进制 比如二进制 八进制 十进制 十六进制
  * bitSize 表示返回结果的bit大小 也就是int8 int16 int32 int64

* `strconv.FormatInt(i int64, base int) string` 将数字转换为字符串。其中bitSize表示转换后的字符串所占用的位数。
  * i 表示要转换的数字
  * base 表示转换后的字符串的进制

### 浮点类型和字符串类型

* `strconv.ParseFloat(s string, bitSize int) (float64, error)` 将字符串转换成`float64`型
* `strconv.FormatFloat(f float64, fmt byte, prec, bitSize int) string` 将`float64`型的数字转换成字符串。其中fmt表示格式（其值可以是'b', 'e', 'f', 或'g'）, prec表示精度，bitSize则使用32表示`float32`，用64表示`float64`

### 布尔类型和字符串类型

* `strconv.ParseBool(str string) (bool, error)` 将字符串的 Bool类型转换成 Bool类型
* `strconv.FormatBool(b bool) string` 将Bool类型转换成字符串类型