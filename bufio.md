# bufio — 缓存IO #

bufio 包实现了缓存IO。它包装了 io.Reader 和 io.Writer 对象，创建了另外的Reader和Writer对象，它们也实现了 io.Reader 和 io.Writer 接口，不过它们是有缓存的。该包同时为文本I/O提供了一些便利操作。

## 1.4.1 Reader 类型和方法 ##

bufio.Reader 结构包装了一个 io.Reader 对象，提供缓存功能，同时实现了 io.Reader 接口。

Reader 结构没有任何导出的字段，结构定义如下：
```go
	type Reader struct {
		buf          []byte		// 缓存
		rd           io.Reader	// 底层的io.Reader
		// r:从buf中读走的字节（偏移）；w:buf中填充内容的偏移；
		// w - r 是buf中可被读的长度（缓存数据的大小），也是Buffered()方法的返回值
		r, w         int
		err          error		// 读过程中遇到的错误
		lastByte     int		// 最后一次读到的字节（ReadByte/UnreadByte)
		lastRuneSize int		// 最后一次读到的Rune的大小 (ReadRune/UnreadRune)
	}
```

### 1.4.1.1 实例化 ###

bufio 包提供了两个实例化 bufio.Reader 对象的函数：NewReader 和 NewReaderSize。其中，NewReader 函数是调用 NewReaderSize 函数实现的：
```go
	func NewReader(rd io.Reader) *Reader {
		// 默认缓存大小：defaultBufSize=4096
		return NewReaderSize(rd, defaultBufSize)
	}
```
我们看一下NewReaderSize的源码：
```go
	func NewReaderSize(rd io.Reader, size int) *Reader {
		// 已经是bufio.Reader类型，且缓存大小不小于 size，则直接返回
		b, ok := rd.(*Reader)
		if ok && len(b.buf) >= size {
			return b
		}
		// 缓存大小不会小于 minReadBufferSize （16字节）
		if size < minReadBufferSize {
			size = minReadBufferSize
		}
		// 构造一个bufio.Reader实例
		return &Reader{
			buf:          make([]byte, size),
			rd:           rd,
			lastByte:     -1,
			lastRuneSize: -1,
		}
	}
```
### 1.4.1.2 ReadSlice、ReadBytes、ReadString 和 ReadLine 方法 ###

之所以将这几个方法放在一起，是因为他们有着类似的行为。事实上，后三个方法最终都是调用ReadSlice来实现的。所以，我们先来看看ReadSlice方法。(感觉这一段直接看源码较好)

**ReadSlice方法签名**如下：
```go
	func (b *Reader) ReadSlice(delim byte) (line []byte, err error)
```
ReadSlice 从输入中读取，直到遇到第一个界定符（delim）为止，返回一个指向缓存中字节的 slice，在下次调用读操作（read）时，这些字节会无效。举例说明：
```go
	reader := bufio.NewReader(strings.NewReader("http://studygolang.com. \nIt is the home of gophers"))
	line, _ := reader.ReadSlice('\n')
	fmt.Printf("the line:%s\n", line)
	// 这里可以换上任意的 bufio 的 Read/Write 操作
	n, _ := reader.ReadSlice('\n')
	fmt.Printf("the line:%s\n", line)
	fmt.Println(string(n))
```
输出：
```bash
	the line:http://studygolang.com. 
	
	the line:It is the home of gophers
	It is the home of gophers
```
从结果可以看出，第一次ReadSlice的结果（line），在第二次调用读操作后，内容发生了变化。也就是说，ReadSlice 返回的 []byte 是指向 Reader 中的 buffer ，而不是 copy 一份返回。正因为ReadSlice 返回的数据会被下次的 I/O 操作重写，因此许多的客户端会选择使用 ReadBytes 或者 ReadString 来代替。读者可以将上面代码中的 ReadSlice 改为 ReadBytes 或 ReadString ，看看结果有什么不同。

注意，这里的界定符可以是任意的字符，可以将上面代码中的'\n'改为'm'试试。同时，返回的结果是包含界定符本身的，上例中，输出结果有一空行就是'\n'本身(line携带一个'\n',printf又追加了一个'\n')。

如果 ReadSlice 在找到界定符之前遇到了 error ，它就会返回缓存中所有的数据和错误本身（经常是 io.EOF）。如果在找到界定符之前缓存已经满了，ReadSlice 会返回 bufio.ErrBufferFull 错误。当且仅当返回的结果（line）没有以界定符结束的时候，ReadSlice 返回err != nil，也就是说，如果ReadSlice 返回的结果 line 不是以界定符 delim 结尾，那么返回的 er r也一定不等于 nil（可能是bufio.ErrBufferFull或io.EOF）。
例子代码：
```go
	reader := bufio.NewReaderSize(strings.NewReader("http://studygolang.com"),16)
	line, err := reader.ReadSlice('\n')
	fmt.Printf("line:%s\terror:%s\n", line, err)
	line, err = reader.ReadSlice('\n')
	fmt.Printf("line:%s\terror:%s\n", line, err)
```
输出：
```bash
	line:http://studygola	error:bufio: buffer full
	line:ng.com	error:EOF
```
**ReadBytes方法签名**如下：
```go
	func (b *Reader) ReadBytes(delim byte) (line []byte, err error)
```
该方法的参数和返回值类型与 ReadSlice 都一样。 ReadBytes 从输入中读取直到遇到界定符（delim）为止，返回的 slice 包含了从当前到界定符的内容 **（包括界定符）**。如果 ReadBytes 在遇到界定符之前就捕获到一个错误，它会返回遇到错误之前已经读取的数据，和这个捕获到的错误（经常是 io.EOF）。跟 ReadSlice 一样，如果 ReadBytes 返回的结果 line 不是以界定符 delim 结尾，那么返回的 err 也一定不等于 nil（可能是bufio.ErrBufferFull 或 io.EOF）。

从这个说明可以看出，ReadBytes和ReadSlice功能和用法都很像，那他们有什么不同呢？

在讲解ReadSlice时说到，它返回的 []byte 是指向 Reader 中的 buffer，而不是 copy 一份返回，也正因为如此，通常我们会使用 ReadBytes 或 ReadString。很显然，ReadBytes 返回的 []byte 不会是指向 Reader 中的 buffer，通过[查看源码](http://docscn.studygolang.com/src/bufio/bufio.go?s=10277:10340#L338)可以证实这一点。

还是上面的例子，我们将 ReadSlice 改为 ReadBytes：
```go
	reader := bufio.NewReader(strings.NewReader("http://studygolang.com. \nIt is the home of gophers"))
	line, _ := reader.ReadBytes('\n')
	fmt.Printf("the line:%s\n", line)
	// 这里可以换上任意的 bufio 的 Read/Write 操作
	n, _ := reader.ReadBytes('\n')
	fmt.Printf("the line:%s\n", line)
	fmt.Println(string(n))
```
输出：
```bash
	the line:http://studygolang.com. 

	the line:http://studygolang.com. 
	
	It is the home of gophers
```
**ReadString方法**

看一下该方法的源码：
```go
	func (b *Reader) ReadString(delim byte) (line string, err error) {
		bytes, err := b.ReadBytes(delim)
		return string(bytes), err
	}
```
它调用了 ReadBytes 方法，并将结果的 []byte 转为 string 类型。

**ReadLine方法签名**如下
```go
	func (b *Reader) ReadLine() (line []byte, isPrefix bool, err error)
```
ReadLine 是一个底层的原始行读取命令。许多调用者或许会使用 ReadBytes('\n') 或者 ReadString('\n') 来代替这个方法。

ReadLine 尝试返回单独的行，不包括行尾的换行符。如果一行大于缓存，isPrefix 会被设置为 true，同时返回该行的开始部分（等于缓存大小的部分）。该行剩余的部分就会在下次调用的时候返回。当下次调用返回该行剩余部分时，isPrefix 将会是 false 。跟 ReadSlice 一样，返回的 line 只是 buffer 的引用，在下次执行IO操作时，line 会无效。可以将 ReadSlice 中的例子该为 ReadLine 试试。

注意，返回值中，要么 line 不是 nil，要么 err 非 nil，两者不会同时非 nil。

ReadLine 返回的文本不会包含行结尾（"\r\n"或者"\n"）。如果输入中没有行尾标识符，不会返回任何指示或者错误。

从上面的讲解中，我们知道，读取一行，通常会选择 ReadBytes 或 ReadString。不过，正常人的思维，应该用 ReadLine，只是不明白为啥 ReadLine 的实现不是通过 ReadBytes，然后清除掉行尾的\\n（或\r\n），它现在的实现，用不好会出现意想不到的问题，比如丢数据。个人建议可以这么实现读取一行：
```go
	line, err := reader.ReadBytes('\n')
	line = bytes.TrimRight(line, "\r\n")
```
这样既读取了一行，也去掉了行尾结束符（当然，如果你希望留下行尾结束符，只用ReadBytes即可）。

### 1.4.1.3 Peek 方法 ###

从方法的名称可以猜到，该方法只是“窥探”一下 Reader 中没有读取的 n 个字节。好比栈数据结构中的取栈顶元素，但不出栈。

方法的签名如下：
```go
	func (b *Reader) Peek(n int) ([]byte, error)
```
同上面介绍的 ReadSlice一样，返回的 []byte 只是 buffer 中的引用，在下次IO操作后会无效，可见该方法（以及ReadSlice这样的，返回buffer引用的方法）对多 goroutine 是不安全的，也就是在多并发环境下，不能依赖其结果。

我们通过例子来证明一下：
```go
	package main

	import (
	    "bufio"
	    "fmt"
	    "strings"
	    "time"
	)
	
	func main() {
	    reader := bufio.NewReaderSize(strings.NewReader("http://studygolang.com.\t It is the home of gophers"), 14)
	    go Peek(reader)
	    go reader.ReadBytes('\t')
	    time.Sleep(1e8)
	}
	
	func Peek(reader *bufio.Reader) {
	    line, _ := reader.Peek(14)
	    fmt.Printf("%s\n", line)
	    // time.Sleep(1)
	    fmt.Printf("%s\n", line)
	}
```
输出：
```bash
	http://studygo
	http://studygo
```
输出结果和预期的一致。然而，这是由于目前的 goroutine 调度方式导致的结果。如果我们将例子中注释掉的 time.Sleep(1) 取消注释（这样调度其他 goroutine 执行），再次运行，得到的结果为：
```bash
	http://studygo
	ng.com.	 It is
```
另外，Reader 的 Peek 方法如果返回的 []byte 长度小于 n，这时返回的 `err != nil` ，用于解释为啥会小于 n。如果 n 大于 reader 的 buffer 长度，err 会是 ErrBufferFull。

### 1.4.1.4 其他方法 ###

Reader 的其他方法都是实现了 io 包中的接口，它们的使用方法在io包中都有介绍，在此不赘述。

这些方法包括：
```go
	func (b *Reader) Read(p []byte) (n int, err error)
	func (b *Reader) ReadByte() (c byte, err error)
	func (b *Reader) ReadRune() (r rune, size int, err error)
	func (b *Reader) UnreadByte() error
	func (b *Reader) UnreadRune() error
	func (b *Reader) WriteTo(w io.Writer) (n int64, err error)
```
你应该知道它们都是哪个接口的方法吧。

## 1.4.2 Scanner 类型和方法 ##

对于简单的读取一行，在 Reader 类型中，感觉没有让人特别满意的方法。于是，Go1.1增加了一个类型：Scanner。官方关于**Go1.1**增加该类型的说明如下：

> 在 bufio 包中有多种方式获取文本输入，ReadBytes、ReadString 和独特的 ReadLine，对于简单的目的这些都有些过于复杂了。在 Go 1.1 中，添加了一个新类型，Scanner，以便更容易的处理如按行读取输入序列或空格分隔单词等，这类简单的任务。它终结了如输入一个很长的有问题的行这样的输入错误，并且提供了简单的默认行为：基于行的输入，每行都剔除分隔标识。这里的代码展示一次输入一行：
```go
	scanner := bufio.NewScanner(os.Stdin)
	for scanner.Scan() {
	    fmt.Println(scanner.Text()) // Println will add back the final '\n'
	}
	if err := scanner.Err(); err != nil {
	    fmt.Fprintln(os.Stderr, "reading standard input:", err)
	}
```
> 输入的行为可以通过一个函数控制，来控制输入的每个部分（参阅 SplitFunc 的文档），但是对于复杂的问题或持续传递错误的，可能还是需要原有接口。

Scanner 类型和 Reader 类型一样，没有任何导出的字段，同时它也包装了一个 io.Reader 对象，但它没有实现 io.Reader 接口。

Scanner 的结构定义如下：

	type Scanner struct {
		r            io.Reader // The reader provided by the client.
		split        SplitFunc // The function to split the tokens.
		maxTokenSize int       // Maximum size of a token; modified by tests.
		token        []byte    // Last token returned by split.
		buf          []byte    // Buffer used as argument to split.
		start        int       // First non-processed byte in buf.
		end          int       // End of data in buf.
		err          error     // Sticky error.
	}

这里 split、maxTokenSize 和 token 需要讲解一下。

然而，在讲解之前，需要先讲解 split 字段的类型 SplitFunc。

### 1.4.2.1 SplitFunc 类型和实例 ###

**SplitFunc 类型定义**如下：
```go
	type SplitFunc func(data []byte, atEOF bool) (advance int, token []byte, err error)
```
SplitFunc 定义了 用于对输入进行分词的 split 函数的签名。参数 data 是还未处理的数据，atEOF 标识 Reader 是否还有更多数据（是否到了EOF）。返回值 advance 表示从输入中读取的字节数，token 表示下一个结果数据，err 则代表可能的错误。

举例说明一下这里的 token 代表的意思：

	有数据 "studygolang\tpolaris\tgolangchina"，通过"\t"进行分词，那么会得到三个token，它们的内容分别是：studygolang、polaris 和 golangchina。而 SplitFunc 的功能是：进行分词，并返回未处理的数据中第一个 token。对于这个数据，就是返回 studygolang。

如果 data 中没有一个完整的 token，例如，在扫描行（scanning lines）时没有换行符，SplitFunc 会返回(0,nil,nil)通知 Scanner 读取更多数据到 slice 中，然后在这个更大的 slice 中同样的读取点处，从输入中重试读取。如下面要讲解的 split 函数的源码中有这样的代码：
```go
	// Request more data.
	return 0, nil, nil
```
如果 `err != nil`，扫描停止，同时该错误会返回。

如果参数 data 为空的 slice，除非 atEOF 为 true，否则该函数永远不会被调用。如果 atEOF 为 true，这时 data 可以非空，这时的数据是没有处理的。

**bufio 包定义的 split 函数，即 SplitFunc 的实例**

在 bufio 包中预定义了一些 split 函数，也就是说，在 Scanner 结构中的 split 字段，可以通过这些预定义的 split 赋值，同时 Scanner 类型的 Split 方法也可以接收这些预定义函数作为参数。所以，我们可以说，这些预定义 split 函数都是 SplitFunc 类型的实例。这些函数包括：ScanBytes、ScanRunes、ScanWords 和 ScanLines。（由于都是 SplitFunc 的实例，自然这些函数的签名都和 SplitFunc 一样）

**ScanBytes** 返回单个字节作为一个 token。

**ScanRunes** 返回单个 UTF-8 编码的 rune 作为一个 token。返回的 rune 序列（token）和 range string类型 返回的序列是等价的，也就是说，对于无效的 UTF-8 编码会解释为 U+FFFD = "\xef\xbf\xbd"。

**ScanWords** 返回通过“空格”分词的单词。如：study golang，调用会返回study。注意，这里的“空格”是 `unicode.IsSpace()`，即包括：'\t', '\n', '\v', '\f', '\r', ' ', U+0085 (NEL), U+00A0 (NBSP)。

**ScanLines** 返回一行文本，不包括行尾的换行符。这里的换行包括了Windows下的"\r\n"和Unix下的"\n"。

一般地，我们不会单独使用这些函数，而是提供给 Scanner 实例使用。现在我们回到 Scanner 的 split、maxTokenSize 和 token 字段上来。

**split 字段**（SplitFunc 类型实例），很显然，代表了当前 Scanner 使用的分词策略，可以使用上面介绍的预定义 SplitFunc 实例赋值，也可以自定义 SplitFunc 实例。（当然，要给 split 字段赋值，必须调用 Scanner 的 Split 方法）

**maxTokenSize 字段** 表示通过 split 分词后的一个 token 允许的最大长度。在该包中定义了一个常量 MaxScanTokenSize = 64 * 1024，这是允许的最大 token 长度（64k）。

**token 字段** 上文已经解释了这个是什么意思。

### 1.4.2.2 Scanner 的实例化 ###

Scanner 没有导出任何字段，而它需要有外部的 io.Reader 对象，因此，我们不能直接实例化 Scanner 对象，必须通过 bufio 包提供的实例化函数来实例化。实例化函数签名以及内部实现：
```go
	func NewScanner(r io.Reader) *Scanner {
		return &Scanner{
			r:            r,
			split:        ScanLines,
			maxTokenSize: MaxScanTokenSize,
			buf:          make([]byte, 4096), // Plausible starting size; needn't be large.
		}
	}
```
可见，返回的 Scanner 实例默认的 split 函数是 ScanLines。

### 1.4.2.2 Scanner 的方法 ###

**Split 方法** 前面我们提到过可以通过 Split 方法为 Scanner 实例设置分词行为。由于 Scanner 实例的默认 split 总是 ScanLines，如果我们想要用其他的 split，可以通过 Split 方法做到。

比如，我们想要统计一段英文有多少个单词（不排除重复），我们可以这么做：
```go
	const input = "This is The Golang Standard Library.\nWelcome you!"
	scanner := bufio.NewScanner(strings.NewReader(input))
	scanner.Split(bufio.ScanWords)
	count := 0
	for scanner.Scan() {
		count++
	}
	if err := scanner.Err(); err != nil {
		fmt.Fprintln(os.Stderr, "reading input:", err)
	}
	fmt.Println(count)
```
输出：
```bash
	8
```
我们实例化 Scanner 后，通过调用 scanner.Split(bufio.ScanWords) 来更改 split 函数。注意，我们应该在调用 Scan 方法之前调用 Split 方法。

**Scan 方法** 该方法好比 iterator 中的 Next 方法，它用于将 Scanner 获取下一个 token，以便 Bytes 和 Text 方法可用。当扫描停止时，它返回false，这时候，要么是到了输入的末尾要么是遇到了一个错误。注意，当 Scan 返回 false 时，通过 Err 方法可以获取第一个遇到的错误（但如果错误是 io.EOF，Err 方法会返回 nil）。

**Bytes 和 Text 方法** 这两个方法的行为一致，都是返回最近的 token，无非 Bytes 返回的是 []byte，Text 返回的是 string。该方法应该在 Scan 调用后调用，而且，下次调用 Scan 会覆盖这次的 token。比如：
```go
	scanner := bufio.NewScanner(strings.NewReader("http://studygolang.com. \nIt is the home of gophers"))
	if scanner.Scan() {
		scanner.Scan()
		fmt.Printf("%s", scanner.Text())
	}
```
返回的是：`It is the home of gophers` 而不是 `http://studygolang.com.`

**Err 方法** 前面已经提到，通过 Err 方法可以获取第一个遇到的错误（但如果错误是 io.EOF，Err 方法会返回 nil）。

下面，我们通过一个完整的示例来演示 Scanner 类型的使用。

### 1.4.2.3 Scanner 使用示例 ###

我们经常会有这样的需求：读取文件中的数据，一次读取一行。在学习了 Reader 类型，我们可以使用它的 ReadBytes 或 ReadString来实现，甚至使用 ReadLine 来实现。然而，在 Go1.1 中，我们可以使用 Scanner 来做这件事，而且更简单好用。
```go
	file, err := os.Create("scanner.txt")
	if err != nil {
	    panic(err)
	}
	defer file.Close()
	file.WriteString("http://studygolang.com.\nIt is the home of gophers.\nIf you are studying golang, welcome you!")
	// 将文件 offset 设置到文件开头
	file.Seek(0, os.SEEK_SET)
	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}
```
输出结果：
```bash
	http://studygolang.com.
	It is the home of gophers.
	If you are studying golang, welcome you!
```
## 1.4.3 Writer 类型和方法 ##

bufio.Writer 结构包装了一个 io.Writer 对象，提供缓存功能，同时实现了 io.Writer 接口。

Writer 结构没有任何导出的字段，结构定义如下：
```go
	type Writer struct {
		err error		// 写过程中遇到的错误
		buf []byte		// 缓存
		n   int			// 当前缓存中的字节数
		wr  io.Writer	// 底层的 io.Writer 对象
	}
```
相比 bufio.Reader, bufio.Writer 结构定义简单很多。

注意：如果在写数据到 Writer 的时候出现了一个错误，不会再允许有数据被写进来了，并且所有随后的写操作都会返回该错误。

### 1.4.3.1 实例化 ###

和 Reader 类型一样，bufio 包提供了两个实例化 bufio.Writer 对象的函数：NewWriter 和 NewWriterSize。其中，NewWriter 函数是调用 NewWriterSize 函数实现的：
```go
	func NewWriter(wr io.Writer) *Writer {
		// 默认缓存大小：defaultBufSize=4096
		return NewWriterSize(wr, defaultBufSize)
	}
```
我们看一下 NewWriterSize 的源码：
```go
	func NewWriterSize(wr io.Writer, size int) *Writer {
		// 已经是 bufio.Writer 类型，且缓存大小不小于 size，则直接返回
		b, ok := wr.(*Writer)
		if ok && len(b.buf) >= size {
			return b
		}
		if size <= 0 {
			size = defaultBufSize
		}
		return &Writer{
			buf: make([]byte, size),
			wr:  w,
		}
	}
```
### 1.4.3.2 Available 和 Buffered 方法 ###

Available 方法获取缓存中还未使用的字节数（缓存大小 - 字段 n 的值）；Buffered 方法获取写入当前缓存中的字节数（字段 n 的值）

### 1.4.3.3 Flush 方法 ###

该方法将缓存中的所有数据写入底层的 io.Writer 对象中。使用 bufio.Writer 时，在所有的 Write 操作完成之后，应该调用 Flush 方法使得缓存都写入 io.Writer 对象中。

### 1.4.3.4 其他方法 ###

Writer 类型其他方法是一些实际的写方法：
```go
	// 实现了 io.ReaderFrom 接口
	func (b *Writer) ReadFrom(r io.Reader) (n int64, err error)
	
	// 实现了 io.Writer 接口
	func (b *Writer) Write(p []byte) (nn int, err error)
	
	// 实现了 io.ByteWriter 接口
	func (b *Writer) WriteByte(c byte) error
	
	// io 中没有该方法的接口，它用于写入单个 Unicode 码点，返回写入的字节数（码点占用的字节），内部实现会根据当前 rune 的范围调用 WriteByte 或 WriteString
	func (b *Writer) WriteRune(r rune) (size int, err error)
	
	// 写入字符串，如果返回写入的字节数比 len(s) 小，返回的error会解释原因
	func (b *Writer) WriteString(s string) (int, error)
```
这些写方法在缓存满了时会调用 Flush 方法。另外，这些写方法源码开始处，有这样的代码：
```go
	if b.err != nil {
		return b.err
	}
```
也就是说，只要写的过程中遇到了错误，再次调用写操作会直接返回该错误。

## 1.4.4 ReadWriter 类型和实例化 ##

ReadWriter 结构存储了 bufio.Reader 和 bufio.Writer 类型的指针（内嵌），它实现了 io.ReadWriter 结构。
```
	type ReadWriter struct {
		*Reader
		*Writer
	}
```
ReadWriter 的实例化可以跟普通结构类型一样，也可以通过调用 bufio.NewReadWriter 函数来实现：只是简单的实例化 ReadWriter
```
	func NewReadWriter(r *Reader, w *Writer) *ReadWriter {
		return &ReadWriter{r, w}
	}
```
# 导航 #