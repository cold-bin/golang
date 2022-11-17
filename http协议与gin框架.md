@[TOC](http协议与gin框架)

## [一、Goland搭建gin框架](https://www.jb51.net/article/199466.htm#_label2"出处")

### 1. 安装go软件包

下载地址：https://studygolang.com/dl

![img](https://img.jbzj.com/file_images/article/202011/2020111111363252.png)

下载后，双击安装即可。



### 2. 配置系统变量

这里需要配置2个系统变量，一个是GOROOT，一个是GOPATH

![img](https://img.jbzj.com/file_images/article/202011/2020111111363253.png)

![img](https://img.jbzj.com/file_images/article/202011/2020111111363254.png)

**注意：GOROOT和GOPATH不能在同一路径下，且变量名必须是GOROOT和GOPATH.**



3. 安装git

下载地址：https://git-scm.com/

下载后，直接双击安装即可。.

cmd窗口中，执行go get命令时，必须要有git环境的支持。



### 4. 设置go代理

在下载gin框架之前，我们还需要配置go公共代理镜像，目的是解决github无法访问或者访问速度慢的问题，在cmd窗口中执行命令：

```
go ``env` `-w GOPROXY=https:``//goproxy``.io,direct
```

如图：

![img](https://img.jbzj.com/file_images/article/202011/2020111111363255.png)

这样设置后，我们的go命令将会从公共代理镜像中快速拉取代码了。



### 5. 下载gin框架

cmd窗口中执行命令：

```
go get -u github.com``/gin-gonic/gin
```

等待片刻后，在GOPATH目录的src目录下，将会看到一个github.com目录，gin框架相关的所有文件，都在这个目录里面。

**注意点1：**

**如果是在goland的命令行中执行上面的命令，则不需要安装Git，即第3步可以省略，因为goland已经内置了git插件。**

**注意点2：**

**由于网络原因，此处与google相关的包无法访问，需要单独下载。**

在GOPATH目录的src目录下，新建文件夹google.golang.org，然后cmd窗口中，切换到该目录下，执行命令：

```
git clone https:``//github``.com``/protocolbuffers/protobuf-go``.git
```

如图：

![img](https://img.jbzj.com/file_images/article/202011/2020111111363256.png)

下载完成后，将protobuf-go目录重命名为protobuf即可。

![img](https://img.jbzj.com/file_images/article/202011/2020111111363357.png)

**注意点3：**

**由于网络原因，此处与golang.org相关的包也无法下载，解决方式与【注意点2】类似。**

在GOPATH目录的src目录下，新建文件夹golang.org，然后cmd窗口中，切换到该目录下，执行命令：

```
git clone https:``//github``.com``/golang/tools``.git
```

如图：

![img](https://img.jbzj.com/file_images/article/202011/2020111111363358.png)

下载完成后，将tools目录重命名为x即可。

![img](https://img.jbzj.com/file_images/article/202011/2020111111363359.png)

进入x目录，继续执行命令：

```
git clone https:``//github``.com``/golang/crypto``.git
```

如图：

![img](https://img.jbzj.com/file_images/article/202011/2020111111363360.png)



### 6. 创建项目

打开goland软件，新建一个项目，如图：

![img](https://img.jbzj.com/file_images/article/202011/2020111111363361.png)

创建完成后，我们还需要在项目根目录下手动创建 bin，pkg和src三个文件夹。

bin目录：用来存放编译后的exe二进制文件。

pkg目录：用来存放自定义包，也就是import的来源。

src目录：用来存放项目源文件，也就是我们的开发文件。

如图：

![img](https://img.jbzj.com/file_images/article/202011/2020111111363462.png)

然后，在src目录下，我们新建一个go文件，名称为main.go，内容如下：

```
package main``import` `(``  ``"github.com/gin-gonic/gin"``  ``"net/http"``)``func main() {``  ``router := gin.Default()``  ``router.GET(``"/"``, func(c *gin.Context) {``    ``c.String(http.StatusOK,``"Hello！欢迎来到GO世界！"``)``  ``})``  ``//` `默认端口是8080,也可以指定端口 r.Run(``":80"``)``  ``router.Run()``}
```

![img](https://img.jbzj.com/file_images/article/202011/2020111111363463.png)



### 7. 编译部署

7.1 禁用【Go Modules】

![img](https://img.jbzj.com/file_images/article/202011/2020111111363464.png)

7.2 配置调试器

点击goland软件右上角【Add Configuration...】，打开后如图所示：

![img](https://img.jbzj.com/file_images/article/202011/2020111111363465.png)

![img](https://img.jbzj.com/file_images/article/202011/2020111111363466.png)

配置完成后，点击软件右上角的运行或调试图标，即可访问项目。如图：

![img](https://img.jbzj.com/file_images/article/202011/2020111111363567.png)

同时，在bin目录下，会自动生成exe文件，只要把这个exe文件复制到服务器上，然后做成系统服务启动，就可以提供对外访问了。



### 8. 访问

在浏览器地址栏中，输入http://127.0.0.1:8080，即可进行访问。

## [二、http协议](https://mp.weixin.qq.com/s?src=11&timestamp=1634959179&ver=3391&signature=CrgCjTBYblVIdUSmgu1VJcwQq0*39qsComkVWgX5dKl8zmt0ZgvkS4-F*DWBGy4HzXm2xWH4FXm9SiauVbyJfjj8*XAO9Kn9pATGnCNYRYT-8c*YrBG9Bq4NpQk8*5CW&new=1 "来源出处" )

### 1. 综述

超文本传输协议（`HyperText Transfer Protocol，HTTP`），是互联网应用最广泛的网络协议。最初的HTTP协议设计用于发送和接收HTML。经过发展演变成为客户端和服务器进行请求(`Request`)和响应(`Response`)的标准协议。要了解HTTP的运行过程，需要了解大体OSI模型

![图片](https://mmbiz.qpic.cn/mmbiz_png/gjnldtnoHOriap6gMCed0XdsQTt7wcDAEh6cv3YWQgXSGUkxQibCNibMiclAapmcsquoBVMdfTy7iaVRQGH04u8sHzg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

由下到上，OSI模型描述了整个网络架构中从基础设置到上层应用的框架。

HTTP是建立在TCP/IP协议之上的应用层协议，HTTP规定了请求和响应的报文格式，由TCP/IP协议簇进行转发，送达目标(服务器)之后，按照HTTP协议可以解析得到报文的意义，并进行响应处理。

![图片](https://mmbiz.qpic.cn/mmbiz_png/gjnldtnoHOriap6gMCed0XdsQTt7wcDAE0GicoeEncz5hMibeRws8RmuJk0eZmy0QU47ObwW6pJ3hk5EzQgjTH19A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上图描述的是一个简单的HTTP请求过程，其中HTTP主要起到的作用，就是告知客户端和服务器，如何构建报文，如何解析报文，而传输则交给下层`TCP/IP`协议进行。此外还有一些与通信有关系的程序，也做一下介绍：

> - 代理：接收客户端发送的请求后直接转发给其他服务器(可能会被发送到下一个代理)，不会改变URI。（如果有级联的多台代理服务器，当客户端请求到达第一个代理服务器时，该服务器会在自己发出的请求里面添加via头部，并填上自己的相关信息，当下一个代理服务器 收到第一个代理服务器的请求时，会在自己发出的请求里面复制前一个代理服务器的请求的Via头部，并把自己的相关信息加到后面。）缓存代理：请求的内容直接从代理服务器中获取，而不必从源服务器获取。

> - 网关：作用与代理类似，但是可以改变请求的协议，比如讲HTTP协议转换成为私有协议或者其他协议，在两个网关之间传输，达到加密的作用。
> - 隧道：通过和HTTP CONNECT方法，构建服务器和客户端之间的通信线路，可以增加安全线

### 2. HTTP报文的构建：

发出的请求信息包括以下几个

> - 请求行 方法(GET,POST等) URI(统一资源标识符) HTTP/1.1，表示请求URI代表的资源
> - 请求首部字段 Host: www.qiongsong.com Accept-Language: cn Connection: keep-alive ……
> - 空行
> - 其他消息体

请求行和标题必须以作为结尾。空行内必须只有而无其他空格。在HTTP/1.1协议中，所有的请求头，除Host外，都是可选的。
返回的响应消息包括下面的几个部分

> - 协议版本号(HTTP/1.1) 状态码(200、404) 状态码原因短语(OK)
> - 响应主体

HTTP1.1 中的请求方法有以下几种

> - GET: 获取资源
> - POST: 传输实体主体
> - PUT: 传输文件，一般不使用，因为不带验证机制，任何人都可以使用PUT来进行文件上传，可以考虑结合表征状态转移(REST Representational State Transfer)或者增加额外的验证机制来配合使用
> - HEAD: 获取报文首部
> - DELETE: 删除资源，与PUT一样不带有验证机制
> - OPTIONS：询问支持的方法
> - TRACE：追踪路径，让服务器端将之前的请求通信环回路给客户端，主要用于跟踪问题
> - CONNECT：要求用隧道协议链接代理，一般配合(SSL、TLS) 使用。

状态码则主要有以下几种类别，每一种类别又有细分的状态码

> - 1xx消息——请求已被服务器接收，继续处理
> - 2xx成功——请求已成功被服务器接收、理解、并接受
> - 3xx重定向——需要后续操作才能完成这一请求(304为资源未修改，与3XX有一点差异)
> - 4xx请求错误——请求含有词法错误或者无法被执行
> - 5xx服务器错误——服务器在处理某个正确请求时发生错误

### 3. 无状态和Cookie

HTTP 的一种重要的特性就是协议本身是无状态的，也就是服务器在接受同一个用户发起的请求报文的时候，并无法知道这些请求是由同一个用户发送过来，也无法得知这些请求的上下文关系。而我们平时浏览网页特别是需要进行用户验证类的网页，每一次要求用户进行身份验证并不合理。为解决这个问题，就发展出Cookie技术。

Cookie技术通过在请求和响应报文中写入cookie信息来控制客户端的状态。从服务器返回的报文中，通过`set-cookie`首部字段，添加cookie(由服务器生成，并通常做加密处理)，客户端收到返回报文之后会保存cookie，当下一次客户端向服务器发送请求的时候，会在请求中加入cookie值后发送。服务器获取到cookie之后，会对比之前服务器上的记录(session)，从而对请求进行状态识别。
cookie具有长度限制，不同的浏览器之间有细微的差异，一般不能超过4097个字节，超过这个数量的cookie在请求的时候，浏览器会直接忽略。

对于 Cookie 的安全保护是 Web 安全的一个重要因素，常见的将 Cookie 设置为 Httponly，避免js 脚本直接获取 Cookie，同时采用加密传输方式来传输内容，避免中间人攻击。

### 4. 性能

性能对于传输协议而言至关重要，剥离底层 TCP 协议对传输性能的影响，在 HTTP 层面对性能影响的点有

- 浏览器：浏览器对于请求的发起可能会限制对同一域名的连接数量，导致并发请求被堵塞。
- DNS：域名到 IP 的转换速度，通常通过 DNS 缓存来解决
- 建立连接：HTTP 需要通过 TCP 三次握手建立连接，连接的高效复用，是影响 HTTP 性能的关键因素。

现在最流行的 HTTP1.1 对于程序员而言，相比于 HTTP1.0 的最大改进在于对长连接的支持，HTTP1.1 默认支持长连接和请求流水线，可以在一个 TCP 连接上传输多个 HTTP 请求，减少建立连接的延迟，体现在报文中是 Connection:keep-alive 的默认开启。

而对于内存，通过引入过期时间等，可以避免重复请求，提高相应速度。

### 5. HTTPs

HTTP 在网络上传输采用的是明文信息，可能会被恶意截获从而产生安全问题。为了解决 HTTP 的传输安全性，增加了一个 SSL （Secure Socket Layer）层，HTTPs = HTTP + SSL。

下面简单描述一下 HTTPs 是如何保证数据传输安全的。为了保证数据传输不被截获，那么需要对数据进行加密，传输过程中采用密文进行传输。加密的方法有两类：一类是对称加密，密钥只有一个，内容通过密钥加密后，只能通过这个密钥解密；另一类是非对称加密，密文通过公钥加密后，通过另一个私钥解密，密钥是成对出现的。对称加密算法效率较高，非对称加密算法较为复杂同时效率也比较低。因此 HTTPs 将两者结合起来，实现了在网络中的安全传输，大致原理是，首先通过非对称加密算法协商对称加密的密钥，后续通过协商出来的密钥进行数据的加解密和传输。

![图片](https://mmbiz.qpic.cn/mmbiz_png/gjnldtnoHOriap6gMCed0XdsQTt7wcDAEbwr6Tu6GacMFBmRotSc7zTOy0URs7IvFAibcoRtYXdmP406OA6HHVkw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

其中有几点要补充说明

- 为什么客户端和服务端都需要生成随机数

SSL协议不信任每个主机都能产生完全随机的随机数，如果随机数不随机，那么pre-master secret就有可能猜测出来，那么仅适用pre-master secret作为密钥就不合适了，因此必须引入新的随机因素，那么客户端随机数和服务器随机数加上pre-master secret三个随机数一同生成的密钥就不容易被猜出了，一个伪随机数可能完全不随机，可是三个伪随机数就十分接近随机了。

- Change Cipher Spec 的意思

意思是告知对方，采用协商后的通信密钥进行加密通信，之后 Encrypted Handshake Message 是发送的第一个加密数据。内容就是把当前（准备发送Encrypted Handshake Message）前，自己收到的数据和发送的数据进行一次简单运算 (hash+加密)，在解密握手信息的时候，需要双方密钥一致，并且接收方也需要模拟计算握手信息，校验整个通信数据没有被篡改，

- HTTPs 是否绝对安全

绝对安全是一个相对说法，上面的 HTTPs 的机制是单项认证，也就是保证客户端只能访问正确的服务器（通过证书保证）但是无法保证服务器只能被正确的客户端访问，要做到这一点需要增加双项认证，也就是客户端也需要提供自己的证书。客户端校验服务端证书，服务端也校验客户端证书，这在金融行业等会引入，例如网银的USB密钥。相对安全性会高很多。

- HTTPs 可以防御中间人攻击和篡改吗

是的，HTTPs 的主要功能就是这个，但是需要严格管理 CA 证书，安全问题是一步错、步步错。

- HTTPS 可以防御重放攻击吗

可以，每一个通信的 Socket 都会协商产生随机数，除非可以完全复制客户端的 Socket 否则无法进行重放攻击。如果是重复提交，需要在系统端进行幂等保证。

`soursce:https://fzsens.github.io/web/2015/03/07/brief-http/`

## 三、gin框架

### [1. a easy example](https://www.bilibili.com/video/BV1WL4y1h7EW?p=4&share_source=copy_web "gin框架快速入门"):

```go
package main
import (
"github.com/gin-gonic/gin"
"net/http"
)
func main() {
    //使用gin默认中间件创建一个gin路由器
    //默认中间件使用了logger()和recovery()中间件
    //logger中间件用于日志的打印，recovery中间件用
    //于程序遇到错误时返回客户端一些信息。
    //也可以使用gin.New()创建，但是没有logger和recovery中间件，无法获得一些信息的反馈
	router := gin.Default()
    //GET方法里第一个参数为url，一般访问ip后添加这个url，可以指定访问哪个Get
	router.GET("/strong", func(c *gin.Context) {
		c.String(http.StatusOK,"难呐呐呐呐呐-")
	})
    router.GET("/", func(c *gin.Context) {
		c.String(http.StatusOK,"东大东大大大大~")
	})
	// 默认端口是8080,也可以指定端口 r.Run(":8081")
	err := router.Run()
	if err != nil {
		panic(err)
	}
}
```

###  2. gin的路由分组

当使用的路由变多，为了管理的方便采取路由分组。

```go
package main

import (
	"github.com/gin-gonic/gin"
	"log"
	"net/http"
)

func main()  {
	//创建路由器
	router:= gin.Default()
	//路由分组
    //使用大括号括起
	routerGroup := router.Group("/v1")
	{
         //访问url：http://127.0.0.1:8081/v1/
		routerGroup.GET("/", func(c *gin.Context) {
			c.String(200,"空")
		})
        //访问url：http://127.0.0.1:8081/v1/login
		routerGroup.GET("/login", func(c *gin.Context) {
			c.String(http.StatusOK,"登陆成功~")
		})
		routerGroup.GET("/register", func(c *gin.Context) {
			c.String(http.StatusOK,"注册成功~")
		})
	}
	err := router.Run(":8081")
	if err != nil {
		log.Println(err.Error())
	}
}
```

### 3. 获取url里的变量

```go
import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main()  {
	router:= gin.Default()
	routerGroup := router.Group("/goods")
	{
		//url里【/:name/:price】表示取变量，这样在输入的时候，可以无限制使用任意字符都可以访问到该url
		routerGroup.GET("/:name/*price", func(c *gin.Context) {
			//获取参数，方法里面的值，需要和【/:name】或【/*price】里的字符【name】或【price】相同，这样才能获取数据
			//总之，一方面可以使用url的特殊格式可以实现某种访问目的，另一方面，也可以动态获取里面的数据
			name := c.Param("name")
			price:= c.Param("price")
			//返回数据（json渲染）
			c.JSON(http.StatusOK,gin.H{
				"name":name,
				"price":price,
			})
		})
		//url里【/*name】表示后面填写任意数据都可以进行访问。但是后面如果有其他【诸如/:】，
		//则读取开始位置到此处结束，后面继续匹配
		/*routerGroup.GET("/*number", func(c *gin.Context) {
			number:= c.Param("number")
			//返回数据（json渲染）
			c.JSON(http.StatusOK,gin.H{
				"number":number,
			})
		})*/
	}
	router.Run(":8081")
}
```

我们可以通过【/:name】和【/*name】配合context的Param（）方法使用任意字符实现对该url的访问。但是问题，也显现：任意输入都可进行访问，某些输入是需要指定类型的，那么无限制的输入就成了非法输入。

**通过参数绑定及反射实现数据传入限制（获取url变量最佳实践）**

```go
import (
	"github.com/gin-gonic/gin"
)

type person struct {
	//限制：required指必须存在，uuid：数据为这个格式
	//uri和url里的变量name、id名字保持一致
	id string `uri:"id" binding:"required,uuid"`
	name string `uri:"name" binding:"required"`
}

func main() {
	router:= gin.Default()
	var person person
	router.GET("/:name/:id", func(c *gin.Context) {
		/*person.name=c.Param("name")
		person.id=c.Param("id")*/
		//绑定参数
		if err:=c.ShouldBindUri(&person);err!=nil{
			c.Status(404)
			return
		}
		c.JSON(200,gin.H{
			"name":person.name,
			"id":person.id,
		})
	})
	err := router.Run(":8082")
	if err != nil {
		panic(err)
	}
}
```

### 4. 获取参数

#### 1）获取GET\POST参数和混合获取参数

```go
import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router:=gin.Default()
	router.GET("/get",getMSG)
	router.POST("/post",postMSG)
	router.POST("/getAndPost",getPostMSG)
	err := router.Run(":8081")
	if err != nil {
		return
	}
}

func getPostMSG(c *gin.Context) {
	id:=c.DefaultPostForm("id", "23333~")
	name := c.PostForm("name")
	firstName:= c.DefaultQuery("firstName", "admin")
	secondName := c.Query("secondName")
	/*c.JSON(http.StatusOK,gin.H{
		"firstName":firstName,
		"secondName": secondName,
		/*"id": id,
		"name":name,*/
	fmt.Println(id,"\t",name,"\t",firstName,"\t",secondName)
}

func postMSG(c *gin.Context) {
	//获取post请求里的参数
	id := c.DefaultPostForm("id", "2020214341")
	name:= c.DefaultPostForm("name", "阿冰")
	/*//json渲染
	c.JSON(http.StatusOK,gin.H{
		"id":id,
		"name":name,
	})*/
	fmt.Println("id",id," ","name",name)
}

func getMSG(c *gin.Context) {

	//从GET请求里获取参数：http://127.0.0.1:8081/get?firstName=lhb&secondName=ab
	//【?firstName=lhb&secondName=ab】用于参数添加，获取get参数时，需要指定相同key
	firstName := c.DefaultQuery("firstName", "firstName")
	secondName := c.DefaultQuery("secondName", "secondName")
	//获取参数json渲染
	c.JSON(http.StatusOK,gin.H{
		"firstName":firstName,
		"secondName":secondName,
	})
}
```

**注意：**POST请求获取步骤：

```markdown
- post指的是服务器向客户端发出响应
	1.打开【Postman】软件。

    2.请求方式选择【Post】，右侧输入【请求URL】。

	3.在请求头中输入【请求头的数据】。

	4.在Body请求头中输入【请求头的数据】。

	5.点击【Send】选项发送请求。

	6.在下方中可以看到响应的数据。
```

### 5. 重定向

客户端请求到达后，将请求转发到指定的其他web服务

#### 1）HTTP重定向

- - 实现

    `ctx.Redirect()`

```go
func main() {
	router := gin.Default()
	router.GET("/test", func(c *gin.Context) {
		c.Redirect(http.StatusMovedPermanently, "http://www.baidu.com/")
	})
	router.Run(":8000")//貌似端口为8081、8080等无法访问？？？
}
```

- - `http.StatusMovedPermanently`为301状态码，即永久移动

#### 1）路由重定向

- - 实现

- - - `ctx.Request.URL.Path`指定跳转路径
    - `router.HandleContext`进行重定向

- - - 示例

```go
func main() {
	router := gin.Default()
	router.GET("/truck", func(c *gin.Context) {
		// 指定重定向的URL
		c.Request.URL.Path = "/differentWorld"
		router.HandleContext(c)
	})
    
	router.GET("/differentWorld", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"message": "呐呐呐！你被传送到了异世界哦~"})
	})
	router.Run(":8080")
}
```

### 6. 中间件

中间件命名以`Middleware`为例，

**多个中间件执行顺序**

![image-20211024155832457](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20211024155832457.png)

![image-20211024160121854](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20211024160121854.png)

![image-20211024160036893](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20211024160036893.png)

#### 1） 为全局路由注册中间件

```go
router.Use(Middleware())
//Use里的中间件（可以有多个）是按照顺序执行的：从左至右依次执行中间件。
//中间件调用next函数时，表示直接跳转到本中间件后面的中间件或者后续其他函数。。。
```

#### 2）为某个路由注册中间件

```go
router.GET("/differentWorld", Middleware, func(c *gin.Context) {
    //get请求默认第一个参数为RelativePath，后续所有参数都为handlerFunc，也是顺序执行的机制
		c.JSON(http.StatusOK, gin.H{"message": "呐呐呐！你被传送到了异世界哦~"})
	})
```

以Cookie鉴权为例

#### 3）设置Cookie

**注意对同一个cookie对象的检查和设置，只能在同一个路由，而且顺序只能是: 1. 设置cookie->2. 检查cookie**

- - 利用方法

```go
cookie := &http.Cookie{}
http.SetCookie()
```

- - 示例

```go
router.GET("/login", func(c *gin.Context) {
    //从get请求获取参数
		account := c.Query("account")
		password := c.Query("password")
		cookie := &http.Cookie{
			Name:     account,
			Value:    password,
			Path:     "/",
			HttpOnly: true,
		}
		http.SetCookie(c.Writer, cookie)
		c.String(http.StatusOK, "Login successful")
	})
```

#### 4）检查Cookie

- - 利用方法

```go
c.Request.Cookie("session_id") // 获取cookie
cookie.Value // 获取cookie值
```

- - 示例

```go
func Middleare() gin.HandlerFunc {
    return func(c *gin.Context) {
        
        cookie, err := c.Request.Cookie("account")
        if err == nil {
            value := cookie.Value
            fmt.Println(value)
            if value == "123" {
                c.Next() // 调用该请求的后续函数
                return
            }
        }
        c.JSON(http.StatusUnauthorized, gin.H{
            "error": "",
        })
        c.Abort() // 不执行该请求的后续函数
        return
    }
}
```

### 附：文件上传简单前端示例代码

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <title>上传文件示例</title>
</head>
<body>
<form action="/upload" method="post" enctype="multipart/form-data">
    <input type="file" name="f1">
    <input type="submit" value="上传">
</form>
</body>
</html>
```

## 三、基于gin框架和gorm的web开发实战 

### 1. template初识

