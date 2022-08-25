# 基于`golang`的`selenium`使用详解

什么是`selenium`？我理解成：`selenium`是一种程序员使用地、自动化地、可以通过代码来操控指定浏览器的一种集成工具。在使用`go`语言`colly`框架爬取需要登录的网站时，遇到了问题，我必须输入并提交账号密码(`colly`这个还做不出来)，才能访问网站后面的资源。在网上百度了许久，于是看到了`selenium`可以做自动化地控制浏览器操作，于是乎便学习了基于`go`语言实现的`selenium`->[第三方框架:`tebeka/selenium`](https://github.com/tebeka/selenium.git)。刚开始学习地时候搜索大量教程，也去官网逛了一圈，觉得这些教程都不是很好，大多只是贴源码，没有更为详尽地解说，于是乎，自己便将自己的学习分享出来。

## `selenium`使用

## 安装

目前我正在使用的一个依赖库是 [`tebeka/selenium`](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Ftebeka%2Fselenium)，功能较完整且处于维护中。

```bash
go get -u github.com/tebeka/selenium
```

另外，我们需要对应不同类型的浏览器进行安装 `WebDriver`，`Google Chrome` 需要安装 [`ChromeDriver`](https://link.juejin.cn?target=https%3A%2F%2Fchromedriver.chromium.org%2F)，`Firefox` 则需要安装 [`geckodriver`](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fmozilla%2Fgeckodriver)。

## 使用实例(`chorme`为例)

```go
import (
	"fmt"
	"os"
	"strings"
	"time"

	"github.com/tebeka/selenium"
)

const (
    //指定对应浏览器的driver
	chromeDriverPath = "D:/selenium/chromedriver.exe"
	port             = 8080
)

func main() {
	//ServiceOption 配置服务实例
	opts := []selenium.ServiceOption{
		selenium.Output(os.Stderr), // Output debug information to STDERR.
	}
    //SetDebug 设置调试模式
	selenium.SetDebug(true)
    //在后台启动一个ChromeDriver实例
	service, err := selenium.NewChromeDriverService(chromeDriverPath, port, opts...)
	if err != nil {
		panic(err) // panic is used only as an example and is not otherwise recommended.
	}
	defer service.Stop()

	//连接到本地运行的 WebDriver 实例
    //这里的map键值只能为browserName，源码里需要获取这个键的值，来确定连接的是哪个浏览器
	caps := selenium.Capabilities{"browserName": "chrome"}
    //NewRemote 创建新的远程客户端，这也将启动一个新会话。 urlPrefix 是 Selenium 服务器的 URL，必须以协议 (http, https, ...) 为前缀。为urlPrefix提供空字符串会导致使用 DefaultURLPrefix,默认访问4444端口，所以最好自定义，避免端口已经被抢占。后面的路由还是照旧DefaultURLPrefix写
	wd, err := selenium.NewRemote(caps, fmt.Sprintf("http://localhost:%d/wd/hub", port))
	if err != nil {
		panic(err)
	}
	defer wd.Quit()

	// 导航到指定网站页面，浏览器默认get方法
	if err := wd.Get("http://play.golang.org/?simple=1"); err != nil {
		panic(err)
	}
	// 此步操作为定位元素
	// 在当前页面的 DOM 中准确找到一个元素，使用css选择器，使用id选择器选择文本框
	// 值得注意的是，WebElement接口定义了几乎所有操作Web元素支持的方法，例如清除元素、移动鼠标、截图、鼠标点击、提交按钮等操作
	// 此处对于浏览器的自动化操作可以应用于爬虫登录需要输入密码或者其他应用
	elem, err := wd.FindElement(selenium.ByCSSSelector, "#code")
	if err != nil {
		panic(err)
	}
	// 操作元素，删除文本框中已有的样板代码
	if err := elem.Clear(); err != nil {
		panic(err)
	}

	// 在文本框中输入一些新代码
	err = elem.SendKeys(`
        package main
        import "fmt"
        func main() {
            fmt.Println("Hello WebDriver!")
        }
    `)
	if err != nil {
		panic(err)
	}

	// css选择器定位按钮控件
	btn, err := wd.FindElement(selenium.ByCSSSelector, "#run")
	if err != nil {
		panic(err)
	}
	if err := btn.Click(); err != nil {
		panic(err)
	}

	// 等待程序完成运行并获得输出
	outputDiv, err := wd.FindElement(selenium.ByCSSSelector, "#output")
	if err != nil {
		panic(err)
	}

	var output string
	for {
		output, err = outputDiv.Text()
		if err != nil {
			panic(err)
		}
		if output != "Waiting for remote server..." {
			break
		}
		time.Sleep(time.Millisecond * 100)
	}

	fmt.Printf("%s", strings.Replace(output, "\n\n", "\n", -1))

	// Example Output:
	// Hello WebDriver!
	//
	// Program exited.
}

```

使用[`css`选择器](##css选择器)，需要熟悉其语法规则。达到定位的目的。之后就是使用`WebElement`对应的方法实现业务逻辑处理ok了。

## 陷阱

**注意：在使用`css`选择器的同时一定不能使用右键检查查看源代码，这样的源代码是已经经过`js`渲染的的最终源代码;而查看网页源代码则是服务器发送到浏览器的原封不动的源代码，不包括页面动态渲染的内容。所以，使用`css`选择器应该使用查看网页源代码。**

## `css`选择器

| **选择器**                                                   | **例子**              | **例子描述**                                            |
| ------------------------------------------------------------ | --------------------- | ------------------------------------------------------- |
| [.*class*](http://www.w3school.com.cn/cssref/selector_class.asp) | `.intro`              | 选择 `class="intro"` 的所有元素。                       |
| [#*id*](http://www.w3school.com.cn/cssref/selector_id.asp)   | `#firstname`          | 选择 `id="firstname" `的所有元素。                      |
| [*](http://www.w3school.com.cn/cssref/selector_all.asp)      | `*`                   | 选择所有元素。                                          |
| [*element*](http://www.w3school.com.cn/cssref/selector_element.asp) | `p`                   | 选择所有 `<p> `元素。                                   |
| [*element*,*element*](http://www.w3school.com.cn/cssref/selector_element_comma.asp) | `div,p`               | 选择所有 `<div> `元素和所有 `<p>` 元素。                |
| [*element* *element*](http://www.w3school.com.cn/cssref/selector_element_element.asp) | `div p`               | 选择` <div> `元素内部的所有` <p> `元素。                |
| [*element*>*element*](http://www.w3school.com.cn/cssref/selector_element_gt.asp) | `div>p`               | 选择父元素为 `<div> `元素的所有 `<p>` 元素。            |
| [*element*+*element*](http://www.w3school.com.cn/cssref/selector_element_plus.asp) | `div+p`               | 选择紧接在 `<div>` 元素之后的所有 `<p> `元素。          |
| [[*attribute*\]](http://www.w3school.com.cn/cssref/selector_attribute.asp) | `[target]`            | 选择带有` target` 属性所有元素。                        |
| [[*attribute*=*value*\]](http://www.w3school.com.cn/cssref/selector_attribute_value.asp) | `[target=_blank]`     | 选择 `target="_blank" `的所有元素。                     |
| [[*attribute*~=*value*\]](http://www.w3school.com.cn/cssref/selector_attribute_value_contain.asp) | `[title~=flower]`     | 选择` title `属性包含单词 `"flower" `的所有元素。       |
| [*element1*~*element2*](http://www.w3school.com.cn/cssref/selector_gen_sibling.asp) | `p~ul`                | 选择前面有 `<p>` 元素的每个 `<ul>` 元素。               |
| [[*attribute*^=*value*\]](http://www.w3school.com.cn/cssref/selector_attr_begin.asp) | `a[src^="https"]`     | 选择其 `src` 属性值以 `"https" `开头的每个` <a>` 元素。 |
| [[*attribute*$=*value*\]](http://www.w3school.com.cn/cssref/selector_attr_end.asp) | `a[src$=".pdf"]`      | 选择其 `src `属性以` ".pdf"` 结尾的所有` <a> `元素。    |
| [[*attribute**=*value*\]](http://www.w3school.com.cn/cssref/selector_attr_contain.asp) | `a[src*="abc"]`       | 选择其 `src` 属性中包含 `"abc" `子串的每个` <a> `元素。 |
| [:first-of-type](http://www.w3school.com.cn/cssref/selector_first-of-type.asp) | `p:first-of-type`     | 选择属于其父元素的首个 `<p>` 元素的每个 `<p> `元素。    |
| [:last-of-type](http://www.w3school.com.cn/cssref/selector_last-of-type.asp) | `p:last-of-type`      | 选择属于其父元素的最后 `<p> `元素的每个 `<p>` 元素。    |
| [:only-of-type](http://www.w3school.com.cn/cssref/selector_only-of-type.asp) | `p:only-of-type`      | 选择属于其父元素唯一的` <p> `元素的每个` <p> `元素。    |
| [:only-child](http://www.w3school.com.cn/cssref/selector_only-child.asp) | `p:only-child`        | 选择属于其父元素的唯一子元素的每个` <p> `元素。         |
| [:nth-child(*n*)](http://www.w3school.com.cn/cssref/selector_nth-child.asp) | `p:nth-child(2)`      | 选择属于其父元素的第二个子元素的每个 `<p> `元素。       |
| [:nth-last-child(*n*)](http://www.w3school.com.cn/cssref/selector_nth-last-child.asp) | `p:nth-last-child(2)` | 同上，从最后一个子元素开始计数。                        |
| [:nth-of-type(*n*)](http://www.w3school.com.cn/cssref/selector_nth-of-type.asp) | `p:nth-of-type(2)`    | 选择属于其父元素第二个 `<p> `元素的每个` <p> `元素。    |
| [:nth-last-of-type(*n*)](http://www.w3school.com.cn/cssref/selector_nth-last-of-type.asp) | `p:nth-last-of-type(` | 同上，但是从最后一个子元素开始计数。                    |
| [:last-child](http://www.w3school.com.cn/cssref/selector_last-child.asp) | `p:last-child`        | 选择属于其父元素最后一个子元素每个 `<p>` 元素。         |
| [:root](http://www.w3school.com.cn/cssref/selector_root.asp) | `:root`               | 选择文档的根元素。                                      |
| [:empty](http://www.w3school.com.cn/cssref/selector_empty.asp) | `p:empty`             | 选择没有子元素的每个 `<p> `元素（包括文本节点）。       |
| [:not(*selector*)](http://www.w3school.com.cn/cssref/selector_not.asp) | `:not(p)`             | 选择非` <p> `元素的每个元素。                           |

## `webElement`接口

```go
// WebElement defines method supported by web elements.
type WebElement interface {
	// Click clicks on the element.
	Click() error
	// SendKeys types into the element.
	SendKeys(keys string) error
	// Submit submits the button.
	Submit() error
	// Clear clears the element.
	Clear() error
	// MoveTo moves the mouse to relative coordinates from center of element, If
	// the element is not visible, it will be scrolled into view.
	MoveTo(xOffset, yOffset int) error

	// FindElement finds a child element.
	FindElement(by, value string) (WebElement, error)
	// FindElement finds multiple children elements.
	FindElements(by, value string) ([]WebElement, error)

	// TagName returns the element's name.
	TagName() (string, error)
	// Text returns the text of the element.
	Text() (string, error)
	// IsSelected returns true if element is selected.
	IsSelected() (bool, error)
	// IsEnabled returns true if the element is enabled.
	IsEnabled() (bool, error)
	// IsDisplayed returns true if the element is displayed.
	IsDisplayed() (bool, error)
	// GetAttribute returns the named attribute of the element.
	GetAttribute(name string) (string, error)
	// Location returns the element's location.
	Location() (*Point, error)
	// LocationInView returns the element's location once it has been scrolled
	// into view.
	LocationInView() (*Point, error)
	// Size returns the element's size.
	Size() (*Size, error)
	// CSSProperty returns the value of the specified CSS property of the
	// element.
	CSSProperty(name string) (string, error)
	// Screenshot takes a screenshot of the attribute scroll'ing if necessary.
	Screenshot(scroll bool) ([]byte, error)
}
```

源码如上，可见这个`webElement`封装了一些操作特定元素的一些方法，可以达到自动化操作浏览器的目的，如清除元素，填充元素。。。