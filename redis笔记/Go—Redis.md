# Go—Redis

## 一、Redis基本介绍、安装及原理

### 1. 基本介绍：

①Redis是NoSQL数据库（非关系型数据库）💨[官网](https//redis.cn)

②Redis(**Remote Dlctionary Server**),意即`远程字典服务器`，单机能达到15w qps，适合做缓存，也可以持久化。

③Redis是高性能的（key/value)分布式内存数据库，基于内存运行并支持持久化的NoSQL数据库，也成为数据结构服务器

### 2. Redis安装

直接进入官网下载最新版本，至充足磁盘位置处，然后即可使用Redis了。

为了使用方便先可以为Redis包配置系统路径，这样cmd可以实现不在Redis目录下，启动Redis服务，然后再将Redis服务注册为windows的系统服务即可使用。

### 3. Redis的原理

### 1. 原理图

Redis默认有16个数据库，初始默认使用0号库，编号是0...15，可以使用select关键字切换数据库编号，但是每个数据的数据互不影响。

![image-20211129232657504](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20211129232657504.png)

### 2. Redis五大数据类型

String（字符串）、Hash（哈希）、List（列表）、Set（集合）、zset（sorted set：有序集合）。

## 二、Redis的基本使用

### 1. 原生Redis命令使用详解

#### ①String

- string介绍

  string是Redis最基本的数据类型，一个key对应一个value。string类型是二进制安全的。除普通字符外，也可以存放图片等数据。Redis里字符串value最大时512M

- string-crud

  - set[如果存在就相当于修改，不存在就就是添加]/get/del

  - mset[同时设置一个或多个key-value对]

  - mget[同时获取多个key-value]

  - 演示

    ![image-20211203204256060](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20211203204256060.png)
  
    ![image-20211203204308118](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20211203204308118.png)
  
    ![image-20211203204324418](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20211203204324418.png)
  
    ![image-20211203204333875](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20211203204333875.png)

#### ②Hash

- Hash的基本介绍

  Redis Hash 是一个string类型的field和value的映射表，Hash特别适合储存**对象**。 

- Hash-crud

  - hset/hget/hgetall/hdel
  
  - hmset/hmget/hlen[统计hash有多少个元素]/hexists key feild[查看哈希表key里，定域field是否存在]
  
  - 演示
  
    ![image-20211203204111212](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20211203204111212.png)
  
    ![image-20211203204140139](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20211203204140139.png)
  
    ![image-20211203204151920](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20211203204151920.png)

#### ③List

- List基本介绍

  列表是最简单的字符串列表，按照插入顺序排序。可以在头部或尾部添加元素。

  List本质是个链表，是有序的，元素的值可以重复。

  如果List数据全移除，对应的键也会消失，或者直接删除列表名

- List-crud

  - lpush/rpush/lrange/lpop/rpop/del/llen key
  
    lpush：表示从列表头部依次添加一个或多个元素
  
    rpush：表示从列表尾部依次一个或多个添加元素
  
    lrange：表示遍历列表，index从0开始，-1表示最后一个元素，-2表示倒数第二个元素，依次类推...
  
    lpop：从左边取出一个元素，并从列表删除
  
    rpop：从右边取出一个元素，并从列表删除
  
    del：删除
  
  - 演示
  
    ![image-20211203203808401](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20211203203808401.png)
  
    ![image-20211203201544298](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20211203201544298.png)
  
    ![image-20211203203828429](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20211203203828429.png)

#### ④Set

- Set基本介绍

  - Redis的Set是string类型的无序集合。
  - 底层是HashTable数据结构，Set存放很多字符串元素，并且是无序的，不能有重复

- Set-crud

  - sadd/smembers/sismember/srem[删除指定值]

  - 演示

    ![image-20211203203736140](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20211203203736140.png)

    ![image-20211203203657097](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20211203203657097.png)

[更多Redis命令](http://redisdoc.com/)

### 2. go-redis库使用

区别于另一个比较常用的Go语言redis client库：`github.com/gomodule/redigo`，我们这里采用`github.com/go-redis/redis`连接Redis数据库并进行操作，因为`go-redis`支持连接哨兵及集群模式的Redis。`go-redis`库本质上是使用go语言对redis命令进行封装成一些方法，api，方便程序员操作redis数据库。

#### ①go连接Redis

注：go-redis支持普通模式、*<u>哨兵模式、集群模式</u>*

```go
//initRedisClient 初始化redis连接
func initRedisClient() (err error) {
	rds = redis.NewClient(&redis.Options{
		DB:       0,//选择0-15号redis数据库，
		Addr:     "localhost:6379",//Redis服务监听的端口号
		Password: "",
	})
	result, err := rds.Ping().Result()
	if err != nil {
		return err
	}
	fmt.Println(result)
	return nil
}
```

#### ②go操作Redis

##### （1）crud

```go
//example1 演示string类型的set/get的用法
func example1(addData interface{}) (err error) {
	err = rds.Set("score", addData, 0).Err()
	if err != nil {
		return err
	}
	result, err := rds.Get("score").Result()
	if err != nil {
		return err
	}
	fmt.Println("添加数据：",result)
	return nil
}
//example2 演示zset（有序集合）的相关用法
func example2() (err error) {
	//zset的key
	zset:= "scoreRank"
	//ZAdd添加多个数据
	scores := []redis.Z{
		redis.Z{
			Score: 91,
			Member: "刘海斌",
		},
		redis.Z{
			Score: 99,
			Member: "包文杰",
		},
		redis.Z{
			Score: 90,
			Member: "王铁霖",
		},
		redis.Z{
			Score: 97,
			Member: "牟国柱",
		},
	}
	fmt.Println(scores)
	result1, err := rds.ZAdd(zset, scores...).Result()
	if err != nil {
		fmt.Println(err)
		return err
	}
	fmt.Println("zadd：",result1)
	//修改分数,在原地分数基础上添加
	result2, err := rds.ZIncrBy(zset, 1, "包文杰").Result()
	if err != nil {
		return err
	}
	fmt.Println("成功修改：",result2)
	//取分数最高的三个
	result3, err := rds.ZRevRangeWithScores(zset,0,2).Result()
	if err != nil {
		return err
	}
	fmt.Println("分数最高的三个：",result3)
	//取分数最低的三个
	result4, err := rds.ZRangeWithScores(zset,0,2).Result()
	if err != nil {
		return err
	}
	fmt.Println("分数最低的三个：",result4)
	//取95~100分的数据
	zRangeBy := redis.ZRangeBy{
		Min: "95",
		Max: "100",
	}
	result5, err := rds.ZRangeByScoreWithScores(zset, zRangeBy).Result()
	if err != nil {
		return err
	}
	fmt.Println("95~100:：",result5)
	//rds.ZRevRangeByScore(zset,zRangeBy)
	return nil
}
```

##### （2）pipline网络优化

- go-redis库使用redis的过程图

  RTT表示网络传输的时间+redis服务器解析处理的时间。严格来说未经优化的redis命令，会一条一条的传输redis命令至redis服务器进行解析，期间网络往返时间花费较多。**注：redis是单线程的，因此每次只能传输一个命令进行解析。**

  ![image-20211204124642583](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20211204124642583.png)

- pipline网络优化思路

  它本质上意味着客户端缓冲一堆命令并一次性将它们发送到服务器。这些命令不能保证在事务中执行。这样做的好处是节省了每个命令的网络往返时间（RTT）。

- pipline优化实例

  ```go
pipe := rdb.Pipeline()
  
  incr := pipe.Incr("pipeline_counter")
  pipe.Expire("pipeline_counter", time.Hour)
  
  _, err := pipe.Exec()
  fmt.Println(incr.Val(), err)
  ```
  
  上面的代码相当于将以下两个命令一次发给redis server端执行，与不使用`Pipeline`相比能减少一次RTT。

  ```bash
INCR pipeline_counter
  EXPIRE pipeline_counts 3600
  ```
  
  也可以使用`Pipelined`：

  ```go
var incr *redis.IntCmd
  _, err := rdb.Pipelined(func(pipe redis.Pipeliner) error {
  	incr = pipe.Incr("pipelined_counter")
  	pipe.Expire("pipelined_counter", time.Hour)
  	return nil
  })
  fmt.Println(incr.Val(), err)
  ```
  
  在某些场景下，当我们有多条命令要执行时，就可以考虑使用pipeline来优化。

- pipline缺点

  pipline是将命令打包成一组一次性发送执行，不适合两个命令具有依赖关系的场景。

##### （3）事务

Redis是单线程的，因此单个命令始终是原子的，但是来自不同客户端的两个给定命令可以依次执行，例如在它们之间交替执行。但是，`Multi/exec`能够确保在`multi/exec`两个语句之间的命令之间没有其他客户端正在执行命令。

在这种场景我们需要使用`TxPipeline`。`TxPipeline`总体上类似于上面的`Pipeline`，但是它内部会使用`MULTI/EXEC`包裹排队的命令。例如：

```go
pipe := rdb.TxPipeline()

incr := pipe.Incr("tx_pipeline_counter")
pipe.Expire("tx_pipeline_counter", time.Hour)

_, err := pipe.Exec()
fmt.Println(incr.Val(), err)
```

上面代码相当于在一个RTT下执行了下面的redis命令：

```bash
MULTI
INCR pipeline_counter
EXPIRE pipeline_counts 3600
EXEC
```

还有一个与上文类似的`TxPipelined`方法，使用方法如下：

```go
var incr *redis.IntCmd
_, err := rdb.TxPipelined(func(pipe redis.Pipeliner) error {
	incr = pipe.Incr("tx_pipelined_counter")
	pipe.Expire("tx_pipelined_counter", time.Hour)
	return nil
})
fmt.Println(incr.Val(), err)
```

##### （4）Watch

在某些场景下，我们除了要使用`MULTI/EXEC`命令外，还需要配合使用`WATCH`命令。在用户使用`WATCH`命令监视某个键之后，直到该用户执行`EXEC`命令的这段时间里，如果有其他用户抢先对被监视的键进行了替换、更新、删除等操作，那么当用户尝试执行`EXEC`的时候，事务将失败并返回一个错误，用户可以根据这个错误选择重试事务或者放弃事务。

```go
Watch(fn func(*Tx) error, keys ...string) error
```

Watch方法接收一个函数和一个或多个key作为参数。基本使用示例如下：

```go
// 监视watch_count的值，并在值不变的前提下将其值+1
key := "watch_count"
err = client.Watch(func(tx *redis.Tx) error {
	n, err := tx.Get(key).Int()
	if err != nil && err != redis.Nil {
		return err
	}
	_, err = tx.Pipelined(func(pipe redis.Pipeliner) error {
		pipe.Set(key, n+1, 0)
		return nil
	})
	return err
}, key)
```

最后看一个V8版本官方文档中使用GET和SET命令以事务方式递增Key的值的示例，仅当Key的值不发生变化时提交一个事务。

```go
func transactionDemo() {
	var (
		maxRetries   = 1000
		routineCount = 10
	)
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	// Increment 使用GET和SET命令以事务方式递增Key的值
	increment := func(key string) error {
		// 事务函数
		txf := func(tx *redis.Tx) error {
			// 获得key的当前值或零值
			n, err := tx.Get(ctx, key).Int()
			if err != nil && err != redis.Nil {
				return err
			}

			// 实际的操作代码（乐观锁定中的本地操作）
			n++

			// 操作仅在 Watch 的 Key 没发生变化的情况下提交
			_, err = tx.TxPipelined(ctx, func(pipe redis.Pipeliner) error {
				pipe.Set(ctx, key, n, 0)
				return nil
			})
			return err
		}

		// 最多重试 maxRetries 次
		for i := 0; i < maxRetries; i++ {
			err := rdb.Watch(ctx, txf, key)
			if err == nil {
				// 成功
				return nil
			}
			if err == redis.TxFailedErr {
				// 乐观锁丢失 重试
				continue
			}
			// 返回其他的错误
			return err
		}

		return errors.New("increment reached maximum number of retries")
	}

	// 模拟 routineCount 个并发同时去修改 counter3 的值
	var wg sync.WaitGroup
	wg.Add(routineCount)
	for i := 0; i < routineCount; i++ {
		go func() {
			defer wg.Done()
			if err := increment("counter3"); err != nil {
				fmt.Println("increment error:", err)
			}
		}()
	}
	wg.Wait()

	n, err := rdb.Get(context.TODO(), "counter3").Int()
	fmt.Println("ended with", n, err)
}
```

更多详情请查阅[文档](https://pkg.go.dev/github.com/go-redis/redis)。

## 三、redis原生命令详解

前提引入

在了解redis之前我们需要简要知道一个概念，什么是关系型数据库，什么是非关系型数据库。

- NoSQL:最常见的解释是“non-relational”，"Not Only SQL"也被许多人接受，NoSQL也仅仅是一个概念，泛指非关系型数据库，也区别于关系型数据库。他们并不会保证关系数据的**ACID**特性。NoSQL是一项全新的数据库革命性运动，其拥护者们提倡运用非关系型的数据存储。

**关系型数据库**：关系型数据库最典型的**数据结构是表**，由**二维表**及其**之间的联系**所组成的一个数据组织。在数据库中有个比较重要的概念,**关系要比表结构重要**

**非关系型数据库**：**是一种数据结构化存储方法的集合，可以是文档或者键值对等**，非关系型数据库提出另一种理念，例如，以键值对存储，且结构不固定，每一个元组可以有不一样的字段，每个元组可以根据需要增加一些自己的键值对，这 样就不会局限于固定的结构，可以减少一些时间和空间的开销。文档型数据库。

## 什么是redis？

Redis是现在最受欢迎的NoSQL数据库之一，Redis是一个使用ANSI C编写的开源、包含多种数据结构、支持网络、基于内存、可选持久性的键值对存储数据库，其具备如下特性：

- 基于内存运行，性能高效

- 支持分布式，理论上可以无限扩展

- key-value存储系统

- 开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API

**常见的使场景**

1.缓存系统（热点数据：高频读取，低频写）

- 由于redis访问速度块、支持的数据类型比较丰富，所以redis很适合用来存储热点数据，另外结合expire，我们可以设置过期时间然后再进行缓存更新操作，这个功能最为常见

2.排行榜

- 关系型数据库在排行榜方面查询速度普遍偏慢，所以可以借助redis的SortedSet进行热点数据的排序。

3.MQ（消息队列）

- 由于redis有list push和list pop这样的命令，所以能够很方便的执行队列操作。

略。。使用场景太多了，还有什么发布订阅啥的，好友关系储存啥的

## redis为什么快？

**首先，采用了多路复用io阻塞机制**
**然后，数据结构简单，操作节省时间**
**最后，运行在内存中，自然速度快**

提问：有人知道IO阻塞机制吗？

## redis的基本命令

### 字符串

#### SET

语法

```
SET key value [EX seconds] [PX milliseconds] [NX|XX]
```

这里就是一组典型的键值对

**EX_seconds**:表明过期时间，单位是秒

**PX**:单位毫秒

**NX** ： 只在键不存在时， 才对键进行设置操作

**XX**： 与NX相反只在键已经存在时， 才对键进行设置操作

基本操作，注意当键值对已经存在时候，会替换掉其对应的value

```plain
127.0.0.1:6379> SET key "value"
OK
127.0.0.1:6379> get key
"value"
```

使用EX、PX选项,TTL 命令显示过期时间s(PTTL显示ms)，这里无法同时使用EX,PX选项

```plain
127.0.0.1:6379> SET key "hello" EX 1000
OK
127.0.0.1:6379> TTL key
(integer) 992
127.0.0.1:6379> get key
"hello"
```

使用了XX与NX选项，NX就是当这个kv不存在的时候才会被设置 ,XX确实在这个kv存在的时候才会被设置

```plain
127.0.0.1:6379> SET key "test" NX
(nil)
127.0.0.1:6379> GET key
"hello"

127.0.0.1:6379> SET AAA "SSS" XX
(nil)
```

#### MSET

语法：`MEST k1 v1 k2 v2`

同时设置多个键值对，并且总是返回OK，（有MSET就肯定会有MGET来得到多个值）

```plain
127.0.0.1:6379> MSET k1 v1 k2 v2
OK
127.0.0.1:6379> MGET k1 k2
1) "v1"
2) "v2"
```

#### **SETNX**

SETNX（set if not exists）跟上述加上NX选项的SET功能一样

```plain
127.0.0.1:6379> SETNX key3 "value"
(integer) 1
127.0.0.1:6379> exists key3
(integer) 1
127.0.0.1:6379> SETNX key3 "worry"
(integer) 0
127.0.0.1:6379> GET key3
"value"
```

#### SETEX

语法:`SETEX key seconds value`

先跟时间再跟值

```plain
127.0.0.1:6379> SETEX id 60 12318
OK
127.0.0.1:6379> TTL id
(integer) 46
127.0.0.1:6379> GET id
"12318"
```

#### PSETEX

跟SETEX差不过，只不过这个是以ms为单位了

#### GET

语法：`GET key`

返回值：若value存在，则返回value，若不存在则返回nil，若该键对应的值不为字符串类型，则返回错误

#### DEL

删除

返回值：返回删除的key数量

```plain
127.0.0.1:6379> DEL k1
(integer) 1
```

#### GETSET

语法：GETSET key value

设置当前的key-value，并返回之前的值，如果之前的key并不存在，就返回nil

这里如果原来就不存在这个KV，则会返回nil

```plain
127.0.0.1:6379> GETSET k2 "value"
"v2"
127.0.0.1:6379> GET k2
"value"
```

#### STRLEN

语法：`STRLEN key`

跟上述的SET语法类似，只是`STRLEN` 命令返回字符串值的长度。

当键 `key` 不存在时， 命令返回 `0` 。

当 `key` 储存的不是字符串值时， 返回一个错误。

```plain
127.0.0.1:6379> GET k2
"value"
127.0.0.1:6379> STRLEN k2
(integer) 5
```

#### APPEND

语法：`APPEND key value`

追加字符串，若该key已存在，则往后追加，若不存在就跟简单的执行`SET`一样

返回追加后key的长度

```plain
127.0.0.1:6379> GET k2
"value"
127.0.0.1:6379> APPEND k2 "is good"
(integer) 12
127.0.0.1:6379> get k2
"valueis good"
```

#### SETRANGE

语法：`SETRANGE key offset value`

从偏移量`offset`后覆盖原来的value，并返回当前key对应字符串的长度

```plain
127.0.0.1:6379> SETRANGE k2 8 "golang"
(integer) 14
127.0.0.1:6379> GET k2
"valueis golang"
```

#### GETRANGE

语法：`GETRANGE key start end`

获取[start,end]的子字符串，允许负偏移量，即-1代表最后一个，-2代表倒数第二个

数字规则和前面一样

```plain
127.0.0.1:6379> GETRANGE k2 0 -1
"valueis golang"
```

#### INCR

语法：INCR key

为键 `key` 储存的数字值加上一。

如果键 `key` 不存在， 那么它的值会先被初始化为 `0` ， 然后再执行 `INCR` 命令。

如果键 `key` 储存的值不能被**解释**为数字（因为redis中并没有专用的整数类型）， 那么 `INCR` 命令将返回一个错误。

```plain
127.0.0.1:6379> SET n 1
OK
127.0.0.1:6379> GET n
"1"
127.0.0.1:6379> INCR n
(integer) 2
127.0.0.1:6379> get n
"2"
```

### 哈希

#### HSET

语法：`HEST hash field value`

将hash表中field域设置为value，与之相配的为HGET(`HEGT hash field`)

返回值：当 `HSET` 命令在哈希表中新创建 `field` 域并成功为它设置值时， 命令返回 `1` ； 如果已经存在，那么命令返回 `0` 。

如：

```plain
127.0.0.1:6379> HSET map k1 v1
(integer) 1
127.0.0.1:6379> HGET map k1
"v1"
```

#### HLEN

语法：`HLEN hash`

返回哈希表hash中域的数量，若hash不存在，则返回0

如：

```plain
127.0.0.1:6379> HLEN map
(integer) 1
```

#### HDEL

语法：`HDEL key field1 field2...`

删除哈希表key中的域，并返回成功删除的域的数量

如：

```plain
127.0.0.1:6379> HDEL map k1
(integer) 1
127.0.0.1:6379> hgetall map
(empty list or set)
```

其他的哈希操作你们自己去试试吧。

### 队列

#### LPUSH与LRANGE

将一个或者多个值插入**列表头部**，如果为空列表会被创建执行LPUSH，当key存在但是却不是列表，就返回错误

LRANGE这命令中0标识列表第一个元素，1则是第二个。

-1代表最后一个元素，-2则代表倒数第二个。

```plain
127.0.0.1:6379> LPUSH list java c++ go
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1
1) "go"
2) "c++"
3) "java"
```

#### LPOP

语法：`LPOP key`

移除列表key的表头元素，即出队列

```plain
127.0.0.1:6379> LPOP list
"go"
127.0.0.1:6379> LRANGE list 0 -1
1) "c++"
2) "java"
```

#### LSET

语法：`LSET key index value`

将列表 key下标为 index 的元素的值设置为 value

若 `index` (这里的下标是从0开始的，也允许负数)参数超出范围，或对一个空列表( `key` 不存在)进行操作时，则会返回一个错误，成功返回OK

```plain
127.0.0.1:6379> LSET list 1 "go"
OK
127.0.0.1:6379> LRANGE list 0 -1
1) "c++"
2) "go"
```

其他队列操作自己去试试吧

### 集合

#### SADD

语法：`SADD key m1 m2...`

将一个或多个 `member` 元素加入到集合 `key` 当中，已经存在于集合的 `member` 元素将被忽略（而列表则不会忽略）。并返回成功添加到集合的数目

与之对应的SMEMBERS（`SMEMBERS key`）则是获取该集合

如：

```plain
127.0.0.1:6379> SADD arr m1 m2 m2
(integer) 2
127.0.0.1:6379> SMEMBERS arr
1) "m2"
2) "m1"
```

#### SPOP

语法：`SPOP key`

移除并返回集合中的一个**随机**元素,并返回被移除的元素，若key不存在或该集合为空时，就返回nil

```plain
127.0.0.1:6379> SPOP arr
"m1"
```

#### SMOVE

语法：`SMOVE source destination member`

如果 `source` 集合不存在或不包含指定的 `member` 元素，则该命令不执行任何操作，仅返回 `0` 。否则， `member` 元素从 `source` 集合中被**移除**，并**添加**到 `destination` 集合中去。

当 `destination` 集合已经包含 `member` 元素时，该命令只是简单地将 `source` 集合中的 `member` 元素删除。

如：

```plain
127.0.0.1:6379> SADD k1 m1 m2
(integer) 2
127.0.0.1:6379> SADD k3 m3
(integer) 1
127.0.0.1:6379> SMOVE k1 k3 m1
(integer) 1
127.0.0.1:6379> smembers k1
1) "m2"
127.0.0.1:6379> smembers k3
1) "m3"
2) "m1"
```

### 有序集合

#### ZADD

语法：`ZADD key score1 member1 score2 member2…`

将一个或多个 `member` 元素及其 `score` 值加入到有序集 `key` 当中，注意这里的member和score是必须配对的，而且score必须是浮点数或者整型，添加成功后返回被成功添加的新成员的数量

如：

```plain
127.0.0.1:6379> ZADD z 1 m1 2 m2
(integer) 2
127.0.0.1:6379> ZRANGE z 0 -1 WITHSCORES
1) "m1"
2) "1"
3) "m2"
4) "2"
127.0.0.1:6379> ZRANGE z 0 -1
1) "m1"
2) "m2"
127.0.0.1:6379> zrem z m1
(integer) 1
127.0.0.1:6379> ZRANGE z 0 -1
1) "m2"
```

#### ZRANGE

语法：`ZRANGE key start stop [WITHSCORES]`

返回有序集 `key` 中，并且其中成员的位置按 `score` 值递增(从小到大)来排序（`ZREVRANGE`则按从大到小排列）。

例子如上，这里便不举例了

#### ZREM

语法:`ZREM key member1 member2`

移除有序集 `key` 中的一个或多个成员，不存在的成员将被忽略，并返回成功移除的数量

例子如上，这里便不举例了

### Redis事务

我们开始也说了NOSQL是不会保证关系数据的ACID的，详见

https://www.cnblogs.com/chenpingzhao/archive/2015/11/27/5001894.html

redis中如何执行事务？

在redis中`multi`标志着事务的开始，`exec`即执行事务

```plain
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v1
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
```

思考一下哪里有问题

```plain
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v2
QUEUED
127.0.0.1:6379> lpush k2 v2
QUEUED
127.0.0.1:6379> exec
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
127.0.0.1:6379> get k1
"v2"
```

很明显，第一条执行成功，而第二条失败了，这并不复合事务的原子性。

因此，我们的redis事务是不满足平时所说的ACDI性质的，其没有回滚的机制（即事务中的命名执行错误，回滚到事务执行之前）
