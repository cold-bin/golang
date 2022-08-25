# redigo的使用

- redigo与go-redis不同之处在于，go-redis是将redis命令封装成方法或函数，进行调用，再解析成redis命令传到服务器，之后返回结果再转化；而redigo只是实现了传输redis命令至服务器，然后解析redis服务器返回的结果。

```golang
package main

import (
   "fmt"
   "github.com/gomodule/redigo/redis"
)

func main()  {
   // 官方地址:https://godoc.org/github.com/gomodule/redigo/redis#pkg-examples
   /*****************************  redis连接  *********************************/
   // option := redis.DialPassword("123456") 如果redis有密码,放在第三个选项
   c, err := redis.Dial("tcp", "127.0.0.1:6379")
   if err != nil {
      fmt.Println(err)
      return
   }
   defer c.Close()

   /*****************************  Do方法  *************************************/
   // 执行redis语句的通用方法
   // 可执行的命令参考 :https://redis.io/commands

   // set name card 成功返回ok
   res,err := c.Do("SET", "NAME", "card")
   if err != nil {
      fmt.Println(err)
   }
   fmt.Println(res)

   // get name 成功返回interface{}
   res, err = c.Do("GET", "NAME")
   if err != nil {
      fmt.Println(err)
   }
   // 进行类型断言
   switch t := res.(type) {
   case []byte:
      fmt.Println(string(t))
   }

   /*****************************  管道  *************************************/
   // redigo支持管道操作Send(),Flush(),Receive()

   // Send()  发送命令到输出缓冲区。
   // Send(commandName string, args ...interface{}) error

   // Flush() 写入命令并刷新输出缓冲区。
   // Flush() error

   // Receive()接收服务器的返回值
   // Receive() (reply interface{}, err error)

   c.Send("SET", "foo", "bar")    // 设置语句到缓冲区
   c.Send("GET", "foo")         // 设置语句到缓冲区
   c.Flush()                                // 发送语句
   res,_ =c.Receive()                         // 接收第一个Send()的返回值
   fmt.Println(res)
   res,_ = c.Receive()                            // 接收第二个Send()的返回值
   fmt.Println(res)

   // Do方法结合了Send，Flush和Receive方法的功能。
   // Do方法首先写入命令并刷新输出缓冲区。
   // 接下来，Do方法接收所有待处理的回复，包括Do执行的命令的回复。
   // 如果收到的任何回复都是错误，则Do返回错误。 如果没有错误，则Do返回最后一个返回值。
   // 如果Do方法的命令参数是“”，则Do方法将刷新输出缓冲区并接收挂起的回复而不发送命令。

   // MULTI类似于事务的begin commit事务处理
   c.Send("MULTI")
   c.Send("INCR", "money")
   c.Send("INCR", "count")
   r, err := c.Do("EXEC")
   fmt.Println(r) // [1,1]


   /*****************************  并发  *************************************/
   // redigo支持一个并发调用Receive()方法和一个并发调用Send()和Flush()方法
   // 不支持其他并发,包括对Do方法的并发调用
   // 如果需要完全并发访问redis,需要使用线程安全池goroutine中获取,使用和释放连接。
   // 从线程中返回具有前一段时间描述的并发限制
   // 关于redis连接池会单独写一个

   /*****************************  发布和订阅  *******************************/
   // 扩展:SUBSCRIBE、UNSUBSCRIBE和PUBLISH 三个命令实现了发布与订阅信息泛型.
   // 在这个实现中， 发送者（发送信息的客户端）不是将信息直接发送给特定的接收者（接收信息的客户端），
   // 而是将信息发送给频道（channel）， 然后由频道将信息转发给所有对这个频道感兴趣的订阅者。 
   // 也就是说发送者无须知道任何关于订阅者的信息， 而订阅者也无须知道是那个客户端给它发送信息，
   // 它只要关注自己感兴趣的频道即可
   // 对发布者和订阅者进行解构（decoupling），可以极大地提高系统的扩展性（scalability），
   // 并得到一个更动态的网络拓扑（network topology）。
   // redis可以订阅任意数量的频道。

   // 通过使用Send(),Flush(),和Receive()方法实现 Pub / Sub订阅者。

   // 订阅者
   // c.Send("SUBSCRIBE", "example")
   // c.Flush()
   // for {
   //     reply, err := c.Receive()
   //     if err != nil{
   //        fmt.Println(err)
   // }
   // // 处理接收到的信息
   // fmt.Println(reply)
   // }

   // PubSubConn类型使用便捷方法包装Conn以实现订阅者。
   // Subscribe，PSubscribe，Unsubscribe和PUnsubscribe方法发送和刷新订阅管理命令。
   // receive方法将推送的消息转换为方便的类型，以便在switch中使用。
   // psc := redis.PubSubConn{Conn:c}
   // psc.Subscribe("example")
   // for {
   //      switch v := psc.Receive().(type) {
   //      case redis.Message:
   //     fmt.Printf("%s: message: %s \n", v.Channel, v.Data)
   //  case redis.Subscription:
   //     fmt.Printf("%s: %s %d\n", v.Channel, v.Kind, v.Count)
   //  case error:
   //     fmt.Println(v)
   //  }
   // }

   /*****************************  返回值助手函数  *******************************/
   // Bool，Int，Bytes，String，Strings和Values函数将返回值转换为特定类型的值。
   // 为了方便地包含对连接Do和Receive方法的调用，这些函数采用了类型error的第二个参数。
   // 如果错误是非nil，则辅助函数返回错误。 如果错误为nil，则该函数将回复转换为指定的类型
   exists, err := redis.Bool(c.Do("EXISTS", "foo"))
   if err != nil {
      // handle error return from c.Do or type conversion error.
   }
   fmt.Println(exists)
}
```

