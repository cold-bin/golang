# Go爬虫

- 爬虫就是模拟客户端程序访问服务端某个`url`下的资源，再将资源进行部分提取（使用正则筛选`html`等），然后在存储至某端，或者放在某网页。
- 流程：
  - 明确目标`url`
  - 发送请求，获取应答数据包
  - 保存、过滤数据。提取有用信息（考虑使用正则表达式对`html`元素进行筛选）
  - 使用、分析得到的数据信息

## 一、入门案例

```go
import (
	"fmt"
	"net/http"
)

//该例子用于简单的模拟了浏览器的请求，即爬虫原理
func main() {
	//创建一个http请求
	req, _ := http.NewRequest("GET", "https://www.baidu.com/", nil)
	//编写请求头
	req.Header.Add("UserAgent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.97 Safari/537.36")
	//初始化一个客户端
	client := &http.Client{}
	//客户端发起请求
	resp, err := client.Do(req)
	if err != nil {
		return
	}
	//获得回应
	fmt.Println(resp)
}
```

## 二、正则表达式

[正则表达式教程](https://www.runoob.com/regexp/regexp-tutorial.html)

## 三、常用爬虫正则表达式

### 1. 爬取`html`标签里的内容

```html
<div class="navbar-header">
    <a href="/" class="navbar-brand" title="Go语言中文网"><img alt="Go语言中文网" src="https://static.studygolang.com/static/img/logo.png" style="margin-top: -7px; height: 45px;">
    </a>
	<button class="navbar-toggle" type="button" data-toggle="collapse" data-target="#navbar-main">
					<span class="icon-bar"></span>
					<span class="icon-bar"></span>
					<span class="icon-bar"></span>
	</button>
</div>
```

```go
//此正则表示匹配div标签里的所有数据，及时里面是空
//?s->是正则表达式的模式修饰符，即单行模式。表示更改【.】的含义，使其与每一个字符匹配，包括换行符。
//*？表示非贪婪，即重复>=0次匹配x，越少越好（优先跳出重复）
//
html := "<a>hhh</a><div>你好，我来自哪里？\n\r我来自火星。</div></br><div>你好，我来自哪里？\n\n我来自域外。</div></br><div>你好，我来自哪里？\n\n我来自你不知道的地方。</div><div></div>"
regex := `<div>(?s:(.+?))</div>`
c, err := regexp.Compile(regex)
	if err != nil {
		fmt.Println(fmt.Sprint(err))
		return
	}
fmt.Println(c.FindAllStringSubmatch(html, -1))
//<div>?s:(.+?)</div>只表示一次正则提取。
//<div>(?s:(.+?))</div>表示先提取整个表达式匹配的，然后再提取括号内满足的表达式

```

### 2. 双向爬取

#### ①横向爬取

所谓横向爬取，是指在爬取的网站页面中，以“页”为单位，找寻该网站的分页规律。一页一页的爬取网站数据信息。大多数网站，采用分页管理模式。针对这类网站，首先需要确立横向爬取方法

#### ②纵向爬取

纵向爬取，是指在一个网页中，按不同的“条目”为单位，找寻个条目之间的规律。一条一条的爬取一个网页的数据信息。也就是爬取一个页面内不同类别数据

## 四、使用`colly`框架

上文举一个例子，通过`go`的`http`库实现了获取指定网站的`html`数据，但是不是每个网站都能这样爬取，有些网站会有一些防爬措施来辨别访问对象是否是爬虫还是正常客户端。按照正常地流程:伪造客户端，设置合理的请求头属性，提交某种特定格式的参数才能对网站资源进行访问，而且在访问后还需要使用正则表达式对获取的html内容进行筛选。❌

我们简化一下流程，让我们可以更加轻松且快速的使用爬虫：

**使用`colly`框架：分析爬取网页`html`结构；选择合适的`selector`来筛选网页数据**

`colly`框架内部的`OnHtml`方法实现了对html的监听，需要使用`goquery`库实现的`jquery` `go`语法。这样使用特定的选择器可以实现对特定`html`标签元素的监听，达到过滤数据的目的。当然还有提取`xml`标签内容的库

详见[xml的go实现](https://github.com/antchfx/xmlquery)

`goquery`对爬取到的HTML进行选择和查找匹配的内容时，`goquery`的[选择器](https://so.csdn.net/so/search?q=选择器&spm=1001.2101.3001.7020)让解析`html操作更简洁`，而且还有很多不常用但又很有用的选择器，这里总结下，以供参考。

[**以下内容转载自飞雪无情的 `golang` `goquery selector`(选择器) 示例大全**](https://cloud.tencent.com/developer/article/1196783)

如果大家以前做过前端开发，对`jquery`不会陌生，`goquery`类似`jquery`，它是`jquery`的go版本实现。使用它，可以很方便的对HTML进行处理。

## 基于`HTML Element`元素的选择器

这个比较简单，就是基于`a,p`等这些`HTML`的基本元素进行选择，这种直接使用Element名称作为选择器即可。比如`dom.Find("div")。`

```go
func main() {
	html := `<body>

				<div>DIV1</div>
				<div>DIV2</div>
				<span>SPAN</span>

			</body>
			`
	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("div").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
}
123456789101112131415161718
```

以上示例，可以把`div`元素筛选出来，而`body`,`span`并不会被筛选。

### `ID` 选择器

这个是使用频次最多的，类似于上面的例子，有两个`div`元素，其实我们只需要其中的一个，那么我们只需要给这个标记一个唯一的id即可，这样我们就可以使用id选择器，精确定位了。

```go
func main() {
	html := `<body>

				<div id="div1">DIV1</div>
				<div>DIV2</div>
				<span>SPAN</span>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("#div1").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
}
12345678910111213141516171819
```

### `Element ID` 选择器

`id选择器以#开头`，紧跟着元素id的值，使用语法为`dom.Find(#id),`后面的例子我会简写为`Find(#id)`,大家知道这是代表`goquery`选择器的即可。

```go
func main() {
	html := `<body>

				<div id="div1">DIV1</div>
				<div>DIV2</div>
				<span>SPAN</span>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("#div1").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
}
12345678910111213141516171819
```

如果有相同的`ID`，但是它们又分别属于不同的`HTML`元素怎么办？有好办法，和`Element`结合起来。比如我们筛选元素为`div`,并且`id`是`div1`的元素，就可以使用`Find(div#div1)`这样的筛选器进行筛选。

所以这类筛选器的语法为`Find(element#id)`，这是常用的组合方法，比如后面讲的过滤器也可以采用这种方式组合使用。

### `Class`选择器

```
class`也是`HTML`中常用的属性，我们可以通过`class`选择器来快速的筛选需要的`HTML`元素，它的用法和ID选择器类似，为`Find(".class")。
func main() {
	html := `<body>

				<div id="div1">DIV1</div>
				<div class="name">DIV2</div>
				<span>SPAN</span>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find(".name").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
}
12345678910111213141516171819
```

以上示例中，就筛选出来`class`为`name`的这个`div`元素。

### `Element Class` 选择器

`class`选择器和id选择器一样，也可以结合着`HTML`元素使用，他们的语法也类似`Find(element.class)`，这样就可以`筛选特定element`、并且`指定class的元素`。

### 属性选择器

一个`HTML`元素都有自己的属性以及属性值，所以我们也可以通过属性和值筛选元素。

```go
func main() {
	html := `<body>

				<div>DIV1</div>
				<div class="name">DIV2</div>
				<span>SPAN</span>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("div[class]").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
}
12345678910111213141516171819
```

示例中我们通过`div[class]`这个选择器，筛选出`Element`为`div`并且有`class`这个属性的，所以第一个`div`没有被筛选到。

刚刚上面这个示例是采用是否存在某个属性为筛选器，同理，我们可以筛选出属性为某个值的元素。

```go
    dom.Find("div[class=name]").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
123
```

这样我们就可以筛选出class这个属性值为name的div元素。

当然我们这里以`class`属性为例，还可以用其他属性，比如`href`等很多，自定义属性也是可以的。

除了完全相等，还有其他匹配方式，使用方式类似，这里统一列举下，不再举例

| 选择器                | 说明                                                  |
| --------------------- | ----------------------------------------------------- |
| Find(“div[lang]“)     | 筛选含有lang属性的div元素                             |
| Find(“div[lang=zh]“)  | 筛选lang属性为zh的div元素                             |
| Find(“div[lang!=zh]“) | 筛选lang属性不等于zh的div元素                         |
| Find(“div[lang¦=zh]“) | 筛选lang属性为zh或者zh-开头的div元素                  |
| Find(“div[lang*=zh]“) | 筛选lang属性包含zh这个字符串的div元素                 |
| Find(“div[lang~=zh]“) | 筛选lang属性包含zh这个单词的div元素，单词以空格分开的 |
| Find(“div[lang$=zh]“) | 筛选lang属性以zh结尾的div元素，区分大小写             |
| Find(“div[lang^=zh]“) | 筛选lang属性以zh开头的div元素，区分大小写             |

**`（如果=后面的值是拼接的用单引号包裹即可）`**
`示例`

```go
	func main() {
	//spiderTop250()
	//spoderMoiveInfo("https://movie.douban.com/subject/3011235/")
	html := `<body>
				<script type="application/ld+json">JSDKK</script>
			</body>
			`
	dom, err := goquery.NewDocumentFromReader(strings.NewReader(html))
	if err != nil {
		log.Fatalln(err)
	}

	dom.Find("script[type='application/ld+json']").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
}
```

以上是属性筛选器的用法，都是以一个属性筛选器为例，当然你也可以使用多个属性筛选器组合使用，比如： `Find("div[id][lang=zh]")`,用多个中括号连起来即可。当有多个属性筛选器的时候，要同时满足这些筛选器的元素才能被筛选出来。

### `parent>child`选择器

如果我们想筛选出某个元素下符合条件的子元素，我们就可以使用子元素筛选器，它的语法为`Find("parent>child")`,表示筛选`parent`这个父元素下，符合`child`这个条件的最直接（一级）的子元素。

```go
func main() {
	html := `<body>

				<div lang="ZH">DIV1</div>
				<div lang="zh-cn">DIV2</div>
				<div lang="en">DIV3</div>
				<span>
					<div>DIV4</div>
				</span>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("body>div").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
}
12345678910111213141516171819202122
```

以上示例，筛选出`body`这个父元素下，符合条件的最直接的子元素`div`，结果是`DIV1、DIV2、DIV3`，虽然`DIV4`也是`body`的子元素，但不是一级的，所以不会被筛选到。

那么问题来了，我就是想把`DIV4`也筛选出来怎么办?就是要筛选`body`下所有的`div`元素，不管是一级、二级还是N级。有办法的，`goquery`考虑到了，只需要把`大于号(>)`改为`空格`就好了。比如上面的例子，改为如下选择器即可。

```go
    dom.Find("body div").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
123
```

### `prev+next`相邻选择器

假设我们要筛选的元素没有规律，但是该元素的上一个元素有规律，我们就可以使用这种下一个相邻选择器来进行选择。

```go
func main() {
	html := `<body>

				<div lang="zh">DIV1</div>
				<p>P1</p>
				<div lang="zh-cn">DIV2</div>
				<div lang="en">DIV3</div>
				<span>
					<div>DIV4</div>
				</span>
				<p>P2</p>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("div[lang=zh]+p").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
}
```

这个示例演示了这种用法，我们想选择`<p>P1</p>`这个元素，但是没啥规律，我们发现它前面的`<div lang="zh">DIV1</div>`很有规律，可以选择，所以我们就可以采用`Find("div[lang=zh]+p")`达到选择P元素的目的。

这种选择器的语法是`("prev+next")`，中间是一个加号`(+)`，`+号`前后也是选择器。

### `prev~next`兄弟选择器

有相邻就有兄弟，兄弟选择器就不一定要求相邻了，只要他们共有一个父元素就可以。

```go
	dom.Find("div[lang=zh]~p").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
123
```

刚刚的例子，只需要把`+号`换成`~号`,就可以把`P2`也筛选出来，因为`P2、P1和DIV1`都是兄弟。

兄弟选择器的语法是`("prev~next"),`也就是`相邻选择器的+换成了~`。

### 内容过滤器

有时候我们使用选择器选择出来后后，希望再过滤一下，这时候就用到过滤器了，过滤器有很多，我们先讲内容过滤器这一种。

```go
	dom.Find("div:contains(DIV2)").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
123
```

`Find(":contains(text)")`表示筛选出的元素要包含指定的文本，我们例子中要求选择出的`div`元素要包含`DIV2`文本，那么只有一个`DIV2`元素满足要求。

此外还有`Find(":empty")`表示筛选出的元素都不能有子元素（包括文本元素），只筛选那些不包含任何子元素的元素。

`Find(":has(selector)")`和`contains`差不多，只不过这个是包含的是元素节点。

```go
    dom.Find("span:has(div)").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
123
```

以上示例表示筛选出`包含div元素的span节点。`

### `:first-child`过滤器

`:first-child过滤器`，语法为`Find(":first-child")`，表示筛选出的元素要是他们的父元素的第一个子元素，如果不是，则不会被筛选出来。

```go
func main() {
	html := `<body>

				<div lang="zh">DIV1</div>
				<p>P1</p>
				<div lang="zh-cn">DIV2</div>
				<div lang="en">DIV3</div>
				<span>
					<div style="display:none;">DIV4</div>
					<div>DIV5</div>
				</span>
				<p>P2</p>
				<div></div>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("div:first-child").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Html())
	})
}
```

以上例子中，我们使用`Find("div")`会筛选出所有的div元素，但是我们加了`:first-child`后，就只有`DIV1和DIV4`了，因为只有这两个是他们父元素的第一个子元素，其他的DIV都不满足。

### `:first-of-type`过滤器

`:first-child`选择器限制的比较死，必须得是第一个子元素，如果该元素前有其他在前面，就不能用`:first-child`了，这时候`:first-of-type`就派上用场了，它要求只要是这个类型的第一个就可以，我们把上面的例子微调下。

```go
func main() {
	html := `<body>

				<div lang="zh">DIV1</div>
				<p>P1</p>
				<div lang="zh-cn">DIV2</div>
				<div lang="en">DIV3</div>
				<span>
					<p>P2</p>
					<div>DIV5</div>
				</span>
				<div></div>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("div:first-of-type").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Html())
	})
}
```

改动很简单，把原来的`DIV4`换成了`P2`，如果我们还使用`:first-child`,`DIV5`是不能被筛选出来的，因为它不是第一个子元素，它前面还有一个`P2`。这时候我们使用`:first-of-type`就可以达到目的，因为它要求是同类型第一个就可以。`DIV5`就是这个`div`类型的第一个元素，`P2`不是`div`类型，被忽略。

### `:last-child 和 :last-of-type`过滤器

这两个正好和上面的`:first-child、:first-of-type`相反，表示最后一个，这里不再举例，大家可以自己试试。

### `:nth-child(n) `过滤器

这个表示筛选出的元素是其父元素的第n个元素，n以1开始。所以我们可以知道`:first-child和:nth-child(1)`是相等的。通过指定n，我们就很灵活的筛选出我们需要的元素。

```go
func main() {
	html := `<body>

				<div lang="zh">DIV1</div>
				<p>P1</p>
				<div lang="zh-cn">DIV2</div>
				<div lang="en">DIV3</div>
				<span>
					<p>P2</p>
					<div>DIV5</div>
				</span>
				<div></div>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("div:nth-child(3)").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Html())
	})
}
```

这个示例会筛选出`DIV2`，因为`DIV2`是其父元素`body`的第三个子元素。

### `:nth-of-type(n)` 过滤器

`:nth-of-type(n)和 :nth-child(n)`类似，只不过它表示的是同类型元素的第n个,所以`:nth-of-type(1)`和`:first-of-type`是相等的，大家可以自己试试，这里不再举例。

### `nth-last-child(n) `和`:nth-last-of-type(n) `过滤器

这两个和上面的类似，只不过是倒序开始计算的，最后一个元素被当成了第一个。大家自己测试下看看效果，很明显。

### `:only-child` 过滤器

`Find(":only-child")`过滤器，从字面上看，可以猜测出来，它表示筛选的元素，在其父元素中，只有它自己，它的父元素没有其他子元素，才会被匹配筛选出来。

```go
func main() {
	html := `<body>
				<div lang="zh">DIV1</div>
				<span>
					<div>DIV5</div>
				</span>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("div:only-child").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Html())
	})
}
12345678910111213141516171819
```

示例中`DIV5`就可以被筛选出来，因为它是它的父元素`span`达到唯一子元素，但`DIV1`就不是，所以不能呗筛选出来。

### `:only-of-type` 过滤器

上面的例子，如果想筛选出`DIV1`怎么办？可以使用`Find(":only-of-type")`,因为它是它的父元素中，唯一的`div`元素，这就是`:only-of-type`过滤器所要做的，同类型元素只要只有一个，就可以被筛选出来。大家把上面的例子改成`:only-of-type`试试，看看是否有`DIV1`。

### 选择器或(|)运算

如果我们想同时筛选出`div,span`等元素怎么办？这时候可以采用多个选择器进行组合使用，并且以`逗号(,)分割`，`Find("selector1, selector2, selectorN")`表示，只要满足其中一个选择器就可以被筛选出来，也就是`选择器的或(|)运算操作`。

```go
func main() {
	html := `<body>
				<div lang="zh">DIV1</div>
				<span>
					<div>DIV5</div>
				</span>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("div,span").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Html())
	})
}
```

小结
`goquery` 是解析`HTML`网页必备的利器，在爬虫抓取网页的过程中，灵活的使用`goquery`不同的选择器，可以让我们的抓取工作事半功倍，大大提升爬虫的效率。

选择器问题解决了。再看看关于爬虫的一些设置，以让爬虫可以更好的伪造客户端进行数据爬取。

下面就是一些伪造方法。

## `colly`常见配置及存储

## 1. 爬取简书首页文章列表

通过浏览器开发者工具查看jianshu.com结构如下



![img](https:////upload-images.jianshu.io/upload_images/15604519-559c5b754294bf04.png?imageMogr2/auto-orient/strip|imageView2/2/w/593/format/webp)

colly-jianshu-dom.png

文章列表为ul标签，中间每一项是li标签，li中包含content，content中包含title，abstract和meta标签



```go
package main

import (
    "fmt"
    "github.com/gocolly/colly"
    "github.com/gocolly/colly/debug"
)

func main() {
    c := colly.NewCollector(colly.UserAgent("Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.163 Safari/537.36"), colly.MaxDepth(1), colly.Debugger(&debug.LogDebugger{}))
        //文章列表
    c.OnHTML("ul[class='note-list']", func(e *colly.HTMLElement) {
            //列表中每一项
            e.ForEach("li", func(i int, item *colly.HTMLElement) {
            //文章链接
            href := item.ChildAttr("div[class='content'] > a[class='title']", "href")
            //文章标题
            title := item.ChildText("div[class='content'] > a[class='title']")
            //文章摘要
            summary := item.ChildText("div[class='content'] > p[class='abstract']")
            fmt.Println(title, href)
            fmt.Println(summary)
            fmt.Println()
        })
    })

    err := c.Visit("https://www.jianshu.com")
    if err != nil {
        fmt.Println(err.Error())
    }
}
```

## 2. 爬取文章列表和详情

> 文章列表和1方式一样，文章详情通过创建新的采集器访问详情页面



```go
package main

import (
    "fmt"
    "github.com/gocolly/colly"
    "time"
)

func main() {
    c1 := colly.NewCollector(colly.UserAgent("Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.163 Safari/537.36"), colly.MaxDepth(1))
    c2 := c1.Clone()

    //异步
    c2.Async = true
    //限速
    c2.Limit(&colly.LimitRule{
        DomainRegexp: "",
        DomainGlob:   "*.jianshu.com/p/*",
        Delay:        10 * time.Second,
        RandomDelay:  0,
        Parallelism:  1,
    })
    //采集器1，获取文章列表
    c1.OnHTML("ul[class='note-list']", func(e *colly.HTMLElement) {
        e.ForEach("li", func(i int, item *colly.HTMLElement) {
            href := item.ChildAttr("div[class='content'] > a[class='title']", "href")
            title := item.ChildText("div[class='content'] > a[class='title']")
            summary := item.ChildText("div[class='content'] > p[class='abstract']")
            ctx := colly.NewContext()
            ctx.Put("href", href)
            ctx.Put("title", title)
            ctx.Put("summary", summary)
            //通过Context上下文对象将采集器1采集到的数据传递到采集器2
            c2.Request("GET", "https://www.jianshu.com" + href, nil, ctx, nil)
        })
    })
    //采集器2，获取文章详情
    c2.OnHTML("article", func(e *colly.HTMLElement) {
        href := e.Request.Ctx.Get("href")
        title := e.Request.Ctx.Get("title")
        summary := e.Request.Ctx.Get("summary")
        detail := e.Text

        fmt.Println("----------" + title + "----------")
        fmt.Println(href)
        fmt.Println(summary)
        fmt.Println(detail)
        fmt.Println()
    })

    c2.OnRequest(func(r *colly.Request) {
        fmt.Println("c2爬取页面：", r.URL)
    })

    c1.OnRequest(func(r *colly.Request) {
        fmt.Println("c1爬取页面：", r.URL)
    })

    c1.OnError(func(r *colly.Response, err error) {
        fmt.Println("Request URL:", r.Request.URL, "failed with response:", r, "\nError:", err)
    })

    err := c1.Visit("https://www.jianshu.com")
    if err != nil {
        fmt.Println(err.Error())
    }

    c2.Wait()
}
```

## 3. 爬取需要登录的网页

> 官网提供登录页处理的例子，但是大多数涉及验证码，不好处理，目前方式是手动登录，复制cookie写到爬虫请求头里



```go
package main

import (
    "fmt"
    "github.com/gocolly/colly"
    "github.com/gocolly/colly/debug"
    "github.com/gocolly/colly/extensions"
    _ "github.com/gocolly/colly/extensions"
    "net/http"
)

func main() {
    url := "https://www.jianshu.com"

    c := colly.NewCollector(colly.UserAgent("Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.163 Safari/537.36"), colly.MaxDepth(1), colly.Debugger(&debug.LogDebugger{}))
    c.OnHTML("ul[class='note-list']", func(e *colly.HTMLElement) {
        e.ForEach("li", func(i int, item *colly.HTMLElement) {
            href := item.ChildAttr("div[class='content'] > a[class='title']", "href")
            title := item.ChildText("div[class='content'] > a[class='title']")
            summary := item.ChildText("div[class='content'] > p[class='abstract']")
            fmt.Println(title, href)
            fmt.Println(summary)
            fmt.Println()
        })
    })

    //设置随机useragent
    extensions.RandomUserAgent(c)
    //设置登录cookie
    c.SetCookies(url, []*http.Cookie{
        &http.Cookie{
            Name:     "remember_user_token",
            Value:    "wNDUxOV0sIiQyYSQxMSRwdkhqWVhHYmxXaDJ6dEU3NzJwbmsuIiwiMTU",
            Path:     "/",
            Domain:   ".jianshu.com",
            Secure:   true,
            HttpOnly: true,
        },
    })

    c.OnRequest(func(r *colly.Request) {
        fmt.Println("爬取页面：", r.URL)
    })

    c.OnError(func(r *colly.Response, err error) {
        fmt.Println("Request URL:", r.Request.URL, "failed with response:", r, "\nError:", err)
    })
    err := c.Visit(url)
    if err != nil {
        fmt.Println(err.Error())
    }
}
```

## 4. 内存任务队列

> 将需要爬取的连接放入队列中，设置队列并发数，可以并行爬取连接



```go
package main

import (
    "fmt"
    "github.com/gocolly/colly"
    "github.com/gocolly/colly/debug"
    "github.com/gocolly/colly/queue"
)

func main() {
    c := colly.NewCollector(colly.UserAgent("Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.163 Safari/537.36"), colly.MaxDepth(3), colly.Debugger(&debug.LogDebugger{}))

//创建内存队列，大小10000，goroutine数量 5
    q, _ := queue.New(5, &queue.InMemoryQueueStorage{MaxSize: 10000})

    c.OnHTML("a", func(element *colly.HTMLElement) {
        element.Request.Visit(element.Attr("href"))
    })

    c.OnRequest(func(r *colly.Request) {
        fmt.Println("爬取页面：", r.URL)
    })

    c.OnError(func(r *colly.Response, err error) {
        fmt.Println("Request URL:", r.Request.URL, "failed with response:", r, "\nError:", err)
    })
    q.AddURL("https://www.jianshu.com")

    q.Run(c)
}
```

## 5. redis任务队列

> 设置redis存储后，队列中URL存储在redis中，访问页面的cookie及访问记录也会保存在redis中



```go
package main

import (
    "fmt"
    "github.com/gocolly/colly"
    "github.com/gocolly/colly/debug"
    "github.com/gocolly/colly/queue"
    "github.com/gocolly/redisstorage"
)

func main() {
    c := colly.NewCollector(colly.UserAgent("Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.163 Safari/537.36"), colly.MaxDepth(3), colly.Debugger(&debug.LogDebugger{}))

    storage := &redisstorage.Storage{
        Address:  "192.168.1.10:6379",
        Password: "123456",
        DB:       0,
        Prefix:   "colly",
        Client:   nil,
        Expires:  0,
    }

    c.SetStorage(storage)

    err := storage.Clear()
    if err != nil{
        panic(err)
    }

    defer storage.Client.Close()

    q, _ := queue.New(5, storage)

    c.OnHTML("a", func(element *colly.HTMLElement) {
        element.Request.Visit(element.Attr("href"))
    })

    c.OnRequest(func(r *colly.Request) {
        fmt.Println("爬取页面：", r.URL)
    })

    c.OnError(func(r *colly.Response, err error) {
        fmt.Println("Request URL:", r.Request.URL, "failed with response:", r, "\nError:", err)
    })
    q.AddURL("https://www.jianshu.com")

    q.Run(c)
}
```

![img](https:////upload-images.jianshu.io/upload_images/15604519-7bd4f8220808532c.png?imageMogr2/auto-orient/strip|imageView2/2/w/442/format/webp)

redis中数据

## 6.配置代理



```go
package main

import (
    "bytes"
    "log"

    "github.com/gocolly/colly"
    "github.com/gocolly/colly/proxy"
)

func main() {
    c := colly.NewCollector()

    //配置两个代理
    rp, err := proxy.RoundRobinProxySwitcher("http://127.0.0.1:1080", "socks5://127.0.0.1:1338")
    if err != nil {
        log.Fatal(err)
    }
    c.SetProxyFunc(rp)

    c.OnResponse(func(r *colly.Response) {
        log.Printf("%s\n", bytes.Replace(r.Body, []byte("\n"), nil, -1))
    })

    c.Visit("https://httpbin.org/ip")
}
```

