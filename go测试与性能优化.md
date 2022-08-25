# `go`测试与性能优化

# 一、单元测试

不写测试的开发不是好程序员。我个人非常崇尚TDD（Test Driven Development）的，然而可惜的是国内的程序员都不太关注测试这一部分。 这篇文章主要介绍下在Go语言中如何做单元测试和基准测试。

## `go test`工具

Go语言中的测试依赖`go test`命令。编写测试代码和编写普通的Go代码过程是类似的，并不需要学习新的语法、规则或工具。

go test命令是一个按照一定约定和组织的测试代码的驱动程序。在包目录内，所有以`_test.go`为后缀名的源代码文件都是`go test`测试的一部分，不会被`go build`编译到最终的可执行文件中。

在`*_test.go`文件中有三种类型的函数，单元测试函数、基准测试函数和示例函数。

|   类型   |         格式          |              作用              |
| :------: | :-------------------: | :----------------------------: |
| 测试函数 |   函数名前缀为Test    | 测试程序的一些逻辑行为是否正确 |
| 基准函数 | 函数名前缀为Benchmark |         测试函数的性能         |
| 示例函数 |  函数名前缀为Example  |       为文档提供示例文档       |

`go test`命令会遍历所有的`*_test.go`文件中符合上述命名规则的函数，然后生成一个临时的main包用于调用相应的测试函数，然后构建并运行、报告测试结果，最后清理测试中生成的临时文件。

## 测试函数

### 格式

每个测试函数必须导入`testing`包，测试函数的基本格式（签名）如下：

```go
func TestName(t *testing.T){
    // ...
}
```

测试函数的名字必须以`Test`开头，可选的后缀名必须以大写字母开头，举几个例子：

```go
func TestAdd(t *testing.T){ ... }
func TestSum(t *testing.T){ ... }
func TestLog(t *testing.T){ ... }
```

其中参数`t`用于报告测试失败和附加的日志信息。 `testing.T`的拥有的方法如下：

```go
func (c *T) Cleanup(func())
func (c *T) Error(args ...interface{})
func (c *T) Errorf(format string, args ...interface{})
func (c *T) Fail()
func (c *T) FailNow()
func (c *T) Failed() bool
func (c *T) Fatal(args ...interface{})
func (c *T) Fatalf(format string, args ...interface{})
func (c *T) Helper()
func (c *T) Log(args ...interface{})
func (c *T) Logf(format string, args ...interface{})
func (c *T) Name() string
func (c *T) Skip(args ...interface{})
func (c *T) SkipNow()
func (c *T) Skipf(format string, args ...interface{})
func (c *T) Skipped() bool
func (c *T) TempDir() string
```

### 单元测试示例

就像细胞是构成我们身体的基本单位，一个软件程序也是由很多单元组件构成的。单元组件可以是函数、结构体、方法和最终用户可能依赖的任意东西。总之我们需要确保这些组件是能够正常运行的。单元测试是一些利用各种方法测试单元组件的程序，它会将结果与预期输出进行比较。

接下来，我们在`base_demo`包中定义了一个`Split`函数，具体实现如下：

```go
// base_demo/split.go

package base_demo

import "strings"

// Split 把字符串s按照给定的分隔符sep进行分割返回字符串切片
func Split(s, sep string) (result []string) {
	i := strings.Index(s, sep)

	for i > -1 {
		result = append(result, s[:i])
		s = s[i+1:]
		i = strings.Index(s, sep)
	}
	result = append(result, s)
	return
}
```

在当前目录下，我们创建一个`split_test.go`的测试文件，并定义一个测试函数如下：

```go
// split/split_test.go

package split

import (
	"reflect"
	"testing"
)

func TestSplit(t *testing.T) { // 测试函数名必须以Test开头，必须接收一个*testing.T类型参数
	got := Split("a:b:c", ":")         // 程序输出的结果
	want := []string{"a", "b", "c"}    // 期望的结果
	if !reflect.DeepEqual(want, got) { // 因为slice不能比较直接，借助反射包中的方法比较
		t.Errorf("expected:%v, got:%v", want, got) // 测试失败输出错误提示
	}
}
```

在当前路径下执行`go test`命令，可以看到输出结果如下：

```bash
❯ go test
PASS
ok      golang-unit-test-demo/base_demo       0.005s
```

### `go test -v`

现在我们有多个测试用例了，为了能更好的在输出结果中看到每个测试用例的执行情况，我们可以为`go test`命令添加`-v`参数，让它输出完整的测试结果（测试所有测试用例）。

```bash
❯ go test -v
=== RUN   TestSplit
--- PASS: TestSplit (0.00s)
=== RUN   TestSplitWithComplexSep
    split_test.go:20: expected:[a d], got:[a cd]
--- FAIL: TestSplitWithComplexSep (0.00s)
FAIL
exit status 1
FAIL    golang-unit-test-demo/base_demo 0.009s
```

从上面的输出结果我们能清楚的看到是`TestSplitWithComplexSep`这个测试用例没有测试通过。

### `go test -run`

在执行`go test`命令的时候可以添加`-run`参数，它对应一个正则表达式，只有函数名匹配上的测试函数才会被`go test`命令执行。

例如通过给`go test`添加`-run=Sep`参数来告诉它本次测试只运行`TestSplitWithComplexSep`这个测试用例：

```bash
❯ go test -run=Sep -v
=== RUN   TestSplitWithComplexSep
--- PASS: TestSplitWithComplexSep (0.00s)
PASS
ok      golang-unit-test-demo/base_demo 0.010s
```

### 回归测试

我们修改了代码之后仅仅执行那些失败的测试用例或新引入的测试用例是错误且危险的，**正确的做法应该是完整运行所有的测试用例，保证不会因为修改代码而引入新的问题**。

```bash
❯ go test -v
=== RUN   TestSplit
--- PASS: TestSplit (0.00s)
=== RUN   TestSplitWithComplexSep
--- PASS: TestSplitWithComplexSep (0.00s)
PASS
ok      golang-unit-test-demo/base_demo 0.011s
```

### 跳过某些测试用例

为了节省时间支持在单元测试时跳过某些耗时的测试用例。

```go
func TestTimeConsuming(t *testing.T) {
    if testing.Short() {
        t.Skip("short模式下会跳过该测试用例")
    }
    ...
}
```

当执行`go test -short`时就不会执行上面的`TestTimeConsuming`测试用例。

### 子测试

在上面的示例中我们为每一个测试数据编写了一个测试函数，而通常单元测试中需要多组测试数据保证测试的效果。Go1.7+中新增了子测试，支持在测试函数中使用`t.Run`执行一组测试用例，这样就不需要为不同的测试数据定义多个测试函数了。

```go
func TestXXX(t *testing.T){
  t.Run("case1", func(t *testing.T){...})
  t.Run("case2", func(t *testing.T){...})
  t.Run("case3", func(t *testing.T){...})
}
```

### 表格驱动测试

#### 介绍

表格驱动测试不是工具、包或其他任何东西，它只是编写更清晰测试的一种方式和视角。

编写好的测试并非易事，但在许多情况下，表格驱动测试可以涵盖很多方面：表格里的每一个条目都是一个完整的测试用例，包含输入和预期结果，有时还包含测试名称等附加信息，以使测试输出易于阅读。

使用表格驱动测试能够很方便的维护多个测试用例，避免在编写单元测试时频繁的复制粘贴。

表格驱动测试的步骤通常是定义一个测试用例表格，然后遍历表格，并使用`t.Run`对每个条目执行必要的测试。

#### 示例

官方标准库中有很多表格驱动测试的示例，例如fmt包中便有如下测试代码：

```go
var flagtests = []struct {
	in  string
	out string
}{
	{"%a", "[%a]"},
	{"%-a", "[%-a]"},
	{"%+a", "[%+a]"},
	{"%#a", "[%#a]"},
	{"% a", "[% a]"},
	{"%0a", "[%0a]"},
	{"%1.2a", "[%1.2a]"},
	{"%-1.2a", "[%-1.2a]"},
	{"%+1.2a", "[%+1.2a]"},
	{"%-+1.2a", "[%+-1.2a]"},
	{"%-+1.2abc", "[%+-1.2a]bc"},
	{"%-1.2abc", "[%-1.2a]bc"},
}
func TestFlagParser(t *testing.T) {
	var flagprinter flagPrinter
	for _, tt := range flagtests {
		t.Run(tt.in, func(t *testing.T) {
			s := Sprintf(tt.in, &flagprinter)
			if s != tt.out {
				t.Errorf("got %q, want %q", s, tt.out)
			}
		})
	}
}
```

通常表格是匿名结构体切片，可以定义结构体或使用已经存在的结构进行结构体数组声明。name属性用来描述特定的测试用例。

接下来让我们试着自己编写表格驱动测试：

```go
func TestSplitAll(t *testing.T) {
	// 定义测试表格
	// 这里使用匿名结构体定义了若干个测试用例
	// 并且为每个测试用例设置了一个名称
	tests := []struct {
		name  string
		input string
		sep   string
		want  []string
	}{
		{"base case", "a:b:c", ":", []string{"a", "b", "c"}},
		{"wrong sep", "a:b:c", ",", []string{"a:b:c"}},
		{"more sep", "abcd", "bc", []string{"a", "d"}},
		{"leading sep", "沙河有沙又有河", "沙", []string{"", "河有", "又有河"}},
	}
	// 遍历测试用例
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) { // 使用t.Run()执行子测试
			got := Split(tt.input, tt.sep)
			if !reflect.DeepEqual(got, tt.want) {
				t.Errorf("expected:%#v, got:%#v", tt.want, got)
			}
		})
	}
}
```

在终端执行`go test -v`，会得到如下测试输出结果：

```bash
❯ go test -v
=== RUN   TestSplit
--- PASS: TestSplit (0.00s)
=== RUN   TestSplitWithComplexSep
--- PASS: TestSplitWithComplexSep (0.00s)
=== RUN   TestSplitAll
=== RUN   TestSplitAll/base_case
=== RUN   TestSplitAll/wrong_sep
=== RUN   TestSplitAll/more_sep
=== RUN   TestSplitAll/leading_sep
--- PASS: TestSplitAll (0.00s)
    --- PASS: TestSplitAll/base_case (0.00s)
    --- PASS: TestSplitAll/wrong_sep (0.00s)
    --- PASS: TestSplitAll/more_sep (0.00s)
    --- PASS: TestSplitAll/leading_sep (0.00s)
PASS
ok      golang-unit-test-demo/base_demo 0.010s
```

#### 并行测试

表格驱动测试中通常会定义比较多的测试用例，而Go语言又天生支持并发，所以很容易发挥自身并发优势将表格驱动测试并行化。 想要在单元测试过程中使用并行测试，可以像下面的代码示例中那样通过添加`t.Parallel()`来实现。

```go
func TestSplitAll(t *testing.T) {
	t.Parallel()  // 将 TLog 标记为能够与其他测试并行运行
	// 定义测试表格
	// 这里使用匿名结构体定义了若干个测试用例
	// 并且为每个测试用例设置了一个名称
	tests := []struct {
		name  string
		input string
		sep   string
		want  []string
	}{
		{"base case", "a:b:c", ":", []string{"a", "b", "c"}},
		{"wrong sep", "a:b:c", ",", []string{"a:b:c"}},
		{"more sep", "abcd", "bc", []string{"a", "d"}},
		{"leading sep", "沙河有沙又有河", "沙", []string{"", "河有", "又有河"}},
	}
	// 遍历测试用例
	for _, tt := range tests {
		tt := tt  // 注意这里重新声明tt变量（避免多个goroutine中使用了相同的变量）
		t.Run(tt.name, func(t *testing.T) { // 使用t.Run()执行子测试
			t.Parallel()  // 将每个测试用例标记为能够彼此并行运行
			got := Split(tt.input, tt.sep)
			if !reflect.DeepEqual(got, tt.want) {
				t.Errorf("expected:%#v, got:%#v", tt.want, got)
			}
		})
	}
}
```

这样我们执行`go test -v`的时候就会看到每个测试用例并不是按照我们定义的顺序执行，而是互相并行了。

#### 使用工具生成测试代码

社区里有很多自动生成表格驱动测试函数的工具，比如[gotests](https://github.com/cweill/gotests)等，很多编辑器如Goland也支持快速生成测试文件。这里简单演示一下`gotests`的使用。

安装

```bash
go get -u github.com/cweill/gotests/...
```

执行

```bash
gotests -all -w split.go
```

上面的命令表示，为`split.go`文件的所有函数生成测试代码至`split_test.go`文件（目录下如果事先存在这个文件就不再生成）。

生成的测试代码大致如下：

```go
package base_demo

import (
	"reflect"
	"testing"
)

func TestSplit(t *testing.T) {
	type args struct {
		s   string
		sep string
	}
	tests := []struct {
		name       string
		args       args
		wantResult []string
	}{
		// TODO: Add test cases.
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if gotResult := Split(tt.args.s, tt.args.sep); !reflect.DeepEqual(gotResult, tt.wantResult) {
				t.Errorf("Split() = %v, want %v", gotResult, tt.wantResult)
			}
		})
	}
}
```

代码格式与我们上面的类似，只需要在TODO位置添加我们的测试逻辑就可以了。

### 测试覆盖率

测试覆盖率是指代码被测试套件覆盖的百分比。通常我们使用的都是语句的覆盖率，也就是在测试中至少被运行一次的代码占总代码的比例。在公司内部一般会要求测试覆盖率达到80%左右。

Go提供内置功能来检查你的代码覆盖率，即使用`go test -cover`来查看测试覆盖率。

```bash
❯ go test -cover
PASS
coverage: 100.0% of statements
ok      golang-unit-test-demo/base_demo 0.009s
```

从上面的结果可以看到我们的测试用例覆盖了100%的代码。

Go还提供了一个额外的`-coverprofile`参数，用来将覆盖率相关的记录信息输出到一个文件。例如：

```bash
❯ go test -cover -coverprofile=c.out
PASS
coverage: 100.0% of statements
ok      golang-unit-test-demo/base_demo 0.009s
```

上面的命令会将覆盖率相关的信息输出到当前文件夹下面的`c.out`文件中。

```bash
❯ tree .
.
├── c.out
├── split.go
└── split_test.go
```

然后我们执行`go tool cover -html=c.out`，使用`cover`工具来处理生成的记录信息，该命令会打开本地的浏览器窗口生成一个HTML报告。![Go test cover](https://www.liwenzhou.com/images/Go/unit_test/cover.png)上图中每个用绿色标记的语句块表示被覆盖了，而红色的表示没有被覆盖。

### testify/assert

[testify](https://github.com/stretchr/testify)是一个社区非常流行的Go单元测试工具包，其中使用最多的功能就是它提供的断言工具——`testify/assert`或`testify/require`。

### 安装

```bash
go get github.com/stretchr/testify
```

### 使用示例

我们在写单元测试的时候，通常需要使用断言来校验测试结果，但是由于Go语言官方没有提供断言，所以我们会写出很多的`if...else...`语句。而`testify/assert`为我们提供了很多常用的断言函数，并且能够输出友好、易于阅读的错误描述信息。

比如我们之前在`TestSplit`测试函数中就使用了`reflect.DeepEqual`来判断期望结果与实际结果是否一致。

```go
t.Run(tt.name, func(t *testing.T) { // 使用t.Run()执行子测试
	got := Split(tt.input, tt.sep)
	if !reflect.DeepEqual(got, tt.want) {
		t.Errorf("expected:%#v, got:%#v", tt.want, got)
	}
})
```

使用`testify/assert`之后就能将上述判断过程简化如下：

```go
t.Run(tt.name, func(t *testing.T) { // 使用t.Run()执行子测试
	got := Split(tt.input, tt.sep)
	assert.Equal(t, got, tt.want)  // 使用assert提供的断言函数
})
```

当我们有多个断言语句时，还可以使用`assert := assert.New(t)`创建一个assert对象，它拥有前面所有的断言方法，只是不需要再传入`Testing.T`参数了。

```go
func TestSomething(t *testing.T) {
  assert := assert.New(t)

  // assert equality
  assert.Equal(123, 123, "they should be equal")

  // assert inequality
  assert.NotEqual(123, 456, "they should not be equal")

  // assert for nil (good for errors)
  assert.Nil(object)

  // assert for not nil (good when you expect something)
  if assert.NotNil(object) {

    // now we know that object isn't nil, we are safe to make
    // further assertions without causing any errors
    assert.Equal("Something", object.Value)
  }
}
```

`testify/assert`提供了非常多的断言函数，这里没办法一一列举出来，大家可以查看[官方文档](https://pkg.go.dev/github.com/stretchr/testify/assert#pkg-functions)了解。

`testify/require`拥有`testify/assert`所有断言函数，它们的唯一区别就是——`testify/require`遇到失败的用例会立即终止本次测试。

此外，`testify`包还提供了[mock](https://pkg.go.dev/github.com/stretchr/testify/mock)、[http](https://pkg.go.dev/github.com/stretchr/testify/http)等其他测试工具。

## 基准测试

### 基准测试函数格式

基准测试就是在一定的工作负载之下检测程序性能的一种方法。基准测试的基本格式如下：

```go
func BenchmarkName(b *testing.B){
    // ...
}
```

基准测试以`Benchmark`为前缀，需要一个`*testing.B`类型的参数b，基准测试必须要执行`b.N`次，这样的测试才有对照性，`b.N`的值是系统根据实际情况去调整的，从而保证测试的稳定性。 `testing.B`拥有的方法如下：

```go
func (c *B) Error(args ...interface{})
func (c *B) Errorf(format string, args ...interface{})
func (c *B) Fail()
func (c *B) FailNow()
func (c *B) Failed() bool
func (c *B) Fatal(args ...interface{})
func (c *B) Fatalf(format string, args ...interface{})
func (c *B) Log(args ...interface{})
func (c *B) Logf(format string, args ...interface{})
func (c *B) Name() string
func (b *B) ReportAllocs()
func (b *B) ResetTimer()
func (b *B) Run(name string, f func(b *B)) bool
func (b *B) RunParallel(body func(*PB))
func (b *B) SetBytes(n int64)
func (b *B) SetParallelism(p int)
func (c *B) Skip(args ...interface{})
func (c *B) SkipNow()
func (c *B) Skipf(format string, args ...interface{})
func (c *B) Skipped() bool
func (b *B) StartTimer()
func (b *B) StopTimer()
```

### 基准测试示例

我们为split包中的`Split`函数编写基准测试如下：

```go
func BenchmarkSplit(b *testing.B) {
	for i := 0; i < b.N; i++ {
		Split("沙河有沙又有河", "沙")
	}
}
```

基准测试并不会默认执行，需要增加`-bench`参数，所以我们通过执行`go test -bench=Split`命令执行基准测试，输出结果如下：

```bash
split $ go test -bench=Split
goos: darwin
goarch: amd64
pkg: github.com/Q1mi/studygo/code_demo/test_demo/split
BenchmarkSplit-8        10000000               203 ns/op
PASS
ok      github.com/Q1mi/studygo/code_demo/test_demo/split       2.255s
```

其中`BenchmarkSplit-8`表示对Split函数进行基准测试，数字`8`表示`GOMAXPROCS`的值，这个对于并发基准测试很重要。`10000000`和`203ns/op`表示每次调用`Split`函数耗时`203ns`，这个结果是`10000000`次调用的平均值。

我们还可以为基准测试添加`-benchmem`参数，来获得内存分配的统计数据。

```bash
split $ go test -bench=Split -benchmem
goos: darwin
goarch: amd64
pkg: github.com/Q1mi/studygo/code_demo/test_demo/split
BenchmarkSplit-8        10000000               215 ns/op             112 B/op          3 allocs/op
PASS
ok      github.com/Q1mi/studygo/code_demo/test_demo/split       2.394s
```

其中，`112 B/op`表示每次操作内存分配了112字节，`3 allocs/op`则表示每次操作进行了3次内存分配。 我们将我们的`Split`函数优化如下：

```go
func Split(s, sep string) (result []string) {
	result = make([]string, 0, strings.Count(s, sep)+1)
	i := strings.Index(s, sep)
	for i > -1 {
		result = append(result, s[:i])
		s = s[i+len(sep):] // 这里使用len(sep)获取sep的长度
		i = strings.Index(s, sep)
	}
	result = append(result, s)
	return
}
```

这一次我们提前使用make函数将result初始化为一个容量足够大的切片，而不再像之前一样通过调用append函数来追加。我们来看一下这个改进会带来多大的性能提升：

```bash
split $ go test -bench=Split -benchmem
goos: darwin
goarch: amd64
pkg: github.com/Q1mi/studygo/code_demo/test_demo/split
BenchmarkSplit-8        10000000               127 ns/op              48 B/op          1 allocs/op
PASS
ok      github.com/Q1mi/studygo/code_demo/test_demo/split       1.423s
```

这个使用make函数提前分配内存的改动，减少了2/3的内存分配次数，并且减少了一半的内存分配。

### 性能比较函数

上面的基准测试只能得到给定操作的绝对耗时，但是在很多性能问题是发生在两个不同操作之间的相对耗时，比如同一个函数处理1000个元素的耗时与处理1万甚至100万个元素的耗时的差别是多少？再或者对于同一个任务究竟使用哪种算法性能最佳？我们通常需要对两个不同算法的实现使用相同的输入来进行基准比较测试。

性能比较函数通常是一个带有参数的函数，被多个不同的Benchmark函数传入不同的值来调用。举个例子如下：

```go
func benchmark(b *testing.B, size int){/* ... */}
func Benchmark10(b *testing.B){ benchmark(b, 10) }
func Benchmark100(b *testing.B){ benchmark(b, 100) }
func Benchmark1000(b *testing.B){ benchmark(b, 1000) }
```

例如我们编写了一个计算斐波那契数列的函数如下：

```go
// fib.go

// Fib 是一个计算第n个斐波那契数的函数
func Fib(n int) int {
	if n < 2 {
		return n
	}
	return Fib(n-1) + Fib(n-2)
}
```

我们编写的性能比较函数如下：

```go
// fib_test.go

func benchmarkFib(b *testing.B, n int) {
	for i := 0; i < b.N; i++ {
		Fib(n)
	}
}

func BenchmarkFib1(b *testing.B)  { benchmarkFib(b, 1) }
func BenchmarkFib2(b *testing.B)  { benchmarkFib(b, 2) }
func BenchmarkFib3(b *testing.B)  { benchmarkFib(b, 3) }
func BenchmarkFib10(b *testing.B) { benchmarkFib(b, 10) }
func BenchmarkFib20(b *testing.B) { benchmarkFib(b, 20) }
func BenchmarkFib40(b *testing.B) { benchmarkFib(b, 40) }
```

运行基准测试：

```bash
split $ go test -bench=.
goos: darwin
goarch: amd64
pkg: github.com/Q1mi/studygo/code_demo/test_demo/fib
BenchmarkFib1-8         1000000000               2.03 ns/op
BenchmarkFib2-8         300000000                5.39 ns/op
BenchmarkFib3-8         200000000                9.71 ns/op
BenchmarkFib10-8         5000000               325 ns/op
BenchmarkFib20-8           30000             42460 ns/op
BenchmarkFib40-8               2         638524980 ns/op
PASS
ok      github.com/Q1mi/studygo/code_demo/test_demo/fib 12.944s
```

这里需要注意的是，默认情况下，每个基准测试至少运行1秒。如果在Benchmark函数返回时没有到1秒，则`b.N`的值会按1,2,5,10,20,50，…增加，并且函数再次运行。

最终的`BenchmarkFib40`只运行了两次，每次运行的平均值只有不到一秒。像这种情况下我们应该可以使用`-benchtime`标志增加最小基准时间，以产生更准确的结果。（因为默认基准执行1s,时间没到时，会自动调整，重新执行直到1s，如果某个函数的执行用时超过默认执行用时1s，可能无法得到返回）例如：

```bash
split $ go test -bench=Fib40 -benchtime=20s
goos: darwin
goarch: amd64
pkg: github.com/Q1mi/studygo/code_demo/test_demo/fib
BenchmarkFib40-8              50         663205114 ns/op
PASS
ok      github.com/Q1mi/studygo/code_demo/test_demo/fib 33.849s
```

这一次`BenchmarkFib40`函数运行了50次，结果就会更准确一些了。

使用性能比较函数做测试的时候一个容易犯的错误就是把`b.N`作为输入的大小，例如以下两个例子都是错误的示范：

```go
// 错误示范1
func BenchmarkFibWrong(b *testing.B) {
	for n := 0; n < b.N; n++ {
		Fib(n)
	}
}

// 错误示范2
func BenchmarkFibWrong2(b *testing.B) {
	Fib(b.N)
}
```

### 重置时间

`b.ResetTimer`之前的处理不会放到执行时间里，也不会输出到报告中，所以可以在之前做一些不计划作为测试报告的操作。例如：

```go
func BenchmarkSplit(b *testing.B) {
	time.Sleep(5 * time.Second) // 假设需要做一些耗时的无关操作
	b.ResetTimer()              // 重置计时器
	for i := 0; i < b.N; i++ {
		Split("沙河有沙又有河", "沙")
	}
}
```

### 并行测试

`func (b *B) RunParallel(body func(*PB))`会以并行的方式执行给定的基准测试。

`RunParallel`会创建出多个`goroutine`，并将`b.N`分配给这些`goroutine`执行， 其中`goroutine`数量的默认值为`GOMAXPROCS`。用户如果想要增加非CPU受限（non-CPU-bound）基准测试的并行性， 那么可以在`RunParallel`之前调用`SetParallelism` 。`RunParallel`通常会与`-cpu`标志一同使用。

```go
func BenchmarkSplitParallel(b *testing.B) {
	// b.SetParallelism(1) // 设置使用的CPU数
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			Split("沙河有沙又有河", "沙")
		}
	})
}
```

执行一下基准测试：

```bash
split $ go test -bench=.
goos: darwin
goarch: amd64
pkg: github.com/Q1mi/studygo/code_demo/test_demo/split
BenchmarkSplit-8                10000000               131 ns/op
BenchmarkSplitParallel-8        50000000                36.1 ns/op
PASS
ok      github.com/Q1mi/studygo/code_demo/test_demo/split       3.308s
```

还可以通过在测试命令后添加`-cpu`参数如`go test -bench=. -cpu 1`来指定使用的CPU数量。

## `Setup`与`TearDown`

测试程序有时需要在测试之前进行额外的设置（setup）或在测试之后进行拆卸（teardown）。

### `TestMain`

通过在`*_test.go`文件中定义`TestMain`函数来可以在测试之前进行额外的设置（setup）或在测试之后进行拆卸（teardown）操作。

如果测试文件包含函数:`func TestMain(m *testing.M)`那么生成的测试会先调用 TestMain(m)，然后再运行具体测试。`TestMain`运行在主`goroutine`中, 可以在调用 `m.Run`前后做任何设置（setup）和拆卸（teardown）。退出测试的时候应该使用`m.Run`的返回值作为参数调用`os.Exit`。

一个使用`TestMain`来设置Setup和TearDown的示例如下：

```go
func TestMain(m *testing.M) {
	fmt.Println("write setup code here...") // 测试之前的做一些设置
	// 如果 TestMain 使用了 flags，这里应该加上flag.Parse()
	retCode := m.Run()                         // 执行测试
	fmt.Println("write teardown code here...") // 测试之后做一些拆卸工作
	os.Exit(retCode)                           // 退出测试
}
```

需要注意的是：在调用`TestMain`时, `flag.Parse`并没有被调用。所以如果`TestMain` 依赖于command-line标志 (包括 testing 包的标记), 则应该显示的调用`flag.Parse`。

### 子测试的Setup与Teardown

有时候我们可能需要为每个测试集设置Setup与Teardown，也有可能需要为每个子测试设置Setup与Teardown。下面我们定义两个函数工具函数如下：

```go
// 测试集的Setup与Teardown
func setupTestCase(t *testing.T) func(t *testing.T) {
	t.Log("如有需要在此执行:测试之前的setup")
	return func(t *testing.T) {
		t.Log("如有需要在此执行:测试之后的teardown")
	}
}

// 子测试的Setup与Teardown
func setupSubTest(t *testing.T) func(t *testing.T) {
	t.Log("如有需要在此执行:子测试之前的setup")
	return func(t *testing.T) {
		t.Log("如有需要在此执行:子测试之后的teardown")
	}
}
```

使用方式如下：

```go
func TestSplit(t *testing.T) {
	type test struct { // 定义test结构体
		input string
		sep   string
		want  []string
	}
	tests := map[string]test{ // 测试用例使用map存储
		"simple":      {input: "a:b:c", sep: ":", want: []string{"a", "b", "c"}},
		"wrong sep":   {input: "a:b:c", sep: ",", want: []string{"a:b:c"}},
		"more sep":    {input: "abcd", sep: "bc", want: []string{"a", "d"}},
		"leading sep": {input: "沙河有沙又有河", sep: "沙", want: []string{"", "河有", "又有河"}},
	}
	teardownTestCase := setupTestCase(t) // 测试之前执行setup操作
	defer teardownTestCase(t)            // 测试之后执行testdoen操作

	for name, tc := range tests {
		t.Run(name, func(t *testing.T) { // 使用t.Run()执行子测试
			teardownSubTest := setupSubTest(t) // 子测试之前执行setup操作
			defer teardownSubTest(t)           // 测试之后执行testdoen操作
			got := Split(tc.input, tc.sep)
			if !reflect.DeepEqual(got, tc.want) {
				t.Errorf("expected:%#v, got:%#v", tc.want, got)
			}
		})
	}
}
```

测试结果如下：

```bash
split $ go test -v
=== RUN   TestSplit
=== RUN   TestSplit/simple
=== RUN   TestSplit/wrong_sep
=== RUN   TestSplit/more_sep
=== RUN   TestSplit/leading_sep
--- PASS: TestSplit (0.00s)
    split_test.go:71: 如有需要在此执行:测试之前的setup
    --- PASS: TestSplit/simple (0.00s)
        split_test.go:79: 如有需要在此执行:子测试之前的setup
        split_test.go:81: 如有需要在此执行:子测试之后的teardown
    --- PASS: TestSplit/wrong_sep (0.00s)
        split_test.go:79: 如有需要在此执行:子测试之前的setup
        split_test.go:81: 如有需要在此执行:子测试之后的teardown
    --- PASS: TestSplit/more_sep (0.00s)
        split_test.go:79: 如有需要在此执行:子测试之前的setup
        split_test.go:81: 如有需要在此执行:子测试之后的teardown
    --- PASS: TestSplit/leading_sep (0.00s)
        split_test.go:79: 如有需要在此执行:子测试之前的setup
        split_test.go:81: 如有需要在此执行:子测试之后的teardown
    split_test.go:73: 如有需要在此执行:测试之后的teardown
=== RUN   ExampleSplit
--- PASS: ExampleSplit (0.00s)
PASS
ok      github.com/Q1mi/studygo/code_demo/test_demo/split       0.006s
```

## 示例函数

### 示例函数的格式

被`go test`特殊对待的第三种函数就是示例函数，它们的函数名以`Example`为前缀。它们既没有参数也没有返回值。标准格式如下：

```go
func ExampleName() {
    // ...
}
```

### 示例函数示例

下面的代码是我们为`Split`函数编写的一个示例函数：

```go
func ExampleSplit() {
	fmt.Println(split.Split("a:b:c", ":"))
	fmt.Println(split.Split("沙河有沙又有河", "沙"))
	// Output:
	// [a b c]
	// [ 河有 又有河]
}
```

为你的代码编写示例代码有如下三个用处：

1. 示例函数能够作为文档直接使用，例如基于web的godoc中能把示例函数与对应的函数或包相关联。

2. 示例函数只要包含了`// Output:`也是可以通过`go test`运行的可执行测试。

   ```bash
   split $ go test -run Example
   PASS
   ok      github.com/Q1mi/studygo/code_demo/test_demo/split       0.006s
   ```

3. 示例函数提供了可以直接运行的示例代码，可以直接在`golang.org`的`godoc`文档服务器上使用`Go Playground`运行示例代码。下图为`strings.ToUpper`函数在Playground的示例函数效果。

   ![Go Playground](https://www.liwenzhou.com/images/Go/unit_test/example.png)

# 二、网络测试

## `httptest`

在Web开发场景下的单元测试，如果涉及到HTTP请求推荐大家使用Go标准库 `net/http/httptest` 进行测试，能够显著提高测试效率。

在这一小节，我们以常见的gin框架为例，演示如何为http server编写单元测试。

假设我们的业务逻辑是搭建一个http server端，对外提供HTTP服务。我们编写了一个`helloHandler`函数，用来处理用户请求。

```go
// gin.go
package httptest_demo

import (
	"fmt"
	"net/http"

	"github.com/gin-gonic/gin"
)

// Param 请求参数
type Param struct {
	Name string `json:"name"`
}

// helloHandler /hello请求处理函数
func helloHandler(c *gin.Context) {
	var p Param
	if err := c.ShouldBindJSON(&p); err != nil {
		c.JSON(http.StatusOK, gin.H{
			"msg": "we need a name",
		})
		return
	}
	c.JSON(http.StatusOK, gin.H{
		"msg": fmt.Sprintf("hello %s", p.Name),
	})
}

// SetupRouter 路由
func SetupRouter() *gin.Engine {
	router := gin.Default()
	router.POST("/hello", helloHandler)
	return router
}
```

现在我们需要为`helloHandler`函数编写单元测试，这种情况下我们就可以使用`httptest`这个工具mock一个HTTP请求和响应记录器，让我们的server端接收并处理我们mock的HTTP请求，同时使用响应记录器来记录server端返回的响应内容。

单元测试的示例代码如下：

```go
// gin_test.go
package httptest_demo

import (
	"encoding/json"
	"net/http"
	"net/http/httptest"
	"strings"
	"testing"

	"github.com/stretchr/testify/assert"
)

func Test_helloHandler(t *testing.T) {
	// 定义两个测试用例
	tests := []struct {
		name   string
		param  string
		expect string
	}{
		{"base case", `{"name": "liwenzhou"}`, "hello liwenzhou"},
		{"bad case", "", "we need a name"},
	}

	r := SetupRouter()

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			// mock一个HTTP请求
			req := httptest.NewRequest(
				"POST",                      // 请求方法
				"/hello",                    // 请求URL
				strings.NewReader(tt.param), // 请求参数
			)

			// mock一个响应记录器
			w := httptest.NewRecorder()

			// 让server端处理mock请求并记录返回的响应内容
			r.ServeHTTP(w, req)

			// 校验状态码是否符合预期
			assert.Equal(t, http.StatusOK, w.Code)

			// 解析并检验响应内容是否复合预期
			var resp map[string]string
			err := json.Unmarshal([]byte(w.Body.String()), &resp)
			assert.Nil(t, err)
			assert.Equal(t, tt.expect, resp["msg"])
		})
	}
}
```

执行单元测试，查看测试结果。

```bash
❯ go test -v
=== RUN   Test_helloHandler
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] POST   /hello                    --> golang-unit-test-demo/httptest_demo.helloHandler (3 handlers)
=== RUN   Test_helloHandler/base_case
[GIN] 2021/09/14 - 22:00:04 | 200 |     164.839µs |       192.0.2.1 | POST     "/hello"
=== RUN   Test_helloHandler/bad_case
[GIN] 2021/09/14 - 22:00:04 | 200 |      23.723µs |       192.0.2.1 | POST     "/hello"
--- PASS: Test_helloHandler (0.00s)
    --- PASS: Test_helloHandler/base_case (0.00s)
    --- PASS: Test_helloHandler/bad_case (0.00s)
PASS
ok      golang-unit-test-demo/httptest_demo     0.055s
```

通过这个示例我们就掌握了如何使用httptest在HTTP Server服务中为请求处理函数编写单元测试了。

## `gock`

上面的示例介绍了如何在HTTP Server服务类场景下为请求处理函数编写单元测试，那么如果我们是在代码中请求外部API的场景（比如通过API调用其他服务获取返回值）又该怎么编写单元测试呢？

例如，我们有以下业务逻辑代码，依赖外部API：`http://your-api.com/post`提供的数据。

```go
// api.go

// ReqParam API请求参数
type ReqParam struct {
	X int `json:"x"`
}

// Result API返回结果
type Result struct {
	Value int `json:"value"`
}

func GetResultByAPI(x, y int) int {
	p := &ReqParam{X: x}
	b, _ := json.Marshal(p)

	// 调用其他服务的API
	resp, err := http.Post(
		"http://your-api.com/post",
		"application/json",
		bytes.NewBuffer(b),
	)
	if err != nil {
		return -1
	}
	body, _ := ioutil.ReadAll(resp.Body)
	var ret Result
	if err := json.Unmarshal(body, &ret); err != nil {
		return -1
	}
	// 这里是对API返回的数据做一些逻辑处理
	return ret.Value + y
}
```

在对类似上述这类业务代码编写单元测试的时候，如果不想在测试过程中真正去发送请求或者依赖的外部接口还没有开发完成时，我们可以在单元测试中对依赖的API进行mock。

这里推荐使用[gock](https://github.com/h2non/gock)这个库。

### 安装

```bash
go get -u gopkg.in/h2non/gock.v1
```

### 使用示例

使用`gock`对外部API进行mock，即mock指定参数返回约定好的响应内容。 下面的代码中mock了两组数据，组成了两个测试用例。

```go
// api_test.go
package gock_demo

import (
	"testing"

	"github.com/stretchr/testify/assert"
	"gopkg.in/h2non/gock.v1"
)

func TestGetResultByAPI(t *testing.T) {
	defer gock.Off() // 测试执行后刷新挂起的mock

	// mock 请求外部api时传参x=1返回100
	gock.New("http://your-api.com").
		Post("/post").
		MatchType("json").
		JSON(map[string]int{"x": 1}).
		Reply(200).
		JSON(map[string]int{"value": 100})

	// 调用我们的业务函数
	res := GetResultByAPI(1, 1)
	// 校验返回结果是否符合预期
	assert.Equal(t, res, 101)

	// mock 请求外部api时传参x=2返回200
	gock.New("http://your-api.com").
		Post("/post").
		MatchType("json").
		JSON(map[string]int{"x": 2}).
		Reply(200).
		JSON(map[string]int{"value": 200})

	// 调用我们的业务函数
	res = GetResultByAPI(2, 2)
	// 校验返回结果是否符合预期
	assert.Equal(t, res, 202)

	assert.True(t, gock.IsDone()) // 断言mock被触发
}
```

执行上面写好的单元测试，看一下测试结果。

```bash
❯ go test -v
=== RUN   TestGetResultByAPI
--- PASS: TestGetResultByAPI (0.00s)
PASS
ok      golang-unit-test-demo/gock_demo 0.054s
```

测试结果和预期的完全一致。

在这个示例中，为了让大家能够清晰的了解`gock`的使用，我特意没有使用表格驱动测试。给大家留一个小作业：自己动手把这个单元测试改写成表格驱动测试的风格，就当做是对最近两篇教程的复习和测验。

## 总结

在日常工作开发中为代码编写单元测试时如何处理外部依赖是最常见的问题，本文介绍了如何使用`httptest`和`gock`工具mock相关依赖。 在下一篇中，我们将更进一步，详细介绍针对依赖MySQL和Redis的场景如何编写单元测试。
