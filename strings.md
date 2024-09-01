# strings - 字符串操作 #

## 前缀和后缀

  * `HasPrefix()` 判断字符串`s`是否以`prefix`开头：

    ```go
    func HasPrefix(s, prefix string) bool
    ```

  * `HasSuffix()`判断字符串`s`是否以`suffix`结尾：

    ```go
    func HasSuffix(s, suffix string) bool
    ```

## 字符串包含关系

  * `Contains()`判断字符串`s`是否包含`substr`

    ```go
    func Contains(s, substr string) bool
    ```
  * `ContainsAny()` chars中任何一个Unicode代码点在s中，返回true
    ```go
    func ContainsAny(s, chars string) bool
    ```
  * `ContainsRune()` Unicode代码点r在s中，返回true
    ```go
    func ContainsRune(s string, r rune) bool
    ```

## 判断子字符串或字符在父字符串中出现的位置（索引）

* `Index()`返回字符串`str`在字符串`s`中的索引（`str`的第一个字符的索引），-1表示字符串`s`不包含字符串`str`:

  ```go
  func Index(s, substr string) int
  ```

* `LastIndex()`返回字符串`str`在字符串`s`中最后出现的位置的索引（`str`的第一个字符的索引），-1表示字符串`s`不包含字符串`str`:

  ```go
  func LastIndex(s, substr string) int
  ```

* `IndexRune` 查询非ASCII编码的字符在父字符串中的位置

  ```go
  func IndexRune(s string, r rune) int
  ```

## 字符串替换

* `Replace()`用于将字符串`str`中前`n`个字符串`old`替换为字符串`new`，并返回一个新的字符串。
  如果`n = -1`则替换所有的字符串`old`为字符串`new`，这种情况同样可以使用`ReplaceAll()`

  ```go
  func Replace(s, old, new string, n int) string
  ```

## 统计字符串出现的次数

* `Count()`用于计算字符串`str`在字符串`s`中出现的非重叠次数：

  ```go
  func Count(s, substr string) int
  ```

  > 💡 **Tip**
  > 
  > 这里要特 别说明一下的是当 substr 为空时，Count 的返回值是:utf8.RuneCountInString(s) + 1

## 重复字符串

* `Repeat()`用于重复`ount`次字符串`s`并返回一个新的字符串：

  ```go
  func Repeat(s string, count int) string
  ```

## 修改字符串大小写

* `ToLower()`将字符串中的Unicode字符全部转化为相应的小写字符：

  ```go
  func ToLower(s string) string
  ```

* `ToUpper()`将字符串中的Unicode字符全部转化为相应的大写字符：

  ```go
  func ToUpper(s string) string
  ```

## 修剪字符串

* `func TrimSpace(s string) string`剔除字符串中开头和结尾的空白符。
* `func Trim(s, cutset string) string`将`字符串`s`中开头和结尾的`cutset`去除掉。第二个参数可以包含任何字符。
* 如果只想剔除开头或者结尾的字符串，使用`TrimLeft` 或者`TrimRight` 来实现。
* `func TrimSuffix(s, suffix string) string` 用于删除字符串末尾的指定字符串。 `func TrimPrefix(s, prefix string) string` 用于删除字符串开头的指定字符串。

> 💡 **Tip**:
>
> `func TrimPrefix(s, prefix string) string` 函数很简单，就是删掉一样的前缀。
> 
> `func TrimLeft(s, cutset string) string` 函数不仅仅删除一样的前缀，若剩下的字符串中有跟前缀一样的字符，也会删掉。
> 
> `func TrimRight(s, cutset string) string` 函数和 `TrimLeft()` 函数类似，只是删除的是一样的后缀。

  ```go
  package main
  import (
      "fmt"
      "strings"
  )
  func main() {
      fmt.Println("删除前缀字符串 TrimPrefix()函数 测试")
      src := "xxabcdefg"
   
      // 由于src的开头和pre1一致，所以返回删除pre1后的内容，即"defg"
      pre1 := "xxabc"
      result1 := strings.TrimPrefix(src, pre1)
      fmt.Printf("删除前：%v, 前缀：%v, 删除后：%v \n", src, pre1, result1)
   
      // 由于src的开头和pre2不一致，所以不做修改，直接返回原来的内容
      pre2 := "abc"
      result2 := strings.TrimPrefix(src, pre2)
      fmt.Printf("删除前：%v, 前缀：%v, 删除后：%v \n\n", src, pre2, result2)
   
      fmt.Println("删除前缀，且删除剩下内容中的左侧和前缀中一样的字符 TrimLeft()函数 测试")
   
      // 由于srcstr的开头和pre3的一致，第一步：删除pre3，即"mysql"，剩下"myour"; 第二步：剩下的"myour"中，左侧开头的"my"和pre3开头一样所以也删除，剩下"our"
      srcstr := "mysqlmyour"
      pre3 := "mysql"
      result3 := strings.TrimLeft(srcstr, pre3)
      fmt.Printf("删除前：%v, 前缀：%v, 删除后：%v \n", srcstr, pre3, result3)
   
      // 由于srcstr2的开头和pre4的一致，第一步：删除pre4，即"mysql"，剩下"ourmy"; 第二步：剩下的"ourmy"，左侧开头的内容和pre4开头不一样，所以不做操作，直接返回
      srcstr2 := "mysqlourmy"
      pre4 := "mysql"
      result4 := strings.TrimLeft(srcstr2, pre4)
      fmt.Printf("删除前：%v, 前缀：%v, 删除后：%v \n", srcstr2, pre4, result4)
  }
  ```

## 分割字符串

* `func Fields(s string) []string` 将会利用1个或多个空白符来作为动态长度的分隔符将字符串分割成若干个小块，并返回一个 slice。如果 字符串 s 只包含空格，则返回空列表([]string的⻓度为0)

  ```go
  str := "The quick brown fox jumps over the lazy dog"
  sl := strings.Fields(str)
  
  for _, val := range(sl) {
    fmt.Printf("%s - ", val)
  }
  ```

* `func FieldsFunc(s string, f func(rune) bool) []string`
  FieldsFunc 用这样的 Unicode 代码点 c 进行分隔：满足 f(c) 返回 true。该函数返回[]string。如果字符串 s 中所有的代码点 (unicode code points) 都满足 f(c) 或者 s 是空，则 FieldsFunc 返回空 slice。
  
  也就是说，我们可以通过实现一个回调函数来指定分隔字符串 s 的字符。比如上面的例子，我们通过 FieldsFunc 来实现：
  ```go
  fmt.Println(strings.FieldsFunc("  foo bar  baz   ", unicode.IsSpace))
  ```
  
  实际上，Fields 函数就是调用 FieldsFunc 实现的：
  ```go
  func Fields(s string) []string {
    return FieldsFunc(s, unicode.IsSpace)
  }
  ```


* `func Split(s, sep string) []string`用于使用自定义分割符号来对指定字符串进行分割，同样返回slice。

  ```go
  str := "GO1|GO2|GO3"
  sl := strings.Split(str, "|")
  for _, val := range(sl) {
    fmt.Printf("%s - ", val)
  }
  ```

## 拼接slice到字符串

* `Join()`用于将元素类型为string的 slice 使用分隔符号来拼接组成一个字符串。

  ```go
  func Join(elems []string, sep string) string
  ```

  ```go
  sl = []string{"a", "b", "c", "d"}
  str := strings.Join(sl, ";")
  fmt.Printf("sl joined by ;: %s\n", str)
  ```

## 从字符串中读取内容

* `NewReader(str)`用于生成一个`Reader`并读取字符串中的内容，然后返回指向该`Reader`的指针。

* `Read()`从`[]byte`中读取内容。

  ```go
  // Len(): 返回未读的字符串长度
  // Size():返回字符串的长度
  // Read(): 读取字符串信息

  r := strings.NewReader("abcdefghijklmn")
  fmt.Println(r.Len())   // 输出14  初始时，未读长度等于字符串长度

  var buf []byte
  buf = make([]byte, 5)
  readLen, err := r.Read(buf)
  fmt.Println("读取到的长度:", readLen) //读取到的长度5
  if err != nil {
  	fmt.Println("错误:", err)
  }

  fmt.Println(buf)            //abcde
  fmt.Println(r.Len())        //9   读取到了5个 剩余未读是14-5
  fmt.Println(r.Size())       //14   字符串的长度
  ```

* `ReadAt()` 读取偏移off字节后的剩余信息到b中（需要注意的是，ReadAt函数不会影响Len的数值，和Read的数值，off不能为负数，不能大于size（）的长度）

  ```go
  func (r *Reader) ReadAt(b []byte, off int64) (n int, err error)
  ```

  ```go
  r := strings.NewReader("abcdefghijklmn")
   
  var bufAt []byte
  bufAt = make([]byte, 256)
  r.ReadAt(bufAt, 5)
  fmt.Println(string(bufAt))              //fghijklmn
  ```


* `ReadByte()`和`ReadRune()`从字符串中读取下一个`byte`或者`rune`。

  ```go
  r := strings.NewReader("abcdefghijklmn")
  	
  //读取一个字节
  b, _ := r.ReadByte()
  fmt.Println(string(b))					// a
  //int(r.Size()) - r.Len() 已读取字节数
  fmt.Println(int(r.Size()) - r.Len())	// 1
   
  //读取一个字节
  b, _ = r.ReadByte()
  fmt.Println(string(b))					// b
  fmt.Println(int(r.Size()) - r.Len())	// 2
   
  //回退一个字节
  r.UnreadByte()
  fmt.Println(int(r.Size()) - r.Len())	// 1
   
  //读取一个字节
  b, _ = r.ReadByte()
  fmt.Println(string(b))					// b
  ```

* `Seek()` 前面有ReadAt方法可以将字符串偏移多少位读取剩下的字符串内容，但是该方法不会影响正在用Read方法读取的内容，如果相对Read方法读取的内容做偏移就可以使用seek方法， offset是偏移的位置，whence是偏移起始位置，支持三种位置（io.SeekStart起始位，io.SeekCurrent当前位，io.SeekEnd末位）。需要注意的是offset可以未负数，当时偏移起始位 与offset相加得到的值不能小于0或者大于size()的长度

  `func (r *Reader) Seek(offset int64, whence int) (int64, error)`
  
  ```go
  r := strings.NewReader("abcdefghijklmn")
   
  var buf []byte
  buf = make([]byte, 5)
  r.Read(buf)
  fmt.Println(string(buf)) //adcde
   
  buf = make([]byte, 5)
  r.Seek(-2, io.SeekCurrent) //从当前位置向前偏移两位 （5-2)
  r.Read(buf)
  fmt.Println(string(buf)) //defgh
  	
  buf = make([]byte, 5)
  r.Seek(-3, io.SeekEnd) //设置当前位置是末尾前移三位
  r.Read(buf)
  fmt.Println(string(buf)) //lmn
   
  buf = make([]byte, 5)
  r.Seek(3, io.SeekStart) //设置当前位置是起始位后移三位
  r.Read(buf)
  fmt.Println(string(buf)) //defgh
  ```

## 其他

* `Clone()`
  ```go
  // Clone returns a copy of the string s.
  func Clone(s string) string {
    if len(s) == 0 {
      return ""
    }
    b := make([]byte, len(s))
    copy(b, s)
    return unsafe.String(&b[0], len(b))
  }
  ```
  * 通过 copy 函数对原始字符串进行复制，得到一份新的 []byte 数据。
  * 通过 *(*string)(unsafe.Pointer(&b)) 进行指针操作，实现 byte 到 string 的零内存复制转换。

* `Cut()`
  ```go
  // Cut slices s around the first instance of sep,
  // returning the text before and after sep.
  // The found result reports whether sep appears in s.
  // If sep does not appear in s, cut returns s, "", false.
  func Cut(s, sep string) (before, after string, found bool)
  ```
  将字符串 s 在第一个 sep 处切割成两部分，返回分割值 before 和 after。如果 s 中存在 sep，则返回 before，after，true，
  如果 s 中没有 sep，则返回 s,"",false。

* `Map()` 将输入字符串中的每个字符使用函数处理后映射后返回一份字符串的副本，如果函数中的某个字符返回负数则删除对应的字符。
  ```go
  func Map(mapping func(rune) rune, s string) string
  ```
  ```go
  package main
  
  import (
    "fmt";
    "strings"
  )
  
  func main() {
    modified := func(r rune) rune {
      if r == 'e' {
        return '@'
      }
      return r
    }
  
    input:= "Hello, Welcome to GeeksforGeeks"
    result := strings.Map(modified, input)
    fmt.Println(result)  // 结果是 H@llo, W@lcom@ to G@@ksforG@@ks
  } 
  ```
