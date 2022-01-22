#### Golang的汇编过程

在程序编译的时候,汇编的目的是把汇编代码转化为机器指令，因为几乎每一条汇编指令都对应着一条机器指令，所以汇编的过程相对而言非常的简单。

汇编操作所生成的文件叫做目标文件（Object File），目标文件的结构与可执行文件是一致的，它们之间只存在着一些细微的差异。目标文件是无法被执行的，它还需要经过链接这一步操作，目标文件被链接之后才可以产生可执行文件。

Golang原生支持用户级协程，交叉编译，跨平台部署运行, 但是go在编译成机器语言交付给CPU执行的过程中，汇编也只是一个中间状态，汇编指令相对于以上的高级语言而言则显得十分拗口。在大部分强类型的语言中，基本上代码在执行前会经历几个阶段：

```markdown
语法分析--->词法分析--->目标码生成
```

Go的汇编是怎么样的？

Go汇编器所用的指令，一部分与目标机器的指令一一对应，而另外一部分则不是。这是因为编译器套件不需要汇编器直接参与常规的编译过程。相反，编译器使用了一种半抽象的指令集，并且部分指令是在代码生成后才被选择的。

汇编器基于这种半抽象的形式工作，所以虽然你看到的是一条MOV指令，但是工具链针对这条指令实际生成可能完全不是一个移动指令，也许会是清除或者加载。也有可能精确的对应目标平台上同名的指令。
由于这种汇编并不对应某种真实的硬件架构，Go编译器会输出一种抽象可移植的汇编代码。

接着我们看一个应用示例:
```go

package main 

//go:noinline
func add(a, b int) (int, bool) { 
  return a + b, true 
}

func main() { 
  add(5, 10) 
}
```

其中 `//go:noinline` 为编译器指令，不是注释，这里应该意为禁止内联，这部分在scan后形成ast树时也会scan到这个记录，在汇编的过程中会读取这个标记，从而控制一些汇编行为。

然后，我们将这段代码编译到汇编:

```bash
> GOOS=linux GOARCH=amd64 go tool compile -S main.go
"".add STEXT nosplit size=20 args=0x10 locals=0x0 funcid=0x0
  0x0000 00000 (main.go:4)        TEXT    "".add(SB), NOSPLIT|ABIInternal, $0-16
  0x0000 00000 (main.go:4)        FUNCDATA        $0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
  0x0000 00000 (main.go:4)        FUNCDATA        $1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
  0x0000 00000 (main.go:5)        MOVL    "".b+12(SP), AX
  0x0004 00004 (main.go:5)        MOVL    "".a+8(SP), CX
  0x0008 00008 (main.go:5)        ADDL    CX, AX
  0x000a 00010 (main.go:5)        MOVL    AX, "".~r2+16(SP)
  0x000e 00014 (main.go:5)        MOVB    $1, "".~r3+20(SP)
  0x0013 00019 (main.go:5)        RET

"".main STEXT size=66 args=0x0 locals=0x18 funcid=0x0
  0x0000 00000 (main.go:8)        TEXT    "".main(SB), ABIInternal, $24-0
  0x0000 00000 (main.go:8)        MOVQ    (TLS), CX
  0x0009 00009 (main.go:8)        CMPQ    SP, 16(CX)
  0x000d 00013 (main.go:8)        PCDATA  $0, $-2
  0x000d 00013 (main.go:8)        JLS     58
  0x000f 00015 (main.go:8)        PCDATA  $0, $-1
  0x000f 00015 (main.go:8)        SUBQ    $24, SP
  0x0013 00019 (main.go:8)        MOVQ    BP, 16(SP)
  0x0018 00024 (main.go:8)        LEAQ    16(SP), BP
  0x001d 00029 (main.go:8)        FUNCDATA        $0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
  0x001d 00029 (main.go:8)        FUNCDATA        $1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
  0x001d 00029 (main.go:9)        MOVQ    $42949672965, AX
  0x0027 00039 (main.go:9)        MOVQ    AX, (SP)
  0x002b 00043 (main.go:9)        PCDATA  $1, $0
  0x002b 00043 (main.go:9)        CALL    "".add(SB)
  0x0030 00048 (main.go:10)       MOVQ    16(SP), BP
  0x0035 00053 (main.go:10)       ADDQ    $24, SP
  0x0039 00057 (main.go:10)       RET
  0x003a 00058 (main.go:10)       NOP
  0x003a 00058 (main.go:8)        PCDATA  $1, $-1
  0x003a 00058 (main.go:8)        PCDATA  $0, $-2
  0x003a 00058 (main.go:8)        CALL    runtime.morestack_noctxt(SB)
  0x003f 00063 (main.go:8)        PCDATA  $0, $-1
  0x003f 00063 (main.go:8)        NOP
  0x0040 00064 (main.go:8)        JMP     0

```
这里的add：

```go
0x0000 00000 (main.go:4)        TEXT    "".add(SB), NOSPLIT|ABIInternal, $0-16

* 0x0000: 当前指令相对于当前函数的偏移量。

* TEXT "".add: TEXT 指令声明了 "".add 是 .text 段(程序代码在运行期会放在内存的 .text 段中)的一部分，并表明跟在这个声明后的是函数的函数体。 在链接期，"" 这个空字符会被替换为当前的包名: 也就是说，"".add 在链接到二进制文件后会变成 main.add。

* (SB): SB 是一个虚拟寄存器，保存了静态基地址(static-base) 指针，即我们程序地址空间的开始地址。 "".add(SB) 表明我们的符号位于某个固定的相对地址空间起始处的偏移位置 (最终是由链接器计算得到的)。换句话来讲，它有一个直接的绝对地址: 是一个全局的函数符号。

* NOSPLIT: 向编译器表明不应该插入 stack-split 的用来检查栈需要扩张的前导指令。 在我们 add 函数的这种情况下，编译器自己帮我们插入了这个标记: 它足够聪明地意识到，由于 add 没有任何局部变量且没有它自己的栈帧，所以一定不会超出当前的栈，因此每次调用函数时在这里执行栈检查就是完全浪费 CPU 循环了。

* $0-16: $0 代表即将分配的栈帧大小；而 16 指定了调用方传入的参数大小。

* 通常帧大小后一般都跟随着一个参数大小，用-分隔。(这不是一个减法操作，只是一种特殊的语法)
帧大小 $24-8 意味着这个函数有24个字节的帧以及8个字节的参数，位于调用者的帧上。如果NOSPLIT没有在TEXT中指定，则必须提供参数大小。

对于Go原型的汇编函数，go vet会检查参数大小是否正确。Go是一个具备gc机制的语言，因此在C，C++里担心的那些问题在Go这都不是问题！

* Go 的调用规约要求每一个参数都通过栈来传递，这部分空间由 caller 在其栈帧(stack frame)上提供。调用其它函数之前，caller 就需要按照参数和返回变量的大小来对应地增长(返回后收缩)栈。
```

```go
0x0000 00000 (main.go:4)        FUNCDATA        $0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
0x0000 00000 (main.go:4)        FUNCDATA        $1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
```

FUNCDATA以及PCDATA指令包含有被gc回收所使用的信息，这些指令是被编译器加入的。

```go
0x0000 00000 (main.go:5)        MOVL    "".b+12(SP), AX
0x0004 00004 (main.go:5)        MOVL    "".a+8(SP), CX
```

Go的调用要求每一个参数都通过栈来传递，这部分空间由caller在其栈帧(stack frame)上提供。调用其它过程之前，caller就需要按照参数和返回变量的大小来对应地增长(返回后收缩)栈。Go编译器不会生成任何 `PUSH或POP` 族的指令: 栈的增长和收缩是通过在栈指针寄存器 SP 上分别执行减法和加法指令来实现的。

```go
SP伪寄存器是虚拟的栈指针，用于引用帧局部变量以及为函数调用准备的参数，它指向局部栈帧的顶部。
"".b+12(SP) 和 "".a+8(SP) 分别指向栈的低12字节和低8字节位置(栈是向低位地址方向增长的！)。
```

```go
 0x0008 00008 (main.go:5)        ADDL    CX, AX
 0x000a 00010 (main.go:5)        MOVL    AX, "".~r2+16(SP)
 0x000e 00014 (main.go:5)        MOVB    $1, "".~r3+20(SP)
```
其中，第一个变量 a 的地址并不是 0(SP)，而是在 8(SP)，这是因为调用方通过使用 CALL 伪指令，把其返回地址保存在了 0(SP) 位置。参数是反序传入的，也就是说，第一个参数和栈顶距离最近。

ADDL 进行实际的加法操作，L 这里代表 Long，4 字节的值（int32)，其将保存在 AX 和 CX 寄存器中的值进行相加，然后再保存进 AX 寄存器中。 这个结果之后被移动到 `"".~r2+16(SP)` 地址处，这是之前调用方专门为返回值预留的栈空间。这一次 `"".~r2` 同样没什么语义上的含义。

为了弄清楚Go 是如何处理多返回值，我们可以同时返回了一个 bool 常量 true。 返回这个 bool 值的方法和之前返回数值的方法是一样的，只是相对于 SP 寄存器的偏移量发生了变化。

最后:

```go
 0x0013 00019 (main.go:5)        RET
```

最后的 RET 伪指令告诉 Go汇编器插入一些指令，这些指令是对应的目标平台中的调用规约所要求的，从子过程中返回时所需要的指令。 一般情况下这样的指令会使在 0(SP) 寄存器中保存的函数返回地址被 pop 出栈，并跳回到该地址。

接着我们看下main:

```go
"".main STEXT size=66 args=0x0 locals=0x18 funcid=0x0
        0x0000 00000 (main.go:8)        TEXT    "".main(SB), ABIInternal, $24-0
        0x0000 00000 (main.go:8)        MOVQ    (TLS), CX
        0x0009 00009 (main.go:8)        CMPQ    SP, 16(CX)
        0x000d 00013 (main.go:8)        PCDATA  $0, $-2
        0x000d 00013 (main.go:8)        JLS     58
        0x000f 00015 (main.go:8)        PCDATA  $0, $-1
        0x000f 00015 (main.go:8)        SUBQ    $24, SP
        0x0013 00019 (main.go:8)        MOVQ    BP, 16(SP)
        0x0018 00024 (main.go:8)        LEAQ    16(SP), BP
        0x001d 00029 (main.go:8)        FUNCDATA        $0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
        0x001d 00029 (main.go:8)        FUNCDATA        $1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
        0x001d 00029 (main.go:9)        MOVQ    $42949672965, AX
        0x0027 00039 (main.go:9)        MOVQ    AX, (SP)
        0x002b 00043 (main.go:9)        PCDATA  $1, $0
        0x002b 00043 (main.go:9)        CALL    "".add(SB)
        0x0030 00048 (main.go:10)       MOVQ    16(SP), BP
        0x0035 00053 (main.go:10)       ADDQ    $24, SP
        0x0039 00057 (main.go:10)       RET
        0x003a 00058 (main.go:10)       NOP
        0x003a 00058 (main.go:8)        PCDATA  $1, $-1
        0x003a 00058 (main.go:8)        PCDATA  $0, $-2
        0x003a 00058 (main.go:8)        CALL    runtime.morestack_noctxt(SB)
        0x003f 00063 (main.go:8)        PCDATA  $0, $-1
        0x003f 00063 (main.go:8)        NOP
        0x0040 00064 (main.go:8)        JMP     0
```


```markdown
"".main (被链接之后名字会变成 main.main) 是一个全局的函数符号，存储在 .text 段中，该函数的地址是相对于整个地址空间起始位置的一个固定的偏移量。
它分配了 24 字节的栈帧，且不接收参数，不返回值。
* main 作为调用者，通过对虚拟栈指针(stack-pointer)寄存器做减法，将其栈帧大小增加了24个字节(回忆一下栈是向低地址方向增长，所以这里的 SUBQ 指令是将栈帧的大小调整得更大了)。 这 24个字节中:

* 8 个字节(16(SP)-24(SP)) 用来存储当前帧指针 BP (这是一个实际存在的寄存器)的值，以支持栈的展开和方便调试
* 1+3 个字节(12(SP)-16(SP)) 是预留出的给第二个返回值 (bool) 的空间，除了类型本身的 1 个字节，在 amd64 平台上还额外需要 3 个字节来做对齐
* 4 个字节(8(SP)-12(SP)) 预留给第一个返回值 (int32)
* 4 个字节(4(SP)-8(SP)) 是预留给传给被调用函数的参数 b (int32)
* 4 个字节(0(SP)-4(SP)) 预留给传入参数 a (int32)
```

最后，跟着栈的增长，`LEAQ` 指令计算出帧指针的新地址，并将其存储到 BP 寄存器中。

```go
 0x001d 00029 (main.go:9)        MOVQ    $42949672965, AX
 0x0027 00039 (main.go:9)        MOVQ    AX, (SP)
```

调用方将被调用方需要的参数作为一个`Quad word`(8 字节值，对应$42949672965)推到了刚刚增长的栈的栈顶。

尽管指令里出现的 42949672965 这个值看起来像是随机的垃圾值，实际上这个值对应的就是 10 和 32 这两个 4 字节值，它们两被连接成了一个 8 字节值。

```bash
> echo 'obase=2;42949672965' | bc 
101000000000000000000000000000000101
```

我们使用相对于 `static-base` 指针的偏移量，来对 add 函数进行 CALL 调用: 这种调用实际上相当于直接跳到一个指定的地址。

注意 CALL 指令还会将函数的返回地址(8 字节值)也推到栈顶；所以每次我们在 add 函数中引用 SP 寄存器的时候还需要额外偏移 8 个字节！ 例如，`"".a` 现在不是 0(SP) 了，而是在 8(SP) 位置。

```go 
  0x0030 00048 (main.go:10)       MOVQ    16(SP), BP
  0x0035 00053 (main.go:10)       ADDQ    $24, SP
  0x0039 00057 (main.go:10)       RET
```

这里的3个指令对应：

* 将帧指针(frame-pointer)下降一个栈帧(stack-frame)的大小(就是“向下”一级).
* 将栈收缩 24 个字节，回收之前分配的栈空间.
* 请求Go汇编器插入子过程返回相关的指令.
