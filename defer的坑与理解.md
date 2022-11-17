# defer的坑与理解

defer语句是Go中一个非常有用的特性，可以将一个方法延迟到包裹该方法的方法返回时执行，在实际应用中，defer语句可以充当其他语言中try…catch…的角色，也可以用来处理关闭文件句柄等收尾操作。

```text
defer的机制：将defer后的函数或方法压入栈内，
每个defer顺序压入，这时被压入的函数和方法以及
其后的语句不会执行，直到遇到以下三种情况会实现
出栈并执行defer后的方法或函数：
①执行到函数或方法返回值；
②执行到函数或方法末尾
③遇到panic时
```

## defer触发时机

> A "defer" statement invokes a function whose execution is deferred to the moment the surrounding function returns, either because the surrounding function executed a return statement, reached the end of its function body, or because the corresponding goroutine is panicking.

Go官方文档中对defer的执行时机做了阐述，分别是。 - 包裹defer的函数返回时 - 包裹defer的函数执行到末尾时 - 所在的goroutine发生panic时

## defer执行顺序

当一个方法中有多个defer时， defer会将要延迟执行的方法“压栈”，当defer被触发时，将所有“压栈”的方法“出栈”并执行。所以defer的执行顺序是LIFO的。

所以下面这段代码的输出不是1 2 3，而是3 2 1。

```go
func stackingDefers() {
    defer func() {
        fmt.Println("1")
    }()
    defer func() {
        fmt.Println("2")
    }()
    defer func() {
        fmt.Println("3")
    }()
}
```

## 坑1：defer在匿名返回值和命名返回值函数中的不同表现

先看下面两个方法执行的结果。

```go
func returnValues() int {
    var result int
    defer func() {
        result++
        fmt.Println("defer")
    }()
    return result
}

func namedReturnValues() (result int) {
    defer func() {
        result++
        fmt.Println("defer")
    }()
    return result
}
```

# Golang中defer、return、返回值之间执行顺序的坑

# 前言

Go语言中延迟函数defer充当着 try…catch 的重任，使用起来也非常简便，然而在实际应用中，很多gopher并没有真正搞明白defer、return和返回值之间的执行顺序，从而掉进坑中，今天我们就来揭开它的神秘面纱！

先来运行下面两段代码：

A. 匿名返回值的情况

```go
package main

import (
	"fmt"
)
func main() {
	fmt.Println("a return:", a()) // 打印结果为 a return: 0
}
func a() int {
	var i int
	defer func() {
	      i++
	      fmt.Println("a defer2:", i) // 打印结果为 a defer2: 2
	}()
	defer func() {
		i++
		fmt.Println("a defer1:", i) // 打印结果为 a defer1: 1
	}()
	return i
}
```

B. 有名返回值的情况

```go
package main
import (
	"fmt"
)
func main() {
	fmt.Println("b return:", b()) // 打印结果为 b return: 2
}
func b() (i int) {
	defer func() {
		i++
		fmt.Println("b defer2:", i) // 打印结果为 b defer2: 2
	}()
	defer func() {
		i++
		fmt.Println("b defer1:", i) // 打印结果为 b defer1: 1
	}()
	return i // 或者直接 return 效果相同
}
```

# 结论

先来假设出结论（这是正确结论），帮助大家理解原因：

- 多个defer的执行顺序为“后进先出”；
- 所有函数在执行RET返回指令之前，都会先检查是否存在defer语句，若存在则先逆序调用defer语句进行收尾工作再退出返回；
- 匿名返回值是在return执行时被声明，有名返回值则是在函数声明的同时被声明，因此在defer语句中只能访问有名返回值，而不能直接访问匿名返回值；
- return其实应该包含前后两个步骤：第一步是给返回值赋值（若为有名返回值则直接赋值，若为匿名返回值则先声明再赋值）；第二步是调用RET返回指令并传入返回值，而RET则会检查defer是否存在，若存在就先逆序插播defer语句，最后RET携带返回值退出函数；
- 因此，‍‍defer、return、返回值三者的执行顺序应该是：return最先给返回值赋值；接着defer开始执行一些收尾工作；最后RET指令携带返回值退出函数。

## 如何解释两种结果的不同：

上面两段代码的返回结果之所以不同，其实从上面的结论中已经很好理解了。

a()int 函数的返回值没有被提前声明，其值来自于其他变量的赋值，而defer中修改的也是其他变量（其实该defer根本无法直接访问到返回值），因此函数退出时返回值并没有被修改。

b()(i int) 函数的返回值被提前声明，这使得defer可以访问该返回值，因此在return赋值返回值 i 之后，defer调用返回值 i 并进行了修改，最后致使return调用RET退出函数后的返回值才会是defer修改过的值。

C. 下面我们再来看第三个例子，验证上面的结论：

```go
package main
import (
	"fmt"
)
func main() {
	c:=c()
	fmt.Println("c return:", *c, c) // 打印结果为 c return: 2 0xc082008340
}
func c() *int {
	var i int
	defer func() {
		i++
		fmt.Println("c defer2:", i, &i) // 打印结果为 c defer2: 2 0xc082008340
	}()
	defer func() {
		i++
		fmt.Println("c defer1:", i, &i) // 打印结果为 c defer1: 1 0xc082008340
	}()
	return &i
}
```

虽然 c()*int 的返回值没有被提前声明，但是由于 c()*int 的返回值是指针变量，那么在return将变量 i 的地址赋给返回值后，defer再次修改了 i 在内存中的实际值，因此return调用RET退出函数时返回值虽然依旧是原来的指针地址，但是其指向的内存实际值已经被成功修改了。

即，我们假设的结论是正确的！

# 补充

------------------------------------补充--------------------------------------

D.defer声明时会先计算确定参数的值，defer推迟执行的仅是其函数体。

```go
package main
import (
	"fmt"
	"time"
)
func main() {
	defer P(time.Now())
	time.Sleep(5e9)
    fmt.Println("1", time.Now())
}
func P(t time.Time) {
	fmt.Println("2", t)
	fmt.Println("3", time.Now())
}
// 输出结果：
// 1 2017-08-01 14:59:47.547597041 +0800 CST
// 2 2017-08-01 14:59:42.545136374 +0800 CST
// 3 2017-08-01 14:59:47.548833586 +0800 CST
```

E. defer的作用域

1. defer只对当前协程有效（main可以看作是主协程）；
2. 当panic发生时依然会执行当前（主）协程中已声明的defer，但如果所有defer都未调用recover()进行异常恢复，则会在执行完所有defer后引发整个进程崩溃；
3. 主动调用os.Exit(int)退出进程时，已声明的defer将不再被执行。

上面的方法会输出0，下面的方法输出1。上面的方法使用了匿名返回值，下面的使用了命名返回值，除此之外其他的逻辑均相同，为什么输出的结果会有区别呢？

要搞清这个问题首先需要了解defer的执行逻辑，文档中说defer语句在方法返回“时”触发，也就是说return和defer是“同时”执行的。以匿名返回值方法举例，过程如下。

- 将result赋值给返回值（可以理解成Go自动创建了一个返回值retValue，相当于执行retValue = result）
- 然后检查是否有defer，如果有则执行
- 返回刚才创建的返回值（retValue）

在这种情况下，defer中的修改是对result执行的，而不是retValue，所以defer返回的依然是retValue。在命名返回值方法中，由于返回值在方法定义时已经被定义，所以没有创建retValue的过程，result就是retValue，defer对于result的修改也会被直接返回。

## 坑2：在for循环中使用defer可能导致的性能问题

看下面的代码

```go
func deferInLoops() {
    for i := 0; i < 100; i++ {
        f, _ := os.Open("/etc/hosts")
        defer f.Close()
    }
}
```

defer在紧邻创建资源的语句后生命力，看上去逻辑没有什么问题。但是和直接调用相比，defer的执行存在着额外的开销，例如defer会对其后需要的参数进行内存拷贝，还需要对defer结构进行压栈出栈操作。所以在循环中定义defer可能导致大量的资源开销，在本例中，可以将f.Close()语句前的defer去掉，来减少大量defer导致的额外资源消耗。

## 坑3：判断执行没有err之后，再defer释放资源

一些获取资源的操作可能会返回err参数，我们可以选择忽略返回的err参数，但是如果要使用defer进行延迟释放的的话，需要在使用defer之前先判断是否存在err，如果资源没有获取成功，即没有必要也不应该再对资源执行释放操作。如果不判断获取资源是否成功就执行释放操作的话，还有可能导致释放方法执行错误。

正确写法如下。

```go
resp, err := http.Get(url)
// 先判断操作是否成功
if err != nil {
    return err
}
// 如果操作成功，再进行Close操作
defer resp.Body.Close()
```

## 坑4：调用os.Exit时defer不会被执行

当发生panic时，所在goroutine的所有defer会被执行，但是当调用os.Exit()方法退出程序时，defer并不会被执行。

```go
func deferExit() {
    defer func() {
        fmt.Println("defer")
    }()
    os.Exit(0)
}
```

上面的defer并不会输出。