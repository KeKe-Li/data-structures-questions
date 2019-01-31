### Golang面试问题汇总

通常我们去面试肯定会有些不错的Golang的面试题目的，所以总结下，让其他Golang开发者也可以查看到，同时也用来检测自己的能力和提醒自己的不足之处．

Golang面试问题汇总:

1. 除了 mutex 以外还有那些方式安全读写共享变量？

2. 无缓冲 chan 的发送和接收是否同步?

3. go语言的并发机制以及它所使用的CSP并发模型．

4. golang 中常用的并发模式？

5. JSON 标准库对 nil slice 和 空 slice 的处理是一致的吗？　

6. 协程，线程，进程的区别。

7. 互斥锁，读写锁，死锁问题是怎么解决。

8. golang的内存模型，为什么小对象多了会造成gc压力。

9. Data Race问题怎么解决？能不能不加锁解决这个问题？

10. 什么是channel，为什么它可以做到线程安全？

11. goroutine 如何调度?

12. Golang GC 时会发生什么?

13. Golang 中 goroutine 的调度.

14. 并发编程概念是什么？

15. 负载均衡原理是什么

16. lvs相关

17. 微服务架构是什么样子的

18. 分布式锁实现原理，用过吗？

19. etcd和redis怎么实现分布式锁

20. redis的数据结构有哪些，以及实现场景

21. mysql高可用方案有哪些

22. 说下go的并发模型

23. goroutine和channel的作用分别是什么

24. 怎么查看goroutine的数量

25. 说下go中的锁有哪些?三种锁，读写锁，互斥锁，还有map的安全的锁

26. 读写锁或者互斥锁读的时候能写吗

27. 怎么限制goroutine的数量

28. channel是同步的还是异步的

29. 说一下异步和非阻塞的区别

30. log包线程安全吗？

31. goroutine和线程的区别

32. 滑动窗口的概念以及应用

33. 怎么做弹性扩缩容，原理是什么

34. 让你设计一个web框架，你要怎么设计，说一下步骤

35. 说一下中间件原理

36. 怎么设计orm，让你写你要怎么写

37. epoll原理

38. 用过原生的http包吗？

39. 一个非常大的数组，让其中两个数想加等于1000怎么算

40. 各个系统出问题怎么监控报警

41. 常用测试工具，压测工具，方法
```go
goconvey,vegeta
```
42. 复杂的单元测试怎么测试，比如有外部接口mysql接口的情况

43. redis集群，哨兵，持久化，事务

44. mysql和redis区别

45. 高可用软件是什么

46. 怎么搞一个并发服务程序

47. 讲解一下你做过的项目，然后找问题问实现细节

48. mysql事务说一下

49. 怎么做一个自动化配置平台系统

50. grpc遵循什么协议

51. grpc内部原理是什么

52. http2的特点是什么，与http1.1的对比

    | HTTP1.1                    | HTTP2       | QUIC                        |
    | -------------------------- | ----------- | --------------------------- |
    | 持久连接                       | 二进制分帧       | 基于UDP的多路传输（单连接下）            |
    | 请求管道化                      | 多路复用（或连接共享） | 极低的等待时延（相比于TCP的三次握手）        |
    | 增加缓存处理（新的字段如cache-control） | 头部压缩        | QUIC为 传输层 协议 ，成为更多应用层的高性能选择 |
    | 增加Host字段、支持断点传输等（把文件分成几部分） | 服务器推送       |                             |


#### 面试总结

1. go的调度

* [Go 调度器: M, P 和 G](https://colobu.com/2017/05/04/go-scheduler/)

* [Go语言实战笔记（十二）| Go goroutine](http://www.flysnow.org/2017/04/11/go-in-action-go-goroutine.html)

* [Golang调度器源码分析](http://ga0.github.io/golang/2015/09/20/golang-runtime-scheduler.html)

2. go struct能不能比较

* 相同struct类型的可以比较

* 不同struct类型的不可以比较,编译都不过，类型不匹配

```go
package main
import "fmt"
func main() {
    type A struct {
        a int
    }
    type B struct {
        a int
    }
    a := A{1}
    //b := A{1}
    b := B{1}
    if a == b {
        fmt.Println("a == b")
    }else{
        fmt.Println("a != b")
    }
} 
// output
// command-line-arguments [command-line-arguments.test]
// ./.go:14:7: invalid operation: a == b (mismatched types A and B) 
```

3. go defer（for defer）

* [Go 关键字 defer 的一些坑](https://deepzz.com/post/how-to-use-defer-in-golang.html)

4. select可以用于什么

Go的select主要是处理多个channel的操作

* [Go语言并发模型：使用 select](https://segmentfault.com/a/1190000006815341)

5. context包的用途

godoc: https://golang.org/pkg/context/

* [Go Context的踩坑经历](https://zhuanlan.zhihu.com/p/34417106)

* [Go语言实战笔记（二十）| Go Context](http://www.flysnow.org/2017/05/12/go-in-action-go-context.html)

6. client如何实现长连接

* [TCP协议的KeepAlive机制与HeartBeat心跳包](http://www.nowamagic.net/academy/detail/23350382#)

* [HTTP Keep-Alive是什么？如何工作？](http://www.nowamagic.net/academy/detail/23350305)

7. 主协程如何等其余协程完再操作

* [Go并发：利用sync.WaitGroup实现协程同步](https://blog.csdn.net/u011304970/article/details/72722044)

* [Go语言重点笔记-深入了解sync.WaitGroup](http://yoojia.xyz/2018/04/13/golang-waitgroup/)

8. slice，len，cap，共享，扩容

9. map如何顺序读取

可以通过sort中的排序包进行对map中的key进行排序

* [golang: 使用 sort 来排序](https://www.jianshu.com/p/6e52bad56e06)

```go
package main

import (
    "fmt"
    "sort"
)

func main() {
    var m = map[string]int{
        "hello":         0,
        "morning":       1,
        "my":            2,
        "girl":   		3,
    }
    var keys []string
    for k := range m {
        keys = append(keys, k)
    }
    sort.Strings(keys)
    for _, k := range keys {
        fmt.Println("Key:", k, "Value:", m[k])
    }
}
```

10. 实现set

根据go中map的keys的无序性和唯一性，可以将其作为set

* [golang实现set集合,变相实现切片去重](https://studygolang.com/articles/3291)

11. 实现消息队列（多生产者，多消费者）

根据Goroutine和channel的读写可以实现消息队列，

* [golang channel多生产者和多消费者实例](https://blog.csdn.net/phpduang/article/details/80143626)

12. 大文件排序

* [【算法】对一个20GB大的文件排序](https://blog.csdn.net/michellechouu/article/details/47002393)

#### 13.基本排序，哪些是稳定的

选择排序、快速排序、希尔排序、堆排序不是稳定的排序算法，

冒泡排序、插入排序、归并排序和基数排序是稳定的排序算法

* [稳定排序和不稳定排序](https://www.cnblogs.com/codingmylife/archive/2012/10/21/2732980.html)

14. Http get跟head

get:获取由Request-URI标识的任何信息(以实体的形式)，如果Request-URI引用某个数据处理过程，则应该以它产生的数据作为在响应中的实体，而不是该过程的源代码文本，除非该过程碰巧输出该文本。

head: 除了服务器不能在响应中返回消息体，HEAD方法与GET相同。用来获取暗示实体的元信息，而不需要传输实体本身。常用于测试超文本链接的有效性、可用性和最近的修改。

* [Http介绍](https://github.com/xuelangZF/CS_Offer/blob/master/Network/HTTP.md)

15. Http 401,403

**401 Unauthorized**： 该HTTP状态码表示认证错误，它是为了认证设计的，而不是为了授权设计的。收到401响应，**表示请求没有被认证—压根没有认证或者认证不正确—但是请重新认证和重试。**（一般在响应头部包含一个*WWW-Authenticate*来描述如何认证）。通常由web服务器返回，而不是web应用。从性质上来说是临时的东西。（服务器要求客户端重试）

**403 Forbidden**：该HTTP状态码是关于授权方面的。从性质上来说是永久的东西，和应用的业务逻辑相关联。它比401更具体，更实际。收到403响应表示**服务器完成认证过程，但是客户端请求没有权限去访问要求的资源。**

总的来说，**401 Unauthorized**响应应该用来表示缺失或错误的认证；**403 Forbidden**响应应该在这之后用，当用户被认证后，但用户没有被授权在特定资源上执行操作。

* [HTTP响应码403 Forbidden和401 Unauthorized对比](https://www.jianshu.com/p/6dceeebbde5b)

16.Http keep-alive


17. Http能不能一次连接多次请求，不等后端返回



18. TCP 和 UDP 有什么区别,适用场景

* TCP 是面向连接的，UDP 是面向无连接的；故 TCP 需要建立连接和断开连接，UDP 不需要。

* TCP 是流协议，UDP 是数据包协议；故 TCP 数据没有大小限制，UDP 数据报有大小限制（UDP 协议本身限制、数据链路层的 MTU、缓存区大小）。

* TCP 是可靠协议，UDP 是不可靠协议；故 TCP 会处理数据丢包重发以及乱序等情况，UDP 则不会处理。


`UDP 的特点及使用场景`:

UDP 不提供复杂的控制机制，利用 IP 提供面向无连接的通信服务，随时都可以发送数据，处理简单且高效，经常用于以下场景：

包总量较小的通信（DNS、SNMP）

视频、音频等多媒体通信（即时通信）

广播通信

`TCP 的特点及使用场景`:

相对于 UDP，TCP 实现了数据传输过程中的各种控制，可以进行丢包时的重发控制，还可以对次序乱掉的分包进行顺序控制。

在对可靠性要求较高的情况下，可以使用 TCP，即不考虑 UDP 的时候，都可以选择 TCP。

* [iOS 面试题 TCP UDP 有什么区别？TCP 为什么要三次握手，四次挥手？](https://mp.weixin.qq.com/s/jLkhjM7wOpZuWgJdAXis1A) 

19. time-wait的作用

* [TCP/IP状态图的TIME_WAIT作用](https://www.iteblog.com/archives/169.html)

20. 数据库如何建索引

* [正确合理的建立MySQL数据库索引](https://blog.csdn.net/nanaMasuda/article/details/52358114)

21. 孤儿进程，僵尸进程

* 孤儿进程：一个父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。孤儿进程将被init进程(进程号为1)所收养，并由init进程对它们完成状态收集工作。

* 僵尸进程：一个进程使用fork创建子进程，如果子进程退出，而父进程并没有调用wait或waitpid获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中。这种进程称之为僵死进程。

22. 死锁条件，如何避免

23. linux命令，查看端口占用，cpu负载，内存占用，如何发送信号给一个进程

24. git文件版本，使用顺序，merge跟rebase

25. 通常一般会用到哪些数据结构?

26. 链表和数组相比, 有什么优缺点?

27. 如何判断两个无环单链表有没有交叉点?

28. 如何判断一个单链表有没有环, 并找出入环点?

29. 描述一下 TCP 四次挥手的过程中

30. TCP 有哪些状态?

31. TCP 的 LISTEN 状态是什么?

32. TCP 的 CLOSE_WAIT 状态是什么?

33. 建立一个 socket 连接要经过哪些步骤?
34. 常见的 HTTP 状态码有哪些?
35. 301和302有什么区别?
36. 504和500有什么区别?
37. HTTPS 和 HTTP 有什么区别?
38. 写一个算法题: 手写快排

#### Golang面试参考

* [Golang面试](http://m.nowcoder.com/discuss/145338?type=2&order=0&pos=6&page=1&headNav=www&from=singlemessage&isappinstalled=0)
