### Golang面试问题汇总

通常我们去面试肯定会有些不错的Golang的面试题目的，所以总结下，让其他Golang开发者也可以查看到，同时也用来检测自己的能力和提醒自己的不足之处,欢迎大家补充和提交新的面试题目.

Golang面试问题汇总:

1. Golang中除了加Mutex锁以外还有哪些方式安全读写共享变量？

 Goroutine 通过 Channel 进行安全读写共享变量.

2. 无缓冲 Chan 的发送和接收是否同步?

```go
ch := make(chan int)    无缓冲的channel由于没有缓冲发送和接收需要同步
ch := make(chan int, 2) 有缓冲channel不要求发送和接收操作同步 
```
* channel无缓冲时，发送阻塞直到数据被接收，接收阻塞直到读到数据.
* channel有缓冲时，当缓冲满时发送阻塞，当缓冲空时接收阻塞.

3. go语言的并发机制以及它所使用的CSP并发模型．

CSP模型是上个世纪七十年代提出的,不同于传统的多线程通过共享内存来通信，CSP讲究的是“以通信的方式来共享内存”。用于描述两个独立的并发实体通过共享的通讯 channel(管道)进行通信的并发模型。 CSP中channel是第一类对象，它不关注发送消息的实体，而关注与发送消息时使用的channel。

Golang中channel 是被单独创建并且可以在进程之间传递，它的通信模式类似于 boss-worker 模式的，一个实体通过将消息发送到channel 中，然后又监听这个 channel 的实体处理，两个实体之间是匿名的，这个就实现实体中间的解耦，其中 channel 是同步的一个消息被发送到 channel 中，最终是一定要被另外的实体消费掉的，在实现原理上其实类似一个阻塞的消息队列。

Goroutine 是Golang实际并发执行的实体，它底层是使用协程(coroutine)实现并发，coroutine是一种运行在用户态的用户线程，类似于 greenthread，go底层选择使用coroutine的出发点是因为，它具有以下特点：

* 用户空间 避免了内核态和用户态的切换导致的成本
* 可以由语言和框架层进行调度
* 更小的栈空间允许创建大量的实例

Golang中的Goroutine的特性:

Golang内部有三个对象： P对象(processor) 代表上下文（或者可以认为是cpu），M(work thread)代表工作线程，G对象（goroutine）.

正常情况下一个cpu对象启一个工作线程对象，线程去检查并执行goroutine对象。碰到goroutine对象阻塞的时候，会启动一个新的工作线程，以充分利用cpu资源。
所有有时候线程对象会比处理器对象多很多

我们用如下图分别表示P、M、G:

<p align="center">
<img width="300" align="center" src="../images/59.jpg" />
</p>

G（Goroutine） ：我们所说的协程，为用户级的轻量级线程，每个Goroutine对象中的sched保存着其上下文信息

M（Machine） ：对内核级线程的封装，数量对应真实的CPU数（真正干活的对象）

P（Processor） ：即为G和M的调度对象，用来调度G和M之间的关联关系，其数量可通过GOMAXPROCS()来设置，默认为核心数

在单核情况下，所有Goroutine运行在同一个线程（M0）中，每一个线程维护一个上下文（P），任何时刻，一个上下文中只有一个Goroutine，其他Goroutine在runqueue中等待。
一个Goroutine运行完自己的时间片后，让出上下文，自己回到runqueue中（如下图所示）。

当正在运行的G0阻塞的时候（可以需要IO），会再创建一个线程（M1），P转到新的线程中去运行。

<p align="center">
<img width="300" align="center" src="../images/60.jpg" />
</p>

当M0返回时，它会尝试从其他线程中“偷”一个上下文过来，如果没有偷到，会把Goroutine放到Global runqueue中去，然后把自己放入线程缓存中。
上下文会定时检查Global runqueue。

Golang是为并发而生的语言，Go语言是为数不多的在语言层面实现并发的语言；也正是Go语言的并发特性，吸引了全球无数的开发者。

Golang的CSP并发模型，是通过Goroutine和Channel来实现的。

Goroutine 是Go语言中并发的执行单位。有点抽象，其实就是和传统概念上的”线程“类似，可以理解为”线程“。
Channel是Go语言中各个并发结构体(Goroutine)之前的通信机制。通常Channel，是各个Goroutine之间通信的”管道“，有点类似于Linux中的管道。

通信机制channel也很方便，传数据用channel <- data，取数据用<-channel。

在通信过程中，传数据channel <- data和取数据<-channel必然会成对出现，因为这边传，那边取，两个goroutine之间才会实现通信。

而且不管传还是取，必阻塞，直到另外的goroutine传或者取为止。

4. Golang 中常用的并发模型？

Golang 中常用的并发模型有三种:

* 通过channel通知实现并发控制

无缓冲的通道指的是通道的大小为0，也就是说，这种类型的通道在接收前没有能力保存任何值，它要求发送 goroutine 和接收 goroutine 同时准备好，才可以完成发送和接收操作。

从上面无缓冲的通道定义来看，发送 goroutine 和接收 gouroutine 必须是同步的，同时准备后，如果没有同时准备好的话，先执行的操作就会阻塞等待，直到另一个相对应的操作准备好为止。这种无缓冲的通道我们也称之为同步通道。

```go
func main() {
    ch := make(chan struct{})
    go func() {
        fmt.Println("start working")
        time.Sleep(time.Second * 1)
        ch <- struct{}{}
    }()

    <-ch

    fmt.Println("finished")
}

```
当主 goroutine 运行到 <-ch 接受 channel 的值的时候，如果该  channel 中没有数据，就会一直阻塞等待，直到有值。 这样就可以简单实现并发控制

* 通过sync包中的WaitGroup实现并发控制

在 sync 包中，提供了 WaitGroup ，它会等待它收集的所有 goroutine 任务全部完成。在WaitGroup里主要有三个方法:

* Add, 可以添加或减少 goroutine的数量
* Done, 相当于Add(-1)
* Wait, 执行后会堵塞主线程，直到WaitGroup 里的值减至0

在主 goroutine 中 Add(delta int) 索要等待goroutine 的数量。
在每一个 goroutine 完成后 Done() 表示这一个goroutine 已经完成，当所有的 goroutine 都完成后，在主 goroutine 中 WaitGroup 返回返回。

```go
func main(){
    var wg sync.WaitGroup
    var urls = []string{
        "http://www.golang.org/",
        "http://www.google.com/",
    }
    for _, url := range urls {
        wg.Add(1)
        go func(url string) {
            defer wg.Done()
            http.Get(url)
        }(url)
    }
    wg.Wait()
}
```

在Golang官网中对于WaitGroup介绍是`A WaitGroup must not be copied after first use`,在 WaitGroup 第一次使用后，不能被拷贝

应用示例:
```go
func main(){
 wg := sync.WaitGroup{}
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(wg sync.WaitGroup, i int) {
            fmt.Printf("i:%d", i)
            wg.Done()
        }(wg, i)
    }
    wg.Wait()
    fmt.Println("exit")
}

```
运行:
```go
i:1i:3i:2i:0i:4fatal error: all goroutines are asleep - deadlock!

goroutine 1 [semacquire]:
sync.runtime_Semacquire(0xc000094018)
        /home/keke/soft/go/src/runtime/sema.go:56 +0x39
sync.(*WaitGroup).Wait(0xc000094010)
        /home/keke/soft/go/src/sync/waitgroup.go:130 +0x64
main.main()
        /home/keke/go/Test/wait.go:17 +0xab
exit status 2

```

它提示所有的 goroutine 都已经睡眠了，出现了死锁。这是因为 wg 给拷贝传递到了  goroutine 中，导致只有 Add 操作，其实 Done操作是在 wg 的副本执行的。
因此 Wait 就死锁了。

这个第一个修改方式:将匿名函数中 wg 的传入类型改为 *sync.WaitGrou,这样就能引用到正确的WaitGroup了。
这个第二个修改方式:将匿名函数中的 wg 的传入参数去掉，因为Go支持闭包类型，在匿名函数中可以直接使用外面的 wg 变量

* 在Go 1.7 以后引进的强大的Context上下文，实现并发控制

通常,在一些简单场景下使用 channel 和 WaitGroup 已经足够了，但是当面临一些复杂多变的网络并发场景下 channel 和 WaitGroup 显得有些力不从心了。
比如一个网络请求 Request，每个 Request 都需要开启一个 goroutine 做一些事情，这些 goroutine 又可能会开启其他的 goroutine，比如数据库和RPC服务。
所以我们需要一种可以跟踪 goroutine 的方案，才可以达到控制他们的目的，这就是Go语言为我们提供的 Context，称之为上下文非常贴切，它就是goroutine 的上下文。
它是包括一个程序的运行环境、现场和快照等。每个程序要运行时，都需要知道当前程序的运行状态，通常Go 将这些封装在一个 Context 里，再将它传给要执行的 goroutine 。

context 包主要是用来处理多个 goroutine 之间共享数据，及多个 goroutine 的管理。

context 包的核心是 struct Context，接口声明如下：
```go
// A Context carries a deadline, cancelation signal, and request-scoped values
// across API boundaries. Its methods are safe for simultaneous use by multiple
// goroutines.
type Context interface {
    // Done returns a channel that is closed when this `Context` is canceled
    // or times out.
    Done() <-chan struct{}

    // Err indicates why this Context was canceled, after the Done channel
    // is closed.
    Err() error

    // Deadline returns the time when this Context will be canceled, if any.
    Deadline() (deadline time.Time, ok bool)

    // Value returns the value associated with key or nil if none.
    Value(key interface{}) interface{}
}
```

Done() 返回一个只能接受数据的channel类型，当该context关闭或者超时时间到了的时候，该channel就会有一个取消信号

Err() 在Done() 之后，返回context 取消的原因。

Deadline() 设置该context cancel的时间点

Value() 方法允许 Context 对象携带request作用域的数据，该数据必须是线程安全的。

Context 对象是线程安全的，你可以把一个 Context 对象传递给任意个数的 gorotuine，对它执行 取消 操作时，所有 goroutine 都会接收到取消信号。

一个 Context 不能拥有 Cancel 方法，同时我们也只能 Done channel 接收数据。
其中的原因是一致的：接收取消信号的函数和发送信号的函数通常不是一个。
典型的场景是：父操作为子操作操作启动 goroutine，子操作也就不能取消父操作。

5. JSON 标准库对 nil slice 和 空 slice 的处理是一致的吗？　

首先JSON 标准库对 nil slice 和 空 slice 的处理是不一致.

通常错误的用法，会报数组越界的错误，因为只是声明了slice，却没有给实例化的对象。
```go
var slice []int
slice[1] = 0
```
此时slice的值是nil，这种情况可以用于需要返回slice的函数，当函数出现异常的时候，保证函数依然会有nil的返回值。

empty slice 是指slice不为nil，但是slice没有值，slice的底层的空间是空的，此时的定义如下：
```go
slice := make([]int,0）
slice := []int{}
```
当我们查询或者处理一个空的列表的时候，这非常有用，它会告诉我们返回的是一个列表，但是列表内没有任何值。

总之，nil slice 和 empty slice是不同的东西,需要我们加以区分的.

6. 协程，线程，进程的区别。

* 进程

进程是具有一定独立功能的程序关于某个数据集合上的一次运行活动,进程是系统进行资源分配和调度的一个独立单位。每个进程都有自己的独立内存空间，不同进程通过进程间通信来通信。由于进程比较重量，占据独立的内存，所以上下文进程间的切换开销（栈、寄存器、虚拟内存、文件句柄等）比较大，但相对比较稳定安全。

* 线程

线程是进程的一个实体,是CPU调度和分派的基本单位,它是比进程更小的能独立运行的基本单位.线程自己基本上不拥有系统资源,只拥有一点在运行中必不可少的资源(如程序计数器,一组寄存器和栈),但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源。线程间通信主要通过共享内存，上下文切换很快，资源开销较少，但相比进程不够稳定容易丢失数据。

* 协程

协程是一种用户态的轻量级线程，协程的调度完全由用户控制。协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈，直接操作栈则基本没有内核切换的开销，可以不加锁的访问全局变量，所以上下文的切换非常快。

7. 互斥锁，读写锁，死锁问题是怎么解决。

* 互斥锁

互斥锁就是互斥变量mutex，用来锁住临界区的. 

条件锁就是条件变量，当进程的某些资源要求不满足时就进入休眠，也就是锁住了。当资源被分配到了，条件锁打开，进程继续运行；读写锁，也类似，用于缓冲区等临界资源能互斥访问的。

* 读写锁

通常有些公共数据修改的机会很少，但其读的机会很多。并且在读的过程中会伴随着查找，给这种代码加锁会降低我们的程序效率。读写锁可以解决这个问题。
<p align="center">
<img width="300" align="center" src="../images/61.jpg" />
</p>

注意：写独占，读共享，写锁优先级高


* 死锁

一般情况下，如果同一个线程先后两次调用lock，在第二次调用时，由于锁已经被占用，该线程会挂起等待别的线程释放锁，然而锁正是被自己占用着的，该线程又被挂起而没有机会释放锁，因此就永远处于挂起等待状态了，这叫做死锁（Deadlock）。
另外一种情况是：若线程A获得了锁1，线程B获得了锁2，这时线程A调用lock试图获得锁2，结果是需要挂起等待线程B释放锁2，而这时线程B也调用lock试图获得锁1，结果是需要挂起等待线程A释放锁1，于是线程A和B都永远处于挂起状态了。

死锁产生的四个必要条件:
(1) 互斥条件：一个资源每次只能被一个进程使用
(2) 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
(4) 不剥夺条件:进程已获得的资源，在末使用完之前，不能强行剥夺。
(5) 循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。
这四个条件是死锁的必要条件，只要系统发生死锁，这些条件必然成立，而只要上述条件之一不满足，就不会发生死锁。

a. 预防死锁

可以把资源一次性分配：（破坏请求和保持条件）

然后剥夺资源：即当某进程新的资源未满足时，释放已占有的资源（破坏不可剥夺条件）

资源有序分配法：系统给每类资源赋予一个编号，每一个进程按编号递增的顺序请求资源，释放则相反（破坏环路等待条件）

b. 避免死锁

预防死锁的几种策略，会严重地损害系统性能。因此在避免死锁时，要施加较弱的限制，从而获得 较满意的系统性能。由于在避免死锁的策略中，允许进程动态地申请资源。因而，系统在进行资源分配之前预先计算资源分配的安全性。若此次分配不会导致系统进入不安全状态，则将资源分配给进程；否则，进程等待。其中最具有代表性的避免死锁算法是银行家算法。

c. 检测死锁

首先为每个进程和每个资源指定一个唯一的号码,然后建立资源分配表和进程等待表.

d. 解除死锁

当发现有进程死锁后，便应立即把它从死锁状态中解脱出来，常采用的方法有.

e. 剥夺资源

从其它进程剥夺足够数量的资源给死锁进程，以解除死锁状态.

f. 撤消进程

可以直接撤消死锁进程或撤消代价最小的进程，直至有足够的资源可用，死锁状态.消除为止.所谓代价是指优先级、运行代价、进程的重要性和价值等。

8. Golang的内存模型，为什么小对象多了会造成gc压力。

通常小对象过多会导致GC三色法消耗过多的GPU。优化思路是，减少对象分配.

9. data Race问题怎么解决？能不能不加锁解决这个问题？

同步访问共享数据是处理数据竞争的一种有效的方法.golang在1.1之后引入了竞争检测机制，可以使用 go run -race 或者 go build -race来进行静态检测。 
其在内部的实现是,开启多个协程执行同一个命令， 并且记录下每个变量的状态.

可以使用互斥锁sync.Mutex,解决 数据竞争(data race),也可以使用管道解决.

10. 什么是channel，为什么它可以做到线程安全？

11. Epoll原理.

12. Golang GC 时会发生什么?

13. Golang 中 Goroutine 如何调度?

14. 并发编程概念是什么？

15. 负载均衡原理是什么

16. LVS相关

17. 微服务架构是什么样子的?

18. 分布式锁实现原理，用过吗？

19. Etcd和Redis怎么实现分布式锁?

20. Redis的数据结构有哪些，以及实现场景?

21. Mysql高可用方案有哪些?

22. 说下Go的并发模型.

23. Goroutine和Channel的作用分别是什么?

24. 怎么查看Goroutine的数量.

25. 说下Go中的锁有哪些?三种锁，读写锁，互斥锁，还有map的安全的锁?

26. 读写锁或者互斥锁读的时候能写吗?

27. 怎么限制Goroutine的数量.

28. Channel是同步的还是异步的

29. 说一下异步和非阻塞的区别?

30. Log包线程安全吗？

31. Goroutine和线程的区别?

32. 滑动窗口的概念以及应用?

33. 怎么做弹性扩缩容，原理是什么?

34. 让你设计一个web框架，你要怎么设计，说一下步骤.

35. 说一下中间件原理.

36. 怎么设计orm，让你写,你会怎么写?

37. 用过原生的http包吗？

38. 一个非常大的数组，让其中两个数想加等于1000怎么算.

39. 各个系统出问题怎么监控报警.

40.  常用测试工具，压测工具，方法?

```go
goconvey,vegeta
```
41. 复杂的单元测试怎么测试，比如有外部接口mysql接口的情况

42. redis集群，哨兵，持久化，事务

43. mysql和redis区别

44. 高可用软件是什么

45. 怎么搞一个并发服务程序

46. 讲解一下你做过的项目，然后找问题问实现细节

47. mysql事务说下.

48. 怎么做一个自动化配置平台系统

49. grpc遵循什么协议

50. grpc内部原理是什么

51. http2的特点是什么，与http1.1的对比

    | HTTP1.1                    | HTTP2       | QUIC                        |
    | -------------------------- | ----------- | --------------------------- |
    | 持久连接                       | 二进制分帧       | 基于UDP的多路传输（单连接下）            |
    | 请求管道化                      | 多路复用（或连接共享） | 极低的等待时延（相比于TCP的三次握手）        |
    | 增加缓存处理（新的字段如cache-control） | 头部压缩        | QUIC为 传输层 协议 ，成为更多应用层的高性能选择 |
    | 增加Host字段、支持断点传输等（把文件分成几部分） | 服务器推送       |                             |

52. go的调度

* [Go 调度器: M, P 和 G](https://colobu.com/2017/05/04/go-scheduler/)

* [Go语言实战笔记（十二）| Go goroutine](http://www.flysnow.org/2017/04/11/go-in-action-go-goroutine.html)

* [Golang调度器源码分析](http://ga0.github.io/golang/2015/09/20/golang-runtime-scheduler.html)

53. go struct能不能比较

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

54. go defer（for defer）

* [Go 关键字 defer 的一些坑](https://deepzz.com/post/how-to-use-defer-in-golang.html)

55. select可以用于什么

Go的select主要是处理多个channel的操作

* [Go语言并发模型：使用 select](https://segmentfault.com/a/1190000006815341)

56. context包的用途

godoc: https://golang.org/pkg/context/

* [Go Context的踩坑经历](https://zhuanlan.zhihu.com/p/34417106)

* [Go语言实战笔记（二十）| Go Context](http://www.flysnow.org/2017/05/12/go-in-action-go-context.html)

57. client如何实现长连接

* [TCP协议的KeepAlive机制与HeartBeat心跳包](http://www.nowamagic.net/academy/detail/23350382#)

* [HTTP Keep-Alive是什么？如何工作？](http://www.nowamagic.net/academy/detail/23350305)

58. 主协程如何等其余协程完再操作

* [Go并发：利用sync.WaitGroup实现协程同步](https://blog.csdn.net/u011304970/article/details/72722044)

* [Go语言重点笔记-深入了解sync.WaitGroup](http://yoojia.xyz/2018/04/13/golang-waitgroup/)

59. slice，len，cap，共享，扩容

60. map如何顺序读取

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

61. 实现set

根据go中map的keys的无序性和唯一性，可以将其作为set

* [golang实现set集合,变相实现切片去重](https://studygolang.com/articles/3291)

62. 实现消息队列（多生产者，多消费者）

根据Goroutine和channel的读写可以实现消息队列，

* [golang channel多生产者和多消费者实例](https://blog.csdn.net/phpduang/article/details/80143626)

63. 大文件排序

* [【算法】对一个20GB大的文件排序](https://blog.csdn.net/michellechouu/article/details/47002393)

64.基本排序，哪些是稳定的

选择排序、快速排序、希尔排序、堆排序不是稳定的排序算法，

冒泡排序、插入排序、归并排序和基数排序是稳定的排序算法

* [稳定排序和不稳定排序](https://www.cnblogs.com/codingmylife/archive/2012/10/21/2732980.html)

65. Http get跟head

get:获取由Request-URI标识的任何信息(以实体的形式)，如果Request-URI引用某个数据处理过程，则应该以它产生的数据作为在响应中的实体，而不是该过程的源代码文本，除非该过程碰巧输出该文本。

head: 除了服务器不能在响应中返回消息体，HEAD方法与GET相同。用来获取暗示实体的元信息，而不需要传输实体本身。常用于测试超文本链接的有效性、可用性和最近的修改。

* [Http介绍](https://github.com/xuelangZF/CS_Offer/blob/master/Network/HTTP.md)

66. Http 401,403

**401 Unauthorized**： 该HTTP状态码表示认证错误，它是为了认证设计的，而不是为了授权设计的。收到401响应，**表示请求没有被认证—压根没有认证或者认证不正确—但是请重新认证和重试。**（一般在响应头部包含一个*WWW-Authenticate*来描述如何认证）。通常由web服务器返回，而不是web应用。从性质上来说是临时的东西。（服务器要求客户端重试）

**403 Forbidden**：该HTTP状态码是关于授权方面的。从性质上来说是永久的东西，和应用的业务逻辑相关联。它比401更具体，更实际。收到403响应表示**服务器完成认证过程，但是客户端请求没有权限去访问要求的资源。**

总的来说，**401 Unauthorized**响应应该用来表示缺失或错误的认证；**403 Forbidden**响应应该在这之后用，当用户被认证后，但用户没有被授权在特定资源上执行操作。

* [HTTP响应码403 Forbidden和401 Unauthorized对比](https://www.jianshu.com/p/6dceeebbde5b)

67.Http keep-alive


68. Http能不能一次连接多次请求，不等后端返回



69. TCP 和 UDP 有什么区别,适用场景

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

70. time-wait的作用

* [TCP/IP状态图的TIME_WAIT作用](https://www.iteblog.com/archives/169.html)

71. 数据库如何建索引

* [正确合理的建立MySQL数据库索引](https://blog.csdn.net/nanaMasuda/article/details/52358114)

72. 孤儿进程，僵尸进程

* 孤儿进程：一个父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。孤儿进程将被init进程(进程号为1)所收养，并由init进程对它们完成状态收集工作。

* 僵尸进程：一个进程使用fork创建子进程，如果子进程退出，而父进程并没有调用wait或waitpid获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中。这种进程称之为僵死进程。

73. 死锁条件，如何避免

74. linux命令，查看端口占用，cpu负载，内存占用，如何发送信号给一个进程

75. git文件版本，使用顺序，merge跟rebase

76. 通常一般会用到哪些数据结构?

77. 链表和数组相比, 有什么优缺点?

78. 如何判断两个无环单链表有没有交叉点?

79. 如何判断一个单链表有没有环, 并找出入环点?

80. 描述一下 TCP 四次挥手的过程中

81. TCP 有哪些状态?

82. TCP 的 LISTEN 状态是什么?

83. TCP 的 CLOSE_WAIT 状态是什么?

84. 建立一个 socket 连接要经过哪些步骤?
85. 常见的 HTTP 状态码有哪些?
86. 301和302有什么区别?
87. 504和500有什么区别?
88. HTTPS 和 HTTP 有什么区别?
89. 算法题: 手写一个快速排序
```go
func main() {
	var arr = []int{19,8,16,15,23,34,6,3,1,0,2,9,7}
	quickAscendingSort(arr, 0, len(arr)-1)
	fmt.Println("quickAscendingSort:",arr)

	quickDescendingSort(arr, 0, len(arr)-1)
	fmt.Println("quickDescendingSort:",arr)
}

//升序
func quickAscendingSort(arr []int, start, end int) {
	if (start < end) {
		i, j := start, end
		key := arr[(start + end)/2]
		for i <= j {
			for arr[i] < key {
				i++
			}
			for arr[j] > key {
				j--
			}
			if i <= j {
				arr[i], arr[j] = arr[j], arr[i]
				i++
				j--
			}
		}

		if start < j {
			quickAscendingSort(arr, start, j)
		}
		if end > i {
			quickAscendingSort(arr, i, end)
		}
	}
}

//降序
func quickDescendingSort(arr []int, start, end int) {
	if (start < end) {
		i, j := start, end
		key := arr[(start + end)/2]
		for i <= j {
			for arr[i] > key {
				i++
			}
			for arr[j] < key {
				j--
			}
			if i <= j {
				arr[i], arr[j] = arr[j], arr[i]
				i++
				j--
			}
		}

		if start < j {
			quickDescendingSort(arr, start, j)
		}
		if end > i {
			quickDescendingSort(arr, i, end)
		}
	}
}
```
90. Golang 里的逃逸分析是什么？怎么避免内存逃逸？

91. 配置中心如何保证一致性？

92. Golang 的GC触发时机是什么?

93. Redis 里数据结构的实现熟悉吗?

94. Etcd的Raft一致性算法原理?

95. 微服务概念.

96. SLB原理.

97. 分布式一直性原则.

98. 如何保证服务宕机造成的分布式服务节点处理问题?

99. 服务发现怎么实现的.


#### Golang面试参考

* [Golang面试](http://m.nowcoder.com/discuss/145338?type=2&order=0&pos=6&page=1&headNav=www&from=singlemessage&isappinstalled=0)
