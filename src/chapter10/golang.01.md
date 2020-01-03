#### Golang runtime的调度

Golang作为一个为并发而产生的语言, 从Golang产生的那一刻就注定它具有高并发的特性，而 Go 语言中的并发（并行）编程是经由 goroutine 实现的，goroutine 是 golang 最重要的特性之一，具有使用成本低、消耗资源低、能效高等特点，官方宣称原生 goroutine 并发成千上万不成问题，于是它也成为 Gopher 们经常使用的特性。


Goroutine，Go 语言基于并发（并行）编程的核心。goroutine 是什么？通常 goroutine 会被当做 coroutine（协程）的 golang 实现，从比较粗浅的层面来看，这种认知也算是合理，但实际上，goroutine 并非传统意义上的协程，现在主流的线程模型分三种：内核级线程模型、用户级线程模型和两级线程模型（也称混合型线程模型），传统的协程库属于用户级线程模型，而 goroutine 和它的Go Scheduler在底层实现上其实是属于两级线程模型，因此，有时候为了方便理解可以简单把 goroutine 类比成协程，但心里一定要有个清晰的认知 — goroutine 并不等同于协程。