# 彻底搞懂GOROOT、GOPATH、PATH、mod管理和gopath管理项目的区别

## 1、GOPATH 和 GOROOT

不同于其他语言，go中没有项目的说法，只有包, 其中有两个重要的路径，GOROOT 和 GOPATH

Go开发相关的环境变量如下：

GOROOT：GOROOT就是Go的安装目录，（类似于java的JDK）
GOPATH：GOPATH是我们的工作空间,保存go项目代码和第三方依赖包
GOPATH可以设置多个，其中，第一个将会是默认的包目录，使用 go get 下载的包都会在第一个path中的src目录下，使用 go install时，在哪个GOPATH中找到了这个包，就会在哪个GOPATH下的bin目录生成可执行文件

## 2、修改 GOPATH 和 GOROOT

GOROOT
GOROOT是Go的安装路径。GOROOT在绝大多数情况下都不需要修改

Mac中安装Go会自动配置好GOROOT，路径为`/usr/local/go`。Win中默认的GOROOT是在 `C:\Go`中，也可自己指定

【如下图所示则我的GORROT为：`D:\development\go`】，以下是GOROOT目录的内容：



可以看到GOROOT下有bin，doc和src目录。bin目录下有我们熟悉的go和gofmt工具。可以认为GOOROOT和Java里的JDK目录类似。

GOPATH
GOPATH是开发时的工作目录。用于：

保存编译后的二进制文件。
go get和go install命令会下载go代码到GOPATH。
import包时的搜索路径
使用GOPATH时，GO会在以下目录中搜索包：

GOROOT/src：该目录保存了Go标准库代码。
GOPATH/src：该目录保存了应用自身的代码和第三方依赖的代码。
假设程序中引入了如下的包：

`import "Go-Player/src/chapter17/models"`
第一步：Go会先去GOROOT的scr目录中查找，很显然它不是标准库的包，没找到。
第二步：继续在GOPATH的src目录去找，准确说是GOPATH/src/Go-Player/src/chapter17/models这个目录。如果该目录不存在，会报错找不到package。在使用GOPATH管理项目时，需要按照GO寻找package的规范来合理地保存和组织Go代码。

## 3、HelloWord——GOPATH版

### （1）设置并查看GOPATH和GOROOT环境变量

安装go SKD目录：`D:\development\go`
go项目存放目录：`D:\development\jetbrains\goland\workspace`，并且此目录下含有bin、src、pkg三个文件夹，src文件夹用来存放项目代码
当引入module时，首先在GOROOT的src目录下查找，然后再GPOPATH的src目录下查找



### （2）GOLand环境配置

在`D:\development\jetbrains\goland\workspace\src`目录下新建项目GO-Player
bin：存放编译后的exe文件

pkg：存放自定义包的目录

src：存放项目源文件的目录



按如下指令进行配置


可在Settings中选择SDK和添加GOPATH

### （3）测试

```go
models：Student.go

main：hello.go
package main

import (
	//"./models"  //相对路径
	"Go-Player/src/ademo/models"  //根据GOPATH找
    //根据GOPATH：D:\development\jetbrains\goland\workspace，在其src目录下查找
    //即GOPATH/src/Go-Player/src/ademo/models
	"fmt"
)

func main() {
	stu := models.Student{
		Name: "张三",
	}
	fmt.Println(stu)
}
```

此篇文章仅介绍网上大部分GOPATH版本。Go语言Hello World都只简单地介绍了GOPATH版本。但是从Go的1.11版本之后，已不再推荐使用GOPATH来构建应用了。也就是说GOPATH被认为是废弃的，错误的做法。

## 4、一些踩坑经验

当你开启了GO111MODULE，仍然使用GOPATH模式的方法，在引入自定义模块时会报错。go mod具体使用将在下一篇介绍

```txt
GO111MODULE 有三个值：off, on和auto（默认值）。

GO111MODULE=off，go命令行将不会支持module功能，寻找依赖包的方式将会沿用旧版本那种通过vendor目录或者GOPATH模式来查找。
GO111MODULE=on，go命令行会使用modules，而一点也不会去GOPATH目录下查找。
GO111MODULE=auto，默认值，go命令行将会根据当前目录来决定是否启用module功能。这种情况下可以分为两种情形：
当前目录在GOPATH/src之外且该目录包含go.mod文件，即使用go mod对项目的第三方依赖进行管理，不再使用gopath的方式
当前文件在包含go.mod文件的目录下面。
```

当modules 功能启用时，依赖包的存放位置变更为$GOPATH/pkg，允许同一个package多个版本并存，且多个项目可以共享缓存的 module。

### （1）使用了了相对路径：import "./models" 

报错：`build command-line-arguments: cannot find module for path _/D_/dev`这里后面一堆本地路径
这是因为在go module下 你源码中 `impot …/ `这样的引入形式不支持了， 应该改成 impot 模块名/ 。 这样就ok了

### （2）使用结合了GOPATH的形式：`import "Go-Player/src/ademo/models"` 

于是我们把上面的import改成了结合GOPATH的如上形式

报错：`package Go-Player/src/ademo/models is not in GOROOT D:/development/go/src/GPlayer/src/ademo/models`

### （3）彻底解决方法：用go env -u 恢复初始设置

不再使用go mod：

`go env -w GO111MODULE=off`  或者  `go env -w GO111MODULE=auto`
`go env -u GO111MODULE`
      区别在于，如果GO111MODULE=on或者auto，在go get下载包时候，会下载到GOPATH/pkg/mod，引入时也是同样的从这个目录开始。如果这行了上述命令，那么在go get下载包时候，会下载到GOPATH/src 目录下

## 4、最佳实践

- 如果在GOPATH目录下的src文件下编写项目代码，可以不使用mod管理方式，直接沿用旧版本的GOPATH管理；

- 如果在GOPATH以外的任意目录下要编译并运行项目代码，为了方便管理第三方依赖，使用go mod进行管理。

  ```txt
  #初始化并创建以“当前项目根目录”为moudles名称的go.mod文件
  go mod init 当前项目根目录
  #剔除无用包，拿取有用包，准备代码所需环境（第三方依赖）
  go mod tidy
  #编译运行即可
  #注意：当前mod文件里的moudles名称必须与当前项目目录名保持一致，否则会报错！
  ```

  


